---
title: Windows 容器业务流程概述
description: 了解 Windows 容器协调器。
keywords: docker, 容器
author: Heidilohr
ms.author: helohr
ms.date: 05/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 23dd1e56ba68a679945779f5e7dbc15225412934
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/06/2020
ms.locfileid: "78853902"
---
# <a name="windows-container-orchestration-overview"></a>Windows 容器业务流程概述

由于容器的大小和应用程序方向较小，因此容器非常适合敏捷交付环境和基于微服务的体系结构。 然而，使用容器和微服务的环境可以有几百个或数千个要跟踪的组件。 你可能能够手动管理几个虚拟机或物理服务器，但无法在没有自动化的情况下正确管理生产规模容器环境。 此任务应属于 orchestrator，这是一个自动化和管理大量容器以及它们彼此交互的方式的过程。

协调器执行以下任务：

- 计划：当给定容器映像和资源请求时，orchestrator 将查找要在其上运行容器的合适计算机。
- 相关性/反相关性：指定一组容器是否应该彼此接近以提高性能或在不同的可用性上运行。
- 运行状况监控：监视容器故障并自动重新调度。
- 故障转移：跟踪每台计算机上运行的内容，并将容器从失败的计算机重新调度到正常的节点。
- 缩放：添加或删除容器实例以匹配需求，手动或自动。
- 网络：提供覆盖网络来协调容器，以便在多台主机之间进行通信。
- 服务发现：让容器即使在主机之间迁移且 IP 地址已改变时，也能实现相互间的自动定位。
- 协调的应用程序升级：管理容器升级，避免应用程序关闭，并在出现错误时启用回滚。

## <a name="orchestrator-types"></a>Orchestrator 类型

Azure 提供两个容器协调器： Azure Kubernetes 服务（AKS）和 Service Fabric。

使用[Azure Kubernetes 服务（AKS）](/azure/aks/) ，可以轻松地创建、配置和管理预配置的虚拟机群集，以运行容器化应用程序。 这使您可以使用您现有的技能，并起草大而不断增长的社区专业知识，在 Microsoft Azure 上部署和管理基于容器的应用程序。 通过使用 AKS，你可以利用 Azure 的企业级功能，同时通过 Kubernetes 和 Docker 映像格式维护应用程序可移植性。

[Azure Service Fabric](/azure/service-fabric/) 是一种分布式系统平台，可用于轻松打包、部署和管理可伸缩的可靠微服务和容器。 Service Fabric 解决了开发和管理云本机应用程序的重大挑战。 开发人员和管理员不仅可以避免复杂的基础结构问题，而且可以专注于实现可伸缩、可靠、可管理的高要求任务关键型工作负荷。 Service Fabric 新一代平台的代表，用于生成和管理在容器中运行的企业级单层云级别应用程序。

## <a name="getting-started"></a>入门

若要开始部署 Azure Kubernetes 服务，请参阅[Kubernetes 安装指南](../kubernetes/getting-started-kubernetes-windows.md)。

若要开始部署 Azure Service Fabric，请参阅[Service Fabric 快速入门](/azure/service-fabric/service-fabric-quickstart-containers.md)。
