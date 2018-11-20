---
title: Windows 上的 Linux 容器
description: 了解可用于 HYPER-V 在 Windows 上运行 Linux 容器，就像它们是本机的不同方法。
keywords: LCOW，linux 容器，docker，容器
author: scooley
ms.date: 11/02/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: edfd11c8-ee99-42d8-9878-efc126fe1826
ms.openlocfilehash: 7db0135e5d5079d3b8cce815d051ecd6a7cb896b
ms.sourcegitcommit: 614e3ca3e6f4373b999a501a2829adbaa61de4c4
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/19/2018
ms.locfileid: "7277531"
---
# <a name="linux-containers-on-windows"></a>Windows 上的 Linux 容器

Linux 容器构成了巨大百分比的整个容器生态系统，是开发人员体验和生产环境的基础。  由于容器与容器主机共享内核，但是，直接在 Windows 上运行 Linux 容器无法选择[*](linux-containers.md#other-options-we-considered)。  这是虚拟化进入图片。

现在，有两种方法来与适用于 Windows 和 Hyper-v： 运行 Linux 容器

1. 这是什么 Docker 通常像现在在完整的 Linux VM 内运行 Linux 容器。
1. 这是用于 Windows 的 Docker 中的新选项使用[HYPER-V 隔离](../manage-containers/hyperv-container.md)(LCOW)-运行 Linux 容器。

本文将概述每种方法的工作原理、 提供有关何时选择哪种解决方案，指南和共享正在进行的工作。

## <a name="linux-containers-in-a-moby-vm"></a>Moby 虚拟机中的 Linux 容器

若要在 Linux 虚拟机中运行 Linux 容器，请遵循[Docker 的-入门指南](https://docs.docker.com/docker-for-windows/)中的说明。

Docker 已能够在 Windows 上运行 Linux 容器桌面，因为它首次发布于 2016年 （HYPER-V 隔离或 LCOW 推出之前） 使用[LinuxKit](https://github.com/linuxkit/linuxkit)基于 HYPER-V 上运行的虚拟机。

在此模型中，Docker 客户端上运行 Windows 桌面版，但调用到 Linux VM 上的 Docker 守护程序。

![Moby 与容器主机的虚拟机](media/MobyVM.png)

在此模型中，所有 Linux 容器都共享单个基于 Linux 的容器主机和所有 Linux 容器：

* 彼此和 Moby 虚拟机，但不是与 Windows 主机共享内核。
* 具有一致的存储和网络属性 （因为它们正在运行 Linux VM 上） 运行 Linux 上的 Linux 容器。

这还意味着 Linux 容器主机 （Moby 虚拟机） 需要运行 Docker 守护程序及其所有 Docker 守护程序的依赖项。

若要查看是否你正在使用 Moby 虚拟机中运行，请 HYPER-V 管理器 Moby 虚拟机使用 HYPER-V 管理器 UI 或通过运行`Get-VM`提升的 PowerShell 窗口中。

## <a name="linux-containers-with-hyper-v-isolation"></a>使用 HYPER-V 隔离的 Linux 容器

若要尝试 LCOW，请遵循[此-入门指南](../quick-start/quick-start-windows-10.md)中的 Linux 容器说明

使用 HYPER-V 隔离的 Linux 容器与正好的操作系统运行容器优化的 Linux 虚拟机中运行每个 Linux 容器 (LCOW)。  与最 Moby 虚拟机的方法，每个 LCOW 都有其自己的内核和其自己的虚拟机沙盒。  它们是也受 Windows 上的 Docker 直接。

![使用 HYPER-V 隔离 (LCOW) 的 Linux 容器](media/lcow-approach.png)

采用探讨容器管理 Moby 虚拟机的方法和 LCOW 之间有何，在 LCOW 模型容器管理将停留在 Windows 上，并通过 GRPC 和 containerd 发生的每个 LCOW 管理。  Linux 发行版容器用于 LCOW 这意味着可以有很多较小的清单。  右现在，我们将使用 LinuxKit 优化发行版容器使用，但如 Kata 其他项目生成类似高度优化 Linux 发行 (清除 Linux) 以及。

下面是每个 LCOW 探讨：

![LCOW 体系结构](media/lcow.png)

若要查看是否正在运行 LCOW，导航到`C:\Program Files\Linux Containers`。  如果 Docker 配置为使用 LCOW，将出现下面包含在每个 HYPER-V 容器中运行的最小 LinuxKit 发行版的几个文件。  请注意的优化的虚拟机组件是小于 100 MB，比 Moby 虚拟机中的 LinuxKit 图像小得多。

### <a name="work-in-progress"></a>正在进行的工作

LCOW 处于活动开发状态。  在[GitHub](https://github.com/moby/moby/issues/33850)上跟踪 Moby 项目的进度

#### <a name="bind-mounts"></a>绑定挂载

带有 `docker run -v ...` 的绑定挂载卷将文件存储于 Windows NTFS 文件系统，因此对于 POSIX 操作需要进行一些转换。 有些文件系统操作当前只得到部分实现或者未实现，这可能导致一些应用无法兼容。

以下操作目前对绑定挂载卷无效：

* MkNod
* XAttrWalk
* XAttrCreate
* 锁定
* Getlock
* Auth
* 刷新
* INotify

还有一些操作未能完全实现：

* GetAttr - Nlink 计数一直报告为 2
* 打开 - 仅 ReadWrite、WriteOnly 和 ReadOnly 标志得到实现

这些应用程序所有要求卷映射并不会开始菜单或运行正常。

* MySQL
* PostgreSQL
* WordPress
* Jenkins
* MariaDB
* RabbitMQ

### <a name="extra-information"></a>额外信息

[描述 LCOW docker 博客](https://blog.docker.com/2017/11/docker-for-windows-17-11/)

[Linux 容器视频](https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4)

[LinuxKit LCOW 内核以及生成说明](https://github.com/linuxkit/lcow)

## <a name="when-to-use-moby-vm-vs-lcow"></a>何时使用 Moby 虚拟机 vs LCOW

### <a name="when-to-use-moby-vm"></a>何时使用 Moby 虚拟机

右现在，我们建议了一运行 Linux 容器人员 Moby 虚拟机方法：

1. 想要在稳定容器环境。  这是用于 Windows 的 Docker 默认值。
1. 运行 Windows 或 Linux 容器，但很少都在同一时间。
1. 具有复杂或自定义网络之间的 Linux 容器的需求。
1. 无需内核隔离 （HYPER-V 隔离） 之间的 Linux 容器。

### <a name="when-to-use-lcow"></a>何时使用 LCOW

右现在，我们建议了一 LCOW 以人员：

1. 想要测试我们的最新技术。
1. 同时运行 Windows 和 Linux 容器。
1. 需要内核隔离 （HYPER-V 隔离） 之间的 Linux 容器。

## <a name="other-options-we-considered"></a>我们考虑其他选项

当我们已看看如何在 Windows 上运行 Linux 容器时，我们认为 WSL。 最终，我们选择的基于虚拟化的方法，以便 Windows 上的 Linux 容器可以与 Linux 上的 Linux 容器保持一致。 使用 HYPER-V 还可使 LCOW 更安全。 我们可能会重新评估将来，但现在，LCOW 将继续使用 HYPER-V。

如果你有想法，请发送通过 GitHub 或 UserVoice 的反馈。  特别感谢你希望看到的特定体验有关的反馈。
