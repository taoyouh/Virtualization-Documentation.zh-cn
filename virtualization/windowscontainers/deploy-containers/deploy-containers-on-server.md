---
title: 在 Windows Server 上部署 Windows 容器
description: 在 Windows Server 上部署 Windows 容器
keywords: docker, 容器
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
ms.openlocfilehash: b80dd0d231d0f9435b7cc1c5e2b35bbf5a59d793
ms.sourcegitcommit: a287211a0ed9cac7ebfe1718e3a46f0f26fc8843
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/14/2018
ms.locfileid: "2748883"
---
# <a name="container-host-deployment---windows-server"></a>容器主机部署 - Windows Server

部署 Windows 容器主机的步骤会有所不同，具体取决于操作系统和主机系统类型（物理或虚拟）。 本文档中详细介绍将 Windows 容器主机部署到物理或虚拟系统上的 Windows Server 2016 或 Windows Server Core 2016 的相关内容。

## <a name="install-docker"></a>安装 Docker

若要使用 Window 容器，则需要安装 Docker。 Docker 由 Docker 引擎和 Docker 客户端组成。 

安装 Docker 将用到 [OneGet 提供程序 PowerShell 模块](https://github.com/OneGet/MicrosoftDockerProvider)。 提供程序将启用计算机上的容器功能，并安装 Docker - 此操作需要重启计算机。 

打开提升的 PowerShell 会话并运行下列命令。

安装 OneGet PowerShell 模块。

```PowerShell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

使用 OneGet 安装最新版的 Docker。

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider
```

完成安装后，重启计算机。

```PowerShell
Restart-Computer -Force
```

## <a name="install-a-specific-version-of-docker"></a>安装 Docker 的特定版本

目前有两个通道适用于 Docker EE Windows Server:

* `17.06` -如果您使用 Docker Enterprise Edition (Docker 引擎，UCP，DTR)，则使用此版本。 `17.06` 是默认按钮。
* `18.03` -如果您运行的单独的 Docker EE 引擎，则使用此版本。

若要安装的特定版本，请使用`RequiredVersion`标记：

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Force -RequiredVersion 18.03
```

安装特定 Docker EE 版本可能需要对以前安装的 DockerMsftProvider 模块的更新。 更新：

```PowerShell
Update-Module DockerMsftProvider
```

## <a name="update-docker"></a>更新 Docker

如果您需要从早期通道的 Docker EE 引擎更新为更高版本的通道，请使用两个`-Update`和`-RequiredVersion`标志：

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Update -Force -RequiredVersion 18.03
```

## <a name="install-base-container-images"></a>安装基本容器映像

使用 Windows 容器前，需安装基本映像。 可通过将 Windows Server Core 或 Nano Server 作为容器操作系统获取基本映像。 有关 Docker 容器映像的详细信息，请参阅[在 docker.com 上生成自己的映像](https://docs.docker.com/engine/tutorials/dockerimages/)。

若要安装 Windows Server Core 基本映像，请运行以下内容：

```PowerShell
docker pull microsoft/windowsservercore
```

若要安装 Nano Server 基本映像，请运行以下内容：

```PowerShell
docker pull microsoft/nanoserver
```

> 可在此处 ([EULA](../images-eula.md)) 阅读 Windows 容器操作系统映像 EULA。

## <a name="hyper-v-container-host"></a>Hyper-V 容器主机

要运行 Hyper-V 容器，需要使用 Hyper-V 角色。 如果 Windows 容器主机本身就是 Hyper-V 虚拟机，则需要在安装 Hyper-V 角色前先启用嵌套虚拟化。 有关嵌套虚拟化的详细信息，请参阅[嵌套虚拟化]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting)。

### <a name="nested-virtualization"></a>嵌套虚拟化

以下脚本将为容器主机配置嵌套虚拟化。 在父 Hyper-V 计算机上运行此脚本。 确保在运行此脚本时，关闭了容器主机虚拟机。

```PowerShell
#replace with the virtual machine name
$vm = "<virtual-machine>"

#configure virtual processor
Set-VMProcessor -VMName $vm -ExposeVirtualizationExtensions $true -Count 2

#disable dynamic memory
Set-VMMemory -VMName $vm -DynamicMemoryEnabled $false

#enable mac spoofing
Get-VMNetworkAdapter -VMName $vm | Set-VMNetworkAdapter -MacAddressSpoofing On
```

### <a name="enable-the-hyper-v-role"></a>启用 Hyper-V 角色

若要使用 PowerShell 启用 Hyper-V 功能，请在提升的 PowerShell 会话中运行以下命令。

```PowerShell
Install-WindowsFeature hyper-v
```
