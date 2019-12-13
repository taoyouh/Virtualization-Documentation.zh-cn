---
title: Dockerfile 和 Windows 容器
description: 创建用于 Windows 容器的 Dockerfile。
keywords: docker, 容器
author: PatrickLang
ms.date: 05/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 75fed138-9239-4da9-bce4-4f2e2ad469a1
ms.openlocfilehash: 9fef74c029dc3efc220b1f9924d2695cdbaa61be
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909657"
---
# <a name="dockerfile-on-windows"></a>Windows 上的 Dockerfile

Docker 引擎包括自动创建容器映像的工具。 尽管可以通过运行 `docker commit` 命令来手动创建容器映像，但采用自动映像创建过程具有许多好处，包括：

- 将容器映像存储为代码。
- 可出于维护和升级的目的快速而精确地重新创建容器映像。
- 容器映像和开发周期之间的持续集成。

驱动实现这一自动化过程的 Docker 组件是 Dockerfile，以及 `docker build` 命令。

Dockerfile 是一个文本文件，其中包含创建新容器映像所需的说明。 这些指令包括对将用作基础的现有映像的标识、将在映像创建过程中运行的命令以及部署容器映像的新实例时将要运行的命令。

Docker build 是使用 Dockerfile 并触发映像创建过程的 Docker 引擎命令。

本主题将说明如何将 Dockerfile 与 Windows 容器一起使用、了解其基本语法以及最常见的 Dockerfile 指令。

本文档将讨论容器映像和容器映像层的概念。 如果要了解有关图像和图像分层的详细信息，请参阅[容器基本映像](../manage-containers/container-base-images.md)。

若要完整了解 Dockerfile，请参阅[Dockerfile 参考](https://docs.docker.com/engine/reference/builder/)。

## <a name="basic-syntax"></a>基本语法

Dockerfile 的最基本形式十分简单。 下面的示例创建了一个新映像，其中包括 IIS 和一个“hello world”站点。 此示例包含了用于解释每个步骤的注释（通过 `#` 指示）。 本文的后续部分将对 Dockerfile 语法规则和 Dockerfile 指令进行更详细的讲解。

>[!NOTE]
>必须创建不带扩展名的 Dockerfile。 若要在 Windows 中执行此操作，请在编辑器中创建所选的文件，并将其保存为 "Dockerfile" 符号（包括引号）。

```dockerfile
# Sample Dockerfile

# Indicates that the windowsservercore image will be used as the base image.
FROM mcr.microsoft.com/windows/servercore:ltsc2019

# Metadata indicating an image maintainer.
LABEL maintainer="jshelton@contoso.com"

# Uses dism.exe to install the IIS role.
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart

# Creates an HTML file and adds content to this file.
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html

# Sets a command or process that will run each time a container is run from the new image.
CMD [ "cmd" ]
```

有关 Dockerfile for Windows 的其他示例，请参阅[Dockerfile For windows repository](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples)。

## <a name="instructions"></a>说明

Dockerfile 指令为 Docker 引擎提供创建容器映像所需的说明。 这些指令逐个执行并按顺序执行。 以下示例是 Dockerfile 中最常使用的说明。 有关 Dockerfile 指令的完整列表，请参阅[Dockerfile 参考](https://docs.docker.com/engine/reference/builder/)。

### <a name="from"></a>FROM

`FROM` 指令用于设置在新映像创建过程期间将使用的容器映像。 例如，使用指令 `FROM mcr.microsoft.com/windows/servercore` 时，所得到的映像派生自 Windows Server Core 基本操作系统映像映像并具有对其的依赖关系。 如果正在进行 Docker 生成过程的系统上不存在指定的映像，Docker 引擎将尝试从公有或私有映像注册表下载该映像。

FROM 指令的格式如下所示：

```dockerfile
FROM <image>
```

下面是 FROM 命令的示例：

若要从 Microsoft 容器注册表（MCR）下载 ltsc2019 版本 windows server core：
```
FROM mcr.microsoft.com/windows/servercore:ltsc2019
```

有关更多详细信息，请参阅 "[从参考](https://docs.docker.com/engine/reference/builder/#from)"。

### <a name="run"></a>RUN

`RUN` 指令指定将要运行并捕获到新容器映像中的命令。 这些命令包括安装软件、创建文件和目录，以及创建环境配置等。

运行指令如下所示：

```dockerfile
# exec form

RUN ["<executable>", "<param 1>", "<param 2>"]

# shell form

RUN <command>
```

Exec 和 shell 窗体之间的区别在于 `RUN` 指令的执行方式。 使用 exec 窗体时，指定的程序显式运行。

下面是 exec 格式的示例：

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN ["powershell", "New-Item", "c:/test"]
```

生成的映像运行 `powershell New-Item c:/test` 命令：

```dockerfile
docker history doc-exe-method

IMAGE               CREATED             CREATED BY                    SIZE                COMMENT
b3452b13e472        2 minutes ago       powershell New-Item c:/test   30.76 MB
```

相反，下面的示例在 shell 窗体中运行相同的操作：

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell New-Item c:\test
```

生成的映像的运行指令为 `cmd /S /C powershell New-Item c:\test`。

```dockerfile
docker history doc-shell-method

IMAGE               CREATED             CREATED BY                              SIZE                COMMENT
062a543374fc        19 seconds ago      cmd /S /C powershell New-Item c:\test   30.76 MB
```

### <a name="considerations-for-using-run-with-windows"></a>使用 Windows 运行时的注意事项

在 Windows 上，使用具有 exec 格式的 `RUN` 指令时，反斜杠必须进行转义。

```dockerfile
RUN ["powershell", "New-Item", "c:\\test"]
```

如果目标程序是 Windows installer，则需要通过 `/x:<directory>` 标志提取安装程序，然后才能启动实际（无提示）安装过程。 还必须等待命令退出，然后再执行任何其他操作。 否则，该过程将提前结束，无需安装任何内容。 有关详细信息，请参阅以下示例。

#### <a name="examples-of-using-run-with-windows"></a>使用 Windows 运行的示例

下面的示例 Dockerfile 使用 DISM 在容器映像中安装 IIS：

```dockerfile
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart
```

此示例安装 Visual Studio 可再发行组件包。 `Start-Process` 和 `-Wait` 参数用于运行安装程序。 这可确保在转到 Dockerfile 中的下一条指令之前安装完成。

```dockerfile
RUN powershell.exe -Command Start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait
```

有关运行指令的详细信息，请参阅[运行引用](https://docs.docker.com/engine/reference/builder/#run)。

### <a name="copy"></a>复制

`COPY` 指令将文件和目录复制到容器的文件系统。 文件和目录必须位于相对于 Dockerfile 的路径中。

`COPY` 指令的格式如下所示：

```dockerfile
COPY <source> <destination>
```

如果源或目标包括空格，请将路径括在方括号和双引号中，如下面的示例中所示：

```dockerfile
COPY ["<source>", "<destination>"]
```

#### <a name="considerations-for-using-copy-with-windows"></a>使用 Windows 副本的注意事项

在 Windows 上，目标格式必须使用正斜杠。 例如，以下是有效 `COPY` 说明：

```dockerfile
COPY test1.txt /temp/
COPY test1.txt c:/temp/
```

同时，以下格式的反斜杠无效：

```dockerfile
COPY test1.txt c:\temp\
```

#### <a name="examples-of-using-copy-with-windows"></a>使用 Windows 副本的示例

下面的示例将源目录的内容添加到容器映像中名为 `sqllite` 的目录：

```dockerfile
COPY source /sqlite/
```

下面的示例将以 config 开头的所有文件添加到容器映像的 `c:\temp` 目录中：

```dockerfile
COPY config* c:/temp/
```

有关 `COPY` 指令的详细信息，请参阅[复制参考](https://docs.docker.com/engine/reference/builder/#copy)。

### <a name="add"></a>添加

ADD 指令与 COPY 指令类似，但具有更多的功能。 除了将文件从主机复制到容器映像，`ADD` 指令还可以使用 URL 规范从远程位置复制文件。

`ADD` 指令的格式如下所示：

```dockerfile
ADD <source> <destination>
```

如果源或目标包括空格，请将路径用方括号括起来，用双引号引起来：

```dockerfile
ADD ["<source>", "<destination>"]
```

#### <a name="considerations-for-running-add-with-windows"></a>有关运行 "添加 Windows" 的注意事项

在 Windows 上，目标格式必须使用正斜杠。 例如，以下是有效 `ADD` 说明：

```dockerfile
ADD test1.txt /temp/
ADD test1.txt c:/temp/
```

同时，以下格式的反斜杠无效：

```dockerfile
ADD test1.txt c:\temp\
```

此外，在 Linux 上 `ADD` 指令将在复制时展开压缩包。 此功能在 Windows 中不可用。

#### <a name="examples-of-using-add-with-windows"></a>使用 ADD with Windows 的示例

下面的示例将源目录的内容添加到容器映像中名为 `sqllite` 的目录：

```dockerfile
ADD source /sqlite/
```

下面的示例将以 "config" 开头的所有文件添加到容器映像的 `c:\temp` 目录中。

```dockerfile
ADD config* c:/temp/
```

下面的示例将在容器映像的 `c:\temp` 目录中下载适用于 Windows 的 Python。

```dockerfile
ADD https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe /temp/python-3.5.1.exe
```

有关 `ADD` 指令的详细信息，请参阅[添加引用](https://docs.docker.com/engine/reference/builder/#add)。

### <a name="workdir"></a>WORKDIR

`WORKDIR` 指令用于为其他 Dockerfile 指令（如 `RUN`、`CMD`）设置一个工作目录，并且还设置用于运行容器映像实例的工作目录。

`WORKDIR` 指令的格式如下所示：

```dockerfile
WORKDIR <path to working directory>
```

#### <a name="considerations-for-using-workdir-with-windows"></a>将 WORKDIR 与 Windows 配合使用时的注意事项

在 Windows 上，如果工作目录包含一个反斜杠，则必须对其进行转义。

```dockerfile
WORKDIR c:\\windows
```

**示例**

```dockerfile
WORKDIR c:\\Apache24\\bin
```

有关 `WORKDIR` 指令的详细信息，请参阅[WORKDIR 参考](https://docs.docker.com/engine/reference/builder/#workdir)。

### <a name="cmd"></a>CMD

`CMD` 指令用于设置部署容器映像的实例时要运行的默认命令。 例如，如果容器将托管 NGINX web 服务器，则 `CMD` 可能包括使用 `nginx.exe`之类的命令来启动 web 服务器的说明。 如果 Dockerfile 中指定了多个 `CMD` 指令，只会计算最后一个指令。

`CMD` 指令的格式如下所示：

```dockerfile
# exec form

CMD ["<executable", "<param>"]

# shell form

CMD <command>
```

#### <a name="considerations-for-using-cmd-with-windows"></a>在 Windows 中使用 CMD 的注意事项

在 Windows 上，在 `CMD` 指令中指定的文件路径必须使用正斜杠或已转义的反斜杠 `\\`。 下面是有效 `CMD` 说明：

```dockerfile
# exec form

CMD ["c:\\Apache24\\bin\\httpd.exe", "-w"]

# shell form

CMD c:\\Apache24\\bin\\httpd.exe -w
```

但是，如果不使用正确的斜杠，将无法使用以下格式：

```dockerfile
CMD c:\Apache24\bin\httpd.exe -w
```

有关 `CMD` 指令的详细信息，请参阅[CMD 引用](https://docs.docker.com/engine/reference/builder/#cmd)。

## <a name="escape-character"></a>转义符

在许多情况下，Dockerfile 指令需要跨多个行。 为此，可以使用转义符。 默认 Dockerfile 转义字符是反斜杠 `\`。 但是，由于反斜杠也是 Windows 中的文件路径分隔符，因此使用它来跨多行会导致问题。 若要解决此情况，可以使用分析器指令更改默认转义符。 有关分析器指令的详细信息，请参阅[分析器指令](https://docs.docker.com/engine/reference/builder/#parser-directives)。

下面的示例显示了使用默认转义符跨多行的单个 RUN 指令：

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
    Remove-Item c:\python-3.5.1.exe -Force
```

要修改转义字符，请在 Dockerfile 最开始的行上放置一个转义分析程序指令。 下面的示例对此进行了介绍。

>[!NOTE]
>只有两个值可用作转义字符： `\` 和 `` ` ``。

```dockerfile
# escape=`

FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command `
    $ErrorActionPreference = 'Stop'; `
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; `
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; `
    Remove-Item c:\python-3.5.1.exe -Force
```

有关 escape 分析器指令的详细信息，请参阅[escape parser 指令](https://docs.docker.com/engine/reference/builder/#escape)。

## <a name="powershell-in-dockerfile"></a>Dockerfile 中的 PowerShell

### <a name="powershell-cmdlets"></a>PowerShell cmdlet

PowerShell cmdlet 可在具有 `RUN` 操作的 Dockerfile 中运行。

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -command Expand-Archive -Path c:\apache.zip -DestinationPath c:\
```

### <a name="rest-calls"></a>REST 调用

在从 web 服务收集信息或文件时，PowerShell 的 `Invoke-WebRequest` cmdlet 会很有用。 例如，如果生成包含 Python 的映像，则可以将 `$ProgressPreference` 设置为 `SilentlyContinue` 以实现更快的下载，如以下示例中所示。

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  $ProgressPreference = 'SilentlyContinue'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

>[!NOTE]
>`Invoke-WebRequest` 也适用于 Nano Server。

在映像创建过程期间，还可以通过 .NET WebClient 库使用 PowerShell 下载文件。 这可以增加下载性能。 下面的示例使用 WebClient 库下载 Python 软件。

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  (New-Object System.Net.WebClient).DownloadFile('https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe','c:\python-3.5.1.exe') ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

>[!NOTE]
>Nano Server 当前不支持 WebClient。

### <a name="powershell-scripts"></a>PowerShell 脚本

在某些情况下，将脚本复制到在映像创建过程中使用的容器可能会很有帮助，然后在容器中运行脚本。

>[!NOTE]
>这将限制任何图像层缓存并降低 Dockerfile 的可读性。

此示例使用 `ADD` 指令将脚本从生成计算机复制到容器中。 然后，此脚本使用 RUN 指令运行。

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019
ADD script.ps1 /windows/temp/script.ps1
RUN powershell.exe -executionpolicy bypass c:\windows\temp\script.ps1
```

## <a name="docker-build"></a>Docker 生成

创建 Dockerfile 并将其保存到磁盘后，可以运行 `docker build` 来创建新映像。 `docker build` 命令采用几个可选参数和指向 Dockerfile 的路径。 有关 Docker Build 的完整文档（包括所有生成选项的列表），请参阅[生成引用](https://docs.docker.com/engine/reference/commandline/build/#build)。

`docker build` 命令的格式如下所示：

```dockerfile
docker build [OPTIONS] PATH
```

例如，以下命令将创建一个名为 "iis" 的映像。

```dockerfile
docker build -t iis .
```

启动生成过程后，输出将指示状态并返回任何引发的错误。

```dockerfile
C:\> docker build -t iis .

Sending build context to Docker daemon 2.048 kB
Step 1 : FROM mcr.microsoft.com/windows/servercore:ltsc2019
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

结果为新的容器映像，在此示例中名为 "iis"。

```dockerfile
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
iis                 latest              e2aafdfbe392        About a minute ago   207.8 MB
windowsservercore   latest              6801d964fda5        4 months ago         0 B
```

## <a name="further-reading-and-references"></a>进一步阅读和引用

- [优化 Windows 的 Dockerfile 和 Docker 生成](optimize-windows-dockerfile.md)
- [Dockerfile 参考](https://docs.docker.com/engine/reference/builder/)
