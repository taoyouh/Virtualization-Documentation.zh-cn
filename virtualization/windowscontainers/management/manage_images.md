---
author: neilpeterson
---

# 容器映像

**这是初步内容，可能还会更改。**

容器映像用于部署容器。 这些映像可以包括操作系统、应用程序以及所有应用程序依赖关系。 例如，你可以开发已使用 Nano Server、IIS 以及在 IIS 中运行的应用程序预配置的容器映像。 然后，此容器可以存储在容器注册表中供以后使用，也可以部署在任何 Windows 容器主机上（可以部署在本地和云中，甚至可以部署到容器服务），还可以用作新容器映像的基础。

有以下两种类型的容器映像：

- **基础操作系统映像** - 这些映像由 Microsoft 提供，并包含核心操作系统组件。
- **容器映像** - 从基础操作系统映像派生的容器映像。

## 基础操作系统映像

### 安装映像

可使用 ContainerProvider PowerShell 模块为 PowerShell 和 Docker 管理找到并安装容器操作系统映像。 需要先安装此模块，然后才能进行使用。 可以使用以下命令安装此模块。

```powershell
PS C:\> Install-PackageProvider ContainerProvider -Force
```

在安装之后，可以使用 `Find-ContainerImage` 返回基础操作系统映像的列表 。

```powershell
PS C:\> Find-ContainerImage

Name                 Version                 Description
----                 -------                 -----------
NanoServer           10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
WindowsServerCore    10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
```

若要下载和安装 Nano Server 基础操作系统映像，请运行以下内容。 `-version` 参数为可选参数。 如果没有指定基础操作系统映像版本，将安装最新版本。

```powershell
PS C:\> Install-ContainerImage -Name NanoServer -Version 10.0.10586.0

Downloaded in 0 hours, 0 minutes, 10 seconds.
```

此外，此命令将下载并安装 Windows Server Core 基础操作系统映像。 `-version` 参数为可选参数。 如果没有指定基础操作系统映像版本，将安装最新版本。

> **问题** Save-ContainerImage 和 Install-ContainerImage cmdlet 可能无法在 PowerShell 远程会话中使用 WindowsServerCore 容器映像。 **解决方法：**使用远程桌面登录计算机并直接使用 Save-ContainerImage cmdlet。

```powershell
PS C:\> Install-ContainerImage -Name WindowsServerCore -Version 10.0.10586.0

Downloaded in 0 hours, 2 minutes, 28 seconds.
```

验证是否已使用 `Get-ContainerImage` 命令安装映像。

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version      IsOSImage
----              ---------    -------      ---------
NanoServer        CN=Microsoft 10.0.10586.0 True
WindowsServerCore CN=Microsoft 10.0.10586.0 True
```

> **Install-ContainerImage** 安装基础操作系统映像以便在 PowerShell 或 Docker 托管的容器中使用。 如果已下载了基础操作系统映像，但未在运行 `docker 映像`时显示，请使用服务控制面板小程序或命令“sc docker stop”和“sc docker start”重新启动 Docker 服务

### 脱机安装

基础操作系统映像也可以在未连接到 Internet 时进行安装。 要执行此操作，请将映像下载到具有 Internet 连接计算机上并复制到目标系统，然后使用 `Install-ContainerOSImages` 命令将映像导入。

下载基础操作系统映像之前，通过运行以下命令为系统准备好容器图像提供程序。

```powershell
PS C:\> Install-PackageProvider ContainerProvider -Force
```

从 PowerShell OneGet 程序包管理器中返回映像列表：

```powershell
PS C:\> Find-ContainerImage

Name                 Version                 Description
----                 -------                 -----------
NanoServer           10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
WindowsServerCore    10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
```

若要下载映像，请使用 `Save-ContainerImage` 命令。

```powershell
PS C:\> Save-ContainerImage -Name NanoServer -Destination c:\container-image\NanoServer.wim
```

下载的容器映像现在可以复制到不同的容器主机上，并可以使用 `Install-ContainerOSImage` 命令进行安装。

```powershell
Install-ContainerOSImage -WimPath C:\container-image\NanoServer.wim -Force
```

### 标记映像

当按名称引用容器映像时，Docker 引擎将搜索最新版本的映像。 如果无法确定最新版本，将引发以下错误。

```powershell
PS C:\> docker run -it windowsservercore cmd

Unable to find image 'windowsservercore:latest' locally
Pulling repository docker.io/library/windowsservercore
C:\Windows\system32\docker.exe: Error: image library/windowsservercore not found.
```

在安装 Windows Server 核心或 Nano Server 基础操作系统映像后，需要将这些映像标记为“最新”版本。 若要执行此操作，请使用 `docker 标记`命令。

有关 `docker 标记`的详细信息，请参阅[在 docker.com 上标记、推送和请求映像](https://docs.docker.com/mac/step_six/)。

```powershell
PS C:\> docker tag <image id> windowsservercore:latest
```

标记后，`docker 映像`的输出将显示相同映像的两个版本，一个具有映像版本标记，另一个具有“最新”标记。 现在可以按名称引用映像。

```powershell
PS C:\> docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14289.1000     df03a4b28c50        2 days ago          783.2 MB
windowsservercore   10.0.14289.1000     290ab6758cec        2 days ago          9.148 GB
windowsservercore   latest              290ab6758cec        2 days ago          9.148 GB
```

### 卸载操作系统映像

可以使用 `Uninstall-ContainerOSImage` 命令卸载基础操作系统映像。 下面的示例将演示卸载 NanoServer 基础操作系统映像。

```powershell
Get-ContainerImage -Name NanoServer | Uninstall-ContainerOSImage
```

## 容器映像 PowerShell

### 列出映像

运行 `Get-ContainerImage` 以返回容器主机上的映像列表。 当使用 `IsOSImage` 属性时，可以区分容器映像类型。

```powershell
PS C:\> Get-ContainerImage

Name                    Publisher       Version         IsOSImage
----                    ---------       -------         ---------
NanoServer              CN=Microsoft    10.0.10586.0    True
WindowsServerCore       CN=Microsoft    10.0.10586.0    True
WindowsServerCoreIIS    CN=Demo         1.0.0.0         False
```

### 创建新映像

可以从现有容器创建新容器映像。 请使用 `New-ContainerImage` 命令执行此操作。

```powershell
PS C:\> New-ContainerImage -Container $container -Publisher Demo -Name DemoImage -Version 1.0
```

### 删除映像

如果有任何容器（即使处于停止状态）与映像存在依赖关系，则不能删除容器映像。

使用 PowerShell 删除单一映像。

```powershell
PS C:\> Get-ContainerImage -Name newimage | Remove-ContainerImage -Force
```

### 映像依赖关系

创建新映像后，它将依赖于从中创建它的映像。 使用 `Get-ContainerImage` 命令可以查看此依赖关系。 如果未列出父映像，则表明该映像是基础操作系统映像。

```powershell
PS C:\> Get-ContainerImage | select Name, ParentImage

Name              ParentImage
----              -----------
NanoServerIIS     ContainerImage (Name = 'NanoServer') [Publisher = 'CN=Microsoft', Version = '10.0.10586.0']
NanoServer
WindowsServerCore
```

### 移动映像存储库

使用 `New-ContainerImage` 命令创建新的容器映像时，此映像默认存储在“C:\ProgramData\Microsoft\Windows\Hyper-V\Container Image Store”。 可以使用 `Move-ContainerImageRepository` 命令移动此存储库。 例如，下列操作将在“c:\container-images”创建新的容器映像存储库。

```powershell
Move-ContainerImageRepository -Path c:\container-images
```
> `Move-ContainerImageRepository` 命令所用的路径在运行该命令时不得已存在。

## 容器映像 Docker

### 列出映像

```powershell
C:\> docker images

REPOSITORY             TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
windowsservercoreiis   latest              ca40b33453f8        About a minute ago   44.88 MB
windowsservercore      10.0.10586.0        6801d964fda5        2 weeks ago          0 B
nanoserver             10.0.10586.0        8572198a60f1        2 weeks ago          0 B
```

### 创建新映像

可以从现有容器创建新容器映像。 若要执行此操作，请使用 `docker commit` 命令。 以下示例将创建一个名为“windowsservercoreiis”的新容器映像。

```powershell
C:\> docker commit 475059caef8f windowsservercoreiis

ca40b33453f803bb2a5737d4d5dd2f887d2b2ad06b55ca681a96de8432b5999d
```

### 删除映像

如果有任何容器（即使处于停止状态）与映像存在依赖关系，则不能删除容器映像。

当使用 Docker 删除映像时，可以按映像名称或 ID 引用这些映像。

```powershell
C:\> docker rmi windowsservercoreiis

Untagged: windowsservercoreiis:latest
Deleted: ca40b33453f803bb2a5737d4d5dd2f887d2b2ad06b55ca681a96de8432b5999d
```

### 映像依赖关系

若要通过 Docker 查看映像依赖关系，可以使用 `docker history` 命令。

```powershell
C:\> docker history windowsservercoreiis

IMAGE               CREATED             CREATED BY          SIZE                COMMENT
2236b49aaaef        3 minutes ago       cmd                 171.2 MB
6801d964fda5        2 weeks ago                             0 B
```

### Docker Hub

Docker Hub 注册表包含预生成的映像，可将这些映像下载到容器主机上。 下载这些映像后，它们可用作 Windows 容器应用程序的基础。

若要查看 Docker Hub 中可用的映像列表，请使用 `docker search` 命令。 请注意：在从停靠程序中心提取这些依赖的容器映像之前，需要安装 Windows Server 核心或 Nano Server 基础操作系统映像。

> 以“nano-”开头的映像依赖于 Nano Server 基础操作系统映像。

```powershell
C:\> docker search *

NAME                    DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
microsoft/aspnet        ASP.NET 5 framework installed in a Windows...   1         [OK]       [OK]
microsoft/django        Django installed in a Windows Server Core ...   1                    [OK]
microsoft/dotnet35      .NET 3.5 Runtime installed in a Windows Se...   1         [OK]       [OK]
microsoft/golang        Go Programming Language installed in a Win...   1                    [OK]
microsoft/httpd         Apache httpd installed in a Windows Server...   1                    [OK]
microsoft/iis           Internet Information Services (IIS) instal...   1         [OK]       [OK]
microsoft/mongodb       MongoDB installed in a Windows Server Core...   1                    [OK]
microsoft/mysql         MySQL installed in a Windows Server Core b...   1                    [OK]
microsoft/nginx         Nginx installed in a Windows Server Core b...   1                    [OK]
microsoft/node          Node installed in a Windows Server Core ba...   1                    [OK]
microsoft/php           PHP running on Internet Information Servic...   1                    [OK]
microsoft/python        Python installed in a Windows Server Core ...   1                    [OK]
microsoft/rails         Ruby on Rails installed in a Windows Serve...   1                    [OK]
microsoft/redis         Redis installed in a Windows Server Core b...   1                    [OK]
microsoft/ruby          Ruby installed in a Windows Server Core ba...   1                    [OK]
microsoft/sqlite        SQLite installed in a Windows Server Core ...   1                    [OK]
microsoft/nano-golang   Go Programming Language installed in a Nan...   1                    [OK]
microsoft/nano-httpd    Apache httpd installed in a Nano Server ba...   1                    [OK]
microsoft/nano-iis      Internet Information Services (IIS) instal...   1         [OK]       [OK]
microsoft/nano-mysql    MySQL installed in a Nano Server based con...   1                    [OK]
microsoft/nano-nginx    Nginx installed in a Nano Server based con...   1                    [OK]
microsoft/nano-node     Node installed in a Nano Server based cont...   1                    [OK]
microsoft/nano-python   Python installed in a Nano Server based co...   1                    [OK]
microsoft/nano-rails    Ruby on Rails installed in a Nano Server b...   1                    [OK]
microsoft/nano-redis    Redis installed in a Nano Server based con...   1                    [OK]
microsoft/nano-ruby     Ruby installed in a Nano Server based cont...   1                    [OK]
```

若要从 Docker Hub 下载映像，请使用 `docker pull`。

```powershell
C:\> docker pull microsoft/aspnet

Using default tag: latest
latest: Pulling from microsoft/aspnet
f9e8a4cc8f6c: Pull complete

b71a5b8be5a2: Download complete
```

当运行 `docker images` 时，可以立即看到该映像。

```powershell
C:\> docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
microsoft/aspnet    latest              b3842ee505e5        5 hours ago         101.7 MB
windowsservercore   10.0.10586.0        6801d964fda5        2 weeks ago         0 B
windowsservercore   latest              6801d964fda5        2 weeks ago         0 B
```






<!--HONumber=Mar16_HO3-->


