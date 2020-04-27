---
title: Windows 容器业务流程概述
description: 了解 Windows 容器业务流程协调程序。
keywords: docker, 容器
author: Heidilohr
ms.author: helohr
ms.date: 05/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 23dd1e56ba68a679945779f5e7dbc15225412934
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/24/2020
ms.locfileid: "78853902"
---
# <a name="windows-container-orchestration-overview"></a>Windows 容器业务流程概述

由于规模较小且面向应用程序，容器非常适合敏捷交付环境和基于微服务的体系结构。 然而，使用容器和微服务的环境可能有数百个或数千个要跟踪的组件。 你也许能够手动管理几十台虚拟机或物理服务器，但却无法在非自动化的情况下正确管理生产级容器环境。 此任务应由业务流程协调程序处理，该程序可自动完成并管理大量容器及其交互方式。

业务流程协调程序执行下列任务：

- 调度：在给定容器映像和资源请求的情况下，业务流程协调程序会找到适合运行该容器的计算机。
- 相关性/反相关性：指定一组容器在运行时是应彼此靠近以提高性能，还是应保持距离以提高可用性。
- 运行状况监视：监视容器故障并自动重新调度。
- 故障转移：跟踪每台计算机上运行的组件，并将容器从有故障的计算机重新调度到正常运行的节点。
- 伸缩：根据需要手动或自动添加或删除容器实例。
- 网络：提供覆盖网络以协调容器，以便跨多台主机通信。
- 服务发现：让容器即使在主机之间迁移且 IP 地址已改变时，也能实现相互间的自动定位。
- 协调的应用程序升级：管理容器升级，避免应用程序关闭，并在出现错误时启用回滚。

## <a name="orchestrator-types"></a>业务流程协调程序类型

Azure 提供两个容器业务流程协调程序：Azure Kubernetes 服务 (AKS) 和 Service Fabric。

[Azure Kubernetes 服务 (AKS)](/azure/aks/) 有助于轻松创建、配置和管理预先配置为运行容器化应用程序的虚拟机群集。 这使你能够使用现有技能并利用大量的社区专业知识，在 Microsoft Azure 上部署和管理基于容器的应用程序。 通过使用 AKS，你可以充分利用 Azure 的企业级功能，同时通过 Kubernetes 和 Docker 映像格式保持应用程序的可移植性。

[Azure Service Fabric](/azure/service-fabric/) 是一种分布式系统平台，可用于轻松打包、部署和管理可伸缩的可靠微服务和容器。 Service Fabric 解决了开发和管理云本机应用程序的重大挑战。 开发人员和管理员不仅可以避免复杂的基础结构问题，而且可以专注于实现可伸缩、可靠、可管理的高要求任务关键型工作负荷。 Service Fabric 新一代平台的代表，用于生成和管理在容器中运行的企业级单层云级别应用程序。

## <a name="getting-started"></a>入门

若要开始部署 Azure Kubernetes 服务，请参阅 [Kubernetes 安装指南](../kubernetes/getting-started-kubernetes-windows.md)。

若要开始部署 Azure Service Fabric，请参阅 [Service Fabric 快速入门](/azure/service-fabric/service-fabric-quickstart-containers.md)。
