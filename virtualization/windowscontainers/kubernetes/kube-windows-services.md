---
title: 以 Windows 服务的形式运行 Kubernetes
author: daschott
ms.author: daschott
ms.date: 02/12/2019
ms.topic: how-to
ms.prod: containers
description: 如何将 Kubernetes 组件作为 Windows 服务运行。
keywords: kubernetes，1.14，windows，入门
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5c18
ms.openlocfilehash: 470538ad796773252c08c7295c5086d0002a55ce
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192484"
---
# <a name="kubernetes-components-as-windows-services"></a>将组件作为 Windows 服务 Kubernetes

某些用户可能想要配置诸如 flanneld.exe、kubelet.exe、kube-proxy.exe 或其他进程，使其作为 Windows 服务运行。 这带来了额外的容错优势，例如，进程在意外进程或节点崩溃时自动重启。


## <a name="prerequisites"></a>先决条件
1. 已将[nssm.exe](https://nssm.cc/download)下载到 `c:\k` 目录
2. 你已将节点加入群集，然后在你的节点上运行[install.ps1](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/install.ps1)或[start.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1)脚本

## <a name="registering-windows-services"></a>注册 Windows 服务
您可以运行[示例脚本](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/register-svc.ps1)，该脚本使用将注册 `kubelet` 、 `kube-proxy` 和 `flanneld.exe` 以在后台运行为 Windows 服务的 nssm.exe：

```
C:\k\register-svc.ps1 -NetworkMode <Network mode> -ManagementIP <Windows Node IP> -ClusterCIDR <Cluster subnet> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Directory to place logs>
```

# <a name="managementip"></a>[ManagementIP](#tab/ManagementIP)
分配给 Windows 节点的 IP 地址。 您可以使用 `ipconfig` 来查找此。

|  |  |
|---------|---------|
|参数     | `-ManagementIP`        |
|默认值    | n.A.        |


# <a name="networkmode"></a>[NetworkMode](#tab/NetworkMode)
`l2bridge`选择为网络解决方案的网络模式（flannel 主机-gw）或 `overlay` （flannel vxlan [network solution](./network-topologies.md)）。

> [!Important]
> `overlay`网络模式（flannel vxlan）要求 Kubernetes v 1.14 二进制文件或更高版本。

|  |  |
|---------|---------|
|参数     | `-NetworkMode`        |
|默认值    | `l2bridge`        |


# <a name="clustercidr"></a>[ClusterCIDR](#tab/ClusterCIDR)
[群集子网范围](./getting-started-kubernetes-windows.md#cluster-subnet-def)。

|  |  |
|---------|---------|
|参数     | `-ClusterCIDR`        |
|默认值    | `10.244.0.0/16`        |


# <a name="kubednsserviceip"></a>[KubeDnsServiceIP](#tab/KubeDnsServiceIP)
[KUBERNETES DNS 服务 IP](./getting-started-kubernetes-windows.md#kube-dns-def)。

|  |  |
|---------|---------|
|参数     | `-KubeDnsServiceIP`        |
|默认值    | `10.96.0.10`        |


# <a name="logdir"></a>[LogDir](#tab/LogDir)
将 kubelet 和 kube 日志重定向到其各自的输出文件的目录。

|  |  |
|---------|---------|
|参数     | `-LogDir`        |
|默认值    | `C:\k`        |

---


> [!TIP]
> 如果出现问题，请参阅[故障排除部分](./common-problems.md#i-have-problems-running-kubernetes-processes-as-windows-services)

## <a name="manual-approach"></a>手动方法
如果[上面引用的脚本](#registering-windows-services)对您不起作用，则本部分提供了一些*示例命令*，可用于逐步手动注册这些服务。

> [!TIP]
> 请参阅[Kubelet 和 kube-proxy 现在可以作为 Windows 服务运行](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services)，以了解有关如何配置 `kubelet` 和 `kube-proxy` 以通过运行本机 Windows 服务的更多详细信息 `sc` 。

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

### <a name="register-kube-proxyexe-l2bridge--host-gw"></a>注册 kube-proxy.exe （l2bridge/主机-gw）
```
nssm install kube-proxy C:\k\kube-proxy.exe
nssm set kube-proxy AppDirectory c:\k
nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --hostname-override=<hostname>--kubeconfig=c:\k\config --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm.exe set kube-proxy AppEnvironmentExtra KUBE_NETWORK=cbr0
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```

### <a name="register-kube-proxyexe-overlay--vxlan"></a>注册 kube-proxy.exe （覆盖/vxlan）
```
PS C:\k> nssm install kube-proxy C:\k\kube-proxy.exe
PS C:\k> nssm set kube-proxy AppDirectory c:\k
PS C:\k> nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --feature-gates="WinOverlay=true" --hostname-override=<hostname> --kubeconfig=c:\k\config --network-name=vxlan0 --source-vip=<source-vip> --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```