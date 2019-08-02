---
title: 网络拓扑
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Windows 和 Linux 上支持的网络拓扑。
keywords: kubernetes、1.14、windows、入门
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 6b0e13258b749ad3dfd5c8349200ca8a54908952
ms.sourcegitcommit: 42cb47ba4f3e22163869d094bd0c9cff415a43b0
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/02/2019
ms.locfileid: "9884978"
---
# <a name="network-solutions"></a>网络解决方案 #

[设置 Kubernetes 主节点](./creating-a-linux-master.md)后, 即可选择网络解决方案。 有多种方法可使虚拟[群集子网](./getting-started-kubernetes-windows.md#cluster-subnet-def)可跨节点进行路由。 在 Windows 今日上为 Kubernetes 选择以下选项之一:

1. 使用 CNI 插件 (如[Flannel](#flannel-in-vxlan-mode) ) 为你设置覆盖网络。
2. 使用 CNI 插件 (如[Flannel](#flannel-in-host-gateway-mode) ) 为你编写路线 (使用 l2bridge 网络模式)。
3. 配置智能的[货架 (ToR) 开关](#configuring-a-tor-switch)以路由子网。

> [!tip]  
> Windows 上有第四个网络解决方案, 它利用开放式 vSwitch (OvS) 和开放虚拟网络 (OVN)。 记录此文档的范围超出范围, 但你可以阅读[这些说明](https://kubernetes.io/docs/getting-started-guides/windows/#for-3-open-vswitch-ovs-open-virtual-network-ovn-with-overlay)进行设置。

## <a name="flannel-in-vxlan-mode"></a>Vxlan 模式下的 Flannel

Vxlan 模式中的 Flannel 可用于设置可配置的虚拟覆盖网络, 该网络使用 VXLAN 隧道在节点之间路由数据包。

### <a name="prepare-kubernetes-master-for-flannel"></a>为 Flannel 准备 Kubernetes 母版
我们的群集中的[Kubernetes 主机](./creating-a-linux-master.md)上建议一些次要准备。 使用 Flannel 时, 建议启用到 iptables 链的桥接 IPv4 流量。 可以使用以下命令执行此操作:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

###  <a name="download--configure-flannel"></a>下载 & 配置 Flannel ###
下载最新的 Flannel 清单:

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

要启用 vxlan 网络后端, 您应该修改两个部分:

1. 在您`net-conf.json` `kube-flannel.yml`的的部分中, 仔细检查:
 * 群集子网 (如 "10.244.0.0/16") 按需设置。
 * VNI 4096 是在后端设置的
 * 端口4789在后端中设置
2. 在您`cni-conf.json` `kube-flannel.yml`的部分中, 将网络名称更改为`"vxlan0"`。

在应用上述步骤后, 您`net-conf.json`应看看如下:
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
> VNI 必须设置为4096和 Flannel 上的端口 4789, 才能与 Windows 上的 Flannel 进行互操作。 即将推出对其他 VNIs 的支持。 有关这些字段的说明, 请参阅[VXLAN](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#vxlan) 。

你`cni-conf.json`应如下所示:
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
> 有关上述选项的详细信息, 请参阅适用于 Linux 的官方 CNI [flannel](https://github.com/containernetworking/plugins/tree/master/plugins/meta/flannel#network-configuration-reference)、 [portmap](https://github.com/containernetworking/plugins/tree/master/plugins/meta/portmap#port-mapping-plugin)和[桥接](https://github.com/containernetworking/plugins/tree/master/plugins/main/bridge#network-configuration-reference)插件文档。

### <a name="launch-flannel--validate"></a>启动 Flannel & 验证 ###
启动 Flannel 使用:

```bash
kubectl apply -f kube-flannel.yml
```

接下来, 由于 Flannel 盒是基于 Linux 的, 因此请将 Linux [NodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml)修补`kube-flannel-ds`程序应用到 DaemonSet, 仅面向 linux (我们稍后将在 Windows 上启动 Flannel "flanneld" host-agent 流程):

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> 如果任何节点不是基于 x86 的, 请`-amd64`将其替换为处理器体系结构。

几分钟后, 如果 Flannel pod 网络已部署, 你应该看到所有的箱都在运行。

```bash
kubectl get pods --all-namespaces
```

![文本](media/kube-master.png)

Flannel DaemonSet 还应应用 NodeSelector `beta.kubernetes.io/os=linux` 。

```bash
kubectl get ds -n kube-system
```

![文本](media/kube-daemonset.png)

> [!tip]  
> 对于剩余的 flannel-* DaemonSets, 它们可以被忽略或删除, 因为如果没有与该处理器体系结构匹配的节点, 则不会安排它们。

> [!tip]  
> <? 下面是 Flannel v 0.11.0 的完整[示例 kube-flannel](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/overlay/manifests/kube-flannel-example.yml) , 这些步骤适用于默认群集子网`10.244.0.0/16`。

成功后, 请继续执行[后续步骤](#next-steps)。

## <a name="flannel-in-host-gateway-mode"></a>主机网关模式下的 Flannel

除了[Flannel vxlan](#flannel-in-vxlan-mode), Flannel 网络的另一种选择是*主机网关模式*(host-gw), 它在每个节点上使用目标节点的主机地址作为下一个跃点, 对其他节点的 pod 子网进行静态路由的编程。

### <a name="prepare-kubernetes-master-for-flannel"></a>为 Flannel 准备 Kubernetes 母版

我们的群集中的[Kubernetes 主机](./creating-a-linux-master.md)上建议一些次要准备。 使用 Flannel 时, 建议启用到 iptables 链的桥接 IPv4 流量。 可以使用以下命令执行此操作:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```


###  <a name="download--configure-flannel"></a>下载 & 配置 Flannel ###
下载最新的 Flannel 清单:

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

您需要更改一个文件才能在两个 Windows/Linux 上启用主机-gw 网络。

在 kube-flannel `net-conf.json`的 yml 部分中, 仔细检查:
1. 要使用的网络后端的类型设置为`host-gw`而不`vxlan`是。
2. 群集子网 (如 "10.244.0.0/16") 按需设置。

应用两个步骤后, `net-conf.json`应如下所示:
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
启动 Flannel 使用:

```bash
kubectl apply -f kube-flannel.yml
```

接下来, 由于 Flannel 箱是基于 Linux 的, 因此请将[](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml)我们的 linux `kube-flannel-ds` NodeSelector 修补程序应用到 DaemonSet, 仅面向 linux (我们稍后将在 Windows 上启动 Flannel "flanneld" host-agent 流程):

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> 如果任何节点不是基于 x86 的, 请`-amd64`将以上替换为所需的处理器架构。

几分钟后, 如果 Flannel pod 网络已部署, 你应该看到所有的箱都在运行。

```bash
kubectl get pods --all-namespaces
```

![文本](media/kube-master.png)

Flannel DaemonSet 还应应用 NodeSelector。

```bash
kubectl get ds -n kube-system
```

![文本](media/kube-daemonset.png)

> [!tip]  
> 对于剩余的 flannel-* DaemonSets, 它们可以被忽略或删除, 因为如果没有与该处理器体系结构匹配的节点, 则不会安排它们。

> [!tip]  
> <? 下面是 Flannel v 0.11.0 的完整[示例 kube-flannel](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/manifests/kube-flannel-example.yml) , 其中包括针对默认群集子网`10.244.0.0/16`预应用的两个步骤的 yml。

成功后, 请继续执行[后续步骤](#next-steps)。

## <a name="configuring-a-tor-switch"></a>配置 ToR 开关 ##
> [!NOTE]
> 如果选择 " [Flannel" 作为网络解决方案](#flannel-in-host-gateway-mode), 则可以跳过此部分。
ToR 切换的配置发生在你的实际节点之外。 有关此操作的更多详细信息, 请参阅[官方 Kubernetes 文档](https://kubernetes.io/docs/getting-started-guides/windows/#upstream-l3-routing-topology)。


## <a name="next-steps"></a>后续步骤 ## 
在本部分中, 我们介绍了如何选择和配置网络解决方案。 现在, 您已准备好执行步骤 4:

> [!div class="nextstepaction"]
> [加入 Windows 工作人员](./joining-windows-workers.md)