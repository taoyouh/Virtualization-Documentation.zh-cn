---
title: "Windows 10 上的 Hyper-V 简介"
description: "Windows 10 上的 Hyper-V 简介。"
keywords: windows 10, hyper-v
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: eb2b827c-4a6c-4327-9354-50d14fee7ed8
translationtype: Human Translation
ms.sourcegitcommit: c3e7cc07ac7e7d4e1c5f1827deb5951daa1e3749
ms.openlocfilehash: ad84961d0a79853e2aadcf9ed0e37e340103835a

---

# Windows 10 上的 Hyper-V 简介

无论你是软件开发人员、IT 专业人员还是技术爱好者，你们中的许多人都需要运行多个操作系统。  Hyper-V 让你可以在 Windows 计算机上以虚拟机 (VM) 的形式运行多个操作系统，而不是将物理硬件专用于每个计算机。

> Microsoft Virtual PC 将在 2017 年 4 月结束使用。 Windows 10 企业版和 Windows 10 专业版上的 Hyper-V 将成为受支持的替代产品。  

## 使用虚拟化的原因
虚拟化使任何人都能够在同一台物理计算机上运行多个操作系统、软件配置和硬件配置。  Hyper-V 提供了用于管理虚拟机的虚拟化和工具。

Hyper-V 可以用于许多方面。 例如：

* 运行需要早期版本的 Windows 操作系统或非 Windows 操作系统的软件。 

* 实验其他操作系统。 通过 Hyper-V，可轻松创建和删除不同的操作系统。

* 使用多个虚拟机在多个操作系统上测试软件。 通过 Hyper-V，可以在一部台式机或便携式计算机上运行所有内容。 可以将这些虚拟机导出并随后导入到任何其他 Hyper-V 系统中，包括 Azure。

* 从任何 Hyper-V 部署中对虚拟机执行排除故障。 你可以从生产环境中导出虚拟机、在运行 Hyper-V 的桌面上将其打开、对虚拟机执行故障排除，然后将其重新导出到生产环境。 

* 使用虚拟网络，你可以创建一个多计算机环境以进行测试/开发/演示，并且同时确保该环境免受生产网络的影响。

## 系统要求
Hyper-V 仅可用于 Windows 8 及更高版本的 Windows 专业版、企业版和教育版。

它需要一个具有二级地址转换 (SLAT) 的 64 位系统。 SLAT 是 Intel 和 AMD 在当前这代的 64 位处理器中提供的一项功能。  你还需要 64 位版本的 Windows。  
也就是说，Hyper-V 不支持虚拟机中的 32 位和 64 位操作系统。

你可以在具有 4GB RAM 的主机上运行 3 或 4 台基本虚拟机，但如果要运行更多的虚拟机则需要更多的资源。 另外，你可能还需要创建有 32 个处理器和 512GB RAM 的较大虚拟机，具体取决于你的物理硬件。

有关 Hyper-V 的操作系统要求以及如何验证 Hyper-V 在计算机上运行的详细信息，请参阅[演练：Windows 10 Hyper-V 系统要求](..\quick_start\walkthrough_install.md)。


## 可以在虚拟机中运行的操作系统
术语“来宾”是指虚拟机，而“主机”是指运行虚拟机的计算机。 Windows 上的 Hyper-V 支持许多不同的来宾操作系统，其中包括各种版本的 Linux、FreeBSD 和 Windows。 

提醒一下，对于你在 VM 中使用的任何操作系统，都需要具有有效的许可证。 

有关 Windows 上的 Hyper-V 中作为来宾支持的操作系统的信息，请参阅[受支持的 Windows 来宾操作系统](supported_guest_os.md)和 [Hyper-V 上的 Linux 和 FreeBSD 虚拟机](https://technet.microsoft.com/library/dn531030.aspx)。 


## Windows 上的 Hyper-V 和 Windows Server 上的 Hyper-V 之间的差异
对于某些功能来说，其工作方式在 Windows 上的 Hyper-V 中和在运行于 Windows Server 上的 Hyper-V 中不同。 

对于 Windows 上的 Hyper-V，内存管理模块不同。 在服务器上，通过假设只有虚拟机在该服务器上运行来管理 Hyper-V 内存。 在 Windows 上的 Hyper-V 中，通过大多数客户端计算机都在运行主机上的软件以及运行虚拟机的预期来管理内存。 例如，开发人员可能在同一台计算机上运行 Visual Studio 以及多个虚拟机。

### 仅在 Windows Server 中可用的 Hyper-V 功能
Windows Server 上的 Hyper-V 中包含的某些功能未包含在 Windows 上的 Hyper-V 中。 这些地方包括：

* 使用 RemoteFX 的虚拟化 GPU 
* 将虚拟机从一台主机实时迁移到另一台主机
* Hyper-V 副本
* 虚拟光纤通道
* SR-IOV 网络
* 共享的 .VHDX

## 限制
使用虚拟化也存在一些限制。 依赖于特定硬件的功能或应用程序不能在虚拟机中良好运行。 例如，需要使用 GPU 进行处理的游戏或应用程序可能无法良好运行。 依赖于子 10 毫秒计时器的应用程序（如实时音乐混合应用程序或高精度时间）在虚拟机中运行时也可能会出问题。

此外，如果已启用了 Hyper-V，这些易受延迟影响的高精度应用程序在主机中运行时可能也会出问题。  这是因为在启用了虚拟化后，主机操作系统也会在 Hyper-V 虚拟化层的顶部运行，就如来宾操作系统那样。 但是，与来宾操作系统不同，主机操作系统在这点上很特殊，它是直接访问所有硬件，这意味着具有特殊硬件要求的应用程序仍然可以在主机操作系统中运行，而不会出问题。

## 下一步
[演练：在 Windows 10 上安装 Hyper-V](..\quick_start\walkthrough_install.md) 

查看 Windows 10 上的 Hyper-V 中的[新增功能](whats_new.md)。




<!--HONumber=Jul16_HO2-->


