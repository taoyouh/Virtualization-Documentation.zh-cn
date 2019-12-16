---
title: 关于 Windows 容器
description: 容器是一种跨本地和云中的不同环境打包和运行应用（包括 Windows 应用）的技术。 本主题讨论 Microsoft、Windows 和 Azure 如何帮助你通过各种方式（包括使用 Docker 和 Azure Kubernetes 服务）在容器中开发和部署应用。
keywords: docker, 容器
author: taylorb-microsoft
ms.date: 10/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8e273856-3620-4e58-9d1a-d1e06550448
ms.openlocfilehash: 4fad299db2c897a6be860ef0cc71e80969c75357
ms.sourcegitcommit: 8dedb887b038fbff872327f51c7416454b301b86
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/06/2019
ms.locfileid: "74909407"
---
# <a name="windows-and-containers"></a>Windows 和容器

容器是一种跨本地和云中的不同环境打包和运行 Windows 和 Linux 应用程序的技术。 容器提供一个轻型隔离环境，使应用更易于开发、部署和管理。 容器可以快速启动和停止，因此适用于需要快速适应不断变化的需求的应用。 容器的轻型性质也使得它们成为一种有用的工具，可以提高基础结构的密度和利用率。

![示意图，显示容器可以在云中或本地运行，支持以几乎任何语言编写的整体式应用或微服务。](media/about-3-box.png)

## <a name="the-microsoft-container-ecosystem"></a>Microsoft 容器生态系统

Microsoft 提供了许多有助于在容器中开发和部署应用的工具和平台：

- <strong>在 Windows 10 上运行基于 Windows 或基于 Linux 的容器</strong>，以便通过 [Docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows) 使用内置到 Windows 中的容器功能进行开发和测试。 也可[在 Windows Server 上以本机方式运行容器](../quick-start/set-up-environment.md?tabs=Windows-Server)。
- 使用 [Visual Studio 中的强大容器支持](https://docs.microsoft.com/visualstudio/containers/overview)和 [Visual Studio Code](https://code.visualstudio.com/docs/azure/docker)（包括对 Docker、Docker Compose、Kubernetes、Helm 和其他有用技术的支持）<strong>开发、测试、发布和部署基于 Windows 的容器</strong>。
- <strong>将应用作为容器映像发布</strong>到公共 DockerHub 供他人使用，或者发布到专用 [Azure 容器注册表](https://azure.microsoft.com/services/container-registry/)供组织进行自己的开发和部署，直接在 Visual Studio 和 Visual Studio Code 中进行推送和拉取。
- <strong>在 Azure 上大规模部署容器</strong>，或者在其他云上这样做：

  - 从容器注册表（例如 Azure 容器注册表）拉取应用（容器映像），然后使用 [Azure Kubernetes 服务 (AKS)](https://docs.microsoft.com/azure/aks/intro-kubernetes)（为基于 Windows 的应用提供预览版）或 [Azure Service Fabric](https://docs.microsoft.com/azure/service-fabric/) 之类的业务流程协调程序对其进行大规模部署和管理。
  - Azure Kubernetes 服务将容器大规模部署到 Azure 虚拟机并对其进行管理，不管容器的数量是数十、数百还是数千。 Azure 虚拟机运行自定义 Windows Server 映像（如果部署基于 Windows 的应用）或自定义 Ubuntu Linux 映像（如果部署基于 Linux 的应用）。
- 通过[将 Azure Stack 与 AKS 引擎配合使用](https://docs.microsoft.com/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)（以预览版方式与 Linux 容器配合使用）或[将 Azure Stack 与 OpenShift 配合使用](https://docs.microsoft.com/azure/virtual-machines/linux/openshift-azure-stack)，<strong>在本地部署容器</strong>。 你也可以在 Windows Server 上自行设置 Kubernetes（请参阅 [Windows 上的 Kubernetes](../kubernetes/getting-started-kubernetes-windows.md)），我们也将致力于提供相关支持，方便你运行 [RedHat OpenShift Container Platform 上的 Windows 容器](https://techcommunity.microsoft.com/t5/Networking-Blog/Managing-Windows-containers-with-Red-Hat-OpenShift-Container/ba-p/339821)。

## <a name="how-containers-work"></a>容器工作原理

容器是一个隔离的轻型接收器，用于在主机操作系统上运行应用程序。 容器在主机操作系统的内核（可以将其视为操作系统的隐藏管道）上构建，如下图所示。

![体系结构图，显示容器如何在内核上运行](media/container-diagram.svg)

容器在共享主机操作系统的内核时，并不能对其进行自由访问， 而只能获取系统的隔离视图，在某些情况下只能获取其虚拟化视图。 例如，容器可以访问文件系统和注册表的虚拟化版本，但任何更改只影响容器，在容器停止后会被丢弃。 若要保存数据，容器可以装载持久性存储，例如 [Azure 磁盘](https://azure.microsoft.com/services/storage/disks/)或文件共享（包括 [Azure 文件存储](https://azure.microsoft.com/services/storage/files/)）。

容器在内核上构建，但内核并不提供应用运行所需的所有 API 和服务 - 大多数 API 和服务由内核之上以用户模式运行的系统文件（库）提供。 容器与主机的用户模式环境隔离，因此需要其自己的用户模式系统文件副本，这些文件在打包后称为基础映像。 基础映像充当构建容器时所在的基础层，为容器提供内核不提供的操作系统服务。 不过，有关容器映像的详细内容，我们会在后面讨论。

## <a name="containers-vs-virtual-machines"></a>容器与虚拟机

与容器不同，虚拟机 (VM) 运行的是完整的操作系统（包括其自己的内核），如下图所示。

![体系结构图，显示 VM 如何运行完整的操作系统（独立于主机操作系统）](media/virtual-machine-diagram.svg)

容器和虚拟机都有自己的用途 - 事实上，许多容器部署使用虚拟机作为主机操作系统，而不是直接运行在设备上，在云中运行容器时尤其如此。

若要更详细地了解这些互补技术的异同，请参阅[容器与虚拟机](containers-vs-vm.md)。

## <a name="container-images"></a>容器映像

所有容器都从容器映像创建。 容器映像是文件的捆绑包，这些文件组织成一个由层组成的堆栈，而这些层驻留在本地计算机或远程容器注册表中。 容器映像包含为应用提供支持所需的用户模式操作系统文件、应用、应用的任何运行时或依赖项，以及应用正常运行所需的任何其他杂项配置文件。

Microsoft 提供多个映像（称为基础映像），你可以从其着手构建自己的容器映像：

* <strong>Windows</strong> - 包含整套 Windows API 和系统服务（服务器角色除外）。
* <strong>Windows Server Core</strong> - 一个较小的映像，包含部分 Windows Server API - 即完整的 .NET Framework。 它还包括大部分服务器角色，但遗憾的是数目太少，不包括传真服务器。
* <strong>Nano Server</strong> - 最小的 Windows Server 映像，支持 .NET Core API 和某些服务器角色。
* <strong>Windows 10 IoT 核心版</strong> - 一个 Windows 版本，由硬件制造商用于小的运行 ARM 或 x86/x64 处理器的物联网设备。

如前所述，容器映像由一系列层组成。 每个层包含一组文件，这些文件重叠在一起时表示容器映像。 由于容器的分层特性，你不需始终以某个基础映像为目标来构建 Windows 容器， 而是可以以另一个已经携带你所需框架的映像为目标。 例如，.NET 团队发布了一个携带 .NET Core 运行时的 [.NET Core 映像](https://hub.docker.com/_/microsoft-dotnet-core)。 有了它，用户就不需重复 .NET Core 安装过程，只需重复使用该容器映像的层即可。 .NET Core 映像本身在 Nano Server 基础上构建。

如需更多详细信息，请参阅[容器基础映像](../manage-containers/container-base-images.md)。

## <a name="container-users"></a>容器用户

### <a name="containers-for-developers"></a>面向开发人员的容器

容器有助于开发人员更快地生成和交付更高质量的应用。 开发人员可以使用容器创建一个在数秒内以相同方式跨环境部署的容器映像。 若要跨团队共享代码，以及在不影响主机文件系统的情况下启动开发环境，不妨使用容器这种可以轻松掌握的机制。

容器可移植且通用，可以运行以任何语言编写的应用，并且兼容任何运行 Windows 10 版本 1607 或更高版本或者 Windows Server 2016 或更高版本的计算机。 开发人员可以在便携式计算机或台式机上通过本地方式创建并测试一个容器，然后将该容器映像部署到其公司的私有云、公有云或服务提供商。 容器的自然敏捷性支持大规模虚拟化云环境中的现代应用开发模式。

### <a name="containers-for-it-professionals"></a>面向 IT 专业人员的容器

容器有助于管理员创建更易于更新和维护且更能充分利用硬件资源的基础结构。 IT 专业人员可以使用容器为其开发、QA 和生产团队提供标准化环境。 通过使用容器，系统管理员抽象出操作系统安装和底层基础结构中的差异。

## <a name="container-orchestration"></a>容器业务流程

设置基于容器的环境时，业务流程协调程序是基础结构的关键部分。 虽然你可以使用 Docker 和 Windows 手动管理数个容器，但应用通常使用五个、十个甚至数百个容器，这种情况下需要业务流程协调程序。

构建容器业务流程协调程序是为了方便大规模地在生产环境中管理容器。 业务流程协调程序提供的功能适用于：

- 大规模部署
- 工作负荷计划
- 运行状况监视
- 在节点故障时进行故障转移
- 纵向扩展或缩减
- 网络
- 服务发现
- 协调应用升级
- 群集节点相关性

可以将许多不同的业务流程协调程序与 Windows 容器配合使用；下面是 Microsoft 提供的选项：
- [Azure Kubernetes 服务 (AKS)](https://docs.microsoft.com/azure/aks/intro-kubernetes) - 使用托管的 Azure Kubernetes 服务
- [Azure Service Fabric](https://docs.microsoft.com/azure/service-fabric/) - 使用托管服务
- [将 Azure Stack 与 AKS 引擎配合使用](https://docs.microsoft.com/azure-stack/user/azure-stack-kubernetes-aks-engine-overview) - 在本地使用 Azure Kubernetes 服务
- [Windows 上的 Kubernetes](../kubernetes/getting-started-kubernetes-windows.md) - 在 Windows 上自行设置 Kubernetes

## <a name="try-containers-on-windows"></a>尝试 Windows 上的容器

有关 Windows Server 或 Windows 10 上的容器的入门，请参阅以下文档：
> [!div class="nextstepaction"]
> [入门：为容器配置环境](../quick-start/set-up-environment.md)

如果需要帮助才能确定哪些 Azure 服务适合自己的方案，请参阅 [Azure 容器服务](https://azure.microsoft.com/product-categories/containers/)和[选择使用哪些 Azure 服务来托管应用程序](https://docs.microsoft.com/azure/architecture/guide/technology-choices/compute-decision-tree)。
