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
ms.openlocfilehash: 9ff6256ab9708533f72e9b3210f8a5fd32f4048a
ms.sourcegitcommit: c48dcfe43f73b96e0ebd661164b6dd164c775bfa
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "9610267"
---
# <a name="dockerfile-on-windows"></a>Windows 上的 Dockerfile

Docker 引擎包含自动创建容器映像的工具。 尽管你可以手动创建容器映像通过运行`docker commit`命令，然而采用自动的映像创建过程方面有许多优势，包括：

- 将容器映像存储为代码。
- 可出于维护和升级的目的快速而精确地重新创建容器映像。
- 容器映像和开发周期之间的持续集成。

驱动实现这一自动化过程的 Docker 组件是 Dockerfile，以及 `docker build` 命令。

Dockerfile 是文本文件，其中包含创建新的容器映像所需的说明。 这些指令包括对将用作基础的现有映像的标识、将在映像创建过程中运行的命令以及部署容器映像的新实例时将要运行的命令。

Docker 版本是使用 Dockerfile 并触发映像创建过程的 Docker 引擎命令。

本主题介绍了如何使用 Windows 容器的 Dockerfile，了解其基本语法和最常用的 Dockerfile 指令是什么。

本文将讨论容器映像和容器映像层的概念。 如果你想要了解有关映像和映像分层的详细信息，请参阅[图像快速入门指南](../quick-start/quick-start-images.md)。

有关 Dockerfile 的完整信息，请参阅[的 Dockerfile 参考](https://docs.docker.com/engine/reference/builder/)。

## <a name="basic-syntax"></a>基本语法

Dockerfile 的最基本形式十分简单。 下面的示例创建了一个新映像，其中包括 IIS 和一个“hello world”站点。 此示例包含了用于解释每个步骤的注释（通过 `#` 指示）。 本文的后续部分将对 Dockerfile 语法规则和 Dockerfile 指令进行更详细的讲解。

>[!NOTE]
>创建的 Dockerfile 不得带扩展名。 若要执行此操作在 Windows 中，使用你的选择的编辑器创建该文件，然后使用表示法"Dockerfile"（包括引号） 保存它。

```dockerfile
# Sample Dockerfile

# Indicates that the windowsservercore image will be used as the base image.
FROM microsoft/windowsservercore

# Metadata indicating an image maintainer.
LABEL maintainer="jshelton@contoso.com"

# Uses dism.exe to install the IIS role.
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart

# Creates an HTML file and adds content to this file.
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html

# Sets a command or process that will run each time a container is run from the new image.
CMD [ "cmd" ]
```

用于 Windows 的 Dockerfile 的其他示例，请参阅[Windows 的 Dockerfile 存储库](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples)。

## <a name="instructions"></a>说明

Dockerfile 指令提供 Docker 引擎创建容器映像所需的说明。 一个地和顺序执行这些说明。 以下示例是 Dockerfile 中的最常使用的说明。 有关 Dockerfile 指令的完整列表，请参阅[的 Dockerfile 参考](https://docs.docker.com/engine/reference/builder/)。

### <a name="from"></a>FROM

`FROM` 指令用于设置在新映像创建过程期间将使用的容器映像。 例如，使用指令 `FROM microsoft/windowsservercore` 时，所得到的映像派生自 Windows Server Core 基本操作系统映像映像并具有对其的依赖关系。 如果正在进行 Docker 生成过程的系统上不存在指定的映像，Docker 引擎将尝试从公有或私有映像注册表下载该映像。

FROM 指令的格式如下所示：

```dockerfile
FROM <image>
```

下面是 FROM 命令的示例：

```dockerfile
FROM microsoft/windowsservercore
```

有关更多详细信息，请参阅[FROM 参考](https://docs.docker.com/engine/reference/builder/#from)。

### <a name="run"></a>RUN

`RUN` 指令指定将要运行并捕获到新容器映像中的命令。 这些命令包括安装软件、创建文件和目录，以及创建环境配置等。

RUN 指令如下所示：

```dockerfile
# exec form

RUN ["<executable>", "<param 1>", "<param 2>"]

# shell form

RUN <command>
```

Exec 与 shell 窗体之间的区别是如何在`RUN`指令执行。 使用 exec 窗体时，指定的程序显式运行。

下面是 exec 窗体的示例：

```dockerfile
FROM microsoft/windowsservercore

RUN ["powershell", "New-Item", "c:/test"]
```

生成的映像运行`powershell New-Item c:/test`命令：

```dockerfile
docker history doc-exe-method

IMAGE               CREATED             CREATED BY                    SIZE                COMMENT
b3452b13e472        2 minutes ago       powershell New-Item c:/test   30.76 MB
```

为了便于对比，下面的示例中是 shell 窗体运行相同的操作：

```dockerfile
FROM microsoft/windowsservercore

RUN powershell New-Item c:\test
```

生成的映像有运行的指令`cmd /S /C powershell New-Item c:\test`。

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

当目标程序是 Windows 安装程序时，你将需要提取通过设置`/x:<directory>`你可以启动实际 （无提示） 安装过程之前标记。 你还必须等待命令退出之前执行其他操作。 否则，此过程将提前结束，而无需安装任何内容。 有关详细信息，请参阅以下示例。

#### <a name="examples-of-using-run-with-windows"></a>使用 Windows 运行使用的示例

下面的示例 Dockerfile 使用 DISM 在容器映像中安装 IIS:

```dockerfile
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart
```

此示例安装 Visual Studio 可再发行组件包。 `Start-Process` 和`-Wait`参数用于运行安装程序。 这可以确保在安装完成之前移动到 Dockerfile 中的下一个指令。

```dockerfile
RUN powershell.exe -Command Start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait
```

有关 RUN 指令的详细信息，请参阅[运行引用](https://docs.docker.com/engine/reference/builder/#run)。

### <a name="copy"></a>复制

`COPY`指令将文件和目录复制到容器的文件系统。 文件和目录必须是相对于 Dockerfile 的路径中。

`COPY`指令的格式如下：

```dockerfile
COPY <source> <destination>
```

如果源或目标包含空格，请将路径括在方括号和双引号，如下面的示例中所示：

```dockerfile
COPY ["<source>", "<destination>"]
```

#### <a name="considerations-for-using-copy-with-windows"></a>与 Windows 使用复制的注意事项

在 Windows 上，目标格式必须使用正斜杠。 例如，这些是有效`COPY`说明：

```dockerfile
COPY test1.txt /temp/
COPY test1.txt c:/temp/
```

同时，将不起作用反斜杠具有以下格式：

```dockerfile
COPY test1.txt c:\temp\
```

#### <a name="examples-of-using-copy-with-windows"></a>与 Windows 使用复制的示例

下面的示例将源目录的内容添加到目录名为`sqllite`在容器映像中：

```dockerfile
COPY source /sqlite/
```

下面的示例会将以 config 到开头的所有文件都添加`c:\temp`目录的容器映像：

```dockerfile
COPY config* c:/temp/
```

有关的更多详细信息`COPY`指令，请参阅[复制引用](https://docs.docker.com/engine/reference/builder/#copy)。

### <a name="add"></a>ADD

ADD 指令与 COPY 指令，如但具有更多的功能。 除了将文件从主机复制到容器映像，`ADD` 指令还可以使用 URL 规范从远程位置复制文件。

`ADD`指令的格式如下：

```dockerfile
ADD <source> <destination>
```

如果源或目标包含空格，请将路径括在方括号和双引号中：

```dockerfile
ADD ["<source>", "<destination>"]
```

#### <a name="considerations-for-running-add-with-windows"></a>与 Windows 运行添加注意事项

在 Windows 上，目标格式必须使用正斜杠。 例如，这些是有效`ADD`说明：

```dockerfile
ADD test1.txt /temp/
ADD test1.txt c:/temp/
```

同时，将不起作用反斜杠具有以下格式：

```dockerfile
ADD test1.txt c:\temp\
```

此外，在 Linux 上 `ADD` 指令将在复制时展开压缩包。 此功能在 Windows 中不可用。

#### <a name="examples-of-using-add-with-windows"></a>与 Windows 使用添加的示例

下面的示例将源目录的内容添加到目录名为`sqllite`在容器映像中：

```dockerfile
ADD source /sqlite/
```

下面的示例会将"配置"开头的所有文件都添加到`c:\temp`目录的容器映像。

```dockerfile
ADD config* c:/temp/
```

下面的示例将 Python for Windows 下载到`c:\temp`目录的容器映像。

```dockerfile
ADD https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe /temp/python-3.5.1.exe
```

有关的更多详细信息`ADD`指令，请参阅[添加引用](https://docs.docker.com/engine/reference/builder/#add)。

### <a name="workdir"></a>WORKDIR

`WORKDIR` 指令用于为其他 Dockerfile 指令（如 `RUN`、`CMD`）设置一个工作目录，并且还设置用于运行容器映像实例的工作目录。

`WORKDIR`指令的格式如下：

```dockerfile
WORKDIR <path to working directory>
```

#### <a name="considerations-for-using-workdir-with-windows"></a>与 Windows 使用 WORKDIR 注意事项

在 Windows 上，如果工作目录包含一个反斜杠，则必须对其进行转义。

```dockerfile
WORKDIR c:\\windows
```

**示例**

```dockerfile
WORKDIR c:\\Apache24\\bin
```

有关详细信息`WORKDIR`指令，请参阅[WORKDIR 参考](https://docs.docker.com/engine/reference/builder/#workdir)。

### <a name="cmd"></a>CMD

`CMD` 指令用于设置部署容器映像的实例时要运行的默认命令。 例如，如果该容器将承载一个 NGINX web 服务器上，`CMD`可能包括如何使用如下命令启动 web 服务器`nginx.exe`。 如果 Dockerfile 中指定了多个 `CMD` 指令，只会计算最后一个指令。

`CMD`指令的格式如下：

```dockerfile
# exec form

CMD ["<executable", "<param>"]

# shell form

CMD <command>
```

#### <a name="considerations-for-using-cmd-with-windows"></a>使用与 Windows CMD 注意事项

在 Windows 上，在 `CMD` 指令中指定的文件路径必须使用正斜杠或已转义的反斜杠 `\\`。 以下是有效`CMD`说明：

```dockerfile
# exec form

CMD ["c:\\Apache24\\bin\\httpd.exe", "-w"]

# shell form

CMD c:\\Apache24\\bin\\httpd.exe -w
```

但是，以下格式不正确斜杠将不起作用：

```dockerfile
CMD c:\Apache24\bin\httpd.exe -w
```

有关的更多详细信息`CMD`指令，请参阅[CMD 参考](https://docs.docker.com/engine/reference/builder/#cmd)。

## <a name="escape-character"></a>转义字符

在许多情况下，Dockerfile 指令需要跨多个行。 若要执行此操作，你可以使用转义字符。 默认 Dockerfile 转义字符是反斜杠 `\`。 但是，因为反斜杠也是 Windows 中的文件路径分隔符，则将其用于跨多个行可能会导致问题。 若要获取解决此，可以使用一个分析程序指令来更改默认转义字符。 有关分析程序指令的详细信息，请参阅[分析程序指令](https://docs.docker.com/engine/reference/builder/#parser-directives)。

下面的示例显示了跨使用默认转义字符的多个行的单个 RUN 指令：

```dockerfile
FROM microsoft/windowsservercore

RUN powershell.exe -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
    Remove-Item c:\python-3.5.1.exe -Force
```

要修改转义字符，请在 Dockerfile 最开始的行上放置一个转义分析程序指令。 这可以在以下示例中所示。

>[!NOTE]
>只有两个值可用作转义字符：`\`和`` ` ``。

```dockerfile
# escape=`

FROM microsoft/windowsservercore

RUN powershell.exe -Command `
    $ErrorActionPreference = 'Stop'; `
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; `
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; `
    Remove-Item c:\python-3.5.1.exe -Force
```

有关转义分析程序指令的详细信息，请参阅[转义分析程序指令](https://docs.docker.com/engine/reference/builder/#escape)。

## <a name="powershell-in-dockerfile"></a>Dockerfile 中的 PowerShell

### <a name="powershell-cmdlets"></a>PowerShell cmdlet

可以使用 Dockerfile 中运行 PowerShell cmdlet`RUN`操作。

```dockerfile
FROM microsoft/windowsservercore

RUN powershell -command Expand-Archive -Path c:\apache.zip -DestinationPath c:\
```

### <a name="rest-calls"></a>REST 调用

PowerShell 的`Invoke-WebRequest`cmdlet 从 web 服务收集信息或文件时可能很有用。 例如，如果你生成包含 Python 的映像，你可以设置`$ProgressPreference`到`SilentlyContinue`若要实现更快的下载速度，如下面的示例中所示。

```dockerfile
FROM microsoft/windowsservercore

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

```dockerfile
FROM microsoft/windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  (New-Object System.Net.WebClient).DownloadFile('https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe','c:\python-3.5.1.exe') ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

>[!NOTE]
>Nano Server 当前不支持 WebClient。

### <a name="powershell-scripts"></a>PowerShell 脚本

在某些情况下，它可能有助于将脚本复制到你在映像创建过程期间使用的容器，然后运行来自在容器中的脚本。

>[!NOTE]
>这将限制任何映像层缓存，并减少 Dockerfile 的可读性。

此示例使用 `ADD` 指令将脚本从生成计算机复制到容器中。 然后，此脚本使用 RUN 指令运行。

```dockerfile
FROM microsoft/windowsservercore
ADD script.ps1 /windows/temp/script.ps1
RUN powershell.exe -executionpolicy bypass c:\windows\temp\script.ps1
```

## <a name="docker-build"></a>Docker 版本

创建 Dockerfile 并将其保存到磁盘后, 可以运行`docker build`若要创建新的映像。 `docker build` 命令采用几个可选参数和指向 Dockerfile 的路径。 Docker Build 的完整文档，包括所有列表生成选项，请参阅[生成引用](https://docs.docker.com/engine/reference/commandline/build/#build)。

格式`docker build`命令如下：

```dockerfile
docker build [OPTIONS] PATH
```

例如，以下命令将创建名为"iis。"的图像

```dockerfile
docker build -t iis .
```

生成过程启动后，输出将指示状态，并返回任何引发的错误。

```dockerfile
C:\> docker build -t iis .

Sending build context to Docker daemon 2.048 kB
Step 1 : FROM micrsoft/windowsservercore
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

结果是一个新的容器映像，其中在此示例中名为"iis"。

```dockerfile
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
iis                 latest              e2aafdfbe392        About a minute ago   207.8 MB
windowsservercore   latest              6801d964fda5        4 months ago         0 B
```

## <a name="further-reading-and-references"></a>进一步阅读和参考

- [优化适用于 Windows 的 Dockerfile 和 Docker 版本](optimize-windows-dockerfile.md)
- [Dockerfile 参考](https://docs.docker.com/engine/reference/builder/)
