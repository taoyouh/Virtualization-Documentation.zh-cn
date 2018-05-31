---
title: Windows 10 上的 Hyper-V 简介
description: Hyper-V、虚拟化和相关技术简介。
keywords: windows 10, hyper-v
author: scooley
ms.date: 04/07/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: eb2b827c-4a6c-4327-9354-50d14fee7ed8
ms.openlocfilehash: 78991d0b6d8b27ea20365fed74f35cee64eb089f
ms.sourcegitcommit: 64c8d5d6f068d385b94db4637259bb3852666efe
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2018
ms.locfileid: "1797673"
---
# <a name="introduction-to-hyper-v-on-windows-10"></a>Windows 10 上的 Hyper-V 简介

> Hyper-V 替换了 Microsoft Virtual PC。

无论你是软件开发人员、IT 专业人员还是技术爱好者，你们中的许多人都需要运行多个操作系统。 Hyper-V 让你可以在 Windows 上以虚拟机形式运行多个操作系统。

![运行 Windows 的虚拟机](media/HyperVNesting.png)

具体来说，Hyper-V 提供硬件虚拟化。  这意味着每个虚拟机都在虚拟硬件上运行。  Hyper-V 允许你创建虚拟硬盘驱动器、虚拟交换机以及许多其他虚拟设备，所有这些都可以添加到虚拟机中。

## <a name="reasons-to-use-virtualization"></a>使用虚拟化的原因

虚拟化允许你：

* 运行需要早期版本的 Windows 操作系统或非 Windows 操作系统的软件。

* 实验其他操作系统。 通过 Hyper-V，可轻松创建和删除不同的操作系统。

* 使用多个虚拟机在多个操作系统上测试软件。 通过 Hyper-V，可以在一部台式机或便携式计算机上运行所有内容。 可以将这些虚拟机导出并随后导入到任何其他 Hyper-V 系统中，包括 Azure。

## <a name="system-requirements"></a>系统要求

Hyper-V 可用于 Windows 8 及更高版本的 64 位 Windows 专业版、企业版和教育版。  它无法用于 Windows 家庭版。

> 打开**设置** > **更新和安全** > **激活**，从 Windows 10 家庭版升级到 Windows 10 专业版。 可以在此处访问应用商店并购买升级。

大多数计算机将运行 Hyper-V，但每个虚拟机都是一个完全独立的操作系统。  通常，你可以在具有 4GB RAM 的计算机上运行一个或多个虚拟机，但是你需要更多的资源以供其他虚拟机使用，或安装和运行资源密集型软件，如游戏、视频编辑或工程设计软件。

有关 Hyper-V 的系统要求以及如何验证 Hyper-V 在计算机上运行的详细信息，请参阅 [Hyper-V 要求参考](..\reference\hyper-v-requirements.md)。

## <a name="operating-systems-you-can-run-in-a-virtual-machine"></a>可以在虚拟机中运行的操作系统

Windows 上的 Hyper-V 支持虚拟机中的许多不同操作系统，其中包括各种版本的 Linux、FreeBSD 和 Windows。

提醒一下，对于你在 VM 中使用的任何操作系统，都需要具有有效的许可证。

有关 Windows 上的 Hyper-V 中作为来宾支持的操作系统的信息，请参阅[受支持的 Windows 来宾操作系统](supported-guest-os.md)和 [受支持的 Linux 来宾操作系统](https://technet.microsoft.com/library/dn531030.aspx)。

## <a name="differences-between-hyper-v-on-windows-and-hyper-v-on-windows-server"></a>Windows 上的 Hyper-V 和 Windows Server 上的 Hyper-V 之间的差异

对于某些功能来说，其工作方式在 Windows 上的 Hyper-V 中和在运行于 Windows Server 上的 Hyper-V 中不同。

仅在 Windows Server 中可用的 Hyper-V 功能：

* 使用 RemoteFX 的虚拟化 GPU
* 将虚拟机从一台主机实时迁移到另一台主机
* Hyper-V 副本
* 虚拟光纤通道
* SR-IOV 网络
* 共享的 .VHDX

仅在 Windows 10 中可用的 Hyper-V 功能：

* 快速创建和 VM 库
* 默认网络（NAT 交换机）

对于 Windows 上的 Hyper-V，内存管理模块不同。 在服务器上，通过假设只有虚拟机在该服务器上运行来管理 Hyper-V 内存。 在 Windows 上的 Hyper-V 中，通过大多数客户端计算机都在运行主机上的软件以及运行虚拟机的预期来管理内存。

## <a name="limitations"></a>限制

依赖于特定硬件的程序不能在虚拟机中良好运行。 例如，需要使用 GPU 进行处理的游戏或应用程序可能无法良好运行。 依赖于子 10 毫秒计时器的应用程序（如实时音乐混合应用程序或高精度时间）在虚拟机中运行时也可能会出问题。

此外，如果已启用了 Hyper-V，这些易受延迟影响的高精度应用程序在主机中运行时可能也会出问题。  这是因为在启用了虚拟化后，主机操作系统也会在 Hyper-V 虚拟化层的顶部运行，就如来宾操作系统那样。 但是，与来宾操作系统不同，主机操作系统在这点上很特殊，它是直接访问所有硬件，这意味着具有特殊硬件要求的应用程序仍然可以在主机操作系统中运行，而不会出问题。

## <a name="next-step"></a>下一步

[在 Windows 10 上安装 Hyper-V](..\quick-start\enable-hyper-v.md)
