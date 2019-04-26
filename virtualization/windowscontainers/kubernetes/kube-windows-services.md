---
title: 作为 Windows 服务运行 Kubernetes
author: daschott
ms.author: daschott
ms.date: 02/12/2019
ms.topic: get-started-article
ms.prod: containers
description: 如何为 Windows 服务运行 Kubernetes 组件。
keywords: kubernetes，1.13，windows，入门
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5c18
ms.openlocfilehash: 6c68edda6e2017640b0a490c3c30f063c81698b3
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/26/2019
ms.locfileid: "9578638"
---
# <a name="kubernetes-components-as-windows-services"></a>作为 Windows 服务的 Kubernetes 组件 

某些用户可能想要配置如 flanneld.exe、 kubelet.exe、 kube proxy.exe 或其他人作为 Windows 服务运行的进程。 这带来了自动重启后意外的进程或节点崩溃的流程如其他容错好处。


## <a name="prerequisites"></a>系统必备
1. 你已下载到[nssm.exe](https://nssm.cc/download) `c:\k`目录
2. 已加入到群集的节点和以前在节点上运行[install.ps1](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/install.ps1)或[start.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1)脚本

## <a name="registering-windows-services"></a>注册 Windows 服务
你可以运行[示例脚本](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/register-svc.ps1)将注册的使用 nssm.exe `kubelet`， `kube-proxy`，并`flanneld.exe`作为 Windows 服务在后台运行：

```
C:\k\register-svc.ps1 -NetworkMode <Network mode> -ManagementIP <Windows Node IP> -ClusterCIDR <Cluster subnet> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Directory to place logs>
```

# [<a name="managementip"></a>ManagementIP](#tab/ManagementIP)
分配给 Windows 节点的 IP 地址。 你可以使用`ipconfig`查找此。

|  |  | 
|---------|---------|
|参数     | `-ManagementIP`        |
|默认值    | n.A.        |


# [<a name="networkmode"></a>NetworkMode](#tab/NetworkMode)
网络模式`l2bridge`(flannel 主机网关) 或`overlay`(flannel vxlan) 选择作为[网络解决方案](./network-topologies.md)。

> [!Important] 
> `overlay` 网络模式 (flannel vxlan) 需要 Kubernetes v1.14 二进制文件或以上。

|  |  | 
|---------|---------|
|参数     | `-NetworkMode`        |
|默认值    | `l2bridge`        |


# [<a name="clustercidr"></a>ClusterCIDR](#tab/ClusterCIDR)
[群集子网范围](./getting-started-kubernetes-windows.md#cluster-subnet-def)。

|  |  | 
|---------|---------|
|参数     | `-ClusterCIDR`        |
|默认值    | `10.244.0.0/16`        |


# [<a name="kubednsserviceip"></a>KubeDnsServiceIP](#tab/KubeDnsServiceIP)
[Kubernetes DNS 服务 IP](./getting-started-kubernetes-windows.md#kube-dns-def)。

|  |  | 
|---------|---------|
|参数     | `-KubeDnsServiceIP`        |
|默认值    | `10.96.0.10`        |


# [<a name="logdir"></a>LogDir](#tab/LogDir)
在其中 kubelet 和 kube 代理日志将重定向到其各自的输出文件的目录。

|  |  | 
|---------|---------|
|参数     | `-LogDir`        |
|默认值    | `C:\k`        |

---


> [!TIP] 
> 应出现了错误，请参阅[疑难解答部分](./common-problems.md#i-have-problems-running-kubernetes-processes-as-windows-services)

## <a name="manual-approach"></a>手动方法
应[上面引用的脚本](#registering-windows-services)不适用于你，本部分提供了一些*示例命令*用于注册手动分步这些服务。

> [!TIP] 
> 有关如何配置的更多详细信息请参见[Kubelet 和 kube 代理现在可以运行为 Windows 服务](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services)`kubelet`和`kube-proxy`作为本机 Windows 服务，通过运行`sc`。

### <a name="register-flanneldexe"></a>注册 flanneld.exe
```
nssm install flanneld C:\flannel\flanneld.exe
nssm set flanneld AppParameters --kubeconfig-file=c:\k\config --iface=<ManagementIP> --ip-masq=1 --kube-subnet-mgr=1
nssm set flanneld AppEnvironmentExtra NODE_NAME=<hostname>
nssm set flanneld AppDirectory C:\flannel
nssm start flanneld
```

### <a name="register-kubeletexe"></a>注册 kubelet.exe
```
nssm install kubelet C:\k\kubelet.exe
nssm set kubelet AppParameters --hostname-override=<hostname> --v=6 --pod-infra-container-image=kubeletwin/pause --resolv-conf="" --allow-privileged=true --enable-debugging-handlers --cluster-dns=<DNS-service-IP> --cluster-domain=cluster.local --kubeconfig=c:\k\config --hairpin-mode=promiscuous-bridge --image-pull-progress-deadline=20m --cgroups-per-qos=false  --log-dir=<log directory> --logtostderr=false --enforce-node-allocatable="" --network-plugin=cni --cni-bin-dir=c:\k\cni --cni-conf-dir=c:\k\cni\config
nssm set kubelet AppDirectory C:\k
nssm start kubelet
```

### <a name="register-kube-proxyexe-l2bridge--host-gw"></a>注册 kube proxy.exe (l2bridge / 主机网关)
```
nssm install kube-proxy C:\k\kube-proxy.exe
nssm set kube-proxy AppDirectory c:\k
nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --hostname-override=<hostname>--kubeconfig=c:\k\config --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm.exe set kube-proxy AppEnvironmentExtra KUBE_NETWORK=cbr0
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```

### <a name="register-kube-proxyexe-overlay--vxlan"></a>注册 kube proxy.exe (覆盖 / vxlan)
```
PS C:\k> nssm install kube-proxy C:\k\kube-proxy.exe
PS C:\k> nssm set kube-proxy AppDirectory c:\k
PS C:\k> nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --feature-gates="WinOverlay=true" --hostname-override=<hostname> --kubeconfig=c:\k\config --network-name=vxlan0 --source-vip=<source-vip> --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```