---
title: Windows 上的 Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: 将 Windows 节点加入到 Kubernetes 群集与 v1.12。
keywords: kubernetes，1.12，windows，入门
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 0e43b2ac5b19d16721c1ba0dd1f34e339223bdaf
ms.sourcegitcommit: 8e9252856869135196fd054e3cb417562f851b51
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/08/2018
ms.locfileid: "6178900"
---
# <a name="kubernetes-on-windows"></a>Windows 上的 Kubernetes #
此页面可用作概述，以便开始使用 Windows 上的 Kubernetes 由 Windows 节点加入到基于 Linux 的群集。 在 Windows Server[版本 1803](https://docs.microsoft.com/en-us/windows-server/get-started/whats-new-in-windows-server-1803#kubernetes) beta 版上的 Kubernetes 1.12 版本，用户可以充分利用[最新功能](https://kubernetes.io/docs/getting-started-guides/windows/#supported-features)在 Kubernetes 中在 Windows 上：

  - **简化了的网络管理**： Flannel 主机网关模式下使用以自动路由节点之间的管理
  - **可扩展性改进**： 享受感谢[为 Windows Server 容器的非设备 vNICs](https://blogs.technet.microsoft.com/networking/2018/04/27/network-start-up-and-performance-improvements-in-windows-10-spring-creators-update-and-windows-server-version-1803/)更快、 更可靠的容器启动时间
  - **hyper-v 隔离 (alpha)**: 协调[的 hyper-v 容器](https://kubernetes.io/docs/getting-started-guides/windows/#hyper-v-containers)与增强的安全 （[请参阅 Windows 容器类型](https://docs.microsoft.com/en-us/virtualization/windowscontainers/about/#windows-container-types)） 的内核模式隔离
  - **存储插件**： [FlexVolume 存储插件](https://github.com/Microsoft/K8s-Storage-Plugins)SMB 和 iSCSI 的支持用于 Windows 容器

> [!TIP] 
> 如果要在 Azure 上部署群集，可以使用开源 ACS-Engine 工具轻松实现。 我们提供了分步[演练](https://github.com/Azure/acs-engine/blob/master/docs/kubernetes/windows.md)供你参考。

## <a name="prerequisites"></a>先决条件 ##

### <a name="plan-ip-addressing-for-your-cluster"></a>规划群集 IP 地址 ###
<a name="definitions"></a>Kubernetes 群集引入了新的子网，pod 和服务，它是一定要确保其中任何一个与你的环境中的任何其他现有网络碰撞。 下面是为了成功部署 Kubernetes 释放所需的所有地址空间：

| 子网地址范围 | 说明 | 默认值 |
| --------- | ------------- | ------------- |
| <a name="service-subnet-def"></a>**服务子网** | 不可路由的纯虚拟子网 pod 中用于无论网络拓扑如何该子网统一访问服务。 它通过节点上运行的 `kube-proxy` 与可路由的地址空间进行相互转换。 | "10.96.0.0/12" |
| <a name="cluster-subnet-def"></a>**群集子网** |  这是由群集中的所有 pod 的全局子网。 每个节点分配较小的/24 子网从此使用其 pod。 它必须足够大，以适应在群集中使用的所有 pod。 若要计算*最小*的子网大小： `(number of nodes) + (number of nodes * maximum pods per node that you configure)` <p/>对于每个节点的 100 pod 的 5 个节点的群集的示例： `(5) + (5 *  100) = 505`。  | "10.244.0.0/16" |
| **Kubernetes DNS 服务 IP** | "Kube dns"服务的 DNS 解析和群集服务发现将使用的 IP 地址。 | "10.96.0.10" |
> [!NOTE]
> 没有其他 Docker 网络 (NAT) 获取创建默认情况下，当你安装 Docker。 它不需要能够按照我们 Ip 分配群集子网改为运行 Windows 上的 Kubernetes。

### <a name="disable-anti-spoofing-protection"></a>禁用反欺骗的防护 ###
> [!Important] 
> 请阅读本节仔细所需的任何人都可以成功使用虚拟机立即部署 Windows 上的 Kubernetes 原样。

确保 MAC 地址欺骗和 Windows 容器主机虚拟机 （来宾） 启用虚拟化。 若要实现此目的，你应托管的虚拟机 （HYPER-V 给定的示例） 在计算机上以管理员身份运行以下内容：

```powershell
Set-VMProcessor -VMName "<name>" -ExposeVirtualizationExtensions $true 
Get-VMNetworkAdapter -VMName "<name>" | Set-VMNetworkAdapter -MacAddressSpoofing On
```
> [!TIP]
> 如果你使用基于 VMware 的产品以满足你的虚拟化，请考虑为启用[混杂模式](https://kb.vmware.com/s/article/1004099)MAC 欺骗要求。

>[!TIP]
> 如果你要自己部署 Kubernetes Azure IaaS 虚拟机上的，请检查支持此要求的[嵌套虚拟化](https://azure.microsoft.com/en-us/blog/nested-virtualization-in-azure/)的 Vm。

## <a name="what-you-will-accomplish"></a>你将实现的目标 ##

本指南结束时，你将实现以下目标：

> [!div class="checklist"]
> * 创建了一个[开始创建 Kubernetes 主机](./creating-a-linux-master.md)节点。  
> * 选择[网络解决方案](./network-topologies.md)。  
> * 加入[Windows 工作者节点](./joining-windows-workers.md)或[Linux 工作者节点](./joining-linux-workers.md)到它。  
> * 部署[示例 Kubernetes 资源](./deploying-resources.md)。  
> * 解决[常见问题和错误](./common-problems.md)。

## <a name="next-steps"></a>后续步骤 ##
在此部分中，我们讨论了重要的先决条件和今天成功部署 Windows 上的 Kubernetes 所需的假设。 若要了解如何设置开始创建 Kubernetes 主机的继续：

> [!div class="nextstepaction"]
> [创建 Kubernetes 主机](./creating-a-linux-master.md)