---
title: "在 Windows Server 上部署 Windows 容器"
description: "在 Windows Server 上部署 Windows 容器"
keywords: "docker, 容器"
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
ms.openlocfilehash: 1db66d5dfe0f0060c56625e396ecff3bb72e3189
ms.sourcegitcommit: 456485f36ed2d412cd708aed671d5a917b934bbe
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/08/2017
---
# <a name="container-host-deployment---windows-server"></a>容器主机部署 - Windows Server

部署 Windows 容器主机的步骤会有所不同，具体取决于操作系统和主机系统类型（物理或虚拟）。 本文档中详细介绍将 Windows 容器主机部署到物理或虚拟系统上的 Windows Server 2016 或 Windows Server Core 2016 的相关内容。

## <a name="install-docker"></a>安装 Docker

若要使用 Window 容器，则需要安装 Docker。 Docker 由 Docker 引擎和 Docker 客户端组成。 

安装 Docker 将用到 [OneGet 提供程序 PowerShell 模块](https://github.com/OneGet/MicrosoftDockerProvider)。 提供程序将启用计算机上的容器功能，并安装 Docker - 此操作需要重启计算机。 

打开提升的 PowerShell 会话并运行下列命令。

安装 OneGet PowerShell 模块。

```
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

使用 OneGet 安装最新版的 Docker。

```
Install-Package -Name docker -ProviderName DockerMsftProvider
```

完成安装后，重启计算机。

```
Restart-Computer -Force
```

## <a name="install-base-container-images"></a>安装基本容器映像

使用 Windows 容器前，需安装基本映像。 可通过将 Windows Server Core 或 Nano Server 作为容器操作系统获取基本映像。 有关 Docker 容器映像的详细信息，请参阅[在 docker.com 上生成自己的映像](https://docs.docker.com/engine/tutorials/dockerimages/)。

若要安装 Windows Server Core 基本映像，请运行以下内容：

```
docker pull microsoft/windowsservercore
```

若要安装 Nano Server 基本映像，请运行以下内容：

```
docker pull microsoft/nanoserver
```

> 可在此处 ([EULA](../images-eula.md)) 阅读 Windows 容器操作系统映像 EULA。

## <a name="hyper-v-container-host"></a>Hyper-V 容器主机

要运行 Hyper-V 容器，需要使用 Hyper-V 角色。 如果 Windows 容器主机本身就是 Hyper-V 虚拟机，则需要在安装 Hyper-V 角色前先启用嵌套虚拟化。 有关嵌套虚拟化的详细信息，请参阅[嵌套虚拟化]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting)。

### <a name="nested-virtualization"></a>嵌套虚拟化

以下脚本将为容器主机配置嵌套虚拟化。 在父 Hyper-V 计算机上运行此脚本。 确保在运行此脚本时，关闭了容器主机虚拟机。

```
#replace with the virtual machine name
$vm = "<virtual-machine>"

#configure virtual processor
Set-VMProcessor -VMName $vm -ExposeVirtualizationExtensions $true -Count 2

#disable dynamic memory
Set-VMMemory $vm -DynamicMemoryEnabled $false

#enable mac spoofing
Get-VMNetworkAdapter -VMName $vm | Set-VMNetworkAdapter -MacAddressSpoofing On
```

### <a name="enable-the-hyper-v-role"></a>启用 Hyper-V 角色

若要使用 PowerShell 启用 Hyper-V 功能，请在提升的 PowerShell 会话中运行以下命令。

```
Install-WindowsFeature hyper-v
```
