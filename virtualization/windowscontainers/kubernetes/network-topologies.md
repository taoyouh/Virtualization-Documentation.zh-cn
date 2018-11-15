---
title: 网络拓扑
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: 在 Windows 和 Linux 上受支持的网络拓扑。
keywords: kubernetes，1.12，windows，入门
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: bcbd7b530b58b663305ea5d8b84a75eaf971f997
ms.sourcegitcommit: 4412583b77f3bb4b2ff834c7d3f1bdabac7aafee
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/15/2018
ms.locfileid: "6948056"
---
# <a name="network-solutions"></a>网络解决方案 #

[设置 Kubernetes 主节点](./creating-a-linux-master.md)后可以随时选择的网络解决方案。 有多种方法能够使虚拟[群集子网](./getting-started-kubernetes-windows.md#cluster-subnet-def)可路由跨节点。 选取适用于 Kubernetes 的以下选项之一在 Windows 上今天：

1. 为你使用第三方的 CNI 插件如[Flannel](network-topologies.md#flannel-in-host-gateway-mode)设置路由到。
1. 配置智能[-顶式 (ToR) 交换机](network-topologies.md#configuring-a-tor-switch)以路由子网。

> [!tip]  
> 没有第三个联网解决方案，它会利用打开 vSwitch (OvS) 在 Windows 和打开虚拟网络 (OVN) 上。 记录此已超出了本文档范围，但你可以阅读[这些说明](https://kubernetes.io/docs/getting-started-guides/windows/#for-3-open-vswitch-ovs-open-virtual-network-ovn-with-overlay)将其设置。

## <a name="flannel-in-host-gateway-mode"></a>在主机网关模式下 flannel

Flannel 网络的可用选项之一是*主机网关模式*（主机网关），该模式要求配置的所有节点上的 pod 子网之间的静态路由。
> [!NOTE]  
> 这是不同于在 Flannel，可使用 VXLAN 封装并处于开发阶段立即*覆盖*网络模式。 观看此空间...

### <a name="prepare-kubernetes-master-for-flannel"></a>为 Flannel 准备开始创建 Kubernetes 主机

在我们的群集中的[开始创建 Kubernetes 主机](./creating-a-linux-master.md)上建议一些细微的准备工作。 建议使用 Flannel 时启用到 iptables 链的桥接的 IPv4 流量。 这可以使用以下命令：

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

###  <a name="download--configure-flannel"></a>下载和配置 Flannel ###
下载最新的 Flannel 清单：

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

有两个事实需要怎么做才能启用网络跨两个 Windows/Linux 主机网关。

在`net-conf.json`部分的 kube flannel.yml 中，仔细检查：
1. 正在使用的网络后端的类型设置为`host-gw`而不是`vxlan`。
2. 群集子网 (例如"10.244.0.0/16") 设置为所需。

在应用的 2 个步骤后在`net-conf.json`应该如下所示：
```json
net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "host-gw"
      }
    }
```

### <a name="launch-flannel--validate"></a>启动 Flannel 和验证 ###
启动 Flannel 使用：

```bash
kubectl apply -f kube-flannel.yml
```

接下来，因为 Flannel pod 都是基于 Linux 的应用到我们 Linux [NodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml)修补程序`kube-flannel-ds`DaemonSet 仅面向 Linux （我们将 Flannel"flanneld"主机代理上的进程 Windows 更高版本时启动加入）：

```
kubectl patch ds/kube-flannel-ds --patch "$(cat node-selector-patch.yml)" -n=kube-system
```

几分钟后，你应该看到与运行如果 Flannel pod 网络的部署的所有 pod。

```bash
kubectl get pods --all-namespaces
```

![文本](media/kube-master.png)

Flannel DaemonSet 还应该具有应用 NodeSelector。

```bash
kubectl get ds -n kube-system
```

![文本](media/kube-daemonset.png)
> [!tip]  
> 混淆？ 下面是完整[的示例 kube flannel.yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/manifests/kube-flannel-example.yml)为 Flannel v0.9.1 2 个步骤预适用于默认群集子网`10.244.0.0/16`。

## <a name="configuring-a-tor-switch"></a>配置 ToR 交换机 ##
> [!NOTE]
> 如果你选择[Flannel 作为你的网络解决方案](#flannel-in-host-gateway-mode)，可以跳过此部分。
在实际节点之外发生 ToR 交换机配置。 有关这方面的更多详细信息，请参阅[官方 Kubernetes 文档](https://kubernetes.io/docs/getting-started-guides/windows/#upstream-l3-routing-topology)。


## <a name="next-steps"></a>后续步骤 ## 
在此部分中，我们将介绍如何选择的网络解决方案。 现在，你已准备好进行第 4 步：

> [!div class="nextstepaction"]
> [加入 Windows 工作人员](./joining-windows-workers.md)