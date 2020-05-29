---
title: Windows 10 上的 Linux 容器
description: 了解使用 Hyper-V 在 Windows 10 上像运行原生容器那样运行 Linux 容器的各种方法。
keywords: LCOW, linux 容器, docker, 容器, windows 10
author: scooley
ms.date: 09/17/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: edfd11c8-ee99-42d8-9878-efc126fe1826
ms.openlocfilehash: 843bd0ab7ccf3a227482ba3a3d2677e36b395b29
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/06/2020
ms.locfileid: "78854011"
---
# <a name="linux-containers-on-windows-10"></a>Windows 10 上的 Linux 容器

Linux 容器占整个容器生态系统的很大一部分，对开发人员体验和生产环境都至关重要。  但是，由于容器与容器主机共享一个内核，因此不能直接在 Windows 上运行 Linux 容器[*](linux-containers.md#other-options-we-considered)。  因此，虚拟化进入了我们的视野中。

目前，可通过两种方法使用 Docker for Windows 和 Hyper-V 运行 Linux 容器：

- 在完全的 Linux VM 中运行 Linux 容器 - 这是 Docker 目前的典型做法。
- 使用 [Hyper-V 隔离](../manage-containers/hyperv-container.md) (LCOW) 运行 Linux 容器 - 这是 Docker for Windows 中的一个新选项。

> _在 Windows Server 操作系统上运行 Linux 容器当前仍处于试验阶段。需要提供 Docker EE 计划的其他许可才能尝试此方法。**本文的剩余部分仅适用于 Windows 10**。_

本文概述了每种方法的工作原理，提供了有关何时选择哪种解决方案的指导，并分享了正在进行的工作。

## <a name="linux-containers-in-a-moby-vm"></a>Moby VM 中的 Linux 容器

若要在 Linux VM 中运行 Linux 容器，请按照 [Docker 入门指南](https://docs.docker.com/docker-for-windows/)中的说明进行操作。

首次于 2016 年发布以来（在 Windows 上的 Hyper-V 隔离或 Linux 容器功能发布之前），Docker 就一直能够使用在 Hyper-V 上运行的基于 [LinuxKit](https://github.com/linuxkit/linuxkit) 的虚拟机在 Windows 桌面上运行 Linux 容器。

在此模型中，Docker 客户端在 Windows 桌面上运行，但调用 Linux VM 上的 Docker 守护程序。

![Moby VM 作为容器主机](media/MobyVM.png)

在此模型中，所有 Linux 容器共享一个基于 Linux 的容器主机，并且所有 Linux 容器具有以下特点：

* 彼此共享内核和 Moby VM，但不与 Windows 主机共享。
* 与在 Linux 上运行的 Linux 容器具有一致的存储和网络属性（因为它们在 Linux VM 上运行）。

这也意味着 Linux 容器主机 (Moby VM) 需要运行 Docker 守护程序和所有 Docker 守护程序依赖项。

若要查看是否正在使用 Moby VM 运行，请使用 Hyper-V 管理器 UI 来检查用于 Moby VM 的 Hyper-V 管理器，或者通过在提升的 PowerShell 窗口中运行 `Get-VM` 来这样做。

## <a name="linux-containers-with-hyper-v-isolation"></a>采用 Hyper-V 隔离的 Linux 容器

若要试用 Windows 10 上的 Linux 容器 (LCOW10)，请按照 [Windows 10 上的 Linux 容器](../quick-start/quick-start-windows-10-linux.md)中的 Linux 容器说明进行操作。 

采用 Hyper-V 隔离的 Linux 容器在经过优化的 Linux VM 中（仅含恰好够运行容器的操作系统）运行每个 Linux 容器。 与 Moby VM 方法相反，每个 Linux 容器都有其自己的内核和自己的 VM 沙盒。 它们也由 Windows 上的 Docker 直接管理。

![采用 Hyper-V 隔离的 Linux 容器 (LCOW)](media/lcow-approach.png)

仔细观察 Moby VM 方法与 LCOW 之间在容器管理方面的差别就可以发现，在 LCOW 模型中，容器管理保持在 Windows 上，每个 LCOW 管理通过 GRPC 和 containerd 进行。  这意味着，用于 LCOW 的 Linux 发行版容器的清单可以小得多。  目前，我们正在使用 LinuxKit 来优化发行版容器的使用，但其他项目（例如 Kata）也在构建类似的高度优化的 Linux 发行版 (Clear Linux)。

下面是每个 LCOW 更详细的视图：

![LCOW 体系结构](media/lcow.png)

若要查看是否正在运行 LCOW，请导航到 `C:\Program Files\Linux Containers`。 如果 Docker 配置为使用 LCOW，则此处会显示一些文件，其中包含在 Hyper-V 隔离下运行的每个容器中运行的最小 LinuxKit 发行版。  请注意，优化的 VM 组件小于 100 MB，比 Moby VM 中的 LinuxKit 映像小得多。

### <a name="work-in-progress"></a>正在进行的工作

正在积极开发 LCOW。 可在 [GitHub](https://github.com/moby/moby/issues/33850) 上跟踪 Moby 项目的当前进展

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

这些应用程序都需要卷映射，不会正常启动或运行。

* MySQL
* PostgreSQL
* WordPress
* Jenkins
* MariaDB
* RabbitMQ

### <a name="extra-information"></a>额外的信息

[介绍 LCOW 的 Docker 博客](https://blog.docker.com/2017/11/docker-for-windows-17-11/)

[Linux 容器视频](https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4)

[LinuxKit LCOW-内核和生成说明](https://github.com/linuxkit/lcow)

## <a name="when-to-use-moby-vm-vs-lcow"></a>何时使用 Moby VM 以及何时使用 LCOW

### <a name="when-to-use-moby-vm"></a>何时使用 Moby VM

目前，我们建议有以下需求的人员使用 Moby VM 方法运行 Linux 容器：

- 需要稳定的容器环境。  这是 Docker for Windows 的默认设置。
- 运行 Windows 或 Linux 容器，但极少情况下会同时运行两者。
- 在 Linux 容器之间有复杂的或自定义的网络要求。
- 在 Linux 容器之间不需要内核隔离（Hyper-V 隔离）。

### <a name="when-to-use-lcow"></a>何时使用 LCOW

目前，我们建议有以下需求的人员使用 LCOW：

- 希望测试我们的最新技术。
- 同时运行 Windows 和 Linux 容器。
- 在 Linux 容器之间需要内核隔离（Hyper-V 隔离）。

## <a name="other-options-we-considered"></a>我们考虑的其他选项

当研究在 Windows 上运行 Linux 容器的方法时，我们曾考虑过 WSL。 最终，我们选择了基于虚拟化的方法，以便 Windows 上的 Linux 容器与 Linux 上的 Linux 容器一致。 使用 Hyper-V 还会使 LCOW 更安全。 将来我们可能会重新评估，但现在，LCOW 将继续使用 Hyper-V。

如果你有想法，请通过 GitHub 或 UserVoice 发送反馈。  我们非常期待你提供反馈，告诉我们你希望看到的具体体验。
