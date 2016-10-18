---
title: "在 Nano Server 上部署 Windows 容器"
description: "在 Nano Server 上部署 Windows 容器"
keywords: "docker, 容器"
author: neilpeterson
manager: timlt
ms.date: 09/28/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b82acdf9-042d-4b5c-8b67-1a8013fa1435
translationtype: Human Translation
ms.sourcegitcommit: a2c78d3945f1d5b0ebe2a4af480802f8c0c656c2
ms.openlocfilehash: a9d398de94cb0d6c54c2e82f4a024bb65de9806d

---

# 容器主机部署 - Nano Server

本文档将详细介绍如何使用 Windows 容器功能进行基本的 Nano Server 部署。 此主题为高级主题，假定读者大致了解 Windows 和 Windows 容器。 有关 Windows 容器的介绍，请参阅 [Windows 容器快速入门](../quick_start/quick_start.md)。

## 准备 Nano Server

以下部分将详细介绍基本的 Nano Server 配置的部署。 有关 Nano Server 的部署和配置选项的更全面的介绍，请参阅 [Getting Started with Nano Server]（[Nano Server 入门]）(https://technet.microsoft.com/en-us/library/mt126167.aspx)。

### 创建 Nano Server VM

首先从[此处](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016)下载 Nano Server 评估 VHD。 在此 VHD 中创建虚拟机，启动虚拟机，并使用 Hyper-V 连接选项或基于正在使用的虚拟化平台（等效）连接到虚拟机。

### 创建远程 PowerShell 会话

由于 Nano Server 没有交互式登录功能，所有管理都将使用 PowerShell 通过远程系统完成。

将 Nano Server 系统添加到远程系统的受信任的主机。 请用此 Nano Server 的 IP 地址替换该 IP 地址。

```none
Set-Item WSMan:\localhost\Client\TrustedHosts 192.168.1.50 -Force
```

创建远程 PowerShell 会话。

```none
Enter-PSSession -ComputerName 192.168.1.50 -Credential ~\Administrator
```

完成这些步骤后，你将位于 Nano Server 系统的远程 PowerShell 会话中。 除非特别指出，此文档的其余部分将通过远程会话发生。

### 安装 Windows 更新

需要安装关键更新，才能让 Windows 容器功能正常运作。 可通过运行以下命令安装这些更新。

```none
$sess = New-CimInstance -Namespace root/Microsoft/Windows/WindowsUpdate -ClassName MSFT_WUOperationsSession
Invoke-CimMethod -InputObject $sess -MethodName ApplyApplicableUpdates
```

应用更新后，请重新启动系统。

```none
Restart-Computer
```

备份后，重新建立远程 PowerShell 连接。

## 安装 Docker

若要使用 Window 容器，则需要安装 Docker。 安装 Docker 将用到 [OneGet 提供程序 PowerShell 模块](https://github.com/oneget/oneget)。 提供程序将启用计算机上的容器功能，并安装 Docker - 此操作需要重启计算机。 

在远程 PowerShell 会话中运行以下命令。

首先，安装 OneGet PowerShell 模块。

```none
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

接下来使用 OneGet 安装最新版的 Docker。

```none
Install-Package -Name docker -ProviderName DockerMsftProvider
```

完成安装后，重启计算机。

```none
Restart-Computer -Force
```

## 安装基本容器映像

基本操作系统映像映像用作任何 Windows Server 或 Hyper-V 容器的基础。 基本操作系统映像可通过同时将 Windows Server Core 和 Nano Server 作为基本操作系统获取，并且可以使用 `docker pull` 进行安装。 有关 Docker 容器映像的详细信息，请参阅[在 docker.com 上生成自己的映像](https://docs.docker.com/engine/tutorials/dockerimages/)。

若要下载和安装 Windows Server 和 Nano Server 基本映像，请运行以下命令。

```none
docker pull microsoft/nanoserver
```

```none
docker pull microsoft/windowsservercore
```

> 可在此处 ([EULA](../Images_EULA.md)) 阅读 Windows 容器操作系统映像 EULA。

## 在 Nano Server 上管理 Docker

为了获得最佳体验，最佳做法是通过远程系统在 Nano Server 上管理 Docker。 为此，需要完成下列各项。

### 准备容器主机

在容器主机上为 Docker 连接创建防火墙规则。 这将是用于不安全连接的端口 `2375`，或用于安全连接的端口 `2376`。

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2375
```

配置 Docker 引擎，使其接受通过 TCP 传入的连接。

首先在 Nano Server 主机上的 `c:\ProgramData\docker\config\daemon.json` 中创建一个 `daemon.json` 文件。

```none
new-item -Type File c:\ProgramData\docker\config\daemon.json
```

接下来，运行以下命令以将连接配置添加到 `daemon.json` 文件中。 这会将 Docker 引擎配置为接受通过 TCP 端口 2375 传入的连接。 这是不安全的连接，因此不建议使用，但可用于隔离的测试。 有关确保连接安全的详细信息，请参阅 [Docker.com 上的保护 Docker 守护程序](https://docs.docker.com/engine/security/https/)。

```none
Add-Content 'c:\programdata\docker\config\daemon.json' '{ "hosts": ["tcp://0.0.0.0:2375", "npipe://"] }'
```

重启 Docker 服务。

```none
Restart-Service docker
```

### 准备远程客户端

在要工作的远程系统上下载 Docker 客户端。

```none
Invoke-WebRequest "https://download.docker.com/components/engine/windows-server/cs-1.12/docker.zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

提取压缩包。

```none
Expand-Archive -Path "$env:TEMP\docker.zip" -DestinationPath $env:ProgramFiles
```

运行以下两个命令以将 Docker 目录添加到系统路径。

```none
# For quick use, does not require shell to be restarted.
$env:path += ";c:\program files\docker"

# For persistent use, will apply even after a reboot. 
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

完成后，可使用 `docker -H` 参数访问远程 Docker 主机。

```none
docker -H tcp://<IPADDRESS>:2375 run -it microsoft/nanoserver cmd
```

可以创建环境变量 `DOCKER_HOST`，这会使 `-H` 参数不再需要。 以下 PowerShell 命令可用于此操作。

```none
$env:DOCKER_HOST = "tcp://<ipaddress of server>:2375"
```

设置此变量后，现在命令将如下所示。

```none
docker run -it microsoft/nanoserver cmd
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



<!--HONumber=Oct16_HO2-->


