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
ms.sourcegitcommit: d0411b05d1ef7328a770766b84fd0743f9d9c237
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2019
ms.locfileid: "10254257"
---
# <a name="windows-container-requirements"></a>Windows 容器要求

本指南列出了 Windows 容器主机的要求。

## <a name="operating-system-requirements"></a>操作系统要求

- Windows 容器功能在 Windows Server （半年度频道）、Windows Server 2019、Windows Server 2016 以及 Windows 10 专业版和企业版（版本1607及更高版本）中可用。
- 必须先安装 Hyper-v 角色，然后才能运行 Hyper-v 隔离
- Windows Server 容器主机必须将 Windows 安装到 c:\。 如果仅部署 Hyper-v 隔离容器，此限制将不适用。

## <a name="virtualized-container-hosts"></a>虚拟化容器主机

如果 Windows 容器主机将从 Hyper-v 虚拟机运行，并且也将托管 Hyper-v 隔离，则需要启用嵌套虚拟化。 嵌套的虚拟化具有以下要求：

- 至少 4 GB RAM 可用于虚拟化的 Hyper-V 主机。
- Windows Server （半年度频道）、Windows Server 2019、Windows Server 2016 或主机系统上的 Windows 10;和 Windows Server （Full 或 Server Core）在虚拟机中。
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

#### <a name="windows-server-semi-annual-channel"></a>Windows Server（半年渠道）

| 基本图像  | Windows Server 容器 | Hyper-V 隔离    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 30 MB                     | 110 MB + 1 GB 页面文件 |
| 服务器核心 | 45 MB                     | 360 MB + 1 GB 页面文件 |

## <a name="see-also"></a>另请参阅

[本地方案中 Windows 容器和 Docker 的支持策略](https://support.microsoft.com/help/4489234/support-policy-for-windows-containers-and-docker-on-premises)