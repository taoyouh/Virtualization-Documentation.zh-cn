---
title: Windows 容器映像
description: 使用 Windows 容器创建和管理容器映像。
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: d8163185-9860-4ee4-9e96-17b40fb508bc
---

# Windows 容器映像

**这是初步内容，可能还会更改。** 

容器映像用于部署容器。 这些映像可以包括操作系统、应用程序以及所有应用程序依赖关系。 例如，你可以开发已使用 Nano Server、IIS 以及在 IIS 中运行的应用程序预配置的容器映像。 然后，此容器映像可以存储在容器注册表中供以后使用，也可以部署在任何 Windows 容器主机上（可以部署在本地、云中，甚至可以部署到容器服务），还可以用作新容器映像的基础。

有以下两种类型的容器映像：

**基本操作系统映像映像** - 这些映像由 Microsoft 提供，并包含核心操作系统组件。 

**容器映像** - 派生自基本操作系统映像映像的自定义容器映像。

## 基本操作系统映像映像

### 安装映像

可使用 ContainerImage PowerShell 模块找到并安装容器操作系统映像。 需要先安装此模块，然后才能进行使用。 可以使用以下命令安装此模块。 有关使用容器映像 OneGet PowerShell 模块的详细信息，请参阅 [容器映像提供程序](https://github.com/PowerShell/ContainerProvider)。 

```none
Install-PackageProvider ContainerImage -Force
```

安装后，可以使用 `Find-ContainerImage` 返回基本操作系统映像映像的列表。

```none
Find-ContainerImage

Name                 Version          Source           Summary
----                 -------          ------           -------
NanoServer           10.0.14300.1010  ContainerImag... Container OS Image of Windows Server 2016 Technical...
WindowsServerCore    10.0.14300.1000  ContainerImag... Container OS Image of Windows Server 2016 Technical...
```

若要下载和安装 Nano Server 基础操作系统映像，请运行以下内容。 `-version` 参数是可选的。 如果没有指定基础操作系统映像版本，将安装最新版本。

```none
Install-ContainerImage -Name NanoServer -Version 10.0.14300.1010
```

此外，此命令将下载并安装 Windows Server Core 基础操作系统映像。 `-version` 参数是可选的。 如果没有指定基础操作系统映像版本，将安装最新版本。

```none
Install-ContainerImage -Name WindowsServerCore -Version 10.0.14300.1000
```

请验证是否已使用 `docker images` 命令安装了映像。 

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14300.1010     40356b90dc80        2 weeks ago         793.3 MB
windowsservercore   10.0.14304.1000     7837d9445187        2 weeks ago         9.176 GB
```  

安装后，你可能还希望使用“最新”标记来标记映像。 在下面的标记部分对这些说明进行了详细阐述。

> 如果已下载了基本操作系统映像映像，但未在运行 `docker images` 时显示，请使用服务控制面板小程序或命令“sc docker stop”和“sc docker start”重启 Docker 服务

### 标记映像

当按名称引用容器映像时，Docker 引擎将搜索最新版本的映像。 如果无法确定最新版本，将引发以下错误。

```none
docker run -it windowsservercore cmd

Unable to find image 'windowsservercore:latest' locally
Pulling repository docker.io/library/windowsservercore
C:\Windows\system32\docker.exe: Error: image library/windowsservercore not found.
```

在安装 Windows Server 核心或 Nano Server 基础操作系统映像后，需要将这些映像标记为“最新”版本。 为此，请使用 `docker tag` 命令。 

有关 `docker tag` 的详细信息，请参阅 [在 docker.com 上标记、推送和请求映像](https://docs.docker.com/mac/step_six/)。 

```none
docker tag <image id> windowsservercore:latest
```

标记后，`docker images` 的输出将显示相同映像的两个版本，一个具有映像版本标记，另一个具有“最新”标记。 现在可以按名称引用映像。

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14300.1010     df03a4b28c50        2 days ago          783.2 MB
windowsservercore   10.0.14300.1000     290ab6758cec        2 days ago          9.148 GB
windowsservercore   latest              290ab6758cec        2 days ago          9.148 GB
```

### 脱机安装

基础操作系统映像也可以在未连接到 Internet 时进行安装。 为此，请在有 Internet 连接的计算机上下载映像，将其复制到目标系统中，然后使用 `Install-ContainerOSImages` 命令导入映像。

在下载基本操作系统映像映像之前，通过运行以下命令为**已连接 Internet** 的系统准备容器映像提供程序。

```none
Install-PackageProvider ContainerImage -Force
```

从 PowerShell OneGet 程序包管理器中返回映像列表：

```none
Find-ContainerImage
```

输出:

```none
Name                 Version                 Description
----                 -------                 -----------
NanoServer           10.0.14300.1010         Container OS Image of Windows Server 2016 Techn...
WindowsServerCore    10.0.14300.1000         Container OS Image of Windows Server 2016 Techn...
```

若要下载映像，请使用 `Save-ContainerImage` 命令。

```none
Save-ContainerImage -Name NanoServer -Path c:\container-image
```

现在可以将下载的容器映像复制到**脱机容器主机**，并可以使用 `Install-ContainerOSImage` 命令进行安装。

```none
Install-ContainerOSImage -WimPath C:\container-image\NanoServer.wim -Force
```

### 卸载操作系统映像

可以使用 `Uninstall-ContainerOSImage` 命令卸载基本操作系统映像映像。 下面的示例将演示卸载 NanoServer 基础操作系统映像。

```none
Uninstall-ContainerOSImage -FullName CN=Microsoft_NanoServer_10.0.14304.1003
```

## 容器映像

### 列出映像

```none
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
windowsservercoreiis   latest              ca40b33453f8        About a minute ago   44.88 MB
windowsservercore      10.0.14300.1000     6801d964fda5        2 weeks ago          0 B
nanoserver             10.0.14300.1010     8572198a60f1        2 weeks ago          0 B
```

### 创建新映像

可以从现有容器创建新容器映像。 为此，请使用 `docker commit` 命令。 以下示例将创建一个名为“windowsservercoreiis”的新容器映像。

```none
docker commit 475059caef8f windowsservercoreiis
```

### 删除映像

如果有任何容器（即使处于停止状态）与映像存在依赖关系，则不能删除容器映像。

当使用 Docker 删除映像时，可以按映像名称或 ID 引用这些映像。

```none
docker rmi windowsservercoreiis
```

### 映像依赖关系

若要通过 Docker 查看映像依赖关系，可以使用 `docker history` 命令。

```none
docker history windowsservercoreiis

IMAGE               CREATED             CREATED BY          SIZE                COMMENT
2236b49aaaef        3 minutes ago       cmd                 171.2 MB
6801d964fda5        2 weeks ago                             0 B
```

### Docker Hub

Docker Hub 注册表包含预生成的映像，可将这些映像下载到容器主机上。 下载这些映像后，它们可用作 Windows 容器应用程序的基础。

若要查看 Docker Hub 中可用的映像列表，请使用 `docker search` 命令。 注意 - 在从 Docker Hub 中提取这些依赖的容器映像之前，需要安装 Windows Server Core 或 Nano Server 基本操作系统映像映像。

其中大多数映像拥有 Windows Server Core 和 Nano Server 版本。 若要获取特定版本，只需添加标记“:windowsservercore”或“:nanoserver”。 默认情况下，“最新”标记将返回 Windows Server Core 版本，除非仅有 Nano Server 版本可用。

> 以“nano-”开头的映像依赖于 Nano Server 基础操作系统映像。

```none
docker search *

NAME                     DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
microsoft/sample-django  Django installed in a Windows Server Core ...   1                    [OK]
microsoft/dotnet35       .NET 3.5 Runtime installed in a Windows Se...   1         [OK]       [OK]
microsoft/sample-golang  Go Programming Language installed in a Win...   1                    [OK]
microsoft/sample-httpd   Apache httpd installed in a Windows Server...   1                    [OK]
microsoft/iis            Internet Information Services (IIS) instal...   1         [OK]       [OK]
microsoft/sample-mongodb MongoDB installed in a Windows Server Core...   1                    [OK]
microsoft/sample-mysql   MySQL installed in a Windows Server Core b...   1                    [OK]
microsoft/sample-nginx   Nginx installed in a Windows Server Core b...   1                    [OK]
microsoft/sample-node    Node installed in a Windows Server Core ba...   1                    [OK]
microsoft/sample-python  Python installed in a Windows Server Core ...   1                    [OK]
microsoft/sample-rails   Ruby on Rails installed in a Windows Serve...   1                    [OK]
microsoft/sample-redis   Redis installed in a Windows Server Core b...   1                    [OK]
microsoft/sample-ruby    Ruby installed in a Windows Server Core ba...   1                    [OK]
microsoft/sample-sqlite  SQLite installed in a Windows Server Core ...   1                    [OK]
```

若要从 Docker Hub 下载映像，请使用 `docker pull`。

```none
docker pull microsoft/aspnet

Using default tag: latest
latest: Pulling from microsoft/aspnet
f9e8a4cc8f6c: Pull complete

b71a5b8be5a2: Download complete
```

现在运行 `docker images` 时可以看到该映像。

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
microsoft/aspnet    latest              b3842ee505e5        5 hours ago         101.7 MB
windowsservercore   10.0.14300.1000     6801d964fda5        2 weeks ago         0 B
windowsservercore   latest              6801d964fda5        2 weeks ago         0 B
```



<!--HONumber=May16_HO4-->


