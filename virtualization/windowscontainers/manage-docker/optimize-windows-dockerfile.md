---
title: 优化 Windows Dockerfile
description: 优化用于 Windows 容器的 Dockerfile。
keywords: docker, 容器
author: cwilhit
ms.date: 05/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb2848ca-683e-4361-a750-0d1d14ec8031
ms.openlocfilehash: d897560061fae23fda6f88ebdad6dd804da9a8f1
ms.sourcegitcommit: c48dcfe43f73b96e0ebd661164b6dd164c775bfa
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "9610337"
---
# <a name="optimize-windows-dockerfiles"></a>优化 Windows Dockerfile

有多种方法来优化 Docker build 过程和生成的 Docker 映像。 本文介绍了 Docker build 过程的工作原理以及如何以最佳方式创建 Windows 容器的图像。

## <a name="image-layers-in-docker-build"></a>在 Docker 生成的映像层

你可以优化 Docker build 之前，你将需要知道如何 Docker build 的工作原理。 在 Docker build 过程中，会占用 Dockerfile 并且在其自身临时的容器中一对一地运行每个可操作的指令。 结果是，每个可操作的指令都有一个新映像层。

例如，下面的示例 Dockerfile 使用`windowsservercore`基本操作系统映像，安装 IIS，然后创建一个简单的网站。

```dockerfile
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

你可能希望此 Dockerfile，将生成具有两个层组成，一个用于容器操作系统映像，第二个包括 IIS 和网站的图像。 但是，在实际图像具有多个层，并且每个层取决于与其前一。

若要使其成为更清楚，让我们运行`docker history`命令针对映像我们的示例中所做的 Dockerfile。

```dockerfile
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

输出向我们显示此图像有四个层： 基层和映射到 Dockerfile 中的每个指令的三个附加层。 底层（本示例中为 `6801d964fda5`）代表基本操作系统映像。 设置的一层是 IIS 安装。 下一层包括新的网站，依次类推。

可以编写 Dockerfiles，以最小化映像层、 优化生成性能和优化通过可读性的辅助功能。 完成相同的映像生成任务基本上有多种方式。 了解如何在 Dockerfile 的格式会影响生成时间和它创建的图像提升自动化体验。

## <a name="optimize-image-size"></a>优化映像大小

根据空间要求，图像大小可以是一个重要因素构建 Docker 容器映像时。 容器映像在注册表和主机之间移动、导出和导入，最终占用了空间。 本部分将告诉你如何在 Windows 容器的 Docker 生成过程期间减小映像大小。

有关 Dockerfile 最佳做法的其他信息，请参阅[编写 Docker.com 上的 Dockerfile 的最佳实践](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)。

### <a name="group-related-actions"></a>组相关的操作

因为每个`RUN`指令在容器映像，将操作分组为一个中创建新的层`RUN`指令可以减少层在 Dockerfile 中的数量。 而最小化层可能不太会影响映像的大小，而对相关操作进行分组则可以，这将在后面的示例中看到。

在此部分中，我们将比较两个示例执行相同操作的 Dockerfile。 但是，一个 Dockerfile 具有一个指令，每个操作，而其他拥有其相关的操作组合在一起。

下面的分组的示例 Dockerfile 下载 Python for Windows，安装它，并安装完成后，将删除下载的安装程序文件。 在此 Dockerfile，每个操作提供其自己`RUN`指令。

```dockerfile
FROM windowsservercore

RUN powershell.exe -Command Invoke-WebRequest "https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe" -OutFile c:\python-3.5.1.exe
RUN powershell.exe -Command Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait
RUN powershell.exe -Command Remove-Item c:\python-3.5.1.exe -Force
```

生成的映像由三个附加层组成，每一个对应一个 `RUN` 指令。

```dockerfile
docker history doc-example-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
a395ca26777f        15 seconds ago      cmd /S /C powershell.exe -Command Remove-Item   24.56 MB
6c137f466d28        28 seconds ago      cmd /S /C powershell.exe -Command Start-Proce   178.6 MB
957147160e8d        3 minutes ago       cmd /S /C powershell.exe -Command Invoke-WebR   125.7 MB
```

第二个示例是 Dockerfile 执行完全相同的操作。 但是，所有相关的操作已被分组在单个`RUN`指令。 的每个步骤`RUN`指令是 Dockerfile 的新行上时 \\ 字符用于换行。

```dockerfile
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

生成的映像有只有一个附加层`RUN`指令。

```dockerfile
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB
```

### <a name="remove-excess-files"></a>删除多余文件

是否已使用你 Dockerfile，例如安装程序，它不需要的文件，你可以删除它以减小映像大小。 将文件复制到映像层的同一个步骤也需要此操作。 这样可以防止文件留在较低级别的映像层。

在下面的示例 Dockerfile，Python 包是下载，执行，然后删除。 在一个 `RUN` 操作中就可以完成以上所有步骤，从而产生单个映像层。

```dockerfile
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

## <a name="optimize-build-speed"></a>优化生成速度

### <a name="multiple-lines"></a>多行

你可以拆分为多个单独指令来优化 Docker build 速度的操作。 多个`RUN`操作会增加缓存有效性，因为每个创建单个层`RUN`指令。 如果在其他 Docker Build 操作中已运行的相同指令，此缓存的操作 （映像层） 重复使用，则导致降低 Docker build 运行时。

在以下示例中，Apache 和 Visual Studio 重新分发包下载、 安装以及然后通过删除不再需要的文件清理。 这可与单个`RUN`指令。 如果下列任一操作更新时，将重新运行所有操作。

```dockerfile
FROM windowsservercore

RUN powershell -Command \
    
  # Download software ; \
    
  wget https://www.apachelounge.com/download/VC11/binaries/httpd-2.4.18-win32-VC11.zip -OutFile c:\apache.zip ; \
  wget "https://download.microsoft.com/download/1/6/B/16B06F60-3B20-4FF2-B699-5E9B7962F9AE/VSU_4/vcredist_x86.exe" -OutFile c:\vcredist.exe ; \
  wget -Uri http://windows.php.net/downloads/releases/php-5.5.33-Win32-VC11-x86.zip -OutFile c:\php.zip ; \
    
  # Install Software ; \
    
  Expand-Archive -Path c:\php.zip -DestinationPath c:\php ; \
  Expand-Archive -Path c:\apache.zip -DestinationPath c:\ ; \
  start-Process c:\vcredist.exe -ArgumentList '/quiet' -Wait ; \
    
  # Remove unneeded files ; \
     
  Remove-Item c:\apache.zip -Force; \
  Remove-Item c:\vcredist.exe -Force; \
  Remove-Item c:\php.zip
```

生成的映像有两个图层，一个用于基本操作系统映像，一个包含单个中的所有操作`RUN`指令。

```dockerfile
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

相比之下，下面是相同的操作拆分为三个`RUN`说明。 在此情况下，每个`RUN`指令缓存在容器映像层，并且只有那些具有更改的需要后续 Dockerfile 重新生成。

```dockerfile
FROM windowsservercore

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.apachelounge.com/download/VC11/binaries/httpd-2.4.18-win32-VC11.zip -OutFile c:\apache.zip ; \
    Expand-Archive -Path c:\apache.zip -DestinationPath c:\ ; \
    Remove-Item c:\apache.zip -Force

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    wget "https://download.microsoft.com/download/1/6/B/16B06F60-3B20-4FF2-B699-5E9B7962F9AE/VSU_4/vcredist_x86.exe" -OutFile c:\vcredist.exe ; \
    start-Process c:\vcredist.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist.exe -Force

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    wget http://windows.php.net/downloads/releases/php-5.5.33-Win32-VC11-x86.zip -OutFile c:\php.zip ; \
    Expand-Archive -Path c:\php.zip -DestinationPath c:\php ; \
    Remove-Item c:\php.zip -Force
```

生成的映像由四个图层;一个用于基本操作系统映像的三个层`RUN`说明。 因为每个`RUN`指令已在它自己的层中运行，此 Dockerfile 的任何后续运行或相同指令集的其他 Dockerfile 中将使用缓存的映像层，减少了生成时间。

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

如何订购说明很重要时使用映像缓存，将在下一节中看到。

### <a name="ordering-instructions"></a>指令排序

从顶到底处理 Dockerfile，针对缓存层比较每条指令。 当找到一条不含缓存层的指令后，会在新的容器映像层处理此指令和所有后续指令。 因此，指令放置的顺序非常重要。 将保持不变的指令放置在 Dockerfile 顶部。 将可能改变的指令放置在 Dockerfile 底部。 这样就降低了取消现有缓存的可能性。

以下示例演示 Dockerfile 指令排序如何影响缓存有效性。 此简单示例 Dockerfile 有四个带编号的文件夹。  

```dockerfile
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```

生成的映像有五个层，一个用于基本操作系统映像，每个`RUN`说明。

```dockerfile
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B
```

此下一步 Dockerfile 具有现在已经过稍许修改，与第三个`RUN`指令更改为新文件。 当针对此 Dockerfile 运行 Docker build 时，与上个示例中相同的前三条指令使用缓存的映像层。 但是，因为更改`RUN`指令不缓存，为更改的指令和所有后续指令创建新的层。

```dockerfile
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

当你进行比较的此部分的第一个示例中的新图像的图像 Id 时，你会注意到共享的前三个图层从底部到顶部，但是第四个和第五个是唯一。

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY               SIZE                COMMENT
c92cc95632fb        28 seconds ago      cmd /S /C mkdir test-4   5.644 MB
2f05e6f5c523        37 seconds ago      cmd /S /C mkdir test-5   5.01 MB
68fda53ce682        3 minutes ago       cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        4 minutes ago       cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                 0 B
```

## <a name="cosmetic-optimization"></a>表面优化

### <a name="instruction-case"></a>指令用例

Dockerfile 指令不区分大小写，但约定是使用大写形式。 通过在指令调用和指令操作之间引入差异，提高了可读性。 下面的两个示例比较首字母未大写和大写 Dockerfile。

以下是首字母未大写的 Dockerfile:

```dockerfile
# Sample Dockerfile

from windowsservercore
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```

下面是使用大写相同 Dockerfile:

```dockerfile
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### <a name="line-wrapping"></a>换行

长而复杂的操作可以由反斜杠分隔到多个行上`\`字符。 以下 Dockerfile 安装了 Visual Studio 可再发行组件包，删除了安装程序文件，然后创建了配置文件。 这三个操作都是在一行上指定的。

```dockerfile
FROM windowsservercore

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```

此命令可分解使用反斜杠因此，每个操作从一个`RUN`在自己的行上指定指令。

```dockerfile
FROM windowsservercore

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## <a name="further-reading-and-references"></a>进一步阅读和参考

[Windows 上的 Dockerfile](manage-windows-dockerfile.md)

[在 Docker.com 上编写 Dockerfile 的最佳实践](https://docs.docker.com/engine/reference/builder/)
