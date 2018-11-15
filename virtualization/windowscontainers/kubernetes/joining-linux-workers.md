---
title: 加入 Linux 节点
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: 将 Linux 节点加入到 Kubernetes 群集与 v1.12。
keywords: kubernetes，1.12，windows，入门
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 97c12d70db9679dbb85877f0985c6053f95fa500
ms.sourcegitcommit: 4412583b77f3bb4b2ff834c7d3f1bdabac7aafee
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/15/2018
ms.locfileid: "6947976"
---
# <a name="joining-linux-nodes-to-a-cluster"></a>Linux 节点加入到群集

一旦[设置 Kubernetes 主节点](creating-a-linux-master.md)并[选择所需的网络解决方案](network-topologies.md)，可以随时将 Linux 节点加入到群集。 这在加入之前需要某些[Linux 节点上的准备工作](joining-linux-workers.md#preparing-a-linux-node)。
> [!tip]
> 面向**Ubuntu 16.04**针对定制的 Linux 说明进行操作。 其他认证运行 Kubernetes 的 linux 还应提供等效命令可以替代。 它们将还互操作成功与 Windows。

## <a name="preparing-a-linux-node"></a>准备 Linux 节点

> [!NOTE]
> 除非明确指定，否则运行的任何命令在**提升权限，根用户 shell**。

首先，获取到根外壳：

```bash
sudo –s
```

请确保你的计算机保持最新状态：

```bash
apt-get update && apt-get upgrade
```

## <a name="install-docker"></a>安装 Docker

若要能够使用容器，你需要一个容器引擎，如 Docker。 若要获取最新版本，你可以使用 Docker 安装[这些说明](https://docs.docker.com/install/linux/docker-ce/ubuntu/)。 你可以验证该安装 docker 时会正确通过运行`hello-world`图像：

```bash
docker run hello-world
```

## <a name="install-kubeadm"></a>安装 kubeadm

下载`kubeadm`为 Linux 分发的二进制文件并初始化群集。

> [!Important]  
> 根据你 Linux 分发，你可能需要替换`kubernetes-xenial`下面与正确[代号](https://wiki.ubuntu.com/Releases)。

``` bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl 
```

## <a name="disable-swap"></a>禁用交换

在 Linux 上的 Kubernetes 需要交换空间来关闭状态：

``` bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a
```

## <a name="flannel-only-enable-bridged-ipv4-traffic-to-iptables"></a>(仅 flannel)启用 iptables 桥接的 IPv4 流量

如果你选择 Flannel 作为建议启用网络解决方案桥接到 iptables 链 IPv4 流量。 你应具有[已完成此主机的](network-topologies.md#flannel-in-host-gateway-mode)并且现在需要重复想要加入的 Linux 节点。 它可使用以下命令：

``` bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

## <a name="copy-kubernetes-certificate"></a>将 Kubernetes 证书复制

**作为常规，（非根） 用户**，请执行以下 3 个步骤。

1. 创建 Kubernetes 的 Linux 目录：

```bash
mkdir -p $HOME/.kube
```

1. 将 Kubernetes 证书文件复制 (`$HOME/.kube/config`)[从主服务器](./creating-a-linux-master.md#collect-cluster-information)并另存为`$HOME/.kube/config`上工作。

1. 设置复制的配置文件的文件所有权，如下所示：

``` bash
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## <a name="joining-node"></a>加入节点

最后，若要加入群集，请运行`kubeadm join`[我们之前所述向下](./creating-a-linux-master.md#initialize-master)**作为根**命令：

```bash
kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>
```

如果成功，你应该会看到与此类似输出：

![文本](./media/node-join.png)

## <a name="next-steps"></a>后续步骤

在此部分中，我们将介绍如何将 Linux 工作者加入到我们的 Kubernetes 群集。 现在，你已准备好步骤 6:
> [!div class="nextstepaction"]
> [部署 Kubernetes 资源](./deploying-resources.md)