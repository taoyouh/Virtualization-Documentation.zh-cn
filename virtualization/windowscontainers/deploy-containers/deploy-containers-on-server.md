---
title: 在 Windows Server 上部署 Windows 容器
description: 在 Windows Server 上部署 Windows 容器
keywords: docker, 容器
author: taylorb-microsoft
ms.date: 09/09/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
ms.openlocfilehash: 9899a2d76bfa1fe312e3bd983f60d09d77c272e9
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/06/2020
ms.locfileid: "78853906"
---
# <a name="container-host-deployment-windows-server"></a>容器主机部署：Windows Server

部署 Windows 容器主机的步骤会有所不同，具体取决于操作系统和主机系统类型（物理或虚拟）。 本文档中详细介绍将 Windows 容器主机部署到物理或虚拟系统上的 Windows Server 2016 或 Windows Server Core 2016 的相关内容。

## <a name="install-docker"></a>安装 Docker

若要使用 Window 容器，则需要安装 Docker。 Docker 由 Docker 引擎和 Docker 客户端组成。

安装 Docker 将用到 [OneGet 提供程序 PowerShell 模块](https://github.com/OneGet/MicrosoftDockerProvider)。 提供程序将启用计算机上的容器功能并安装 Docker，这需要重启计算机。

打开提升的 PowerShell 会话并运行下列 cmdlet。

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

## <a name="install-a-specific-version-of-docker"></a>安装特定版本的 Docker

目前有两个可用于 Docker EE for Windows Server 的渠道：

* `17.06` - 如果使用的是 Docker 企业版（Docker 引擎、UCP、DTR），请使用此版本。 默认值为 `17.06`。
* `18.03` - 如果仅运行 Docker EE 引擎，请使用此版本。

若要安装特定版本，请使用 `RequiredVersion` 标志：

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Force -RequiredVersion 18.03
```

安装特定 Docker EE 版本可能需要更新以前安装的 DockerMsftProvider 模块。 若要进行更新，请执行以下操作：

```PowerShell
Update-Module DockerMsftProvider
```

## <a name="update-docker"></a>更新 Docker

如果需要将 Docker EE 引擎从以前的渠道更新为后来的渠道，请同时使用 `-Update` 和 `-RequiredVersion` 标志：

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Update -Force -RequiredVersion 18.03
```

## <a name="install-base-container-images"></a>安装基础容器映像

使用 Windows 容器前，需安装基本映像。 可通过将 Windows Server Core 或 Nano Server 作为容器操作系统获取基本映像。 有关 Docker 容器映像的详细信息，请参阅[在 docker.com 上生成自己的映像](https://docs.docker.com/engine/tutorials/dockerimages/)。

> [!TIP]
> 自 2018 年 5 月起，我们一直提供一致且可信的获取体验，几乎所有源自 Microsoft 的容器映像都通过 Microsoft 容器注册表 _mcr.microsoft.com_ 来提供，同时保留了通过 [_Docker Hub_](https://hub.docker.com/publishers/microsoftowner) 执行的当前发现流程。

### <a name="windows-server-2019-and-newer"></a>Windows Server 2019 和更高版本

若要安装“Windows Server Core”基础映像，请运行以下命令：

```PowerShell
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

若要安装“Nano Server”基础映像，请运行以下命令：

```PowerShell
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

### <a name="windows-server-2016-versions-1607-1803"></a>Windows Server 2016（版本 1607-1803）

若要安装 Windows Server Core 基本映像，请运行以下内容：

```PowerShell
docker pull mcr.microsoft.com/windows/servercore:1607
```

若要安装 Nano Server 基本映像，请运行以下内容：

```PowerShell
docker pull mcr.microsoft.com/windows/nanoserver:1803
```

> 请在此处 ([EULA](../images-eula.md)) 阅读 Windows 容器操作系统映像 EULA。

## <a name="hyper-v-isolation-host"></a>Hyper-V 隔离主机

必须具有 Hyper-V 角色才能运行 Hyper-V 隔离操作。 如果 Windows 容器主机本身就是 Hyper-V 虚拟机，则需要在安装 Hyper-V 角色前先启用嵌套虚拟化。 有关嵌套虚拟化的详细信息，请参阅[嵌套虚拟化](https://docs.microsoft.com/virtualization/hyper-v-on-windows/user-guide/nested-virtualization)。

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

若要使用 PowerShell 来启用 Hyper-V 功能，请在提升的 PowerShell 会话中运行以下 cmdlet。

```PowerShell
Install-WindowsFeature hyper-v
```
