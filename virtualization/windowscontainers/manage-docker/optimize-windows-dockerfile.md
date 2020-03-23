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
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910147"
---
# <a name="optimize-windows-dockerfiles"></a>优化 Windows Dockerfile

可以使用许多方法来优化 Docker build 过程和生成的 Docker 映像。 本文介绍 Docker build 过程的原理，以及如何以最佳方式为 Windows 容器创建映像。

## <a name="image-layers-in-docker-build"></a>Docker build 中的映像层

在优化 Docker build 之前，需要了解 Docker build 的原理。 在 Docker build 过程中，会占用 Dockerfile 并且在其自身临时的容器中一对一地运行每个可操作的指令。 结果是，每个可操作的指令都有一个新映像层。

例如，以下示例 Dockerfile 使用 `mcr.microsoft.com/windows/servercore:ltsc2019` 基础 OS 映像，在安装 IIS 后创建了一个简单的网站。

```dockerfile
# Sample Dockerfile

FROM mcr.microsoft.com/windows/servercore:ltsc2019
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

可以预期该 Dockerfile 生成的映像会有两个层：一个层用于容器 OS 映像，另一个包含 IIS 和网站。 但是，实际映像有多个层，每个层都依赖于其前面的层。

为了更清楚地说明这一点，让我们对示例 Dockerfile 创建的映像运行 `docker history` 命令。

```dockerfile
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

输出显示，此映像有四个层：一个基础层和三个附加层，分别映射到 Dockerfile 中的每个指令。 底层（本示例中为 `6801d964fda5`）代表基本操作系统映像。 再上一层是 IIS 安装。 下一层包括新的网站，依次类推。

可以在编写 Dockerfiles 时最小化映像层、优化生成性能，以及优化从可访问性到可读性的一系列功能。 完成相同的映像生成任务基本上有多种方式。 了解 Dockerfile 的格式如何影响生成时间，以及它所创建的映像如何改进自动化体验。

## <a name="optimize-image-size"></a>优化映像大小

映像大小可能是生成 Docker 容器映像时的一个重要因素，具体取决于空间要求。 容器映像在注册表和主机之间移动、导出和导入，最终占用了空间。 此部分将介绍如何在 Windows 容器的 Docker build 过程中尽量缩小映像。

有关 Dockerfile 最佳做法的其他信息，请参阅 [Docker.com 上的有关如何编写 Dockerfile 的最佳做法](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)。

### <a name="group-related-actions"></a>组相关的操作

在 Dockerfile 中，由于每个 `RUN` 指令会在容器映像中创建新的层，因此将操作组合成一个 `RUN` 指令可以减少层数。 而最小化层可能不太会影响映像的大小，而对相关操作进行分组则可以，这将在后面的示例中看到。

在此部分，我们将比较两个执行相同操作的示例 Dockerfile。 不过，一个 Dockerfile 会为每个操作设置一个指令，而另一个 Dockerfile 则将其相关操作组合到一起。

下面这个未组合的示例 Dockerfile 会下载并安装 Python for Windows，并会在安装完成后删除下载的安装程序文件。 在此 Dockerfile 中，每个操作都会获得自己的 `RUN` 指令。

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

第二个示例是执行完全相同的操作的 Dockerfile。 但是，所有相关的操作都已组合到单个 `RUN` 指令下。 `RUN` 指令中的每个步骤在 Dockerfile 中各占一行，而“\\”字符则用于换行。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

就 `RUN` 指令来说，生成的映像只有一个附加层。

```dockerfile
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB
```

### <a name="remove-excess-files"></a>删除多余文件

如果 Dockerfile 中有一个文件（例如安装程序）该文件在用完之后就不再需要，则可删除它以减小映像。 将文件复制到映像层的同一个步骤也需要此操作。 这样做可以防止文件留在较低级别的映像层。

在下面的示例 Dockerfile 中，Python 包在下载并执行后就会被删除。 在一个 `RUN` 操作中就可以完成以上所有步骤，从而产生单个映像层。

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

可以将操作拆分成多个单独的指令，优化 Docker build 速度。 由于为每个 `RUN` 指令创建了单独的层，因此多个 `RUN` 操作会提高缓存效率。 如果同一指令已在另一 Docker Build 操作中运行，则会重复使用该缓存的操作（映像层），从而缩短 Docker build 运行时。

在以下示例中，我们下载并安装了 Apache 和 Visual Studio 可再发行包，然后通过删除不再需要的文件来进行清理。 只需单个 `RUN` 指令即可完成所有这些操作。 如果更新下列任一操作，则会重新运行所有操作。

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

生成的映像有两个层，一个用于基础 OS 映像，另一个包含单个 `RUN` 指令中的所有操作。

```dockerfile
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

与之形成对比的是，下面是拆分成三个 `RUN` 指令的相同操作。 在此示例中，每个 `RUN` 指令缓存在容器映像层，只有已经发生更改的指令需要在后续 Dockerfile build 中重新运行。

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

生成的映像由四个层组成，一个层用于基础 OS 映像，另外三个层分别对应于三条 `RUN` 指令。 因为每个 `RUN` 指令已在自身的层中运行，所以该 Dockerfile 或其他 Dockerfile 中的相同指令集的任何后续运行都将使用缓存的映像层，减少生成时间。

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

在使用映像缓存时，如何对指令排序非常重要，如下一部分所示。

### <a name="ordering-instructions"></a>指令排序

从顶到底处理 Dockerfile，针对缓存层比较每条指令。 当找到一条不含缓存层的指令后，会在新的容器映像层处理此指令和所有后续指令。 因此，指令放置的顺序非常重要。 将保持不变的指令放置在 Dockerfile 顶部。 将可能改变的指令放置在 Dockerfile 底部。 这样就降低了取消现有缓存的可能性。

以下示例演示 Dockerfile 指令排序如何影响缓存有效性。 这个简单的示例 Dockerfile 有四个编号文件夹。  

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```

生成的映像有五个层，一个用于基础 OS 映像，另外四个分别对应于四条 `RUN` 指令。

```dockerfile
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B
```

接下来的这个 Dockerfile 现在已略微进行了修改，第三个 `RUN` 指令已更改为新文件。 当针对此 Dockerfile 运行 Docker build 时，与上个示例中相同的前三条指令使用缓存的映像层。 但是，由于更改的 `RUN` 指令未缓存，因此为更改的指令和所有后续指令创建了一个新的层。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

将新映像的映像 ID 与此部分的第一个示例中的相比较，你会注意到共享的是前三个层（从下到上），而第四层和第五层则是独一无二的。

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

Dockerfile 指令不区分大小写，虽然约定使用大写形式。 这样可以区分指令调用和指令操作，提高了可读性。 以下两个示例比较了非大写的和大写的 Dockerfile。

下面是一个非大写的 Dockerfile：

```dockerfile
# Sample Dockerfile

from mcr.microsoft.com/windows/servercore:ltsc2019
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```

下面是同一个 Dockerfile，但使用了大写：

```dockerfile
# Sample Dockerfile

FROM mcr.microsoft.com/windows/servercore:ltsc2019
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### <a name="line-wrapping"></a>换行

可以通过反斜杠 (`\`) 字符将长而复杂的操作分隔成多个行。 以下 Dockerfile 安装了 Visual Studio 可再发行组件包，删除了安装程序文件，然后创建了配置文件。 这三个操作都是在一行上指定的。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```

命令可以使用反斜杠进行拆分，使一个 `RUN` 指令中的每个操作都可以在自己的行中指定。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## <a name="further-reading-and-references"></a>更深入的阅读和参考

[Windows 上的 Dockerfile](manage-windows-dockerfile.md)

[Docker.com 上的有关如何编写 Dockerfile 的最佳做法](https://docs.docker.com/engine/reference/builder/)
