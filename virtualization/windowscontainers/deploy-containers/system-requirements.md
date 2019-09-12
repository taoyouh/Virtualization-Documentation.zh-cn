---
title: Windows 容器要求
description: Windows 容器要求。
keywords: 元数据, 容器
author: taylorb-microsoft
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: df5d8e17d0d512f7f53fcacf6c2c2a2652f3e7c0
ms.sourcegitcommit: 73134bf279f3ed18235d24ae63cdc2e34a20e7b7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/12/2019
ms.locfileid: "10107861"
---
# <a name="windows-container-requirements"></a>Windows 容器要求

本指南列出了 Windows 容器主机的要求。

## <a name="os-requirements"></a>操作系统要求

- Windows 容器功能仅在 Windows Server 2016 （核心和桌面体验）、Windows 10 专业版和企业版（周年纪念日）及更高版本中可用。
- 必须先安装 Hyper-v 角色，然后才能运行 Hyper-v 隔离
- Windows Server 容器主机必须将 Windows 安装到 c:\。 如果仅部署 Hyper-v 隔离容器，此限制将不适用。

## <a name="virtualized-container-hosts"></a>虚拟化容器主机

如果 Windows 容器主机将从 Hyper-v 虚拟机运行，并且也将托管 Hyper-v 隔离，则需要启用嵌套虚拟化。 嵌套的虚拟化具有以下要求：

- 至少 4 GB RAM 可用于虚拟化的 Hyper-V 主机。
- Windows Server 2019、Windows Server 版本1803、Windows server 版本1709、Windows Server 2016 或虚拟机中的 windows Server （Full、Core）。
- 带有 Intel VT-x 处理器（此功能目前只适用于 Intel 处理器）。
- 容器主机 VM 还需要至少两个虚拟处理器。

### <a name="memory-requirements"></a>内存要求

可以通过[资源控制](https://docs.microsoft.com/virtualization/windowscontainers/manage-containers/resource-controls)或重载容器主机来配置容器可用内存限值。  下面列出了启动容器和运行基本命令（ipconfig、dir 等）所需的最小内存量。

>[!NOTE]
>在容器中运行的应用程序中，这些值不考虑容器或要求之间的资源共享。  例如，具有 512MB 可用内存的主机可以在 Hyper-V 隔离下运行多个 Server Core 容器，因为这些容器会共享资源。

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
