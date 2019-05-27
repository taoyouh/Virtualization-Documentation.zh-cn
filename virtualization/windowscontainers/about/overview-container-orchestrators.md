---
title: 关于 Windows 容器 orchestrators
description: 了解有关 Windows 容器 orchestrators 的信息。
keywords: docker, 容器
author: Heidilohr
ms.author: helohr
ms.date: 05/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 1ccf63b0ae55501ba32f8bdd61994e7f8006b5e6
ms.sourcegitcommit: daf1d2b5879c382404fc4d59f1c35c88650e20f7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/23/2019
ms.locfileid: "9674873"
---
# <a name="about-windows-container-orchestrators"></a>关于 Windows 容器 orchestrators

由于容器的大小和应用规模较小, 因此容器非常适合于敏捷交付环境和基于 microservice 的体系结构。 但是, 使用容器和 microservers 的环境可以有成百上千个组件可供跟踪。 你可能能够手动管理几个虚拟机或物理服务器, 但无法在不进行自动化的情况下正确管理生产规模的容器环境。 此任务应属于你的 orchestrator, 后者是自动化和管理大量容器以及它们如何相互交互的过程。

Orchestrators 执行以下任务:

- 计划: 当给定容器图像和资源请求时, orchestrator 将找到运行容器的合适计算机。
- 相关性/反关联: 指定一组容器是否应在更远的位置上为性能或间距, 以实现可用性。
- 运行状况监控：监视容器故障并自动重新调度。
- 故障转移: 跟踪每台计算机上运行的内容, 并将容器从失败的计算机重新安排到运行状况良好的节点。
- 缩放: 添加或删除容器实例, 以匹配需求, 手动或自动。
- 网络: 提供覆盖跨多台主机的容器进行通信的覆盖网络。
- 服务发现：让容器即使在主机之间迁移且 IP 地址已改变时，也能实现相互间的自动定位。
- 协调的应用程序升级：管理容器升级，避免应用程序关闭，并在出现错误时启用回滚。

## <a name="orchestrator-types"></a>Orchestrator 类型

Azure 提供两个容器 orchestrators: Azure Kubernetes 服务 (AKS) 和 Service Fabric。

[Azure Kubernetes 服务 (AKS)](/azure/aks/)使创建、配置和管理已预配置为运行容器的应用程序的虚拟机群集非常简单。 这使你能够使用现有的技能, 并根据社区专业知识的大型和成长的身体, 在 Microsoft Azure 上部署和管理基于容器的应用程序。 通过使用 AKS, 你可以利用 Azure 的企业级功能, 但仍可以通过 Kubernetes 和 Docker 图像格式保持应用程序可移植性。

[Azure Service Fabric](/azure/service-fabric/) 是一种分布式系统平台，可用于轻松打包、部署和管理可伸缩的可靠微服务和容器。 Service Fabric 解决了开发和管理云本机应用程序的重大挑战。 开发人员和管理员不仅可以避免复杂的基础结构问题，而且可以专注于实现可伸缩、可靠、可管理的高要求任务关键型工作负荷。 Service Fabric 新一代平台的代表，用于生成和管理在容器中运行的企业级单层云级别应用程序。

## <a name="getting-started"></a>即刻体验

若要开始部署 Azure Kubernetes 服务, 请参阅[Kubernetes 安装指南](../kubernetes/getting-started-kubernetes-windows.md)。

若要开始部署 Azure 服务结构, 请参阅[Service Fabric 快速入门](/azure/service-fabric/service-fabric-quickstart-containers.md)。