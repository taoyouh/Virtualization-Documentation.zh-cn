---
title: Windows 上的 Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: 将 Windows 节点加入到带有 v 1.14 的 Kubernetes 群集。
keywords: kubernetes，1.14，windows，入门
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 18734f102042ec951255061dcd82229e18d29a15
ms.sourcegitcommit: 8eedfdc1fda9d0abb36e28dc2b5fb39891777364
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/15/2020
ms.locfileid: "79402888"
---
# <a name="kubernetes-on-windows"></a>Windows 上的 Kubernetes

此页概述了如何通过将 Windows 节点加入到基于 Linux 的群集，在 Windows 上开始使用 Kubernetes。 随着 Windows Server[版本 1809](https://docs.microsoft.com/windows-server/get-started/whats-new-in-windows-server-1809#container-networking-with-kubernetes)上的 Kubernetes 1.14 版本的发布，用户可以在 windows 上的 Kubernetes 中利用以下功能：

- **覆盖网络**：使用 vxlan 模式下的 Flannel 配置虚拟覆盖网络
    - 需要安装 Windows Server 2019 with [KB4489899](https://support.microsoft.com/help/4489899)或[Windows Server vNext 有问必答 preview](https://blogs.windows.com/windowsexperience/tag/windows-insider-program/) Build 18317 +
    - 需要 Kubernetes v 1.14 （或更高版本）启用 `WinOverlay` 功能入口
    - 需要 Flannel v 0.11.0 （或更高版本）
- **简化的网络管理**：在主机-网关模式下使用 Flannel 在节点间自动路由管理。
- **可伸缩性改进**： [Deviceless Vnic for Windows Server](https://techcommunity.microsoft.com/t5/Networking-Blog/Network-start-up-and-performance-improvements-in-Windows-10/ba-p/339716)container，享受更快速、更可靠的容器启动时间。
- **Hyper-v 隔离（alpha）** ：通过内核模式隔离来协调[hyper-v 隔离](https://kubernetes.io/docs/getting-started-guides/windows/#hyper-v-containers)，以增强安全性。 有关详细信息，请查看[Windows 容器类型](https://docs.microsoft.com/virtualization/windowscontainers/about/#windows-container-types)。
    - 已启用 `HyperVContainer` 功能入口需要 Kubernetes v 1.10 （或更高版本）。
- **存储插件**：对 Windows 容器使用 SMB 和 iSCSI 支持中的[FlexVolume 存储插件](https://github.com/Microsoft/K8s-Storage-Plugins)。

>[!TIP]
>如果你想要在 Azure 上部署群集，则开源 AKS 工具使其变得简单。 若要了解详细信息，请参阅我们的分步[演练](https://github.com/Azure/aks-engine/blob/master/docs/topics/windows.md)。

## <a name="prerequisites"></a>先决条件

### <a name="plan-ip-addressing-for-your-cluster"></a>规划群集的 IP 寻址

<a name="definitions"></a>由于 Kubernetes 群集引入了用于 pod 和服务的新子网，因此务必确保它们都不会与环境中的任何其他现有网络发生冲突。 下面是为了成功部署 Kubernetes 需要释放的所有地址空间：

| 子网/地址范围 | 说明 | 默认值 |
| --------- | ------------- | ------------- |
| <a name="service-subnet-def"></a>**服务子网** | Pod 用于 uniformally 访问服务而无需关心网络拓扑的不可路由的纯粹虚拟子网。 它通过节点上运行的 `kube-proxy` 与可路由的地址空间进行相互转换。 | "10.96.0.0/12" |
| <a name="cluster-subnet-def"></a>**群集子网** |  这是由群集中的所有 pod 使用的全局子网。 为每个节点分配一个较小的/24 个子网，供其盒使用。 它必须足够大，才能容纳群集中使用的所有 pod。 若要计算*最小*子网大小： `(number of nodes) + (number of nodes * maximum pods per node that you configure)` <p/>每个节点100盒的5节点群集示例： `(5) + (5 *  100) = 505`。  | "10.244.0.0/16" |
| **Kubernetes DNS 服务 IP** | 将用于 DNS 解析 & 群集服务发现的 "kube" 服务的 IP 地址。 | "10.96.0.10" |

> [!NOTE]
> 安装 Docker 时，默认情况下会创建另一个 Docker 网络（NAT）。 不需要在 Windows 上运行 Kubernetes，因为我们改为分配群集子网中的 Ip。

## <a name="what-you-will-accomplish"></a>你将实现的目标

本指南结束时，你将实现以下目标：

> [!div class="checklist"]
> * 创建了[Kubernetes 主](./creating-a-linux-master.md)节点。  
> * 选择了一个[网络解决方案](./network-topologies.md)。  
> * 已将[Windows 辅助角色](./joining-windows-workers.md)节点或[Linux 辅助角色节点](./joining-linux-workers.md)加入到其中。  
> * 部署了[示例 Kubernetes 资源](./deploying-resources.md)。  
> * 解决[常见问题和错误](./common-problems.md)。

## <a name="next-steps"></a>后续步骤

在本部分中，我们讨论了在 Windows 上成功部署 Kubernetes 所需的重要先决条件 & 假设。 继续了解如何设置 Kubernetes master：

>[!div class="nextstepaction"]
>[创建 Kubernetes 主机](./creating-a-linux-master.md)