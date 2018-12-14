---
title: 关于 Windows 容器
description: 了解 Windows 容器。
keywords: docker, 容器
author: taylorb-microsoft
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8e273856-3620-4e58-9d1a-d1e06550448
ms.openlocfilehash: 5dcc2af869ba0433b1d755f6d70f5dae6e18cdc5
ms.sourcegitcommit: 4090d158dd3573ea90799de5b014c131a206b000
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/07/2018
ms.locfileid: "6121607"
---
# <a name="windows-containers"></a>Windows 容器

## <a name="what-are-containers"></a>什么是容器

容器是一种将应用程序包装到其自身隔离空间内的方法。 位于容器中的应用不了解该容器外存在的所有其他应用程序或进程。 应用程序成功运行所需的所有依赖项也存在于此容器内。  无论容器移动到何处，应用程序都将始终得到满足，因为应用程序与其运行所需的一切都已绑定在一起。

这就好像是一个厨房。 我们打包所有的电器和家具、锅碗瓢盆、洗洁精和毛巾。 这便是我们的容器

<center style="margin: 25px">![](media/box1.png)</center>

我们现在可以带着这个容器，将它放在任何喜欢的入住公寓中，厨房还会是这个厨房。 我们所需做的全部工作就是为它接通水电，然后我们便可以立即开始烹饪（因为我们拥有所有需要的器具！）

<center style="margin: 25px">![](media/apartment.png)</center>

容器在很大程度上就像是这个厨房。 可以有不同类型的房间以及许多相同类型的房间。 重要的一点是容器与其所需的一切内容打包在一起。

在此处观看简短概述：[基于 Windows 的容器：使用企业级控制的现代应用开发](https://youtu.be/Ryx3o0rD5lY)。

## <a name="container-fundamentals"></a>容器基础知识

容器是独立的、资源受控制的和可移植的运行时环境，在主机或虚拟机上运行。 在容器中运行的应用程序或进程与所有需要的依赖项和配置文件打包在一起；在它看来，容器之外似乎没有任何其他进程在运行。

容器的主机为容器预配一组资源，且容器只会使用这些资源。 尽可能容器知道，之外什么它已被授予存在任何其他资源，并因此容器不能接触可能已预配为相邻容器的资源。

在你开始创建和使用 Windows 容器时，以下关键概念将会很有用。

**容器主机：** 使用 Windows 容器功能配置的物理或虚拟计算机系统。 容器主机将运行一个或多个 Windows 容器。

**容器映像：** 在对容器文件系统或注册表进行修改时（如软件安装），将在沙盒中捕获这些修改。 在许多情况下，你可能希望捕获此状态，以便可以创建继承这些更改的新容器。 这就是映像的本质：一旦容器停止，你便可以放弃该沙盒，或者可以将其转换为新的容器映像。 例如，让我们想象你已从 Windows Server Core 操作系统映像部署一个容器。 然后你将 MySQL 安装到此容器中。 从此容器创建新映像将充当该容器的可部署版本。 此映像将只包含所做的更改 (MySQL)，但是将充当容器操作系统映像之上的一个层。

**沙盒：** 容器启动后，将在此“沙盒”层中捕获所有的写入操作，如文件系统修改、注册表修改或软件安装。

**容器操作系统映像：** 从映像部署容器。 容器操作系统映像是可能组成容器的许多映像层中的第一层。 此映像提供操作系统环境。 容器操作系统映像是不可变的。 也就是说，不能对其进行修改。

**容器存储库：** 每次创建容器映像时，容器映像及其依赖项都会存储在本地存储库中。 这些映像可以在容器主机上重复使用多次。 容器映像还可以存储在公共或私有注册表（如 DockerHub）中，以便可以在许多不同的容器主机上使用它们。

<center>![](media/containerfund.png)</center>

对于熟悉虚拟机的人员而言，容器可能具有令人难以置信的相似性。 容器在操作系统上运行、具有文件系统，并且可以通过网络访问，就像它是物理或虚拟计算机系统一样。 话虽如此，但容器背后的技术和概念与虚拟机有很大不同。

Microsoft Azure 专家 Mark Russinovich 有[一篇精彩的博客文章](https://azure.microsoft.com/en-us/blog/containers-docker-windows-and-trends/)详述了这些差异。

## <a name="windows-container-types"></a>Windows 容器类型

Windows 容器包括两个不同的容器类型或运行时。

**Windows Server 容器** - 通过进程和命名空间隔离技术提供应用程序隔离。 Windows Server 容器与容器主机和该主机上运行的所有容器共享内核。 这些容器不提供敌对安全边界，不应该用于隔离不受信任的代码。 由于共享内核空间，这些容器要求具有相同的内核版本和配置。

**Hyper-V 隔离** - 通过在高度优化的虚拟机中运行每个容器，在由 Windows Server 容器提供的隔离上扩展。 在此配置中，容器主机的内核不与相同主机上的其他容器共享。 这些容器旨在托管敌对多租户，并且具有与虚拟机相同的安全保证。 由于这些容器与主机或主机上的其他容器不共享内核，它们可运行（与受支持的版本）采用不同版本和配置的内核 - 例如 Windows 10 上的所有 Windows 容器都使用 Hyper-V 隔离以充分利用 Windows Server 内核版本和配置。

在 Windows 上运行容器时是否使用 Hyper-V 隔离将在运行时决定。 你可以在最初选择创建具有 Hyper-V 隔离的容器，而稍后在运行时选择将其作为 Windows Server 容器运行。

## <a name="what-is-docker"></a>什么是 Docker？

阅读有关容器的资料时，你将不可避免地看到有关 Docker 的信息。 Docker 是对容器映像进行打包和传送的容器。 此自动化过程会生成稍后可能在任何地方（在本地、在云中或在个人计算机上）作为容器运行的映像（相当于模板）。

<center>![](media/docker.png)</center>

同任何其他容器一样，可以通过 [Docker](https://www.docker.com) 管理 Windows Server 容器。

## <a name="containers-for-developers"></a>面向开发人员的容器 ##

从开发人员的桌面到测试计算机再到一组生产计算机，可以创建以相同方式在几秒内在任何环境中部署的 Docker 映像。 由此创造出了封装在 Docker 容器中的巨大且持续增长的应用程序生态系统，其中 DockerHub 是 Docker 所维护的公共容器化应用程序注册表，当前已在公共社区存储库中发布超过 180,000 个应用程序。

当你容器化某个应用时，仅该应用以及运行该运用所需的组件将组合到“映像”中。 然后根据你的需要从此映像创建容器。 你还可以使用映像作为创建其他映像的基线，从而使映像创建速度更快。 多个容器可以共享同一个映像，这意味着容器将非常快速地启动，并使用更少的资源。 例如，你可以使用容器为已分配的应用起转轻型和可移植的应用组件（或“微服务”），并快速单独缩放每个服务。

由于容器具有运行应用程序所需的一切，因此它们非常易于移植，并且可在运行 Windows Server 2016 的任何计算机上运行。 你可以本地创建和测试容器，然后将该相同的容器映像部署到你的公司的私有云、公有云或服务提供商。 容器的自然灵活性支持大规模、虚拟化云环境中的现代应用开发模式。

借助容器，开发人员可以采用任何语言生成应用。 这些应用完全可移植，并且可在任何位置（笔记本电脑、台式机、服务器、私有云、公有云或服务提供商）运行，而无需任何代码更改。  

容器有助于开发人员更快地生成和交付更高质量的应用程序。

## <a name="containers-for-it-professionals"></a>面向 IT 专业人员的容器 ##

IT 专业人员可以使用容器来为其开发、QA 和生产团队提供标准化环境。 他们不再需要担心复杂的安装和配置步骤。 通过使用容器，系统管理员抽象出操作系统安装和底层基础结构中的差异。

容器有助于管理员创建更易于更新和维护的基础结构。

## <a name="container-orchestrators"></a>容器 Orchestrator
由于规模较小且面向应用程序，容器非常适合敏捷交付环境和基于微服务的体系结构。 借助容器和微服务，你可以在环境中轻松地部署成百上千个组件。  你可能能够手动管理几十台虚拟机或物理服务器，但却无法在非自动化的情况下管理生产级容器环境。  自动化和管理大量容器及其交互方式的任务称为协调。 

标准定义的协调包含以下任务：

- 调度：在给定容器映像和资源请求的情况下，找到适合运行该容器的计算机。
关联/反-关联：指定一组应该彼此靠近（以提高性能）或保持足够距离（以提高可用性）运行的容器。
- 运行状况监控：监视容器故障并自动重新调度。
- 故障转移：跟踪每台计算机上运行的组件，并将容器从有故障的计算机重新调度到正常运行的节点。
- 伸缩：根据需要手动或自动添加或删除容器实例。
- 联网：提供覆盖网络以协调容器，以便跨多台主机通信。
- 服务发现：让容器即使在主机之间迁移且 IP 地址已改变时，也能实现相互间的自动定位。
- 协调的应用程序升级：管理容器升级，避免应用程序关闭，并在出现错误时启用回滚。

Azure 提供两种容器协调器：Azure 容器服务 (AKS) 和 Service Fabric。

[Azure 容器服务 (AKS)](/azure/aks/) 有助于轻松创建、配置和管理预先配置为运行容器化应用程序的虚拟机群集。 这使你能够使用现有技能，或利用大量的社区专业知识，在 Microsoft Azure 上部署和管理基于容器的应用程序。 通过使用 AKS，你可以充分利用 Azure 的企业级功能，同时通过 Kubernetes 和 Docker 映像格式保持应用程序的可移植性。

[Azure Service Fabric](/azure/service-fabric/) 是一种分布式系统平台，可用于轻松打包、部署和管理可伸缩的可靠微服务和容器。 Service Fabric 解决了开发和管理云本机应用程序的重大挑战。 开发人员和管理员不仅可以避免复杂的基础结构问题，而且可以专注于实现可伸缩、可靠、可管理的高要求任务关键型工作负荷。 Service Fabric 新一代平台的代表，用于生成和管理在容器中运行的企业级单层云级别应用程序。

## <a name="video-overview"></a>视频概述

<iframe src="https://channel9.msdn.com/Blogs/containers/Containers-101-with-Microsoft-and-Docker/player" width="800" height="450" allowFullScreen="true" frameBorder="0" scrolling="no"></iframe>

## <a name="try-windows-server-containers"></a>试用 Windows Server 容器

已准备好开始利用容器的强大功能？ 请点击下方的链接亲自开始部署你的第一个容器： <br/>
对于 Windows Server 上的用户，请转到此处 - [Windows Server 快速入门简介](../quick-start/quick-start-windows-server.md) <br/>
对于 Windows 10 上的用户，请转到此处 - [Windows 10 快速入门简介](../quick-start/quick-start-windows-10.md)
