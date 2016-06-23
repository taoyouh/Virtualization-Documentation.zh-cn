---
title: 在 Nano Server 上部署 Windows 容器
description: 在 Nano Server 上部署 Windows 容器
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b82acdf9-042d-4b5c-8b67-1a8013fa1435
---

# 容器主机部署 - Nano Server

**这是初步内容，可能还会更改。** 

开始在 Nano Server 上配置 Windows 容器之前，你需要一个运行 Nano Server 的系统，并且还需要将此系统与某个远程 PowerShell 相连接。

有关使用 Nano Server 进行部署和连接的详细信息，请参阅 [Getting Started with Nano Server]( https://technet.microsoft.com/en-us/library/mt126167.aspx)（Nano Server 入门）。

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

在这些功能安装完毕后，需要重启 Nano Server 主机。

## 安装 Docker

若要使用 Window 容器，则需要安装 Docker。 Docker 由 Docker 引擎和 Docker 客户端组成。 使用这些步骤安装 Docker 守护程序。

下载 Docker 守护程序并将其复制到容器主机的 `$env:SystemRoot\system32\`。 由于 Nano Server 当前不支持 `Invoke-Webrequest`，因此这需要从远程系统来完成。

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile .\dockerd.exe
```

将 Docker 安装为 Windows 的一个服务。

```none
dockerd.exe --register-service
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

最后，需要使用“最新”版本来标记映像。 为此，请运行以下命令。

```none
docker tag nanoserver:10.0.14300.1010 nanoserver:latest
```

## Hyper-V 容器主机

为部署 Hyper-V 容器，需使用 Hyper-V 角色。 如果 Windows 容器主机本身就是 Hyper-V 虚拟机，则需要在安装 Hyper-V 角色前先启用嵌套虚拟化。 有关嵌套虚拟化的详细信息，请参阅嵌套虚拟化。

### 嵌套虚拟化

以下脚本将为容器主机配置嵌套虚拟化。 在托管容器主机虚拟机的 Hyper-V 计算机上运行此脚本。 确保在运行此脚本时，关闭了容器主机虚拟机。

```none
#replace with the virtual machine name
$vm = "<virtual-machine>"

#configure virtual processor
Set-VMProcessor -VMName $vm -ExposeVirtualizationExtensions $true -Count 2

#disable dynamic memory
Set-VMMemory $vm -DynamicMemoryEnabled $false

#enable mac spoofing
Get-VMNetworkAdapter -VMName $vm | Set-VMNetworkAdapter -MacAddressSpoofing On
```

### 启用 Hyper-V 角色

```none
Install-NanoServerPackage Microsoft-NanoServer-Compute-Package
```

## 在 Nano Server 上管理 Docker

**准备 Docker 守护程序：**

为了获得最佳体验，请从远程系统管理 Nano Server 上的 Docker。 为此，需要完成下列各项。

在容器主机上为 Docker 连接创建防火墙规则。 这将是用于不安全连接的端口 `2375`，或用于安全连接的端口 `2376`。

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2376
```

配置 Docker 守护程序以接受通过 TCP 传入的连接。

首先，在 `c:\ProgramData\docker\config\daemon.json` 创建一个 `daemon.json` 文件。

```none
new-item -Type File c:\ProgramData\docker\config\daemon.json
```

接下来，将此 JSON 复制到该文件。 这会将 Docker 守护程序配置为接受通过端口 2375 传入的连接。 由于这是不安全的连接，因此不建议，但是这可用于隔离的测试。

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

在远程管理系统上下载 Docker 客户端。

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile $env:SystemRoot\system32\docker.exe
```

完成后，可使用 `Docker -H` 参数访问 Docker 守护程序。

```none
docker -H tcp://10.0.0.5:2376 run -it nanoserver cmd
```

可以创建环境变量 `DOCKER_HOST`，这会使 `-H` 参数不再需要。 以下 PowerShell 命令可用于此操作。

```none
$env:DOCKER_HOST = "tcp://<ipaddress of server:2376"
```

设置此变量后，现在命令将如下所示。

```none
docker run -it nanoserver cmd
```

<!--HONumber=May16_HO5-->


