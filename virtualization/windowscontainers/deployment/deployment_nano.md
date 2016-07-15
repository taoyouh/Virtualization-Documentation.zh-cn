---
title: "在 Nano Server 上部署 Windows 容器"
description: "在 Nano Server 上部署 Windows 容器"
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 07/06/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b82acdf9-042d-4b5c-8b67-1a8013fa1435
translationtype: Human Translation
ms.sourcegitcommit: e035a45e22eee04263861d935b338089d8009e92
ms.openlocfilehash: 876ffb4f4da32495fb77b735391203c33c78cff3

---

# 容器主机部署 - Nano Server

**这是初步内容，可能还会更改。** 

本文档将详细介绍如何使用 Windows 容器功能进行基本的 Nano Server 部署。 此主题为高级主题，假定读者大致了解 Windows 和 Windows 容器。 有关 Windows 容器的介绍，请参阅 [Windows 容器快速入门](../quick_start/quick_start.md)。

## 准备 Nano Server

以下部分将详细介绍基本的 Nano Server 配置的部署。 有关 Nano Server 的部署和配置选项的更全面的介绍，请参阅 [Getting Started with Nano Server]（[Nano Server 入门]）(https://technet.microsoft.com/en-us/library/mt126167.aspx)。

### 创建 Nano Server VM

首先从[此处](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/nano_eula)下载 Nano Server 评估 VHD。 在此 VHD 中创建虚拟机，启动虚拟机，并使用 Hyper-V 连接选项或基于正在使用的虚拟化平台（等效）连接到虚拟机。

接下来，需要设置管理密码。 在 Nano Server 恢复控制台上按 `F11` 来设置密码。 将出现更改密码对话框。

### 创建远程 PowerShell 会话

由于 Nano Server 没有交互式登录功能，所有管理将通过远程 PowerShell 会话完成。 若要创建远程会话，请使用 Nano Server 恢复控制台的网络部分获取系统 IP 地址，然后在远程主机上运行以下命令。 将 IPADDRESS 替换为 Nano Server 系统的实际 IP 地址。

将 Nano Server 系统添加到受信任的主机。

```none
set-item WSMan:\localhost\Client\TrustedHosts IPADDRESS -Force
```

创建远程 PowerShell 会话。

```none
Enter-PSSession -ComputerName IPADDRESS -Credential ~\Administrator
```

完成这些步骤后，你将位于 Nano Server 系统的远程 PowerShell 会话中。 除非特别指出，此文档的其余部分将通过远程会话发生。


## 安装容器功能

Nano Server 程序包管理提供程序允许在 Nano Server 上安装角色和功能。 使用此命令安装提供程序。

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

备份后，重新建立远程 PowerShell 连接。

## 安装 Docker

若要使用 Window 容器，则需要安装 Docker。 Docker 由 Docker 引擎和 Docker 客户端组成。 使用这些步骤安装 Docker 守护程序和客户端。

在 Nano Server 主机上为 Docker 可执行文件创建一个文件夹。

```none
New-Item -Type Directory -Path $env:ProgramFiles'\docker\'
```

下载 Docker 守护程序和客户端并将其复制到容器主机的 'C:\Program Files\docker\'。 

**注意** - 由于 Nano Server 当前不支持 `Invoke-WebRequest`，因此需要通过远程系统完成下载，然后将其复制到 Nano Server 主机。

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile .\dockerd.exe
```

下载 Docker 客户端。

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile .\docker.exe
```

Docker 守护程序和客户端下载完成后，将它们复制到 Nano Server 容器主机中的“C:\Program Files\docker\'”文件夹中。 需要将 Nano Server 防火墙配置为允许传入的 SMB 连接。 可使用 PowerShell 或 Nano Server 恢复控制台完成该配置。 

```none
Set-NetFirewallRule -Name FPS-SMB-In-TCP -Enabled True
```

现在可使用标准 SMB 文件复制方法复制文件。

将 dockerd.exe 文件复制到主机后，运行此命令以将 Docker 安装为 Windows 服务。

```none
& $env:ProgramFiles'\docker\dockerd.exe' --register-service
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
& $env:ProgramFiles'\docker\docker.exe' tag nanoserver:10.0.14300.1016 nanoserver:latest
```

## 在 Nano Server 上管理 Docker

为了获得最佳体验，最佳做法是通过远程系统在 Nano Server 上管理 Docker。 为此，需要完成下列各项。

### 准备容器主机

在容器主机上为 Docker 连接创建防火墙规则。 这将是用于不安全连接的端口 `2375`，或用于安全连接的端口 `2376`。

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2376
```

配置 Docker 守护程序以接受通过 TCP 传入的连接。

首先在 Nano Server 主机上的 `c:\ProgramData\docker\config\daemon.json` 中创建一个 `daemon.json` 文件。

```none
new-item -Type File c:\ProgramData\docker\config\daemon.json
```

接下来，运行以下命令以将连接配置添加到 `daemon.json` 文件中。 这会将 Docker 守护程序配置为接受通过端口 2375 传入的连接。 这是不安全的连接，因此不建议使用，但可用于隔离的测试。 有关确保连接安全的详细信息，请参阅 [Docker.com 上的保护 Docker 守护程序](https://docs.docker.com/engine/security/https/)。

```none
Add-Content 'c:\programdata\docker\config\daemon.json' '{ "hosts": ["tcp://0.0.0.0:2375", "npipe://"] }'
```

重启 Docker 服务。

```none
Restart-Service docker
```

### 准备远程客户端

在要工作的远程系统上创建一个目录来保存 Docker 客户端。

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

将 Docker 客户端下载到此目录中。

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile "$env:ProgramFiles\docker\docker.exe"
```

将 Docker 目录添加到系统路径。

```none
$env:Path += ";$env:ProgramFiles\Docker"
```

重启 PowerShell 或命令会话以识别已修改的路径。

完成后，可使用 `docker -H` 参数访问远程 Docker 主机。

```none
docker -H tcp://<IPADDRESS>:2375 run -it nanoserver cmd
```

可以创建环境变量 `DOCKER_HOST`，这会使 `-H` 参数不再需要。 以下 PowerShell 命令可用于此操作。

```none
$env:DOCKER_HOST = "tcp://<ipaddress of server>:2375"
```

设置此变量后，现在命令将如下所示。

```none
docker run -it nanoserver cmd
```

## Hyper-V 容器主机

为部署 Hyper-V 容器，需要在容器主机上使用 Hyper-V 角色。 有关 Hyper-V 容器的详细信息，请参阅 [Hyper-V 容器](../management/hyperv_container.md)。

如果 Windows 容器主机本身是 Hyper-V 虚拟机，则将需要启用嵌套虚拟化。 有关嵌套虚拟化的详细信息，请参阅[嵌套虚拟化](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting)。


在 Nano Server 容器主机上安装 Hyper-V 角色。

```none
Install-NanoServerPackage Microsoft-NanoServer-Compute-Package
```

Hyper-V 角色安装完毕后，需要重启 Nano Server 主机。

```none
Restart-Computer
```


<!--HONumber=Jul16_HO1-->


