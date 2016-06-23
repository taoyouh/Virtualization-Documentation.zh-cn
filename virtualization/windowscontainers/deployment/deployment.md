---
title: 在 Windows Server 上部署 Windows 容器
description: 在 Windows Server 上部署 Windows 容器
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
---

# 容器主机部署 - Windows Server

**这是初步内容，可能还会更改。** 

部署 Windows 容器主机的步骤会有所不同，具体取决于操作系统和主机系统类型（物理或虚拟）。 本文档中详细介绍将 Windows 容器主机部署到物理或虚拟系统上的 Windows Server 2016 或 Windows Server Core 2016 的相关内容。

## 安装容器功能

需要在使用 Windows 容器之前启用容器功能。 要执行此操作，在提升的 PowerShell 会话中运行以下命令。 

```none
Install-WindowsFeature containers
```

功能安装完成后，重启计算机。

## 安装 Docker

若要使用 Window 容器，则需要安装 Docker。 Docker 由 Docker 引擎和 Docker 客户端组成。 此示例中将会安装这两者。

为 Docker 可执行文件创建文件夹。

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

下载 Docker 守护程序。

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile $env:ProgramFiles\docker\dockerd.exe
```

下载 Docker 客户端。

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile $env:ProgramFiles\docker\docker.exe
```

将 Docker 目录添加到系统路径。

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

重启 PowerShell 会话以识别已修改的路径。

若要将 Docker 安装为一个 Windows 服务，请运行以下命令。

```none
dockerd --register-service
```

安装完成后，可以启动该服务。

```none
Start-Service Docker
```

## 安装基本容器映像

需要先下载容器基本操作系统映像，才能部署容器。 下面的示例将下载 Windows Server Core 基本操作系统映像。 安装 Nano Server 基本映像可使用相同的步骤。 安装 Nano Server 基本映像可使用相同的步骤。 有关 Windows 容器映像的详细信息，请参阅[管理容器映像](../management/manage_images.md)。
    
首先，安装容器映像包提供程序。

```none
Install-PackageProvider ContainerImage -Force
```

接着，安装 Windows Server Core 映像。 此过程可能需要花费一些时间，所以此时可以休息一会儿，待下载完成后返回继续。

```none 
Install-ContainerImage -Name WindowsServerCore    
```

安装基本映像之后，需要重启 Docker 服务。

```none
Restart-Service docker
```

最后，需要使用“最新”版本来标记映像。 为此，请运行以下命令。

```none
docker tag windowsservercore:10.0.14300.1000 windowsservercore:latest
```

## Hyper-V 容器主机

为部署 Hyper-V 容器，需使用 Hyper-V 角色。 如果 Windows 容器主机本身就是 Hyper-V 虚拟机，则需要在安装 Hyper-V 角色前先启用嵌套虚拟化。 有关嵌套虚拟化的详细信息，请参阅[嵌套虚拟化]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting)。

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

若要使用 PowerShell 启用 Hyper-V 功能，请在提升的 PowerShell 会话中运行以下命令。

```none
Install-WindowsFeature hyper-v
```



<!--HONumber=May16_HO4-->


