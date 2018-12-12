---
title: 从头开始创建 Kubernetes 主机
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: 开始创建 Kubernetes 群集主机。
keywords: kubernetes，1.12，主机 linux
ms.openlocfilehash: 2bbcf2d382f20d140c73d9b34cf0f13a74debdfa
ms.sourcegitcommit: 8e9252856869135196fd054e3cb417562f851b51
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/08/2018
ms.locfileid: "6178850"
---
# <a name="creating-a-kubernetes-master"></a>开始创建 Kubernetes 主机 #
> [!NOTE]
> 本指南已验证 Kubernetes v1.12 上。 由于易变性的 Kubernetes 版本到版本，本部分可能会使假设未保存适用于适用于所有将来的版本。 找不到官方文档中的针对初始化 Kubernetes 主机使用 kubeadm[下面](https://kubernetes.io/docs/setup/independent/install-kubeadm/)。 只需上启用[混合操作系统计划部分](#enable-mixed-os-scheduling)。

> [!NOTE]  
> 近期已更新的 Linux 计算机所需遵循沿;Kubernetes 主像[kube dns](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)、 [kube 调度程序](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)，以及[kube apiserver](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/)不已移植到 Windows 尚未的资源。 

> [!tip]
> 面向**Ubuntu 16.04**针对定制的 Linux 说明。 其他认证运行 Kubernetes 的 linux 还应提供等效命令可以替代。 它们将还互操作成功与 Windows。


## <a name="initialization-using-kubeadm"></a>使用 kubeadm 初始化 ##
除非明确指定，否则作为**根**运行下面的任何命令。

首先，进入提升的根外壳：

```bash
sudo –s
```

请确保你的计算机保持最新状态：

```bash
apt-get update -y && apt-get upgrade -y
```

### <a name="install-docker"></a>安装 Docker ###
若要能够使用容器，你需要一个容器引擎，如 Docker。 若要获取最新版本，你可以使用 Docker 安装[这些说明](https://docs.docker.com/install/linux/docker-ce/ubuntu/)。 你可以验证该 docker 已正确安装通过运行`hello-world`容器：

```bash
docker run hello-world
```

### <a name="install-kubeadm"></a>安装 kubeadm ###
下载`kubeadm`为 Linux 分发的二进制文件并初始化群集。

> [!Important]  
> 具体取决于你 Linux 分发，你可能需要替换`kubernetes-xenial`下方的正确[代号](https://wiki.ubuntu.com/Releases)。

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl 
```

### <a name="prepare-the-master-node"></a>准备主节点 ###
在 Linux 上的 Kubernetes 需要交换空间来关闭状态：

```bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a 
```

### <a name="initialize-master"></a>初始化大纲 ###
记下你群集子网 (例如 10.244.0.0/16) 和服务子网 (例如 10.96.0.0/12) 并初始化使用 kubeadm 你主机：

```bash
kubeadm init --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12
```

这可能需要几分钟。 完成后，你应该看到像此确认已初始化你的主屏幕：

![文本](media/kubeadm-init.png)

> [!tip]
> 记下 kubeadm 加入命令输出的上方*现在*图片中之前它获取丢失。

> [!tip]
> 如果你拥有所需的 Kubernetes 版本你想要使用，你可以传递`--kubernetes-version`kubeadm 标志。

我们不尚未完成。 若要使用`kubectl`作为常规用户，运行以下__**unelevated、 非根用户 shell 中**__

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
现在可以使用 kubectl 编辑或查看有关你的群集的信息。

### <a name="enable-mixed-os-scheduling"></a>启用混合操作系统计划 ###
默认情况下，它们将在所有节点上计划的方式编写某些 Kubernetes 资源。 但是，在 multi-OS 环境中，我们不希望 Linux 资源会影响或双调度到 Windows 节点，反之亦然。 出于此原因，我们需要将应用[NodeSelector](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector)标签。 

在这里，我们将修补程序 linux kube 代理[DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)仅面向 Linux。

确认更新策略的`kube-proxy`DaemonSet 设置为[RollingUpdate](https://kubernetes.io/docs/tasks/manage-daemon/update-daemon-set/):

```bash
kubectl get ds/kube-proxy -o go-template='{{.spec.updateStrategy.type}}{{"\n"}}' --namespace=kube-system
```

接下来，通过下载[此 nodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml)修补 DaemonSet 并将其仅针对 Linux 应用：

```bash
kubectl patch ds/kube-proxy --patch "$(cat node-selector-patch.yml)" -n=kube-system
```

一旦成功，你应该看到"节点选择器"的`kube-proxy`和任何其他 DaemonSets 设置为 `beta.kubernetes.io/os=linux`

```bash
kubectl get ds -n kube-system
```

![文本](media/kube-proxy-ds.png)

### <a name="collect-cluster-information"></a>收集群集信息 ###
若要成功加入大纲将来的节点，应该记下以下信息：
  1. `kubeadm join` 从输出 （在[此处](#initialize-master)） 的命令
    * 示例： `kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>`
  2. 群集子网期间定义`kubeadm init`（在[此处](#initialize-master)）
    * 示例： `10.244.0.0/16`
  3. 定义期间的服务子网`kubeadm init`（在[此处](#initialize-master)）
    * 示例： `10.96.0.0/12`
    * 此外可以通过使用找到 `kubectl cluster-info dump | grep -i service-cluster-ip-range`
  4. Kube dns 服务 IP 
    * 示例： `10.96.0.10`
    * 可以使用"群集 IP"字段中找到 `kubectl get svc/kube-dns -n kube-system`
  5. Kubernetes`config`生成后文件`kubeadm init`（在[此处](#initialize-master)）。 如果你遵循的说明，这可以在以下路径找到它们：
    * `/etc/kubernetes/admin.conf`
    * `$HOME/.kube/config`

## <a name="verifying-the-master"></a>验证主机 ##
几分钟后，系统应处于以下状态：

  - 在`kubectl get pods -n kube-system`，将为所有[Kubernetes 主组件](https://kubernetes.io/docs/concepts/overview/components/#master-components)中的 pod`Running`状态。
  - 调用`kubectl cluster-info`将显示有关 Kubernetes 主 API 服务器以及 DNS 加载项的信息。

## <a name="next-steps"></a>后续步骤 ## 
在此部分中，我们将介绍如何设置使用 kubeadm 开始创建 Kubernetes 主机。 现在，你已准备好步骤 3:

> [!div class="nextstepaction"]
> [选择网络解决方案](./network-topologies.md)