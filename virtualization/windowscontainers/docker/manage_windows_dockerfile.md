---
title: "Dockerfile 和 Windows 容器"
description: "创建用于 Windows 容器的 Dockerfile。"
keywords: "docker, 容器"
author: PatrickLang
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 75fed138-9239-4da9-bce4-4f2e2ad469a1
translationtype: Human Translation
ms.sourcegitcommit: 31515396358c124212b53540af8a0dcdad3580e4
ms.openlocfilehash: 20dcc6d263488673bf0a025058c3dee8d30168a2

---

# Windows 上的 Dockerfile

Docker 引擎包含用于自动创建容器映像的工具。 尽管可以使用 `docker commit` 命令手动创建容器映像，然而采用自动映像创建过程可获得许多好处，其中包括：

- 将容器映像存储为代码。
- 可出于维护和升级的目的快速而精确地重新创建容器映像。
- 容器映像和开发周期之间的持续集成。

驱动实现这一自动化过程的 Docker 组件是 Dockerfile，以及 `docker build` 命令。

- **Dockerfile** - 一个文本文件，包含创建新容器映像所需的指令。 这些指令包括对将用作基础的现有映像的标识、将在映像创建过程中运行的命令以及部署容器映像的新实例时将要运行的命令。
- **Docker build** - 使用 Dockerfile 并触发映像创建过程的 Docker 引擎命令。

本文档将介绍将 Dockerfile 用于 Windows 容器的相关内容、讨论语法并详细介绍常用的 Dockerfile 指令。 

本文档将通篇讨论容器映像和容器映像层的概念。 有关映像和映像分层的详细信息，请参阅[管理 Windows 容器映像](../management/manage_images.md)。 

有关 Dockerfile 的完整详细信息，请参阅 [docker.com 上的 Dockerfile 参考]( https://docs.docker.com/engine/reference/builder/)。

## Dockerfile 指令

### 基本语法

Dockerfile 的最基本形式十分简单。 下面的示例创建了一个新映像，其中包括 IIS 和一个“hello world”站点。 此示例包含了用于解释每个步骤的注释（通过 `#` 指示）。 本文的后续部分将对 Dockerfile 语法规则和 Dockerfile 指令进行更详细地讲解。

```none
# Sample Dockerfile

# Indicates that the windowsservercore image will be used as the base image.
FROM microsoft/windowsservercore

# Metadata indicating an image maintainer.
MAINTAINER jshelton@contoso.com

# Uses dism.exe to install the IIS role.
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart

# Creates an HTML file and adds content to this file.
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html

# Sets a command or process that will run each time a container is run from the new image.
CMD [ "cmd" ]
```

有关用于 Windows 的 Dockerfile 的其他示例，请参阅 [用于 Windows 的 Dockerfile 存储库] (https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples)。

## 说明

Dockerfile 指令为 Docker 引擎提供创建容器映像所需的步骤。 这些指令按顺序逐一执行。 以下是有关一些基本 Dockerfile 指令的详细信息。 有关 Dockerfile 指令的完整列表，请参阅 [Docker.com 上的 Dockerfile 参考] (https://docs.docker.com/engine/reference/builder/)。

### FROM

`FROM` 指令用于设置在新映像创建过程期间将使用的容器映像。 例如，使用指令 `FROM windowsservercore` 时，所得到的映像派生自 Windows Server Core 基本操作系统映像映像并具有对其的依赖关系。 如果正在进行 Docker 生成过程的系统上不存在指定的映像，Docker 引擎将尝试从公有或私有映像注册表下载该映像。

**格式**

FROM 指令所采用的格式为： 

```
FROM <image>
```

**示例**

```
FROM windowsservercore
```

有关 FROM 指令的详细信息，请参阅 [Docker.com 上的 FROM 参考]( https://docs.docker.com/engine/reference/builder/#from)。 

### RUN

`RUN` 指令指定将要运行并捕获到新容器映像中的命令。 这些命令包括安装软件、创建文件和目录，以及创建环境配置等。

**格式**

RUN 指令所采用的格式为： 

```none
# exec form

RUN ["<executable", "<param 1>", "<param 2>"]

# shell form

RUN <command>
```

Exec 与 Shell 窗体之间的区别在于 `RUN` 指令执行的方式。 使用 exec 窗体时，指定的程序显式运行。 

以下示例使用了 exec 窗体。

```none
FROM windowsservercore

RUN ["powershell", "New-Item", "c:/test"]
```

检查生成的映像，所运行的命令是 `powershell New-Item c:/test`。

```none
docker history doc-exe-method

IMAGE               CREATED             CREATED BY                    SIZE                COMMENT
b3452b13e472        2 minutes ago       powershell New-Item c:/test   30.76 MB
```

为进行比较，下面的示例运行相同的操作，但使用的是 shell 窗体。

```none
FROM windowsservercore

RUN powershell New-Item c:\test
```

这将导致运行指令 `cmd /S /C powershell New-Item c:\test`。 

```none
docker history doc-shell-method

IMAGE               CREATED             CREATED BY                              SIZE                COMMENT
062a543374fc        19 seconds ago      cmd /S /C powershell New-Item c:\test   30.76 MB
```

**Windows 注意事项**
 
在 Windows 上，使用具有 exec 格式的 `RUN` 指令时，反斜杠必须进行转义。

```none
RUN ["powershell", "New-Item", "c:\\test"]
```

当目标程序是 Windows Installer 时，在启动实际（无提示）安装过程之前需要执行一个额外步骤：通过 `/x:<directory>` 标志，提取安装程序。 此外，需要等待此命令退出。 否则，安装过程将在未安装任何内容的情况下提前结束。 有关详细信息，请参阅以下示例。

**示例**

此示例使用 DISM 在容器映像中安装 IIS。
```none
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart
```

此示例安装 Visual Studio 可再发行组件包。 请注意，`Start-Process` 和 `-Wait` 参数用于运行安装程序。 以确保在完成安装后再移动到 Dockerfile 中的第二步。

```none
RUN Start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait
``` 

有关 RUN 指令的详细信息，请参阅 [Docker.com 上的 RUN 参考]( https://docs.docker.com/engine/reference/builder/#run)。 

### 复制

`COPY` 指令将文件和目录复制到容器的文件系统。 文件和目录需位于相对于 Dockerfile 的路径中。

**格式**

`COPY` 指令所采用的格式为： 

```none
COPY <source> <destination>
``` 

如果源或目标包含空格，请将路径括在方括号和双引号中。
 
```none
COPY ["<source>", "<destination>"]
```

**Windows 注意事项**
 
在 Windows 上，目标格式必须使用正斜杠。 例如，以下是有效的 `COPY` 指令。

```none
COPY test1.txt /temp/
COPY test1.txt c:/temp/
```

但是，以下指令不起作用。

```none
COPY test1.txt c:\temp\
```

**示例**

此示例将源目录的内容添加到容器映像中一个名为 `sqllite` 的目录。
```none
COPY source /sqlite/
```

此示例会将以 config 开头的所有文件添加到容器映像的 `c:\temp` 目录中。
```none
COPY config* c:/temp/
```

有关 `COPY` 指令的详细信息，请参阅 [COPY Reference on Docker.com]( https://docs.docker.com/engine/reference/builder/#copy)（Docker.com上的 COPY 参考）。

### 添加

ADD 指令与 COPY 指令非常类似；但它包含更多功能。 除了将文件从主机复制到容器映像，`ADD` 指令还可以使用 URL 规范从远程位置复制文件。

**格式**

`ADD` 指令所采用的格式为： 

```none
ADD <source> <destination>
``` 

如果源或目标包含空格，请将路径括在方括号和双引号中。
 
```none
ADD ["<source>", "<destination>"]
```

**Windows 注意事项**
 
在 Windows 上，目标格式必须使用正斜杠。 例如，以下是有效的 `ADD` 指令。

```none
ADD test1.txt /temp/
ADD test1.txt c:/temp/
```

但是，以下指令不起作用。

```none
ADD test1.txt c:\temp\
```

此外，在 Linux 上 `ADD` 指令将在复制时展开压缩包。 此功能在 Windows 中不可用。

**示例**

此示例将源目录的内容添加到容器映像中一个名为 `sqllite` 的目录。
```none
ADD source /sqlite/
```

此示例会将以 config 开头的所有文件添加到容器映像的 `c:\temp` 目录中。
```none
ADD config* c:/temp/
```

此示例会将 Python for Windows下载到容器映像的 `c:\temp` 目录。
```none
ADD https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe /temp/python-3.5.1.exe
```

有关 `ADD` 指令的详细信息，请参阅 [Docker.com 上的 ADD 参考]( https://docs.docker.com/engine/reference/builder/#add)。 

### WORKDIR

`WORKDIR` 指令用于为其他 Dockerfile 指令（如 `RUN`、`CMD`）设置一个工作目录，并且还设置用于运行容器映像实例的工作目录。

**格式**

`WORKDIR` 指令所采用的格式为： 

```none
WORKDIR <path to working directory>
``` 

**Windows 注意事项**

在 Windows 上，如果工作目录包含一个反斜杠，则必须对其进行转义。

```none
WORKDIR c:\\windows
```

**示例**

```none
WORKDIR c:\\Apache24\\bin
```

有关 `WORKDIR` 指令的详细信息，请参阅 [Docker.com 上的 WORKDIR 参考]( https://docs.docker.com/engine/reference/builder/#workdir)。 

### CMD

`CMD` 指令用于设置部署容器映像的实例时要运行的默认命令。 例如，如果该容器将承载 NGINX Web 服务器，则 `CMD` 可能包括用于启动 Web 服务器的指令，如 `nginx.exe`。 如果 Dockerfile 中指定了多个 `CMD` 指令，只会计算最后一个指令。

**格式**

`CMD` 指令所采用的格式为： 

```none
# exec form

CMD ["<executable", "<param>"]

# shell form

CMD <command>
```

**Windows 注意事项**

在 Windows 上，在 `CMD` 指令中指定的文件路径必须使用正斜杠或已转义的反斜杠 `\\`。 例如，这些是有效的 `CMD` 指令。

```none
# exec form

CMD ["c:\\Apache24\\bin\\httpd.exe", "-w"]

# shell form

CMD c:\\Apache24\\bin\\httpd.exe -w
```
但是，以下指令不起作用。

```none
CMD c:\Apache24\bin\httpd.exe -w
```

有关 `CMD` 指令的详细信息，请参阅 [Docker.com 上的 CMD 参考]( https://docs.docker.com/engine/reference/builder/#cmd)。 

## 转义字符

在许多情况下，Dockerfile 指令需要跨多个行；这可通过转义字符完成。 默认 Dockerfile 转义字符是反斜杠 `\`。 由于反斜杠在 Windows 中也是一个文件路径分隔符，这可能导致出现问题。 要更改默认转义字符，可使用一个分析程序指令。 有关分析程序指令的详细信息，请参阅 [Parser Directives on Docker.com]( https://docs.docker.com/engine/reference/builder/#parser-directives)（Docker.com 上的分析程序指令）。

以下示例显示使用默认转义字符跨多个行的单个 RUN 指令。

```none
FROM windowsservercore

RUN powershell.exe -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
    Remove-Item c:\python-3.5.1.exe -Force
```

要修改转义字符，请在 Dockerfile 最开始的行上放置一个转义分析程序指令。 如以下示例所示。

> 请注意，只有两个值可用作转义字符：`\` 和 `` ` ``。

```none
# escape=`

FROM windowsservercore

RUN powershell.exe -Command `
    $ErrorActionPreference = 'Stop'; `
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; `
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; `
    Remove-Item c:\python-3.5.1.exe -Force
```

有关转义分析程序指令的详细信息，请参阅 [Escape Parser Directive on Docker.com]( https://docs.docker.com/engine/reference/builder/#escape)（Docker.com 上的转义分析程序指令）。

## Dockerfile 中的 PowerShell

### PowerShell 命令

可使用 `RUN` 操作在 Dockerfile 中运行 PowerShell 命令。 

```none
FROM windowsservercore

RUN powershell -command Expand-Archive -Path c:\apache.zip -DestinationPath c:\
```

### REST 调用

当从 Web 服务收集信息或文件时，PowerShell 与 `Invoke-WebRequest` 命令会很有用。 例如，如果要构建包括 Python 的映像，可以使用下面的示例。

```none
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

> Nano Server 中当前不支持 Invoke-WebRequest

有关在映像创建过程期间使用 PowerShell 下载文件的另一个选择是使用 .NET WebClient 库。 这可以增加下载性能。 下面的示例使用 WebClient 库下载 Python 软件。

```none
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  (New-Object System.Net.WebClient).DownloadFile('https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe','c:\python-3.5.1.exe') ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

> Nano Server 中当前不支持 WebClient

### PowerShell 脚本

在某些情况下，执行这样的操作可能会有所帮助：将脚本复制到映像创建过程期间使用的容器中，然后从该容器内运行脚本。 请注意 - 这会限制任何映像层缓存，并降低 Dockerfile 的可读性。

此示例使用 `ADD` 指令将脚本从生成计算机复制到容器中。 然后，此脚本使用 RUN 指令运行。

```
FROM windowsservercore
ADD script.ps1 /windows/temp/script.ps1
RUN powershell.exe -executionpolicy bypass c:\windows\temp\script.ps1
```

## Docker Build 

创建 Dockerfile 并将其保存到磁盘后，即可运行 `docker build` 以创建新映像。 `docker build` 命令采用几个可选参数和指向 Dockerfile 的路径。 有关 Docker Build 的完整文档，包括所有生成选项的列表，请参阅 [build Reference on Docker.com](https://docs.docker.com/engine/reference/commandline/build/#build)（Docker.com 上的 Build 参考）。

```none
Docker build [OPTIONS] PATH
```
例如，以下命令将创建名为“iis”的映像。

```none
docker build -t iis .
```

生成过程启动后，输出将指示状态，并返回任何引发的错误。

```none
C:\> docker build -t iis .

Sending build context to Docker daemon 2.048 kB
Step 1 : FROM windowsservercore
 ---> 6801d964fda5

Step 2 : RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
 ---> Running in ae8759fb47db

Deployment Image Servicing and Management tool
Version: 10.0.10586.0

Image Version: 10.0.10586.0

Enabling feature(s)
The operation completed successfully.

 ---> 4cd675d35444
Removing intermediate container ae8759fb47db

Step 3 : RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
 ---> Running in 9a26b8bcaa3a
 ---> e2aafdfbe392
Removing intermediate container 9a26b8bcaa3a

Successfully built e2aafdfbe392
```

其结果是一个新的容器映像，在此示例中名为“iis”。

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
iis                 latest              e2aafdfbe392        About a minute ago   207.8 MB
windowsservercore   latest              6801d964fda5        4 months ago         0 B
```

## 深度阅读和参考

[优化用于 Windows 的 Dockerfile 和 Docker Build] (./optimize_windows_dockerfile.md)

[Docker.com 上的 Dockerfile 参考](https://docs.docker.com/engine/reference/builder/)



<!--HONumber=Nov16_HO1-->


