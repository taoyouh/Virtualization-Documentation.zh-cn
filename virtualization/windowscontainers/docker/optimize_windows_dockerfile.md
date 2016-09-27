---
title: "优化 Windows Dockerfile"
description: "优化用于 Windows 容器的 Dockerfile。"
keywords: "docker, 容器"
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb2848ca-683e-4361-a750-0d1d14ec8031
translationtype: Human Translation
ms.sourcegitcommit: f721639b1b10ad97cc469df413d457dbf8d13bbe
ms.openlocfilehash: 59f096ce914e0f1cbee5678b7db980297ca89364

---
# 优化 Windows Dockerfile

有几种方法可用来优化 Docker build 过程和生成的 Docker 映像。 本文档详细介绍了 Docker build 过程的操作原理，并演示了使用 Windows 容器创建最佳映像所用的几种策略。

## Docker Build

### 映像层

在检查 Docker build 优化之前，请务必了解 Docker build 的工作原理。 在 Docker build 过程中，会占用 Dockerfile 并且在其自身临时的容器中一对一地运行每个可操作的指令。 结果是，每个可操作的指令都有一个新映像层。 

看一看以下 Dockerfile。 在此示例中，正在使用 `windowsservercore` 基本操作系统映像，安装 IIS，然后创建一个简单的网站。

```none
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

从此 Dockerfile 来看，可预计生成的映像含有两个层：一个用于容器操作系统映像，另一个包括 IIS 和网站，但是这里不是这样的。 新映像由多个层构成，每个层依赖于前者。 若要形象地说明这一点，可以针对新映像运行 `docker history` 命令。 这样可以显示映像由四个层组成：基层和其他三个附加层。每个层在 Dockerfile 中都对应一条指令。

```none
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

每个层都可以映射到 Dockerfile 的一条指令。 底层（本示例中为 `6801d964fda5`）代表基本操作系统映像。 上面一层，可以看到 IIS 安装。 下一层包括新的网站，依次类推。

可以编写 Dockerfiles，以最小化映像层、优化生成性能，也可以优化修饰性的项目，例如可读性等。 完成相同的映像生成任务基本上有多种方式。 了解 Dockerfile 的格式如何影响生成时间和生成的映像，以及如何提升自动化体验。 

## 优化映像大小

构建 Docker 容器映像时，映像大小可能是一个重要因素。 容器映像在注册表和主机之间移动、导出和导入，最终占用了空间。 在 Docker build 过程中，可采用几种策略尽可能减小映像大小。 本部分详细介绍一些特定于 Windows 容器的策略。 

有关 Dockerfile 最佳做法的附加信息，请参阅 [Best practices for writing Dockerfiles on Docker.com]( https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)（在 Docker.com 上编写 Dockerfile 的最佳做法）。

### 组相关的操作

因为每个 `RUN` 指令在容器映像中创建新的层，将操作分组为一个 `RUN` 指令可以减少层数。 而最小化层可能不太会影响映像的大小，而对相关操作进行分组则可以，这将在后面的示例中看到。

下面的两个示例演示相同的操作，这将产生相同容量的容器映像，但两个 Dockerfiles 的构造却不相同。 同时也比较了生成的映像。  

第一个示例下载并安装 Python for Windows，然后通过删除下载的安装程序文件进行清理。 每个操作都按其自身的 `RUN` 指令运行。

```none
FROM windowsservercore

RUN powershell.exe -Command Invoke-WebRequest "https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe" -OutFile c:\python-3.5.1.exe
RUN powershell.exe -Command Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait
RUN powershell.exe -Command Remove-Item c:\python-3.5.1.exe -Force
```

生成的映像由三个附加层组成，每一个对应一个 `RUN` 指令。

```none
docker history doc-example-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
a395ca26777f        15 seconds ago      cmd /S /C powershell.exe -Command Remove-Item   24.56 MB
6c137f466d28        28 seconds ago      cmd /S /C powershell.exe -Command Start-Proce   178.6 MB
957147160e8d        3 minutes ago       cmd /S /C powershell.exe -Command Invoke-WebR   125.7 MB
```

为了进行比较，此处采用相同的操作，但所有步骤都采用相同的 `RUN` 指令运行。 请注意，`RUN` 指令中的每个步骤位于 Dockerfile 新的一行，'\' 字符用于换行。 

```none
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

此处生成的映像由 `RUN` 指令的一个附加层组成。

```none
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB                
```

### 删除多余文件

如果某个文件（例如安装程序）在用完之后不再需要，请删除该文件以减小映像大小。 将文件复制到映像层的同一个步骤也需要此操作。 这样可以防止文件留在较低级别的映像层。

在此示例中，会下载和执行 Python 包执，然后删除可执行文件。 在一个 `RUN` 操作中就可以完成以上所有步骤，从而产生单个映像层。

```none
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

## 优化生成速度

### 多行

在优化 Docker build 速度时，将操作拆分为多个单独指令可能会更有利。 多个 `RUN` 操作会增加缓存有效性。 因为每个 `RUN` 指令会创建单个层，如果在其他 Docker Build 操作中已运行相同的步骤，则此缓存的操作（映像层）会被重复使用。 结果，减少了 Docker Build 运行时。

在下面的示例中，下载、安装了 Apache 和 Visual Studio 重新分发包，然后清理不需要的文件。 这些只需一个 `RUN` 指令就可全部完成。 如果下列任一操作更新时，将重新运行所有的操作。

```none
FROM windowsservercore

RUN powershell -Command \
    
  # Download software ; \
    
  wget https://www.apachelounge.com/download/VC11/binaries/httpd-2.4.18-win32-VC11.zip -OutFile c:\apache.zip ; \
  wget "https://download.microsoft.com/download/1/6/B/16B06F60-3B20-4FF2-B699-5E9B7962F9AE/VSU_4/vcredist_x86.exe" -OutFile c:\vcredist.exe ; \
  wget -Uri http://windows.php.net/downloads/releases/php-5.5.33-Win32-VC11-x86.zip -OutFile c:\php.zip ; \
    
  # Install Software ; \
    
  Expand-Archive -Path c:\php.zip -DestinationPath c:\php ; \
  Expand-Archive -Path c:\apache.zip -DestinationPath c:\ ; \
  start-Process c:\vcredistexe -ArgumentList '/quiet' -Wait ; \
    
  # Remove unneeded files ; \
     
  Remove-Item c:\apache.zip -Force; \
  Remove-Item c:\vcredist.exe -Force
```

生成的映像由两个层组成，一个用于基本操作系统映像，第二个包含单个 `RUN` 指令中的所有操作。

```none
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

为了便于对比，此处将相同的操作分解为三个 `RUN` 指令。 在这种情况下，每个 `RUN` 指令缓存在容器映像层，并且只有已经发生更改的指令需要在以后的 Dockerfile build 中重新运行。

```none
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

生成的映像由四个层组成，一个用于基本操作系统映像，然后另外三个分别针对一条 `RUN` 指令。 因为每个 `RUN` 指令已在自身的层中运行，所以该 Dockerfile 或其他 Dockerfile 中的相同指令集的任何后续运行都将使用缓存的映像层，从而减少了生成时间。 指令排序在使用映像缓存过程中十分重要，有关详细信息，请参阅本文档的下一节。

```none
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

### 指令排序

从顶到底处理 Dockerfile，针对缓存层比较每条指令。 当找到一条不含缓存层的指令后，会在新的容器映像层处理此指令和所有后续指令。 因此，指令放置的顺序非常重要。 将保持不变的指令放置在 Dockerfile 顶部。 将可能改变的指令放置在 Dockerfile 底部。 这样就降低了取消现有缓存的可能性。

此示例的目的是为了演示 Dockerfile 指令排序如何影响缓存有效性。 在此简单 Dockerfile 中，将创建四个带编号的文件夹。  

```none
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```
生成的映像有五个层，一个用于基本操作系统映像，然后另外四个分别针对一条 `RUN` 指令。

```none
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B    
```

Docker 文件现在已经过稍许修改。 请注意，第三条 `RUN` 指令已更改。 当针对此 Dockerfile 运行 Docker build 时，与上个示例中相同的前三条指令使用缓存的映像层。 但是，因为更改的 `RUN` 指令尚未缓存，因此为自身和所有后续指令创建了一个新的层。

```none
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

将新映像的映像 ID 与上个示例中的相比较，你将看到共享的是前三个层（底部到顶部），但是第四个和第五个是唯一的。

```none
docker history doc-sample-2

IMAGE               CREATED             CREATED BY               SIZE                COMMENT
c92cc95632fb        28 seconds ago      cmd /S /C mkdir test-4   5.644 MB
2f05e6f5c523        37 seconds ago      cmd /S /C mkdir test-5   5.01 MB
68fda53ce682        3 minutes ago       cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        4 minutes ago       cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                 0 B
```

## 表面优化

### 指令用例

虽然约定是使用大写形式，但是 Dockerfile 指令不区分大小写。 通过在指令调用和指令操作之间引入差异，提高了可读性。 下面两个示例演示了这一概念。 

小写：
```none
# Sample Dockerfile

from windowsservercore
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```
大写： 
```none
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### 换行

使用反斜杠 `\` 字符可以将长而复杂的操作分隔到多个行上。 以下 Dockerfile 安装了 Visual Studio 可再发行组件包，删除了安装程序文件，然后创建了配置文件。 这三个操作都是在一行上指定的。

```none
FROM windowsservercore

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```
命令可以重新编写，这样一个 `RUN` 指令的每个操作都可以自己的行上指定。 

```none
FROM windowsservercore

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## 深度阅读和参考

[Dockerfile on Windows] (./manage_windows_dockerfile.md)

[在 Docker.com 上编写 Dockerfile 的最佳做法](https://docs.docker.com/engine/reference/builder/)



<!--HONumber=Sep16_HO4-->


