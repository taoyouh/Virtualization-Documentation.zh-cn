---
title: "在 Windows Server 上部署 Windows 容器"
description: "在 Windows Server 上部署 Windows 容器"
keywords: "docker, 容器"
author: neilpeterson
manager: timlt
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
translationtype: Human Translation
ms.sourcegitcommit: f721639b1b10ad97cc469df413d457dbf8d13bbe
ms.openlocfilehash: 4d7e8fb1fcbb7e9680b7d5bd143ef6d59e45035e

---

# 容器主机部署 - Windows Server

部署 Windows 容器主机的步骤会有所不同，具体取决于操作系统和主机系统类型（物理或虚拟）。 本文档中详细介绍将 Windows 容器主机部署到物理或虚拟系统上的 Windows Server 2016 或 Windows Server Core 2016 的相关内容。

## 安装容器功能

需要在使用 Windows 容器之前启用容器功能。 要执行此操作，在提升的 PowerShell 会话中运行以下命令。

```none
Install-WindowsFeature containers
```

功能安装完成后，重启计算机。

```none
Restart-Computer -Force
```

## 安装 Docker

若要使用 Window 容器，则需要安装 Docker。 Docker 由 Docker 引擎和 Docker 客户端组成。 此示例中将会安装这两者。

以 zip 存档形式下载 Docker 引擎和客户端。

```none
Invoke-WebRequest "https://download.docker.com/components/engine/windows-server/cs-1.12/docker.zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

将 zip 存档展开到 Program Files，存档内容已经位于 Docker 目录中。

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

若要将 Docker 安装为一个 Windows 服务，请运行以下命令。

```none
dockerd --register-service
```

安装完成后，可以启动该服务。

```none
Start-Service Docker
```

## 安装基本容器映像

使用 Windows 容器前，需安装基本映像。 可通过将 Windows Server Core 或 Nano Server 作为基础操作系统获取基本映像。 有关 Docker 容器映像的详细信息，请参阅[在 docker.com 上生成自己的映像](https://docs.docker.com/engine/tutorials/dockerimages/)。

若要安装 Windows Server Core 基本映像，请运行以下内容：

```none
docker pull microsoft/windowsservercore
```

若要安装 Nano Server 基本映像，请运行以下内容：

```none
docker pull microsoft/nanoserver
```

> 可在此处 ([EULA](../Images_EULA.md)) 阅读 Windows 容器操作系统映像 EULA。

## Hyper-V 容器主机

要运行 Hyper-V 容器，需要使用 Hyper-V 角色。 如果 Windows 容器主机本身就是 Hyper-V 虚拟机，则需要在安装 Hyper-V 角色前先启用嵌套虚拟化。 有关嵌套虚拟化的详细信息，请参阅[嵌套虚拟化]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting)。

### 嵌套虚拟化

以下脚本将为容器主机配置嵌套虚拟化。 在父 Hyper-V 计算机上运行此脚本。 确保在运行此脚本时，关闭了容器主机虚拟机。

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



<!--HONumber=Sep16_HO4-->


