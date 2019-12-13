---
title: 从头开始创建 Kubernetes 主机
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: 创建 Kubernetes 群集主机。
keywords: kubernetes、1.14、master、linux
ms.openlocfilehash: b1ec23b039ce6f5c42859452ecf3a8a5b35e006c
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910417"
---
# <a name="creating-a-kubernetes-master"></a>创建 Kubernetes Master #
> [!NOTE]
> 本指南已在 Kubernetes v 1.14 上验证。 由于从版本到版本的 Kubernetes 的波动性，此部分可能会假设不会对所有未来版本都提供 true。 可在[此处](https://kubernetes.io/docs/setup/independent/install-kubeadm/)找到使用 Kubeadm 初始化 Kubernetes 的正式文档。 只需在上面启用[混合 OS 计划部分](#enable-mixed-os-scheduling)。

> [!NOTE]  
> 需要使用最近更新的 Linux 计算机进行跟踪;Kubernetes 主资源，如[kube](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)、 [kube](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)和[kube-apiserver](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/)尚未移植到 Windows。 

> [!tip]
> Linux 说明专为**Ubuntu 16.04**定制。 认证为运行 Kubernetes 的其他 Linux 分发还应提供可替代的等效命令。 它们还将成功与 Windows 互操作。


## <a name="initialization-using-kubeadm"></a>使用 kubeadm 初始化 ##
除非显式指定，否则请以**root**身份运行以下命令。

首先，转到提升的根 shell：

```bash
sudo –s
```

请确保您的计算机是最新的：

```bash
apt-get update -y && apt-get upgrade -y
```

### <a name="install-docker"></a>安装 Docker ###
为了能够使用容器，需要一个容器引擎，如 Docker。 若要获取最新版本，你可以使用[这些](https://docs.docker.com/install/linux/docker-ce/ubuntu/)针对 Docker 安装的说明。 可以通过运行 `hello-world` 容器来验证是否正确安装了 docker：

```bash
docker run hello-world
```

### <a name="install-kubeadm"></a>安装 kubeadm ###
下载适用于 Linux 分发版的 `kubeadm` 二进制文件并初始化群集。

> [!Important]  
> 根据 Linux 发行版，可能需要将以下 `kubernetes-xenial` 替换为正确的[codename](https://wiki.ubuntu.com/Releases)。

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl 
```

### <a name="prepare-the-master-node"></a>准备主节点 ###
Linux 上的 Kubernetes 需要关闭交换空间：

```bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a 
```

### <a name="initialize-master"></a>初始化 master ###
记下群集子网（例如 10.244.0.0/16）和服务子网（例如 10.96.0.0/12），并使用 kubeadm 初始化 master：

```bash
kubeadm init --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12
```

这可能需要几分钟。 完成后，应会看到如下所示的屏幕，确认已初始化了 master：

![文本](media/kubeadm-init.png)

> [!tip]
> 应记下此 kubeadm join 命令。 应 kubeadm 令牌过期后，可以使用 `kubeadm token create --print-join-command` 创建新令牌。

> [!tip]
> 如果你有想要使用的所需 Kubernetes 版本，可以将 `--kubernetes-version` 标志传递到 kubeadm。

尚未完成。 若要将 `kubectl` 用作普通用户，请 __**在停用的非根用户 shell 中**运行以下命令：__

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
现在，你可以使用 kubectl 来编辑或查看有关群集的信息。

### <a name="enable-mixed-os-scheduling"></a>启用混合 OS 计划 ###
默认情况下，某些 Kubernetes 资源是以在所有节点上进行计划的方式编写的。 但是，在多 OS 环境中，我们不希望 Linux 资源干扰或计划到 Windows 节点，反之亦然。 出于此原因，我们需要应用[NodeSelector](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector)标签。 

在这方面，我们要将 linux kube-proxy [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)修补为仅面向 linux。

首先，让我们创建一个目录来存储 yaml 清单文件：
```bash
mkdir -p kube/yaml && cd kube/yaml
```

确认 `kube-proxy` DaemonSet 的更新策略设置为[RollingUpdate](https://kubernetes.io/docs/tasks/manage-daemon/update-daemon-set/)：

```bash
kubectl get ds/kube-proxy -o go-template='{{.spec.updateStrategy.type}}{{"\n"}}' --namespace=kube-system
```

接下来，通过下载[此 nodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml)修补 DaemonSet，并将其应用到仅面向 Linux：

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml
kubectl patch ds/kube-proxy --patch "$(cat node-selector-patch.yml)" -n=kube-system
```

成功后，你应该会看到 `kube-proxy` 的 "节点选择器" 和设置为 `beta.kubernetes.io/os=linux` 的任何其他 Daemonset

```bash
kubectl get ds -n kube-system
```

![文本](media/kube-proxy-ds.png)

### <a name="collect-cluster-information"></a>收集群集信息 ###
若要成功地将未来节点加入到主节点，你应跟踪以下信息：
  1. 从输出中 `kubeadm join` 命令（[此处](#initialize-master)）
    * 示例： `kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>`
  2. 在 `kubeadm init` 期间定义的群集子网（[此处](#initialize-master)）
    * 示例： `10.244.0.0/16`
  3. 在 `kubeadm init` 过程中定义的服务子网（[此处](#initialize-master)）
    * 示例： `10.96.0.0/12`
    * 还可以使用 `kubectl cluster-info dump | grep -i service-cluster-ip-range`
  4. Kube-dns 服务 IP 
    * 示例： `10.96.0.10`
    * 可以在 "群集 IP" 字段中找到使用 `kubectl get svc/kube-dns -n kube-system`
  5. 在 `kubeadm init` 后生成的 Kubernetes `config` 文件（[此处](#initialize-master)）。 如果遵循这些说明，可在以下路径中找到此内容：
    * `/etc/kubernetes/admin.conf`
    * `$HOME/.kube/config`

## <a name="verifying-the-master"></a>验证主节点 ##
几分钟后，系统应处于以下状态：

  - 在 `kubectl get pods -n kube-system`下，将在 `Running` 状态下为[Kubernetes 主组件](https://kubernetes.io/docs/concepts/overview/components/#master-components)提供箱。
  - 调用 `kubectl cluster-info` 除了显示 DNS 加载项外，还会显示有关 Kubernetes 主 API 服务器的信息。
  
> [!tip]
> 由于 kubeadm 不设置网络，因此 DNS pod 可能仍处于 `ContainerCreating` 或 `Pending` 状态。 [选择网络解决方案](./network-topologies.md)后，它们将切换到 `Running` 状态。

## <a name="next-steps"></a>后续步骤 ## 
在本部分中，我们介绍了如何使用 kubeadm 设置 Kubernetes 主节点。 现在，你已准备好执行步骤3：

> [!div class="nextstepaction"]
> [选择网络解决方案](./network-topologies.md)