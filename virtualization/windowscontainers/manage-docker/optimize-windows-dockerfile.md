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
ms.openlocfilehash: ae633c7ba5d9672335addcc582988fc47c13ed79
ms.sourcegitcommit: f3b6b470dd9cde8e8cac7b13e7e7d8bf2a39aa34
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/10/2019
ms.locfileid: "10077448"
---
# <a name="optimize-windows-dockerfiles"></a>优化 Windows Dockerfile

可通过多种方法来优化 Docker 生成过程和生成的 Docker 图像。 本文介绍 Docker 生成过程的工作原理，以及如何以最佳方式为 Windows 容器创建图像。

## <a name="image-layers-in-docker-build"></a>Docker 内部的图像图层

在可以优化 Docker 生成之前，需要了解 Docker 构建的工作方式。 在 Docker build 过程中，会占用 Dockerfile 并且在其自身临时的容器中一对一地运行每个可操作的指令。 结果是，每个可操作的指令都有一个新映像层。

例如，下面的示例 Dockerfile 使用`mcr.microsoft.com/windows/servercore:ltsc2019`基本 OS 映像，安装 IIS，然后创建一个简单的网站。

```dockerfile
# Sample Dockerfile

FROM mcr.microsoft.com/windows/servercore:ltsc2019
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

你可能会认为，此 Dockerfile 将生成一个具有两个层的映像，一个用于容器 OS 映像，另一个包含 IIS 和网站。 但是，实际图像具有多个图层，每个图层都依赖于它前面的图层。

若要使其更清晰，让我们`docker history`在示例 Dockerfile 中对图像运行命令。

```dockerfile
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

输出显示我们此图像具有四个图层：基本图层和3个附加层，它们映射到 Dockerfile 中的每个指令。 底层（本示例中为 `6801d964fda5`）代表基本操作系统映像。 一个图层是 IIS 安装。 下一层包括新的网站，依次类推。

Dockerfiles 可以编写为最小化图像层、优化生成性能和通过可读性优化辅助功能。 完成相同的映像生成任务基本上有多种方式。 了解 Dockerfile 的格式如何影响生成时间以及它所创建的图像可提高自动化体验。

## <a name="optimize-image-size"></a>优化图像大小

在构建 Docker 容器图像时，图像大小可能是一个重要因素，具体取决于你的空间需求。 容器映像在注册表和主机之间移动、导出和导入，最终占用了空间。 本部分将告诉你如何在 Windows 容器的 Docker 生成过程中最大程度地减少图像大小。

有关 Dockerfile 最佳做法的其他信息，请参阅[在 Docker.com 上编写 Dockerfiles 的最佳做法](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)。

### <a name="group-related-actions"></a>组相关的操作

由于每`RUN`个指令在容器图像中创建一个新图层，因此将`RUN`操作分组为一个指令可以减少 Dockerfile 中的层数。 而最小化层可能不太会影响映像的大小，而对相关操作进行分组则可以，这将在后面的示例中看到。

在本部分中，我们将比较两个执行相同操作的示例 Dockerfiles。 但是，一个 Dockerfile 对每个操作有一个指令，而另一个则将其相关操作分组在一起。

以下取消组合示例 Dockerfile 下载 Python for Windows，安装它，并在安装完成后删除下载的安装文件。 在此 Dockerfile 中，每个操作都具有`RUN`自己的说明。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

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

第二个示例是执行完全相同的操作的 Dockerfile。 但是，所有相关操作均按单个`RUN`指令分组。 `RUN`指令中的每个步骤均位于 Dockerfile 的新行中，而 "\" 字符用于换行。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

生成的图像只有一个额外的`RUN`图层。

```dockerfile
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB
```

### <a name="remove-excess-files"></a>删除多余文件

如果你的 Dockerfile 中有一个文件（如安装程序）在使用后不需要，可以将其删除以减小图像大小。 将文件复制到映像层的同一个步骤也需要此操作。 这样做会阻止文件在较低级别的图像图层中保留。

在以下示例 Dockerfile 中，将下载、执行并删除 "Python" 程序包。 在一个 `RUN` 操作中就可以完成以上所有步骤，从而产生单个映像层。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

## <a name="optimize-build-speed"></a>优化生成速度

### <a name="multiple-lines"></a>多行

你可以将操作拆分为多个单独的说明，以优化 Docker 生成速度。 由于`RUN`为每条`RUN`指令创建单独的图层，因此多项操作提高了缓存效率。 如果在不同的 Docker 生成操作中已运行相同的指令，则会重复使用此缓存操作（图像层），从而减少放大的 Docker 生成运行时。

在以下示例中，通过删除不再需要的文件，可以下载、安装和清理 Apache 和 Visual Studio 重新发布程序包。 这一切都是通过一条`RUN`指令完成的。 如果更新这些操作中的任何一个，将重新运行所有操作。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

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

生成的图像具有两个层，一个用于基操作系统映像，另一个包含来自单个`RUN`指令的所有操作。

```dockerfile
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

通过比较，下面是与三个`RUN`指令分离的相同操作。 在这种情况下`RUN` ，每个指令都缓存在一个容器图像层中，只有已更改的指令才需要在后续的 Dockerfile 内部版本上运行。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

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

生成的图像包含四个图层;一个图层，基本 OS 映像和三`RUN`个说明中的每一个。 由于每`RUN`个指令都在其自己的层中运行，因此在不同的 Dockerfile 中任何后续运行此 Dockerfile 或相同的指令集将使用缓存的图像层，从而减少生成时间。

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

在使用图像缓存时，如何对说明进行排序非常重要，如下一节中所示。

### <a name="ordering-instructions"></a>排序说明

从顶到底处理 Dockerfile，针对缓存层比较每条指令。 当找到一条不含缓存层的指令后，会在新的容器映像层处理此指令和所有后续指令。 因此，指令放置的顺序非常重要。 将保持不变的指令放置在 Dockerfile 顶部。 将可能改变的指令放置在 Dockerfile 底部。 这样就降低了取消现有缓存的可能性。

以下示例显示了 Dockerfile 指令排序如何影响缓存的有效性。 此简单示例 Dockerfile 具有四个编号的文件夹。  

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```

生成的映像有五个层，一个用于基本 OS 映像，另一个`RUN`说明。

```dockerfile
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B
```

下一个 Dockerfile 现在已略微修改，第三`RUN`条指令更改为新文件。 当针对此 Dockerfile 运行 Docker build 时，与上个示例中相同的前三条指令使用缓存的映像层。 但是，由于不会`RUN`缓存更改的指令，因此会为已更改的指令和所有后续说明创建新的图层。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

将新图像的图像 Id 与本部分第一个示例中的图像 Id 进行比较时，你将注意到从下到上的前三层是共享，但第四和第五层是唯一的。

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY               SIZE                COMMENT
c92cc95632fb        28 seconds ago      cmd /S /C mkdir test-4   5.644 MB
2f05e6f5c523        37 seconds ago      cmd /S /C mkdir test-5   5.01 MB
68fda53ce682        3 minutes ago       cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        4 minutes ago       cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                 0 B
```

## <a name="cosmetic-optimization"></a>修饰优化

### <a name="instruction-case"></a>说明案例

Dockerfile 说明不区分大小写，但约定是使用大写字母。 这通过区分指令调用和指令操作来提高可读性。 以下两个示例比较了 uncapitalized 和大写的 Dockerfile。

以下是 uncapitalized Dockerfile：

```dockerfile
# Sample Dockerfile

from mcr.microsoft.com/windows/servercore:ltsc2019
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```

下面是使用大写字母的相同 Dockerfile：

```dockerfile
# Sample Dockerfile

FROM mcr.microsoft.com/windows/servercore:ltsc2019
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### <a name="line-wrapping"></a>换行

长且复杂的操作可通过反斜杠`\`字符分隔到多行。 以下 Dockerfile 安装了 Visual Studio 可再发行组件包，删除了安装程序文件，然后创建了配置文件。 这三个操作都是在一行上指定的。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```

该命令可以用反斜杠进行中断，以便在单独的行中`RUN`指定每个指令中的每个操作。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## <a name="further-reading-and-references"></a>更多阅读和参考

[Windows 上的 Dockerfile](manage-windows-dockerfile.md)

[在 Docker.com 上编写 Dockerfile 的最佳实践](https://docs.docker.com/engine/reference/builder/)
