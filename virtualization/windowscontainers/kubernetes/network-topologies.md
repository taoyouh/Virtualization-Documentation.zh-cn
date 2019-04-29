---
title: 网络拓扑
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: 在 Windows 和 Linux 上受支持的网络拓扑。
keywords: kubernetes，1.13，windows，入门
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 9f96fcc80c533b74ab46d93beecc7ca8629ce395
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/26/2019
ms.locfileid: "9576128"
---
# <a name="network-solutions"></a>网络解决方案 #

[设置 Kubernetes 主节点](./creating-a-linux-master.md)后可以随时选择的网络解决方案。 有多种方法能够使虚拟[群集子网](./getting-started-kubernetes-windows.md#cluster-subnet-def)可路由跨节点。 选取用于 Kubernetes 的以下选项之一在 Windows 上现在：

1. 使用如[Flannel](#flannel-in-vxlan-mode) CNI 插件为你设置覆盖网络。
2. 为你使用程序路由到如[Flannel](#flannel-in-host-gateway-mode) CNI 插件。
3. 配置智能[顶部的顶式 (ToR) 交换机](#configuring-a-tor-switch)以路由子网。

> [!tip]  
> 没有第四个网络它会利用打开 vSwitch (OvS) 在 Windows 和打开虚拟网络 (OVN) 上的解决方案。 记录此已超出了本文档范围，但你可以阅读[这些说明](https://kubernetes.io/docs/getting-started-guides/windows/#for-3-open-vswitch-ovs-open-virtual-network-ovn-with-overlay)将其设置。

## <a name="flannel-in-vxlan-mode"></a>在 vxlan 模式下的 flannel

在 vxlan 模式下的 flannel 可以用于设置使用 VXLAN 隧道路由节点之间的数据包的可配置的虚拟覆盖网络。

### <a name="prepare-kubernetes-master-for-flannel"></a>为 Flannel 准备开始创建 Kubernetes 主机
在我们的群集中的[开始创建 Kubernetes 主机](./creating-a-linux-master.md)上建议一些细微的准备工作。 建议使用 Flannel 时启用到 iptables 链的桥接的 IPv4 流量。 这可以使用以下命令：

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

###  <a name="download--configure-flannel"></a>下载 & 配置 Flannel ###
下载最新的 Flannel 清单：

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

有两个部分应修改启用 vxlan 网络后端：

1. 在`net-conf.json`部分你`kube-flannel.yml`，仔细检查：
 * 群集子网 (例如"10.244.0.0/16") 设置为所需。
 * 在后端中设置 VNI 4096
 * 端口 4789 设置在后端
2. 在`cni-conf.json`部分你`kube-flannel.yml`，网络名称更改为`"vxlan0"`。

后应用上述步骤中，你`net-conf.json`应该如下所示：
```json
  net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan",
        "VNI" : 4096,
        "Port": 4789
      }
    }
```

> [!NOTE]  
> 必须设置为 4096 和端口 4789 用于在 Linux 上的 Flannel VNI 与在 Windows 上的 Flannel 进行互操作。 对其他 VNIs 支持功能即将发布。 有关这些字段的说明，请参阅[VXLAN](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#vxlan) 。

在`cni-conf.json`应该如下所示：
```json
cni-conf.json: |
    {
      "name": "vxlan0",
      "plugins": [
        {
          "type": "flannel",
          "delegate": {
            "hairpinMode": true,
            "isDefaultGateway": true
          }
        },
        {
          "type": "portmap",
          "capabilities": {
            "portMappings": true
          }
        }
      ]
    }
```
> [!tip]  
> 有关上述选项的详细信息，请参阅官方 CNI [flannel](https://github.com/containernetworking/plugins/tree/master/plugins/meta/flannel#network-configuration-reference)、[端口映射](https://github.com/containernetworking/plugins/tree/master/plugins/meta/portmap#port-mapping-plugin)，以及[桥](https://github.com/containernetworking/plugins/tree/master/plugins/main/bridge#network-configuration-reference)插件文档，linux。

### <a name="launch-flannel--validate"></a>启动 Flannel & 验证 ###
启动 Flannel 使用：

```bash
kubectl apply -f kube-flannel.yml
```

接下来，因为 Flannel pod 都是基于 Linux 的应用到的 Linux [NodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml)修补程序`kube-flannel-ds`DaemonSet 仅面向 Linux （我们将 Flannel"flanneld"主机代理过程在 Windows 上的更高版本时启动加入）：

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> 如果任何节点不是基于 x86-64，替换`-amd64`上面带有处理器体系结构。

几分钟后，你应该看到与运行如果 Flannel pod 网络部署的所有 pod。

```bash
kubectl get pods --all-namespaces
```

![文本](media/kube-master.png)

Flannel DaemonSet 还应该具有 NodeSelector`beta.kubernetes.io/os=linux`应用。

```bash
kubectl get ds -n kube-system
```

![文本](media/kube-daemonset.png)

> [!tip]  
> 对于剩余 flannel-ds-* DaemonSets，它们可以忽略或删除，因为如果没有匹配的处理器体系结构节点将不会进行计划。

> [!tip]  
> 混淆？ 下面是 Flannel v0.11.0 预应用默认群集子网的这些步骤的完整[示例 kube flannel.yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/overlay/manifests/kube-flannel-example.yml) `10.244.0.0/16`。

后成功，继续执行[下一步骤](#next-steps)。

## <a name="flannel-in-host-gateway-mode"></a>在主机网关模式下的 flannel

一起[Flannel vxlan](#flannel-in-vxlan-mode)，Flannel 网络的另一个选项是*主机网关模式*（主机网关），它需要静态路由到其他节点的 pod 子网使用目标节点的主机地址作为下一跃点每个节点上的编程。

### <a name="prepare-kubernetes-master-for-flannel"></a>为 Flannel 准备开始创建 Kubernetes 主机

在我们的群集中的[开始创建 Kubernetes 主机](./creating-a-linux-master.md)上建议一些细微的准备工作。 建议使用 Flannel 时启用到 iptables 链的桥接的 IPv4 流量。 这可以使用以下命令：

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```


###  <a name="download--configure-flannel"></a>下载 & 配置 Flannel ###
下载最新的 Flannel 清单：

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

没有所需更改，以便启用网络跨两个 Windows/Linux 主机网关的一个文件。

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

### <a name="launch-flannel--validate"></a>启动 Flannel & 验证 ###
启动 Flannel 使用：

```bash
kubectl apply -f kube-flannel.yml
```

接下来，因为 Flannel pod 都是基于 Linux 的应用到我们 Linux [NodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml)修补程序`kube-flannel-ds`DaemonSet 仅面向 Linux （我们将 Flannel"flanneld"主机代理过程在 Windows 上的更高版本时启动加入）：

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> 如果任何节点不是基于 x86-64，替换`-amd64`上面使用的所需的处理器体系结构。

几分钟后，你应该看到与运行如果 Flannel pod 网络部署的所有 pod。

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
> 对于剩余 flannel-ds-* DaemonSets，它们可以忽略或删除，因为如果没有匹配的处理器体系结构节点将不会进行计划。

> [!tip]  
> 混淆？ 下面是 2 个步骤预应用默认群集子网的完整[示例 kube flannel.yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/manifests/kube-flannel-example.yml)为 Flannel v0.11.0 `10.244.0.0/16`。

后成功，继续执行[下一步骤](#next-steps)。

## <a name="configuring-a-tor-switch"></a>配置 ToR 交换机 ##
> [!NOTE]
> 如果你选择[Flannel 作为你的网络解决方案](#flannel-in-host-gateway-mode)，可以跳过此部分。
在实际节点之外发生 ToR 交换机配置。 有关这方面的更多详细信息，请参阅[官方 Kubernetes 文档](https://kubernetes.io/docs/getting-started-guides/windows/#upstream-l3-routing-topology)。


## <a name="next-steps"></a>后续步骤 ## 
在此部分中，我们将介绍如何选择和配置网络解决方案。 现在，你可以随时步骤 4:

> [!div class="nextstepaction"]
> [加入 Windows 工作人员](./joining-windows-workers.md)