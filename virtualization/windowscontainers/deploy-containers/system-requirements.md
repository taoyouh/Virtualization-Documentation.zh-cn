---
title: Windows 容器要求
description: Windows 容器要求。
keywords: 元数据, 容器
author: taylorb-microsoft
ms.author: taylorb
ms.date: 10/22/2019
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: 74f501e5efab3a93e60c9d4797464cea283cdc0b
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910497"
---
# <a name="windows-container-requirements"></a>Windows 容器要求

本指南列出了对 Windows 容器主机的要求。

## <a name="operating-system-requirements"></a>操作系统要求

- Windows 容器功能在 Windows Server（半年频道）、Windows Server 2019、Windows Server 2016 和 Windows 10 专业版和企业版（版本 1607 及更高版本）中可用。
- 在运行 Hyper-V 隔离操作之前必须安装 Hyper-V 角色
- Windows Server 容器主机必须将 Windows 安装到 c:\。 如果仅部署 Hyper-V 隔离容器，则没有此限制。

## <a name="virtualized-container-hosts"></a>虚拟化容器主机

如果 Windows 容器主机将从 Hyper-V 虚拟机运行，并且还将承载 Hyper-V 隔离，则需要启用嵌套虚拟化。 嵌套的虚拟化具有以下要求：

- 至少 4 GB RAM 可用于虚拟化的 Hyper-V 主机。
- 主机系统使用 Windows Server（半年频道）、Windows Server 2019、Windows Server 2016 或 Windows 10；虚拟机使用 Windows Server（完整版或 Server Core 版）。
- 带有 Intel VT-x 处理器（此功能目前只适用于 Intel 处理器）。
- 容器主机虚拟机还需要至少两个虚拟处理器。

### <a name="memory-requirements"></a>内存要求

可以通过[资源控制](https://docs.microsoft.com/virtualization/windowscontainers/manage-containers/resource-controls)或重载容器主机来配置容器可用内存限值。  下面列出了启动容器和运行基本命令（ipconfig、dir 等）所需的最小内存量。

>[!NOTE]
>这些值未考虑容器之间的资源共享或来自在容器中运行的应用程序的要求。  例如，具有 512MB 可用内存的主机可以在 Hyper-V 隔离下运行多个 Server Core 容器，因为这些容器会共享资源。

#### <a name="windows-server-2016"></a>Windows Server 2016

| Base image  | Windows Server 容器 | Hyper-V 隔离    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 40 MB                     | 130 MB + 1 GB 页面文件 |
| 服务器核心 | 50 MB                     | 325 MB + 1 GB 页面文件 |

#### <a name="windows-server-semi-annual-channel"></a>Windows Server（半年频道）

| Base image  | Windows Server 容器 | Hyper-V 隔离    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 30 MB                     | 110 MB + 1 GB 页面文件 |
| 服务器核心 | 45 MB                     | 360 MB + 1 GB 页面文件 |

## <a name="see-also"></a>另请参阅

[针对本地方案中 Windows 容器和 Docker 的支持策略](https://support.microsoft.com/help/4489234/support-policy-for-windows-containers-and-docker-on-premises)