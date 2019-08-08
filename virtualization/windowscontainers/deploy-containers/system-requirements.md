---
title: Windows 容器要求
description: Windows 容器要求。
keywords: 元数据, 容器
author: taylorb-microsoft
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: 5fc9b5c9135e87a0d3246952c35c9755e9ad209e
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998464"
---
# <a name="windows-container-requirements"></a>Windows 容器要求

本指南列出了 Windows 容器主机的要求。

## <a name="os-requirements"></a>操作系统要求

- Windows 容器功能仅在 Windows Server 2016 (核心和桌面体验)、Windows 10 专业版和企业版 (周年纪念日) 及更高版本中可用。
- 必须先安装 Hyper-v 角色, 然后才能运行 Hyper-v 隔离
- Windows Server 容器主机必须将 Windows 安装到 c:\。 如果仅部署 Hyper-v 隔离容器, 此限制将不适用。

## <a name="virtualized-container-hosts"></a>虚拟化容器主机

如果 Windows 容器主机将从 Hyper-v 虚拟机运行, 并且也将托管 Hyper-v 隔离, 则需要启用嵌套虚拟化。 嵌套的虚拟化具有以下要求：

- 至少 4 GB RAM 可用于虚拟化的 Hyper-V 主机。
- Windows Server 2019、Windows Server 版本1803、Windows server 版本1709、Windows Server 2016 或虚拟机中的 windows Server (Full、Core)。
- 带有 Intel VT-x 处理器（此功能目前只适用于 Intel 处理器）。
- 容器主机 VM 还需要至少两个虚拟处理器。

## <a name="supported-base-images"></a>支持的基本图像

Windows 容器提供四个容器基映像: Windows Server Core、Nano Server、Windows 和 IoT Core。 并非所有配置都支持这两个操作系统映像。 下表详细介绍所支持的配置。

|主机操作系统|Windows 容器|Hyper-V 隔离|
|---------------------|-----------------|-----------------|
|Windows Server 2016 或 Windows Server 2019 (标准版或数据中心)|服务器核心版、Nano Server、Windows|服务器核心版、Nano Server、Windows|
|Nano Server|Nano Server|服务器核心版、Nano Server、Windows|
|Windows 10 专业版或 Windows 10 企业版|不可用|服务器核心版、Nano Server、Windows|
|IoT Core|IoT Core|不可用|

> [!WARNING]  
> 从 Windows Server 版本1709开始, Nano Server 将不再作为容器主机提供。

### <a name="memory-requirements"></a>内存要求

可以通过[资源控制](https://docs.microsoft.com/virtualization/windowscontainers/manage-containers/resource-controls)或重载容器主机来配置容器可用内存限值。  下面列出了启动容器和运行基本命令 (ipconfig、dir 等) 所需的最小内存量。

>[!NOTE]
>在容器中运行的应用程序中, 这些值不考虑容器或要求之间的资源共享。  例如，具有 512MB 可用内存的主机可以在 Hyper-V 隔离下运行多个 Server Core 容器，因为这些容器会共享资源。

#### <a name="windows-server-2016"></a>WindowsServer 2016

| 基本图像  | Windows Server 容器 | Hyper-V 隔离    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 40 MB                     | 130 MB + 1 GB 页面文件 |
| 服务器核心 | 50 MB                     | 325 MB + 1 GB 页面文件 |

#### <a name="windows-server-version-1709"></a>Windows Server 版本 1709

| 基本图像  | Windows Server 容器 | Hyper-V 隔离    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 30 MB                     | 110 MB + 1 GB 页面文件 |
| 服务器核心 | 45 MB                     | 360 MB + 1 GB 页面文件 |

### <a name="base-image-differences"></a>基本图像差异

如何选择要在其上构建的正确的基本图像？ 尽管你可以随意构建任何内容, 但以下是每个图像的一般指南:

- [Windows Server Core](https://hub.docker.com/_/microsoft-windows-servercore): 如果你的应用程序需要完整的 .net framework, 这是要使用的最佳图像。
- [Nano server](https://hub.docker.com/_/microsoft-windows-nanoserver): 对于仅需要 .net Core 的应用程序, Nano server 将提供大量精简图像。
- [Windows](https://hub.docker.com/_/microsoft-windowsfamily-windows): 你可能会发现你的应用程序依赖于服务器 Core 或 Nano server 映像 (如 GDI 库) 中缺少的某个组件或 .dll。 此映像携带 Windows 的完整依赖集。
- [IoT 核心](https://hub.docker.com/_/microsoft-windows-iotcore): 此图像专为[IoT 应用程序](https://developer.microsoft.com/windows/iot)而构建。 在面向 IoT 核心主机时, 应使用此容器图像。

对于大多数用户, Windows Server Core 或 Nano Server 将是最适合使用的映像。 当你考虑在 Nano Server 上生成时, 请注意以下事项:

- 已删除服务堆栈
- 未包含 .NET Core（但你可以使用 [.NET Core Nano Server 映像](https://hub.docker.com/r/microsoft/dotnet/)）
- 已删除 PowerShell
- 已删除 WMI
- 自 Windows Server 版本 1709 起，在用户上下文环境下运行应用程序时，需要管理员权限的命令将会失败。 你可以通过--用户标志 (如 docker 运行-用户 ContainerAdministrator) 指定容器管理员帐户, 以后我们打算从 NanoServer 中完全删除管理员帐户。

以上是最显著的差异，其他差异并未一一列出。 还有一些缺失的其他组件并未在此列出。 请记住，你始终可以在自己认为合适的条件下在 Nano Server 上添加层。 有关此情况的示例，请查看 [.NET Core Nano Server Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile)。