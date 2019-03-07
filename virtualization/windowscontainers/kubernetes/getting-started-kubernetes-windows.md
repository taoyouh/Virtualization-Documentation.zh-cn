---
title: Windows 上的 Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: 将 Windows 节点加入到 Kubernetes 群集与 v1.13。
keywords: kubernetes，1.13，windows，入门
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 2edc940d4132870a6f7d6714bb50faef201136ba
ms.sourcegitcommit: 817a629f762a4a5d4bcff58302f2bc2408bf8be1
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/07/2019
ms.locfileid: "9149907"
---
# <a name="kubernetes-on-windows"></a>Windows 上的 Kubernetes #
此页面可用作概述，以便开始使用 Windows 上 Kubernetes 的 Windows 节点加入到基于 Linux 的群集。 使用 Windows Server[版本 1809](https://docs.microsoft.com/en-us/windows-server/get-started/whats-new-in-windows-server-1809#container-networking-with-kubernetes)Kubernetes 1.13 的发布，用户可以在 Kubernetes 中充分利用[最新功能](https://kubernetes.io/docs/getting-started-guides/windows/#supported-features)，在 Windows beta 版本上：

  - **覆盖网络**： 使用 Flannel vxlan 模式配置虚拟覆盖网络
    - 与安装的 KB4482887 或[Windows Server vNext Insider Preview](https://blogs.windows.com/windowsexperience/tag/windows-insider-program/)版本 18317 + 需要任何一种 Windows Server 2019
    - 需要 Kubernetes v1.14 （或以上） 与`WinOverlay`已启用的功能门槛
    - 需要 Flannel v0.11.0 （或以上）
  - **简化了的网络管理**： Flannel 在主机网关模式下用于节点间的自动路由管理
  - **可扩展性改进**： 享受借助[适用于 Windows Server 容器的非设备 vNICs](https://blogs.technet.microsoft.com/networking/2018/04/27/network-start-up-and-performance-improvements-in-windows-10-spring-creators-update-and-windows-server-version-1803/)更快、 更可靠的容器启动时间
  - **hyper-v 隔离 (alpha)**: 协调[的 hyper-v 容器](https://kubernetes.io/docs/getting-started-guides/windows/#hyper-v-containers)与增强的安全 （[请参阅 Windows 容器类型](https://docs.microsoft.com/en-us/virtualization/windowscontainers/about/#windows-container-types)） 的内核模式隔离
    - 需要 Kubernetes v1.10 （或以上） 与`HyperVContainer`已启用的功能门槛
  - **存储插件**： [FlexVolume 存储插件](https://github.com/Microsoft/K8s-Storage-Plugins)的 SMB 和 iSCSI 支持用于 Windows 容器

> [!TIP] 
> 如果你想要 Azure 上部署群集，开源 AKS 引擎工具轻松实现。 我们提供了分步[演练](https://github.com/Azure/aks-engine/blob/master/docs/topics/windows.md)供你参考。

## <a name="prerequisites"></a>先决条件 ##

### <a name="plan-ip-addressing-for-your-cluster"></a>规划 IP 地址为群集 ###
<a name="definitions"></a>如 Kubernetes 群集引入了新的子网，pod 和服务，务必确保其中任何一个与你的环境中的任何其他现有网络碰撞。 下面是为了成功部署 Kubernetes 释放所需要的所有地址空间：

| 子网地址范围 | 描述 | 默认值 |
| --------- | ------------- | ------------- |
| <a name="service-subnet-def"></a>**服务子网** | 不可路由的纯虚拟子网 pod 中用于无论网络拓扑如何该子网统一访问服务。 它通过节点上运行的 `kube-proxy` 与可路由的地址空间进行相互转换。 | "10.96.0.0/12" |
| <a name="cluster-subnet-def"></a>**群集子网** |  这是由在群集中的所有 pod 的全局子网。 每个节点分配较小的/24 子网从此使用其 pod。 它必须足够大，以适应在群集中使用的所有 pod。 若要计算*最小*的子网大小： `(number of nodes) + (number of nodes * maximum pods per node that you configure)` <p/>对于每个节点的 100 pod 的 5 个节点的群集的示例： `(5) + (5 *  100) = 505`。  | "10.244.0.0/16" |
| **Kubernetes DNS 服务 IP** | 将用于 DNS 解析 & 群集服务发现的"kube dns"服务的 IP 地址。 | "10.96.0.10" |
> [!NOTE]
> 没有另一个 Docker 网络 (NAT) 时安装 Docker 获取创建默认情况下。 它不需要能够按照我们 Ip 分配群集子网改为运行 Windows 上的 Kubernetes。



## <a name="what-you-will-accomplish"></a>你将实现的目标 ##

本指南结束时，你将实现以下目标：

> [!div class="checklist"]
> * 创建了一个[开始创建 Kubernetes 主机](./creating-a-linux-master.md)节点。  
> * 选择[网络解决方案](./network-topologies.md)。  
> * 加入[Windows 工作者节点](./joining-windows-workers.md)或[Linux 工作者节点](./joining-linux-workers.md)到它。  
> * 部署[示例 Kubernetes 资源](./deploying-resources.md)。  
> * 解决[常见问题和错误](./common-problems.md)。

## <a name="next-steps"></a>后续步骤 ##
在此部分中，我们讨论过重要先决条件今天成功部署 Windows 上的 Kubernetes 所需的 & 假设。 继续以了解如何设置开始创建 Kubernetes 主机：

> [!div class="nextstepaction"]
> [创建 Kubernetes 主机](./creating-a-linux-master.md)