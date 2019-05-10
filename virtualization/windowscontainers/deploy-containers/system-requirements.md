---
title: Windows 容器要求
description: Windows 容器要求。
keywords: 元数据, 容器
author: taylorb-microsoft
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: 19943e45df7847e83010ca31c01c6cb5b18d41cf
ms.sourcegitcommit: 34d8b2ca5eebcbdb6958560b1f4250763bee5b48
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "9621495"
---
# <a name="windows-container-requirements"></a>Windows 容器要求

本指南列出 Windows 容器主机的要求。

## <a name="os-requirements"></a>操作系统要求

- Windows 容器功能仅适用于 Windows Server 2016 (核心和带桌面体验)，Windows 10 专业版和企业版 （周年纪念版） 和更高版本。
- 运行 HYPER-V 隔离之前必须安装 HYPER-V 角色
- Windows Server 容器主机必须将 Windows 安装到 c:\。 此限制不适用于如果仅将部署 HYPER-V 隔离容器。

## <a name="virtualized-container-hosts"></a>虚拟化的容器主机

如果 Windows 容器主机将从 HYPER-V 虚拟机，运行，并且还将承载 HYPER-V 隔离，将需要启用嵌套虚拟化。 嵌套的虚拟化具有以下要求：

- 至少 4 GB RAM 可用于虚拟化的 Hyper-V 主机。
- Windows Server 2019，Windows Server 版本 1803、 Windows Server 版本 1709、 Windows Server 2016 或 Windows 10 上的主机系统和 Windows Server （Full、 Core） 在虚拟机中。
- 带有 Intel VT-x 处理器（此功能目前只适用于 Intel 处理器）。
- 容器主机虚拟机还需要至少两个虚拟处理器。

## <a name="supported-base-images"></a>支持的基本映像

Windows 容器提供四个容器基本映像： Windows Server Core、 Nano Server、 Windows 和 IoT 核心版。 并非所有配置都支持这两个操作系统映像。 下表详细介绍所支持的配置。

|主机操作系统|Windows 容器|Hyper-V 隔离|
|---------------------|-----------------|-----------------|
|Windows Server 2016 或 Windows Server 2019 （Standard 或 Datacenter）|服务器核心 Windows 在 Nano Server|服务器核心 Windows 在 Nano Server|
|Nano Server|Nano Server|服务器核心 Windows 在 Nano Server|
|Windows 10 专业版或 Windows 10 企业版|不可用|服务器核心 Windows 在 Nano Server|
|IoT Core|IoT Core|不可用|

> [!WARNING]  
> 从 Windows Server 版本 1709年开始，Nano Server 不再作为容器主机提供。

### <a name="memory-requirements"></a>内存要求

可以通过[资源控制](https://docs.microsoft.com/virtualization/windowscontainers/manage-containers/resource-controls)或重载容器主机来配置容器可用内存限值。  下面列出的最小启动容器和运行基本命令 （ipconfig、 dir 等） 所需的内存量。

>[!NOTE]
>这些值未考虑容器或来自在容器中运行的应用程序的要求之间共享的资源。  例如，具有 512MB 可用内存的主机可以在 Hyper-V 隔离下运行多个 Server Core 容器，因为这些容器会共享资源。

#### <a name="windows-server-2016"></a>WindowsServer 2016

| 基本映像  | Windows Server 容器 | Hyper-V 隔离    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 40 MB                     | 130mb + 1GB 页面文件 |
| 服务器核心 | 50 MB                     | 325mb + 1GB 页面文件 |

#### <a name="windows-server-version-1709"></a>Windows Server 版本 1709

| 基本映像  | Windows Server 容器 | Hyper-V 隔离    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 30 MB                     | 110mb + 1GB 页面文件 |
| 服务器核心 | 45 MB                     | 360 + 1GB 页面文件 |

### <a name="base-image-differences"></a>基本映像的差异

一个如何选择正确基本映像，以生成后？ 虽然可以随意使用它们进行生成之中，这些是为每个图像的常规准则：

- [Windows Server Core](https://hub.docker.com/_/microsoft-windows-servercore)： 如果你的应用程序需要完整的.NET framework，这是要使用的最佳映像。
- [Nano Server](https://hub.docker.com/_/microsoft-windows-nanoserver): Nano Server 将仅需要.NET Core 应用程序，用于提供更简洁的图像。
- [Windows](https://hub.docker.com/_/microsoft-windowsfamily-windows)： 你可能会发现应用程序所依赖的组件或服务器核心中缺少的.dll 或 Nano Server 映像，如 GDI 库。 此图像带来的完全依赖项一套 Windows。
- [IoT 核心版](https://hub.docker.com/_/microsoft-windows-iotcore)： 此图像是专门针对[IoT 应用程序](https://developer.microsoft.com/windows/iot)构建。 面向 IoT 核心版主机时，你应使用此容器映像。

对于大多数用户，Windows Server Core 或 Nano Server 将是要使用的最合适图像。 以下是需要考虑在 Nano Server 上生成，请记住的事项：

- 已删除服务堆栈
- 未包含 .NET Core（但你可以使用 [.NET Core Nano Server 映像](https://hub.docker.com/r/microsoft/dotnet/)）
- 已删除 PowerShell
- 已删除 WMI
- 自 Windows Server 版本 1709 起，在用户上下文环境下运行应用程序时，需要管理员权限的命令将会失败。 你可以指定容器管理员帐户通过用户标记 （如 docker run-用户 ContainerAdministrator)，不过未来我们打算从 NanoServer 中完全移除管理员帐户。

以上是最显著的差异，其他差异并未一一列出。 还有一些缺失的其他组件并未在此列出。 请记住，你始终可以在自己认为合适的条件下在 Nano Server 上添加层。 有关此情况的示例，请查看 [.NET Core Nano Server Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile)。