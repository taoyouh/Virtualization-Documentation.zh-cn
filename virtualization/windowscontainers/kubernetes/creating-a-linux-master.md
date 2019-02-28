---
title: 从头开始创建 Kubernetes 主机
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: 开始创建 Kubernetes 群集主机。
keywords: kubernetes，1.13，主机 linux
ms.openlocfilehash: 8a3fb073616d115ab84e6cc36f0fb6cedbcf1f7d
ms.sourcegitcommit: 41318edba7459a9f9eeb182bf8519aac0996a7f1
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/28/2019
ms.locfileid: "9120425"
---
# <a name="creating-a-kubernetes-master"></a>开始创建 Kubernetes 主机 #
> [!NOTE]
> 本指南已验证 Kubernetes v1.13 上。 由于易变性的 Kubernetes 版本到版本，本部分可能会使并持有适用于适用于所有的未来版本的假设。 找不到初始化使用 kubeadm Kubernetes 主机的正式文档[在此处](https://kubernetes.io/docs/setup/independent/install-kubeadm/)。 只需除此之外启用[混合操作系统计划部分](#enable-mixed-os-scheduling)。

> [!NOTE]  
> 近期已更新 Linux 计算机所需遵循沿;Kubernetes 主像[kube dns](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)、 [kube 计划程序](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)，以及[kube apiserver](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/)不已移植到 Windows 尚未的资源。 

> [!tip]
> 面向**Ubuntu 16.04**针对定制的 Linux 说明。 其他认证运行 Kubernetes 的 linux 还应提供等效命令可以替代。 它们将还兼容成功与 Windows。


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
若要能够使用容器，你需要一个容器引擎，如 Docker。 若要获取最新版本，你可以使用 Docker 安装[这些说明](https://docs.docker.com/install/linux/docker-ce/ubuntu/)。 你可以验证该 docker 是否已正确安装通过运行`hello-world`容器：

```bash
docker run hello-world
```

### <a name="install-kubeadm"></a>安装 kubeadm ###
下载`kubeadm`为 Linux 分发的二进制文件并初始化群集。

> [!Important]  
> 具体取决于你 Linux 分发，你可能需要替换`kubernetes-xenial`下面与正确[代号](https://wiki.ubuntu.com/Releases)。

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl 
```

### <a name="prepare-the-master-node"></a>准备主节点 ###
在 Linux 上的 Kubernetes 需要交换空间将其关闭：

```bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a 
```

### <a name="initialize-master"></a>初始化大纲 ###
记下你群集子网 (例如 10.244.0.0/16) 和服务子网 (例如 10.96.0.0/12) 并初始化使用 kubeadm 你大纲：

```bash
kubeadm init --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12
```

这可能需要几分钟。 完成后，你应该看到像你大纲已初始化此确认屏幕：

![文本](media/kubeadm-init.png)

> [!tip]
> 你应注意此 kubeadm 加入命令。 应 kubeadm 令牌过期，你可以使用`kubeadm token create --print-join-command`若要创建一个新的令牌。

> [!tip]
> 如果你拥有所需的 Kubernetes 版本你想要使用，你可以传递`--kubernetes-version`kubeadm 标志。

我们不尚未完成。 若要使用`kubectl`作为常规用户，运行以下__**unelevated 的非根用户 shell 中**__

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
现在可以使用 kubectl 编辑或查看有关群集的信息。

### <a name="enable-mixed-os-scheduling"></a>启用混合操作系统计划 ###
默认情况下，某些 Kubernetes 资源是写入它们正在计划的所有节点上的方式。 但是，在 multi-OS 环境中，我们不希望 Linux 资源干扰或双计划到 Windows 节点，反之亦然。 出于此原因，我们需要应用[NodeSelector](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector)标签。 

在这里，我们将修补程序 linux kube 代理[DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)仅面向 Linux。

首先，让我们创建一个目录来存储.yaml 清单文件：
```bash
mkdir -p kube/yaml && cd kube/yaml
```

确认更新策略的`kube-proxy`DaemonSet 设置为[RollingUpdate](https://kubernetes.io/docs/tasks/manage-daemon/update-daemon-set/):

```bash
kubectl get ds/kube-proxy -o go-template='{{.spec.updateStrategy.type}}{{"\n"}}' --namespace=kube-system
```

接下来，通过下载[此 nodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml)修补 DaemonSet 并将其仅面向 Linux 应用：

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml
kubectl patch ds/kube-proxy --patch "$(cat node-selector-patch.yml)" -n=kube-system
```

一旦成功，你应该看到"节点选择器"的`kube-proxy`和任何其他 DaemonSets 设置为 `beta.kubernetes.io/os=linux`

```bash
kubectl get ds -n kube-system
```

![文本](media/kube-proxy-ds.png)

### <a name="collect-cluster-information"></a>收集群集信息 ###
若要成功加入未来节点到主服务器，你应跟踪的以下信息：
  1. `kubeadm join` 从输出 （在[此处](#initialize-master)） 的命令
    * 示例： `kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>`
  2. 群集子网期间定义`kubeadm init`（在[此处](#initialize-master)）
    * 示例： `10.244.0.0/16`
  3. 定义期间的服务子网`kubeadm init`（在[此处](#initialize-master)）
    * 示例： `10.96.0.0/12`
    * 此外可以通过使用找到 `kubectl cluster-info dump | grep -i service-cluster-ip-range`
  4. Kube dns 服务 IP 
    * 示例： `10.96.0.10`
    * 可以在"群集 IP"字段中使用找到 `kubectl get svc/kube-dns -n kube-system`
  5. Kubernetes`config`生成后文件`kubeadm init`（在[此处](#initialize-master)）。 如果你遵循的说明，这可以在以下路径找到：
    * `/etc/kubernetes/admin.conf`
    * `$HOME/.kube/config`

## <a name="verifying-the-master"></a>验证主机 ##
几分钟后，系统应处于以下状态：

  - 在`kubectl get pods -n kube-system`，将有关[Kubernetes 主组件](https://kubernetes.io/docs/concepts/overview/components/#master-components)中的 pod`Running`状态。
  - 调用`kubectl cluster-info`将显示有关 Kubernetes 主 API 服务器以及 DNS 加载项的信息。
  
> [!tip]
> 由于 kubeadm 不设置网络，DNS pod 仍可以在`ContainerCreating`或`Pending`状态。 他们会切换到`Running`状态后[选择的网络解决方案](./network-topologies.md)。

## <a name="next-steps"></a>后续步骤 ## 
在此部分中，我们将介绍如何设置使用 kubeadm Kubernetes 主机。 现在，你可以随时步骤 3:

> [!div class="nextstepaction"]
> [选择网络解决方案](./network-topologies.md)