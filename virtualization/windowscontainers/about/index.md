---
title: 关于 Windows 容器
description: 容器是一种用于打包和运行应用的技术，包括 Windows 应用-在本地和云中的各种环境中。 本主题讨论 Microsoft、Windows 和 Azure 如何帮助你在容器中开发和部署应用，包括使用 Docker 和 Azure Kubernetes 服务。
keywords: docker, 容器
author: taylorb-microsoft
ms.date: 10/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8e273856-3620-4e58-9d1a-d1e06550448
ms.openlocfilehash: acce214cc8991f20c979b6dbe636590416841cb9
ms.sourcegitcommit: d0411b05d1ef7328a770766b84fd0743f9d9c237
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2019
ms.locfileid: "10254287"
---
# <a name="windows-and-containers"></a>Windows 和容器

容器是在本地和云中的各种环境中打包和运行 Windows 和 Linux 应用程序的技术。 容器提供一个轻型的隔离环境，使应用更易于开发、部署和管理。 容器快速启动和停止，使它们非常适合需要快速适应不断变化的需求的应用。 容器的轻量级特性还使它们成为提高基础结构的密度和利用率的有用工具。

![图形显示容器可以在云中或内部部署中运行的方式，支持以几乎任何语言编写的单片应用或 microservices。](media/about-3-box.png)

## <a name="the-microsoft-container-ecosystem"></a>Microsoft 容器生态系统

Microsoft 提供了许多工具和平台，可帮助你在容器中开发和部署应用：

- <strong>在 windows 10 上运行基于 windows 或基于 Linux 的容器</strong>，以便使用[Docker 桌面](https://store.docker.com/editions/community/docker-ce-desktop-windows)进行开发和测试，从而使用内置于 Windows 的容器功能。 你还可以[在 Windows Server 上本机运行容器](../quick-start/set-up-environment.md?tabs=Windows-Server)。
- 使用 Visual Studio 和[Visual Studio 代码](https://code.visualstudio.com/docs/azure/docker)[中强大的容器支持](https://docs.microsoft.com/visualstudio/containers/overview)<strong>开发、测试、发布和部署基于 Windows 的容器</strong>，其中包括对 docker、docker 撰写、Kubernetes、Helm 和其他有用的支持科技.
- 将<strong>你的应用作为容器映像发布</strong>到公共 DockerHub 供其他人使用，或发布到专用[Azure 容器注册表](https://azure.microsoft.com/services/container-registry/)以供你组织自己的开发和部署，直接从 Visual Studio 和 visual studio 代码内推送和提取.
- <strong>在 Azure 或其他云上以缩放方式部署容器</strong>：

  - 从容器注册表（如 Azure 容器注册表）中提取你的应用（容器映像），然后使用 orchestrator （如[Azure Kubernetes Service （AKS）](https://docs.microsoft.com/azure/aks/intro-kubernetes) （在基于 Windows 的应用）或 Azure 服务的应用程序按比例部署和管理你的应用[结构](https://docs.microsoft.com/azure/service-fabric/)。
  - Azure Kubernetes 服务将容器部署到 Azure 虚拟机并按比例进行管理，无论是数十个容器、上百甚至上千个容器。 Azure 虚拟机运行自定义的 Windows Server 映像（如果你部署基于 Windows 的应用）或自定义的 Ubuntu Linux 映像（如果你要部署基于 Linux 的应用）。
- 通过[将 Azure 堆栈与 AKS 引擎](https://docs.microsoft.com/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)（在具有 Linux 容器的预览版中）或使用[OpenShift 的 azure 堆栈](https://docs.microsoft.com/azure/virtual-machines/linux/openshift-azure-stack)一起使用，<strong>在本地部署容器</strong>。 你还可以在 Windows Server 上设置 Kubernetes （请参阅[windows 上的 Kubernetes](../kubernetes/getting-started-kubernetes-windows.md)），并且我们还在[RedHat OpenShift 容器平台](https://techcommunity.microsoft.com/t5/Networking-Blog/Managing-Windows-containers-with-Red-Hat-OpenShift-Container/ba-p/339821)上处理运行 Windows 容器的支持。

## <a name="how-containers-work"></a>容器的工作原理

容器是一个独立的轻型接收器，用于在主机操作系统上运行应用程序。 在主机操作系统的内核（可视为操作系统的掩蔽管道）的顶部构建容器，如下图所示。

![显示如何在内核顶部运行容器的体系结构图表](media/container-diagram.svg)

当容器共享主机操作系统的内核时，容器不会获取对该操作系统的 unfettered 访问权限。 相反，容器将获取隔离的，在某些情况下，虚拟化-系统的视图。 例如，容器可以访问文件系统和注册表的虚拟化版本，但任何更改仅影响容器，在停止时被放弃。 为了保存数据，容器可以装入永久性存储，如[Azure 磁盘](https://azure.microsoft.com/services/storage/disks/)或文件共享（包括[azure 文件](https://azure.microsoft.com/services/storage/files/)）。

容器在内核顶部构建，但内核不提供应用需要运行的所有 Api 和服务-大多数都由在用户模式下运行于内核的系统文件（库）提供。 由于容器独立于主机的用户模式环境，因此容器需要自己的这些用户模式系统文件的副本，这些文件打包为基本映像。 基本映像充当你的容器的基础层，它通过内核未提供的操作系统服务提供。 但稍后我们将详细讨论容器映像。

## <a name="containers-vs-virtual-machines"></a>容器与虚拟机

相对于容器，虚拟机（Vm）运行完整的操作系统（包括它自己的内核），如下图所示。

![展示 Vm 在主机操作系统旁边运行完整操作系统的体系结构图](media/virtual-machine-diagram.svg)

容器和虚拟机每个容器和虚拟机都具有各自的用途-事实上，许多容器部署将虚拟机用作主机操作系统，而不是直接在硬件上运行，尤其是在云中运行容器时。

有关这些补充技术的相似性和差异的更多详细信息，请参阅[容器与虚拟机](containers-vs-vm.md)。

## <a name="container-images"></a>容器图像

从容器图像创建所有容器。 容器图像是组织为驻留在本地计算机或远程容器注册表中的图层堆栈的文件包。 容器映像包含支持你的应用、应用、应用的任何运行时或依赖项的用户模式操作系统文件，以及你的应用需要正确运行的任何其他杂项配置文件。

Microsoft 提供了多个图像（称为基本图像），可用作构建自己的容器图像的起始点：

* <strong>Windows</strong> -包含完整的 windows api 和系统服务集（负服务器角色）。
* <strong>Windows Server Core</strong> -一个较小的图像，其中包含 Windows Server api 的子集，即完整的 .net framework。 它还包括大多数服务器角色，但很遗憾，并非传真服务器。
* <strong>Nano Server</strong> -最小的 Windows Server 映像，支持 .Net Core api 和某些服务器角色。
* <strong>Windows 10 IoT 核心</strong>版-适用于运行 ARM 或 x86/x64 处理器的设备的小型互联网的硬件制造商使用的 windows 版本。

正如前面所述，容器图像由一系列图层组成。 每个层都包含一组文件，这些文件重叠在一起，表示你的容器图像。 由于容器的分层特性，因此无需始终针对基映像来构建 Windows 容器。 相反，你可以指向已携带所需框架的另一个图像。 例如，.NET 团队发布了一个[.net core 映像](https://hub.docker.com/_/microsoft-dotnet-core)，它携带 .net core runtime。 它可使用户不必复制 .NET core 的安装过程-而是可以重复使用此容器图像的图层。 .NET core 映像本身基于 Nano Server 生成。

有关更多详细信息，请参阅[容器基础图像](../manage-containers/container-base-images.md)。

## <a name="container-users"></a>容器用户

### <a name="containers-for-developers"></a>面向开发人员的容器

容器可帮助开发人员更快地构建和交付更高质量的应用程序。 通过容器，开发人员可以创建在数秒内（在不同环境中相同）部署的容器映像。 容器充当跨团队共享代码和引导开发环境而不影响你的主机文件系统的简便机制。

容器是可移植的，可运行以任何语言编写的应用，它们与运行 Windows 10 版本1607或更高版本的任何计算机兼容，或者与运行 windows 2016 或更高版本的任何计算机兼容。 开发人员可以在其笔记本或桌面本地创建和测试容器，然后将该容器映像部署到其公司的专用云、公共云或服务提供商。 容器的自然灵活性支持大规模、虚拟化云环境中的新式应用开发模式。

### <a name="containers-for-it-professionals"></a>面向 IT 专业人员的容器

容器可帮助管理员创建更易于更新和维护的基础结构，并可更充分地利用硬件资源。 IT 专业人员可以使用容器为其开发、QA 和生产团队提供标准化的环境。 通过使用容器，系统管理员会在操作系统安装和底层基础结构中失去差异。

## <a name="container-orchestration"></a>容器业务流程

在设置基于容器的环境时，Orchestrators 是基础结构的关键部分。 虽然你可以使用 Docker 和 Windows 手动管理几个容器，但应用通常使用五个、十个甚至上百个容器，在这里 orchestrators。

容器 orchestrators 的构建旨在帮助在规模和生产环境中管理容器。 Orchestrators 提供以下功能：

- 按比例部署
- 工作负荷计划
- 运行状况监视
- 节点出现故障时进行故障转移
- 向上或向下缩放
- 网络
- 服务发现
- 协调应用程序升级
- 群集节点亲近性

有许多不同的 orchestrators 可用于 Windows 容器;以下是 Microsoft 提供的选项：
- [Azure Kubernetes 服务（AKS）](https://docs.microsoft.com/azure/aks/intro-kubernetes) -使用托管的 Azure Kubernetes 服务
- [Azure 服务结构](https://docs.microsoft.com/azure/service-fabric/)-使用托管服务
- [具有 AKS 引擎的 Azure 堆栈](https://docs.microsoft.com/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)-使用 Azure Kubernetes 服务本地
- [Windows 上的 Kubernetes](../kubernetes/getting-started-kubernetes-windows.md) -在 windows 上设置 Kubernetes

## <a name="try-containers-on-windows"></a>在 Windows 上试用容器

若要开始使用 Windows Server 或 Windows 10 上的容器，请参阅以下内容：
> [!div class="nextstepaction"]
> [入门：为容器配置你的环境](../quick-start/set-up-environment.md)

有关确定哪些 Azure 服务适合你的方案的帮助，请参阅[Azure 容器服务](https://azure.microsoft.com/product-categories/containers/)和[选择用于托管你的应用程序的 azure 服务](https://docs.microsoft.com/azure/architecture/guide/technology-choices/compute-decision-tree)。
