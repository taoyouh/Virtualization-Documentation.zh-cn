---
title: 网络拓扑
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: how-to
ms.prod: containers
description: Windows 和 Linux 上支持的网络拓扑。
keywords: kubernetes，1.14，windows，入门
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: c322edb6a5ead34d7988f83d8cb8fba7c99cec0d
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192534"
---
# <a name="network-solutions"></a>Network Solutions #

[设置 Kubernetes 主节点](./creating-a-linux-master.md)后，就可以选择网络解决方案。 可以通过多种方式跨节点进行虚拟[群集子网](./getting-started-kubernetes-windows.md#cluster-subnet-def)路由。 在今天的 Windows 上选择以下 Kubernetes 选项之一：

1. 使用 CNI 插件（如[Flannel](#flannel-in-vxlan-mode) ）为你设置覆盖网络。
2. 使用 CNI 插件（如[Flannel](#flannel-in-host-gateway-mode) ）为你编写路由（使用 l2bridge 网络模式）。
3. 配置智能[架顶式（ToR）交换机](#configuring-a-tor-switch)来路由子网。

> [!tip]
> Windows 上有第四个网络解决方案，它利用开放 vSwitch （Ovs-es）和开放虚拟网络（OVN）。 记录此文档超出了本文档的范围，但你可以阅读[这些说明](https://kubernetes.io/docs/getting-started-guides/windows/#for-3-open-vswitch-ovs-open-virtual-network-ovn-with-overlay)进行设置。

## <a name="flannel-in-vxlan-mode"></a>Vxlan 模式下的 Flannel

Vxlan 模式下的 Flannel 可用于设置可配置的虚拟覆盖网络，该网络使用 VXLAN 隧道在节点之间路由数据包。

### <a name="prepare-kubernetes-master-for-flannel"></a>为 Flannel 准备 Kubernetes master
建议在群集的[Kubernetes 主机](./creating-a-linux-master.md)上进行一些小部分准备。 使用 Flannel 时，建议启用到 iptables 链的桥接 IPv4 流量。 可以使用以下命令完成此操作：

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

###  <a name="download--configure-flannel"></a>下载 & 配置 Flannel ###
下载最新的 Flannel 清单：

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

若要启用 vxlan 网络后端，应修改两个部分：

1. 在 `net-conf.json` 您的部分 `kube-flannel.yml` ，请仔细检查：
 * 根据需要设置群集子网（例如 "10.244.0.0/16"）。
 * VNI 4096 是在后端中设置的
 * 在后端设置端口4789
2. 在中的 `cni-conf.json` 部分 `kube-flannel.yml` ，将网络名称更改为 `"vxlan0"` 。

应用上述步骤之后， `net-conf.json` 应如下所示：
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
> 对于 Linux 上的 Flannel，VNI 必须设置为4096和端口4789，才能与 Windows 上的 Flannel 进行互操作。 即将推出对其他 VNIs 的支持。 有关这些字段的说明，请参阅[VXLAN](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#vxlan) 。

`cni-conf.json`应该如下所示：
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
> 有关上述选项的详细信息，请参阅适用于 Linux 的官方 CNI [flannel](https://github.com/containernetworking/plugins/tree/master/plugins/meta/flannel#network-configuration-reference)、 [portmap](https://github.com/containernetworking/plugins/tree/master/plugins/meta/portmap#port-mapping-plugin)和[桥接](https://github.com/containernetworking/plugins/tree/master/plugins/main/bridge#network-configuration-reference)插件文档。

### <a name="launch-flannel--validate"></a>启动 Flannel & 验证 ###
使用以下内容启动 Flannel：

```bash
kubectl apply -f kube-flannel.yml
```

接下来，由于 Flannel pod 是基于 Linux 的，因此请将 Linux [NodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml)修补程序应用到 `kube-flannel-ds` DaemonSet，仅面向 linux （在加入后，我们稍后将在 Windows 上启动 Flannel "flanneld" 主机代理进程）：

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]
> 如果任何节点不基于 x86-64，请 `-amd64` 将上面的替换为处理器体系结构。

几分钟后，如果部署了 Flannel pod 网络，则应会看到所有的箱都处于运行状态。

```bash
kubectl get pods --all-namespaces
```

![text](media/kube-master.png)

Flannel DaemonSet 还应 `beta.kubernetes.io/os=linux` 应用 NodeSelector。

```bash
kubectl get ds -n kube-system
```

![text](media/kube-daemonset.png)

> [!tip]
> 对于剩余的 flannel-* Daemonset，可以忽略或删除这些项，因为如果没有与该处理器体系结构匹配的节点，则不会对其进行安排。

> [!tip]
> 搞? 下面是 Flannel v 0.11.0 的完整[示例 kube-flannel docker-compose.override.yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/overlay/manifests/kube-flannel-example.yml) ，这些步骤预先应用于默认群集子网 `10.244.0.0/16` 。

成功后，继续执行[后续步骤](#next-steps)。

## <a name="flannel-in-host-gateway-mode"></a>主机-网关模式下的 Flannel

除了[Flannel vxlan](#flannel-in-vxlan-mode)，Flannel 网络的另一个选项是*主机网关模式*（主机-gw），该模式需要使用目标节点的主机地址作为下一个跃点，将每个节点上的静态路由的编程方式用于其他节点的 pod 子网。

### <a name="prepare-kubernetes-master-for-flannel"></a>为 Flannel 准备 Kubernetes master

建议在群集的[Kubernetes 主机](./creating-a-linux-master.md)上进行一些小部分准备。 使用 Flannel 时，建议启用到 iptables 链的桥接 IPv4 流量。 可以使用以下命令完成此操作：

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```


###  <a name="download--configure-flannel"></a>下载 & 配置 Flannel ###
下载最新的 Flannel 清单：

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

必须更改一个文件，才能在 Windows/Linux 中启用主机-gw 网络。

在 `net-conf.json` kube-flannel. docker-compose.override.yml 的部分中，仔细检查以下内容：
1. 所使用的网络后端的类型设置为 `host-gw` 而不是 `vxlan` 。
2. 根据需要设置群集子网（例如 "10.244.0.0/16"）。

应用2个步骤后， `net-conf.json` 应如下所示：
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
使用以下内容启动 Flannel：

```bash
kubectl apply -f kube-flannel.yml
```

接下来，由于 Flannel pod 是基于 Linux 的，因此请将 Linux [NodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml)修补程序应用到 `kube-flannel-ds` DaemonSet，仅面向 linux （在加入后，我们将在 Windows 上启动 Flannel "flanneld" 主机代理进程）：

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]
> 如果任何节点不基于 x86-64，请 `-amd64` 将上面的替换为所需的处理器体系结构。

几分钟后，如果部署了 Flannel pod 网络，则应会看到所有的箱都处于运行状态。

```bash
kubectl get pods --all-namespaces
```

![text](media/kube-master.png)

Flannel DaemonSet 还应应用 NodeSelector。

```bash
kubectl get ds -n kube-system
```

![text](media/kube-daemonset.png)

> [!tip]
> 对于剩余的 flannel-* Daemonset，可以忽略或删除这些项，因为如果没有与该处理器体系结构匹配的节点，则不会对其进行安排。

> [!tip]
> 搞? 下面是 Flannel v 0.11.0 的完整[示例 kube-flannel. docker-compose.override.yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/manifests/kube-flannel-example.yml) ，其中2个步骤预先应用于默认群集子网 `10.244.0.0/16` 。

成功后，继续执行[后续步骤](#next-steps)。

## <a name="configuring-a-tor-switch"></a>配置 ToR 交换机 ##
> [!NOTE]
> 如果选择 " [Flannel" 作为网络解决方案](#flannel-in-host-gateway-mode)，则可以跳过此部分。
ToR 开关的配置发生在实际节点之外。 有关此内容的详细信息，请参阅[官方 Kubernetes 文档](https://kubernetes.io/docs/getting-started-guides/windows/#upstream-l3-routing-topology)。


## <a name="next-steps"></a>后续步骤 ##
在本部分中，我们介绍了如何选择和配置网络解决方案。 现在，你已准备好执行步骤4：

> [!div class="nextstepaction"]
> [加入 Windows 辅助角色](./joining-windows-workers.md)