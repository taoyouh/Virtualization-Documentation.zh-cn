---
title: Windows 上的 Linux 容器
description: 了解你可以使用 Hyper-v 在 Windows 上运行 Linux 容器的不同方式, 就像它们是本机一样。
keywords: LCOW、linux 容器、docker、容器
author: scooley
ms.date: 11/02/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: edfd11c8-ee99-42d8-9878-efc126fe1826
ms.openlocfilehash: 0426b14c423c06a0f12ea91529ce794f7a972f47
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998474"
---
# <a name="linux-containers-on-windows"></a>Windows 上的 Linux 容器

Linux 容器占整个容器生态系统的巨大百分比, 并且是开发人员体验和生产环境的基础。  但是, 由于容器与容器主机共享内核, 因此在 Windows 上直接运行 Linux 容器不是一个[*](linux-containers.md#other-options-we-considered)选项。  这是虚拟化进入图片的地方。

现在, 有两种方法可以通过 Windows 和 Hyper-v 的 Docker 运行 Linux 容器:

1. 在完整 Linux 虚拟机内运行 Linux 容器-这是由 Docker 目前通常所做的工作。
1. 使用[hyper-v 隔离](../manage-containers/hyperv-container.md)运行 Linux 容器 (LCOW)-这是 Windows 的 Docker 中的新选项。

本文概述了每种方法的工作方式, 提供了有关何时选择哪个解决方案以及正在进行工作的指南。

## <a name="linux-containers-in-a-moby-vm"></a>Moby VM 中的 Linux 容器

若要在 Linux VM 中运行 Linux 容器, 请按照[Docker 的入门指南](https://docs.docker.com/docker-for-windows/)中的说明进行操作。

由于在使用 Hyper-v 运行的基于[LinuxKit](https://github.com/linuxkit/linuxkit)的虚拟机, Docker 在 Windows desktop 上首次发布后, 它就能够在2016中运行 Linux 容器 (使用 hyper-v 隔离或 LCOW 之前)。

在此模型中, Docker 客户端在 Windows 桌面上运行, 但在 Linux VM 上调用到 Docker 后台程序。

![Moby VM 作为容器主机](media/MobyVM.png)

在此模型中, 所有 Linux 容器都共享单个基于 Linux 的容器主机和所有 Linux 容器:

* 彼此共享内核和 Moby VM, 但不与 Windows 主机共享。
* 具有与在 Linux 上运行的 Linux 容器一致的存储和网络属性 (因为它们在 Linux VM 上运行)。

这还意味着 Linux 容器主机 (Moby VM) 需要运行 Docker 后台程序和所有 Docker 后台程序的依赖项。

若要查看是否正在使用 Moby VM 运行, 请使用 Hyper-v 管理器 UI 或在提升的 PowerShell 窗口中运行`Get-VM`来检查 Moby VM 的 Hyper-v 管理器。

## <a name="linux-containers-with-hyper-v-isolation"></a>具有 Hyper-v 隔离的 Linux 容器

若要尝试 LCOW, 请按照[此入门指南](../quick-start/quick-start-windows-10.md)中的 Linux 容器说明进行操作

具有 Hyper-v 隔离的 Linux 容器在已优化的 Linux VM 中运行每个 Linux 容器 (LCOW), 只有足够的操作系统才能运行容器。  与 Moby VM 方法相比, 每个 LCOW 都有其自己的内核和自己的 VM 沙盒。  它们也可以由 Windows 上的 Docker 直接管理。

![具有 Hyper-v 隔离的 Linux 容器 (LCOW)](media/lcow-approach.png)

深入了解 Moby VM 方法和 LCOW 之间的容器管理如何在 Windows 上保留, 以及每个 LCOW 管理通过 GRPC 和 containerd 发生。  这意味着用于 LCOW 的 Linux distro 容器可以有更小的库存空间。  现在, 我们为优化的 distro 容器使用 LinuxKit, 但其他项目 (如 Kata) 也会生成类似高度优化的 Linux distros (明文 Linux)。

下面详细了解每个 LCOW:

![LCOW 体系结构](media/lcow.png)

若要查看是否正在运行 LCOW, 请导航`C:\Program Files\Linux Containers`到。 如果 Docker 配置为使用 LCOW, 则此处将有一些文件, 其中包含在 Hyper-v 隔离下运行的每个容器中运行的最低 LinuxKit distro。  注意, 优化的 VM 组件小于 100 MB, 比 Moby VM 中的 LinuxKit 映像更小。

### <a name="work-in-progress"></a>正在进行的工作

LCOW 处于活动开发下。 跟踪[GitHub](https://github.com/moby/moby/issues/33850)上 Moby 项目中的正在进行的进度

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

这些应用程序都需要卷映射, 并且不会正确启动或运行。

* MySQL
* PostgreSQL
* WordPress
* Jenkins
* MariaDB
* RabbitMQ

### <a name="extra-information"></a>额外信息

[介绍 LCOW 的 Docker 博客](https://blog.docker.com/2017/11/docker-for-windows-17-11/)

[Linux 容器视频](https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4)

[LinuxKit LCOW-内核 + 生成说明](https://github.com/linuxkit/lcow)

## <a name="when-to-use-moby-vm-vs-lcow"></a>何时使用 Moby VM vs LCOW

### <a name="when-to-use-moby-vm"></a>何时使用 Moby VM

现在, 我们将运行 Linux 容器的 Moby VM 方法推荐给以下人员:

- 需要稳定的容器环境。  这是 Windows 的 Docker 默认值。
- 运行 Windows 或 Linux 容器, 但同时很少同时运行。
- 在 Linux 容器之间有复杂或自定义的网络要求。
- 在 Linux 容器之间不需要内核隔离 (Hyper-v 隔离)。

### <a name="when-to-use-lcow"></a>何时使用 LCOW

现在, 我们建议 LCOW 用户:

- 想要测试我们的最新技术。
- 同时运行 Windows 和 Linux 容器。
- 在 Linux 容器之间需要内核隔离 (Hyper-v 隔离)。

## <a name="other-options-we-considered"></a>我们考虑的其他选项

当我们查看在 Windows 上运行 Linux 容器的方法时, 我们认为 WSL。 最终, 我们选择了一个基于虚拟化的方法, 以便 Windows 上的 Linux 容器与 Linux 上的 Linux 容器保持一致。 使用 Hyper-v 还会使 LCOW 更加安全。 我们可能会在将来重新评估, 但目前 LCOW 将继续使用 Hyper-v。

如果你有想法, 请通过 GitHub 或 UserVoice 发送反馈。  我们特别感谢你想要查看的特定体验的反馈。
