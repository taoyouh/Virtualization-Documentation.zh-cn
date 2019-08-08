---
title: 关于 Windows 容器
description: 了解 Windows 容器。
keywords: docker, 容器
author: taylorb-microsoft
ms.date: 05/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8e273856-3620-4e58-9d1a-d1e06550448
ms.openlocfilehash: 1a66ef0fd07162f8bcd78b9bffa159d3f96e4763
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998794"
---
# <a name="about-windows-containers"></a>关于 Windows 容器

这就好像是一个厨房。 在此单个房间内, 你需要能够提供食物的所有功能: oven、pan、水池等。 这是我们的容器。

![黑色框内带有黄色墙纸的完全配备的厨房的插图。](media/box1.png)

现在, 请想象将此厨房放在大楼内, 就像将书籍滑入 bookshelf 一样轻松。 由于厨房需要运行的所有内容都已存在, 因此我们只需连接电能和管道即可。

![由两叠黑色框组成的单元建筑物。 这四个框中的四个框与厨房示例中使用的黄色框相同, 并且在整个建筑物中均位于随机位置, 而其余部分是彩色活房间或为空并灰显。](media/apartment.png)

为什么停止？ 你可以按自己喜欢的方式自定义你的构建;使用很多种类的房间填充它, 使用相同的房间填充它或组合两个房间。

通过以我们在厨房中的方式运行应用来运行容器, 容器的作用类似于此聊天室。 容器将应用以及应用需要运行的所有内容放入其自己的独立的框中。 因此, 独立应用不知道任何其他存在于其容器之外的应用或进程。 由于容器具有应用需要运行的所有内容, 因此容器可以在任何位置移动, 仅使用资源的主机置备, 而不触及为其他容器预配的任何资源。

以下视频将告诉你有关 Windows 容器可以为你执行哪些操作的详细信息, 以及 Microsoft 与 Docker 的合作关系如何帮助为开放源容器开发创建 frictionless 环境:

<iframe width="800" height="450" src="https://www.youtube.com/embed/Ryx3o0rD5lY" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## <a name="container-fundamentals"></a>容器基础知识

在开始使用 Windows 容器时, 我们将了解你将发现的一些术语:

- 容器主机: 使用 Windows 容器功能配置的物理或虚拟计算机系统。 容器主机将运行一个或多个 Windows 容器。
- 沙盒: 捕获在运行时对容器所做的所有更改 (如文件系统修改、注册表修改或软件安装) 的层。
- 基本图像: 提供容器操作系统环境的容器的图像图层中的第一图层。 无法修改基本图像。
- 容器图像: 创建容器的说明的只读模板。 图像可以基于基本的、不改变的操作系统环境, 但也可以从已修改的容器的沙箱创建。 这些修改后的图像在基本图像层顶部分层更改, 并且可以将这些图层复制并重新应用到其他基础图像, 以使用相同的更改创建新图像。
- 容器存储库: 每次创建新图像时存储容器图像及其依赖项的本地存储库。 你可以在容器主机上多次重复使用存储的图像。 你还可以将容器映像存储在公共或专用注册表 (如 Docker 集线器) 中, 以便它们可以跨许多不同的容器主机使用。
- 容器 orchestrator: 自动化和管理大量容器以及它们如何相互交互的过程。 若要了解详细信息, 请参阅[关于 Windows 容器 orchestrators](overview-container-orchestrators.md)。
- Docker: 打包和传递容器图像的自动化过程。 若要了解详细信息, 请参阅[docker 概述](docker-overview.md)、 [Windows 上的 docker 引擎](../manage-docker/configure-docker-daemon.md)或访问[Docker 网站](https://www.docker.com)。

![显示如何创建容器的流程图。 应用程序和基本图像用于创建沙盒和新的应用程序映像, 它们在基本映像的顶部分层以构建新容器。](media/containerfund.png)

熟悉虚拟机的人可能会认为容器和虚拟机看起来类似。 容器运行操作系统, 具有文件系统, 并且可以通过网络访问, 与物理或虚拟计算机系统非常相似。 话虽如此，但容器背后的技术和概念与虚拟机有很大不同。 若要了解有关这些概念的详细信息, 请阅读标记 Russinovich 的[博客文章](https://azure.microsoft.com/blog/containers-docker-windows-and-trends/), 其中详细说明了差异。

### <a name="windows-container-types"></a>Windows 容器类型

你应该知道, 有两个不同的容器类型 (也称为运行时)。

Windows Server 容器通过进程和命名空间隔离技术提供应用程序隔离, 这就是这些容器也称为进程隔离的容器的原因。 Windows Server 容器与容器主机和该主机上运行的所有容器共享内核。 这些进程隔离的容器不提供敌意安全边界, 不应用于隔离不受信任的代码。 由于共享内核空间，这些容器要求具有相同的内核版本和配置。

Hyper-v 隔离通过在高度优化的虚拟机中运行每个容器来扩展 Windows Server 容器提供的隔离。 在此配置中, 容器主机不与同一主机上的其他容器共享其内核。 这些容器旨在托管敌对多租户，并且具有与虚拟机相同的安全保证。 由于这些容器不与主机上的主机或其他容器共享内核, 因此它们可以运行具有不同版本和配置 (受支持版本内) 的内核。 例如, Windows 10 上的所有 Windows 容器都使用 Hyper-v 隔离来利用 Windows Server 内核版本和配置。

使用或不使用 Hyper-v 隔离在 Windows 上运行容器是运行时决策。 你最初可以创建具有 Hyper-v 隔离的容器, 稍后在运行时选择将其作为 Windows Server 容器运行。

## <a name="container-users"></a>容器用户

### <a name="containers-for-developers"></a>面向开发人员的容器

容器可帮助开发人员更快地构建和交付更高质量的应用程序。 开发人员可以创建一个放大图像, 该图像将在几秒钟内完全部署到所有环境中。 在 Docker 容器中打包的应用程序有大量且发展壮大。 DockerHub 是由 Docker 维护的公共应用程序注册表, 它在其公共社区存储库中发布了超过180000的应用程序, 该号码仍在增加。

当开发人员 containerizes 应用时, 仅将其需要运行的应用和组件合并到一个图像中。 然后根据你的需要从此映像创建容器。 你还可以使用映像作为创建其他映像的基线，从而使映像创建速度更快。 多个容器可以共享同一个图像, 这意味着容器启动速度非常快, 并且使用的资源较少。 例如, 开发人员可以使用容器为分布式应用程序旋转轻量级和便携应用组件 (也称为 microservices), 并单独对每个服务进行快速缩放。

容器是可移植和通用的, 可采用任何语言编写, 并且与运行 Windows Server 2016 的任何计算机兼容。 开发人员可以在其笔记本或桌面本地创建和测试容器, 然后将该容器映像部署到其公司的专用云、公共云或服务提供商。 容器的自然灵活性支持大规模、虚拟化云环境中的新式应用开发模式。

### <a name="containers-for-it-professionals"></a>面向 IT 专业人员的容器

容器可帮助管理员创建更易于更新和维护的基础结构。 IT 专业人员可以使用容器为其开发、QA 和生产团队提供标准化的环境。 他们不再需要担心复杂的安装和配置过程。 通过使用容器, 系统管理员会在操作系统安装和底层基础结构中失去差异。

## <a name="containers-101-video-presentation"></a>容器101视频演示文稿

下面的视频演示将为你提供有关 Windows 容器的历史记录和实现的更深入概述。

<iframe src="https://channel9.msdn.com/Blogs/containers/Containers-101-with-Microsoft-and-Docker/player" width="800" height="450" allowFullScreen="true" frameBorder="0" scrolling="no"></iframe>

## <a name="try-windows-server-containers"></a>尝试 Windows Server 容器

已准备好开始利用容器的强大功能？ 以下文章将帮助您入门:

若要在 Windows Server 上设置容器, 请参阅[Windows server 快速入门](../quick-start/quick-start-windows-server.md)。

若要在 Windows 10 上设置容器, 请参阅[Windows 10 快速入门](../quick-start/quick-start-windows-10.md)。