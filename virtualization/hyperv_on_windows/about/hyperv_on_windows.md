---
title: Windows 10 上的 Hyper-V 简介
description: Windows 10 上的 Hyper-V 简介。
keywords: windows 10, hyper-v
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: &86227359 windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: eb2b827c-4a6c-4327-9354-50d14fee7ed8
---

# Windows 10 上的 Hyper-V 简介

无论你是软件开发人员、IT 专业人员还是技术爱好者，你们中的许多人都需要运行多个操作系统（有时需要在多台不同的计算机上运行）。 并非我们中的所有人都可以访问一整套实验室来容纳这些计算机。 与其将物理硬件专用于运行你的每台计算机，你可以将它们作为 Hyper-V *虚拟机* (VM) 运行，从而使用虚拟化技术以节省你的空间和时间。

> Microsoft Virtual PC 将在 2017 年 4 月结束使用。 将支持替换为 Windows 10 上的 HYPER-V。

## 虚拟化的用途

虚拟化使任何人都能够轻松维护多个包含许多操作系统、软件配置和硬件配置的测试环境。 Hyper-V 不仅在 Windows 上提供了虚拟化，还提供了一个简单的机制，用于在这些环境之间快速切换，而不会产生额外硬件成本。

Hyper-V 可以用于许多方面。 例如：

- 可以在一台台式机或笔记本电脑上创建一个包含多个虚拟机的测试环境。 测试完成后，可以将这些虚拟机导出并随后导入到任何其他 Hyper-V 系统中。

- 开发人员可以在他们的计算机上使用 Hyper-V 以在多个操作系统上测试软件。 例如，如果你的应用程序必须要在 Windows 8、Windows 7 和 Linux 操作系统上进行测试，你则可以在开发系统上创建多台虚拟机，每台虚拟机包含其中一个操作系统。

- 你可以使用 Windows 10 上的 Hyper-V 从任何 Hyper-V 部署中进行虚拟机疑难解答。 你可以从生产环境中导出虚拟机、在运行 Hyper-V 的桌面上将其打开、执行所需的疑难解答，然后将其重新导出到生产环境。

- 使用虚拟网络，你可以创建一个多计算机环境以进行测试/开发/演示，并且同时确保该环境免受生产网络的影响。

- 狂热爱好者们可以将 Hyper-V 用于实验其他操作系统。 通过 Hyper-V，可轻松打开和关闭不同的操作系统。

- 你可以在笔记本电脑上使用 Hyper-V 来演示较早版本的 Windows 或非 Windows 操作系统。


## 系统要求

Hyper-V 需要一个具有二级地址转换 (SLAT) 的 64 位系统。 SLAT 是 Intel 和 AMD 在当前这代的 64 位处理器中提供的一项功能。 你还需要一个 64 位版本的 Windows 8 或更高版本，并且至少有 4GB 的 RAM。 Hyper-V 支持在 VM 中同时创建 32 位和 64 位的操作系统。

Hyper-V 的动态内存允许将 VM 需要的内存动态分配和取消分配（你可指定最小值和最大值），并共享 VM 之间未使用的内存。 你可以在一台具有 4GB RAM 的计算机上运行 3 个或 4 个 VM，但是如果要运行 5 个或 5 个以上的 VM，则需要更多的 RAM。 另外，你可能还需要创建有 32 个处理器和 512GB RAM 的较大 VM，具体取决于你的物理硬件。

## 可以在虚拟机中运行的操作系统

术语“来宾”是指虚拟机，而“主机”是指运行虚拟机的计算机。 Windows 上的 Hyper-V 支持许多不同的来宾操作系统，其中包括各种版本的 Linux、FreeBSD 和 Windows。 有关 Windows 上的 Hyper-V 中作为来宾受支持的操作系统的信息，请参阅[受支持的 Windows 来宾操作系统](supported_guest_os.md)和 [Hyper-V 上的 Linux 和 FreeBSD 虚拟机](https://technet.microsoft.com/library/dn531030.aspx)。

## Windows 上的 Hyper-V 和 Windows Server 上的 Hyper-V 之间的差异

对于某些功能来说，其工作方式在 Windows 上的 Hyper-V 中和在运行于 Windows Server 上的 Hyper-V 中不同。 其中包括：

- 对于 Windows 上的 Hyper-V，内存管理模块不同。 在服务器上，通过假设只有虚拟机在该服务器上运行来管理 Hyper-V 内存。 在 Windows 上的 Hyper-V 中，通过大多数客户端计算机都在运行软件以及运行虚拟机的预期来管理内存。 例如，开发人员可能在同一台计算机上运行 Visual Studio 以及多个虚拟机。

- 64 位来宾上的 SR-IOV 仍正常工作，但 32 位将无法正常工作并且不受支持。

### Windows Server 功能在 Windows Hyper-V 中不可用

Windows Server 上的 Hyper-V 中包含的某些功能未包含在 Windows 上的 Hyper-V 中。 其中包括：

- 使用 RemoteFX 的虚拟化 GPU

- 将虚拟机从一台主机实时迁移到另一台主机

- Hyper-V 副本

- 虚拟光纤通道

- SR-IOV 网络

- 共享的 .VHDX

> **警告**：在 Hyper-V 上运行的虚拟机不会自动处理从有线连接移动到无线连接的操作。 必须手动更改虚拟机的网络适配器设置。

## 限制

使用虚拟化也存在一些限制。 依赖于特定硬件的功能或应用程序不能在 VM 中良好运行。 例如，需要使用 GPU 进行处理（不提供软件回退）的游戏或应用程序可能无法良好运行。 依赖于子 10 毫秒计时器的应用程序（如实时音乐混合应用等易受延迟影响的高精度应用）在 VM 中运行时也可能会出问题。

此外，如果已启用了虚拟化，易受延迟影响的高精度应用在主机操作系统中运行时则可能也会出问题。 （这是因为在启用了虚拟化后，主机操作系统也会在 Hyper-V 虚拟化层的顶部运行，就如来宾操作系统那样。 但是，与来宾操作系统不同，主机操作系统在这点上很特殊，它是直接访问所有硬件，这意味着具有特殊硬件要求的应用程序仍然可以在主机操作系统中运行，而不会出问题。）

提醒一下，对于你在 VM 中使用的任何操作系统，都需要具有有效的许可证。

## 下一步

[演练：Windows 10 上的 Hyper-V](..\quick_start\walkthrough.md)

查看 Windows 10 上的 Hyper-V 中的 [新增功能](whats_new.md)。







<!--HONumber=May16_HO2-->


