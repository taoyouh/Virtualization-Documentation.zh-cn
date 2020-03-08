---
title: Windows 10 上的 Linux 容器
description: 了解在 Windows 10 上使用 Hyper-v 以本机方式运行 Linux 容器的不同方法。
keywords: LCOW、linux 容器、docker、容器、windows 10
author: scooley
ms.date: 09/17/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: edfd11c8-ee99-42d8-9878-efc126fe1826
ms.openlocfilehash: 843bd0ab7ccf3a227482ba3a3d2677e36b395b29
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/06/2020
ms.locfileid: "78854011"
---
# <a name="linux-containers-on-windows-10"></a>Windows 10 上的 Linux 容器

Linux 容器占总体容器生态系统的巨大百分比，是开发人员体验和生产环境的基础。  不过，由于容器与容器主机共享内核，因此不能选择[*](linux-containers.md#other-options-we-considered)上直接在 Windows 上运行 Linux 容器。  这就是虚拟化的图片。

目前有两种方法可通过用于 Windows 的 Docker 和 Hyper-v 运行 Linux 容器：

- 在完整的 Linux 虚拟机中运行 Linux 容器-这是 Docker 目前的典型功能。
- 使用[hyper-v 隔离](../manage-containers/hyperv-container.md)运行 Linux 容器（LCOW）-这是用于 Windows 的 Docker 中的新选项。

> _在 Windows Server 操作系统上运行 Linux 容器当前仍处于试验阶段。需要提供 Docker EE 计划的其他许可才能尝试此方案。**本文的其余部分仅适用于 Windows 10**。_

本文概述了每种方法的工作原理，并提供有关何时选择解决方案以及共享正在进行的操作的指导。

## <a name="linux-containers-in-a-moby-vm"></a>小鲸鱼 VM 中的 Linux 容器

若要在 Linux VM 中运行 Linux 容器，请按照[Docker 入门指南](https://docs.docker.com/docker-for-windows/)中的说明进行操作。

Docker 已能够在 Windows 桌面上运行 Linux 容器，因为它是第一次在2016中发布的（在 Windows 上的 Hyper-v 隔离或 Linux 容器可用之前），该虚拟机使用在 Hyper-v 上运行的基于[LinuxKit](https://github.com/linuxkit/linuxkit)的虚拟机。

在此模型中，Docker 客户端在 Windows 桌面上运行，但在 Linux VM 上调用到 Docker 后台程序。

![作为容器主机的小鲸鱼 VM](media/MobyVM.png)

在此模型中，所有 Linux 容器共享一个基于 Linux 的容器主机和所有 Linux 容器：

* 彼此共享内核和小鲸鱼 VM，但不与 Windows 主机共享。
* 在 linux 上运行的 Linux 容器使用一致的存储和网络属性（因为它们在 Linux VM 上运行）。

这也意味着 Linux 容器主机（小鲸鱼 VM）需要运行 Docker 守护程序和所有 Docker 守护程序依赖项。

若要查看是否正在使用小鲸鱼 VM 运行，请使用 Hyper-v 管理器 UI 或通过在提升的 PowerShell 窗口中运行 `Get-VM` 来检查小鲸鱼 VM 的 Hyper-v 管理器。

## <a name="linux-containers-with-hyper-v-isolation"></a>具有 Hyper-v 隔离的 Linux 容器

若要在 Windows 10 上试用 Linux 容器（LCOW10），请按照[Windows 10 上 linux](../quick-start/quick-start-windows-10-linux.md)容器中的 linux 容器说明进行操作。 

使用 Hyper-v 隔离的 linux 容器在经过优化的 Linux VM 中运行每个 Linux 容器，只有足够的操作系统才能运行容器。 与小鲸鱼 VM 方法相比，每个 Linux 容器都有自己的内核和自己的 VM 沙盒。 它们还可以由 Docker 在 Windows 上直接管理。

![具有 Hyper-v 隔离（LCOW）的 Linux 容器](media/lcow-approach.png)

进一步了解容器管理在小鲸鱼 VM 方法和 LCOW 之间的差异，在 LCOW 模型容器管理中，在 Windows 上保留，每个 LCOW 管理通过 GRPC 和 containerd 发生。  这意味着，用于 LCOW 的 Linux 发行版容器使用的清单可能要少得多。  现在，我们将使用 LinuxKit 来优化发行版容器，但其他项目（如 Kata）也会构建类似的高度优化 Linux 发行版（明文 Linux）。

下面是每个 LCOW 的详细介绍：

![LCOW 体系结构](media/lcow.png)

若要查看是否正在运行 LCOW，请导航到 `C:\Program Files\Linux Containers`。 如果 Docker 配置为使用 LCOW，则此处会出现一些文件，其中包含在 Hyper-v 隔离下运行的每个容器中运行的最小 LinuxKit 发行版。  请注意，优化的 VM 组件小于 100 MB，比小鲸鱼 VM 中的 LinuxKit 映像小得多。

### <a name="work-in-progress"></a>正在进行的工作

LCOW 处于积极开发阶段。 跟踪[GitHub](https://github.com/moby/moby/issues/33850)上的小鲸鱼项目中的正在进行的进度

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

这些应用程序都需要卷映射，并且不会正常启动或运行。

* MySQL
* PostgreSQL
* WordPress
* Jenkins
* MariaDB
* RabbitMQ

### <a name="extra-information"></a>额外信息

[介绍 LCOW 的 Docker 博客](https://blog.docker.com/2017/11/docker-for-windows-17-11/)

[Linux 容器视频](https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4)

[LinuxKit LCOW-内核和生成说明](https://github.com/linuxkit/lcow)

## <a name="when-to-use-moby-vm-vs-lcow"></a>何时使用小鲸鱼 VM vs LCOW

### <a name="when-to-use-moby-vm"></a>何时使用小鲸鱼 VM

现在，我们建议将 Linux 容器的小鲸鱼 VM 方法用于以下人员：

- 需要稳定的容器环境。  这是用于 Windows 的 Docker 默认值。
- 运行 Windows 或 Linux 容器，但这种情况很少发生。
- Linux 容器之间存在复杂的或自定义的网络要求。
- 不需要 Linux 容器之间的内核隔离（Hyper-v 隔离）。

### <a name="when-to-use-lcow"></a>何时使用 LCOW

现在，我们建议 LCOW：

- 想要测试我们的最新技术。
- 同时运行 Windows 和 Linux 容器。
- 需要 Linux 容器之间的内核隔离（Hyper-v 隔离）。

## <a name="other-options-we-considered"></a>我们考虑的其他选项

当我们查看在 Windows 上运行 Linux 容器的方法时，我们考虑到了 WSL。 最终，我们选择了基于虚拟化的方法，以便 Windows 上的 Linux 容器与 Linux 上的 Linux 容器一致。 使用 Hyper-v 还会使 LCOW 更安全。 我们可能会在将来重新评估，但现在，LCOW 将继续使用 Hyper-v。

如果你有想法，请通过 GitHub 或 UserVoice 发送反馈。  我们特别感谢你要查看的特定体验的反馈。
