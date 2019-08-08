---
title: Windows 上的 Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: 将 Windows 节点加入带有 v 1.14 的 Kubernetes 群集。
keywords: kubernetes、1.14、windows、入门
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 18734f102042ec951255061dcd82229e18d29a15
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998384"
---
# <a name="kubernetes-on-windows"></a>Windows 上的 Kubernetes

通过将 Windows 节点加入到基于 Linux 的群集, 此页面可用作 Windows 上的 Kubernetes 入门概述。 在 Windows Server[版本 1809](https://docs.microsoft.com/windows-server/get-started/whats-new-in-windows-server-1809#container-networking-with-kubernetes)上发布 Kubernetes 1.14 后, 用户可以利用 windows Kubernetes 中的以下功能:

- **覆盖网络**: 在 vxlan 模式下使用 Flannel 配置虚拟覆盖网络
    - 需要安装有[KB4489899](https://support.microsoft.com/help/4489899)的 windows server 2019 或[windows Server VNext 预览体验成员预览版](https://blogs.windows.com/windowsexperience/tag/windows-insider-program/)18317 +
    - 需要 Kubernetes v 1.14 (或更高版本`WinOverlay` ) 启用功能门
    - 需要 Flannel v 0.11.0 (或更高版本)
- **简化了网络管理**: 在主网关模式中使用 Flannel 在节点之间自动路由管理。
- **可伸缩性改进**: 由于[Deviceless vNICs Windows Server 容器](https://techcommunity.microsoft.com/t5/Networking-Blog/Network-start-up-and-performance-improvements-in-Windows-10/ba-p/339716), 享受更快、更可靠的容器启动时间。
- **Hyper-v 隔离 (alpha)**: 通过内核模式隔离协调[hyper-v 隔离](https://kubernetes.io/docs/getting-started-guides/windows/#hyper-v-containers), 提高了安全性。 有关详细信息, 请查看[Windows 容器类型](https://docs.microsoft.com/virtualization/windowscontainers/about/#windows-container-types)。
    - 在启用了功能门时`HyperVContainer`需要 Kubernetes v 1.10 (或更高版本)。
- **存储插件**: 将[FlexVolume 存储插件](https://github.com/Microsoft/K8s-Storage-Plugins)与适用于 Windows 容器的 SMB 和 iSCSI 支持配合使用。

>[!TIP]
>如果你想要在 Azure 上部署群集, 则开放源代码 AKS 引擎工具可轻松实现此操作。 若要了解详细信息, 请参阅我们的分步[演练](https://github.com/Azure/aks-engine/blob/master/docs/topics/windows.md)。

## <a name="prerequisites"></a>系统必备

### <a name="plan-ip-addressing-for-your-cluster"></a>为您的群集规划 IP 寻址

<a name="definitions"></a>由于 Kubernetes 群集为箱和服务引入了新的子网, 因此确保它们不会与你环境中的任何其他现有网络相冲突非常重要。 以下是需要释放的所有地址空间, 以便成功部署 Kubernetes:

| 子网/地址范围 | 说明 | 默认值 |
| --------- | ------------- | ------------- |
| <a name="service-subnet-def"></a>**服务子网** | 一个不可路由的纯虚拟子网, 可由盒式虚拟子网用于 uniformally access 服务, 而无需国度网络拓扑。 它通过节点上运行的 `kube-proxy` 与可路由的地址空间进行相互转换。 | "10.96.0.0/12" |
| <a name="cluster-subnet-def"></a>**群集子网** |  这是由群集中的所有箱使用的全局子网。 将为每个节点分配一个小的/24 个子网, 供其盒中使用。 它必须足够大才能容纳您的群集中使用的所有盒。 若要计算*最小*子网大小: `(number of nodes) + (number of nodes * maximum pods per node that you configure)` <p/>每个节点100箱的5节点群集的示例: `(5) + (5 *  100) = 505`。  | "10.244.0.0/16" |
| **Kubernetes DNS 服务 IP** | 将用于 & 群集服务发现的 DNS 解析的 "kube" 服务的 IP 地址。 | "10.96.0.10" |

> [!NOTE]
> 在安装 Docker 时, 会默认创建另一个 Docker 网络 (NAT)。 由于我们改为从群集子网分配 Ip, 因此不需要在 Windows 上运行 Kubernetes。

## <a name="what-you-will-accomplish"></a>你将实现的目标

本指南结束时，你将实现以下目标：

> [!div class="checklist"]
> * 创建了[Kubernetes 主](./creating-a-linux-master.md)节点。  
> * 已选择[网络解决方案](./network-topologies.md)。  
> * 已将[Windows worker 节点](./joining-windows-workers.md)或[Linux 辅助节点](./joining-linux-workers.md)加入到其中。  
> * 已部署[示例 Kubernetes 资源](./deploying-resources.md)。  
> * 解决[常见问题和错误](./common-problems.md)。

## <a name="next-steps"></a>后续步骤

在本部分中, 我们将讨论今天在 Windows 上成功部署 Kubernetes 所需的重要先决条件 & 假设。 继续了解如何设置 Kubernetes 母版:

>[!div class="nextstepaction"]
>[创建 Kubernetes 母版](./creating-a-linux-master.md)