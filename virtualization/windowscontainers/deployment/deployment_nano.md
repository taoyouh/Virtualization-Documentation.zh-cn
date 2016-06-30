---
title: "在 Nano Server 上部署 Windows 容器"
description: "在 Nano Server 上部署 Windows 容器"
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 06/17/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b82acdf9-042d-4b5c-8b67-1a8013fa1435
translationtype: Human Translation
ms.sourcegitcommit: 1ba6af300d0a3eba3fc6d27598044f983a4c9168
ms.openlocfilehash: f2790186aa641378b1981a1f946665ca46fdbd73

---

# 容器主机部署 - Nano Server

**这是初步内容，可能还会更改。** 

开始在 Nano Server 上配置 Windows 容器之前，你需要一个运行 Nano Server 的系统，并且还需要将此系统与某个远程 PowerShell 相连接。 有关使用 Nano Server 进行部署和连接的详细信息，请参阅 [Getting Started with Nano Server]( https://technet.microsoft.com/en-us/library/mt126167.aspx)（Nano Server 入门）。

[此处](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/nano_eula)提供 Nano Server 的评估副本。

## 安装容器功能

安装 Nano Server 包管理提供程序。

```none
Install-PackageProvider NanoServerPackage
```

在包提供程序安装完毕后，请安装容器功能。

```none
Install-NanoServerPackage -Name Microsoft-NanoServer-Containers-Package
```

容器功能安装完毕后，需要重启 Nano Server 主机。

```none
Restart-Computer
```

## 安装 Docker

若要使用 Window 容器，则需要安装 Docker。 Docker 由 Docker 引擎和 Docker 客户端组成。 使用这些步骤安装 Docker 守护程序和客户端。

在 Nano Server 主机上为 Docker 可执行文件创建一个文件夹。

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

下载 Docker 守护程序和客户端并将其复制到容器主机的 'C:\Program Files\docker\'。 

**注意** - 由于 Nano Server 当前不支持 `Invoke-WebRequest`，因此下载需要从远程系统来完成。

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile .\dockerd.exe
```

下载 Docker 客户端。

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile .\docker.exe
```

下载 Docker 守护程序和客户端并将其复制到 Nano Server 容器主机后，在主机上运行此命令以安装 Docker 作为一项 Windows 服务。

```none
& 'C:\Program Files\docker\dockerd.exe' --register-service
```

启动 Docker 服务。

```none
Start-Service Docker
```

## 安装基本容器映像

基本操作系统映像映像用作任何 Windows Server 或 Hyper-V 容器的基础。 基本操作系统映像映像随 Windows Server Core 和 Nano Server 作为基本操作系统映像提供，并且可以使用容器映像提供程序进行安装。 有关 Windows 容器映像的详细信息，请参阅[管理容器映像](../management/manage_images.md)。

可以使用以下命令来安装容器映像提供程序。

```none
Install-PackageProvider ContainerImage -Force
```

若要下载和安装 Nano Server 基本映像，请运行以下内容：

```none
Install-ContainerImage -Name NanoServer
```

**注意** - 此时，只有 Nano Server 基本映像与 Nano Server 容器主机兼容。

重启 Docker 服务。

```none
Restart-Service Docker
```

将 Nano Server 基本映像标记为最新映像。

```none
& 'C:\Program Files\docker\docker.exe' tag nanoserver:10.0.14300.1016 nanoserver:latest
```

## 在 Nano Server 上管理 Docker

为了获得最佳体验，最佳做法是通过远程系统在 Nano Server 上管理 Docker。 为此，需要完成下列各项。

**准备 Docker 守护程序：**

在容器主机上为 Docker 连接创建防火墙规则。 这将是用于不安全连接的端口 `2375`，或用于安全连接的端口 `2376`。

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2376
```

配置 Docker 守护程序以接受通过 TCP 传入的连接。

首先，在 `c:\ProgramData\docker\config\daemon.json` 创建一个 `daemon.json` 文件。

```none
new-item -Type File c:\ProgramData\docker\config\daemon.json
```

接下来，将此 JSON 复制到配置文件。 这会将 Docker 守护程序配置为接受通过端口 2375 传入的连接。 由于这是不安全的连接，因此不建议，但是这可用于隔离的测试。

```none
{
    "hosts": ["tcp://0.0.0.0:2375", "npipe://"]
}
```

以下示例将配置安全的远程连接。 需要创建 TLS 证书并将其复制到正确的位置。 有关详细信息，请参阅 [Windows 上的 Docker 守护程序](./docker_windows.md)。

```none
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

重启 Docker 服务。

```none
Restart-Service docker
```

**准备 Docker 客户端：**

创建一个目录来保存 Docker 客户端。

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

在远程管理系统上下载 Docker 客户端。

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile "C:\Program Files\docker\docker.exe"
```

将 Docker 目录添加到系统路径。

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

重启 PowerShell 或命令会话以识别已修改的路径。

完成后，可使用 `docker -H` 参数访问远程 Docker 主机。

```none
docker -H tcp://10.0.0.5:2375 run -it nanoserver cmd
```

可以创建环境变量 `DOCKER_HOST`，这会使 `-H` 参数不再需要。 以下 PowerShell 命令可用于此操作。

```none
$env:DOCKER_HOST = "tcp://<ipaddress of server:2375"
```

设置此变量后，现在命令将如下所示。

```none
docker run -it nanoserver cmd
```

## Hyper-V 容器主机

为部署 Hyper-V 容器，需使用 Hyper-V 角色。 有关 Hyper-V 容器的详细信息，请参阅 [Hyper-V 容器](../management/hyperv_container.md)。

如果 Windows 容器主机本身是 Hyper-V 虚拟机，则将需要启用嵌套虚拟化。 有关嵌套虚拟化的详细信息，请参阅[嵌套虚拟化](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting)。


安装 Hyper-V 角色：

```none
Install-NanoServerPackage Microsoft-NanoServer-Compute-Package
```

Hyper-V 角色安装完毕后，需要重启 Nano Server 主机。

```none
Restart-Computer
```






<!--HONumber=Jun16_HO4-->


