---
title: 加入 Linux 节点
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: how-to
ms.prod: containers
description: 使用 v 1.14 将 Linux 节点加入 Kubernetes 群集。
keywords: kubernetes，1.14，windows，入门
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 88ad448796702b3cebe71bb9d0189ea86f72635e
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192774"
---
# <a name="joining-linux-nodes-to-a-cluster"></a>将 Linux 节点加入群集

[设置 Kubernetes 主节点](creating-a-linux-master.md)并选择所需的[网络解决方案](network-topologies.md)后，便可以将 Linux 节点加入群集。 这需要在加入之前[在 Linux 节点上做好一些准备](joining-linux-workers.md#preparing-a-linux-node)。
> [!tip]
> Linux 说明专为**Ubuntu 16.04**定制。 认证为运行 Kubernetes 的其他 Linux 分发还应提供可替代的等效命令。 它们还将成功与 Windows 互操作。

## <a name="preparing-a-linux-node"></a>准备 Linux 节点

> [!NOTE]
> 除非显式指定，否则在**提升的根用户 shell**中运行任何命令。

首先，转到根 shell：

```bash
sudo –s
```

请确保您的计算机是最新的：

```bash
apt-get update && apt-get upgrade
```

## <a name="install-docker"></a>安装 Docker

为了能够使用容器，需要一个容器引擎，如 Docker。 若要获取最新版本，你可以使用[这些](https://docs.docker.com/install/linux/docker-ce/ubuntu/)针对 Docker 安装的说明。 可以通过运行映像来验证是否正确安装了 docker `hello-world` ：

```bash
docker run hello-world
```

## <a name="install-kubeadm"></a>安装 kubeadm

下载 `kubeadm` 适用于 Linux 分发版的二进制文件并初始化群集。

> [!Important]
> 根据 Linux 发行版，可能需要将 `kubernetes-xenial` 下面的替换为正确的[codename](https://wiki.ubuntu.com/Releases)。

``` bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl
```

## <a name="disable-swap"></a>禁用交换

Linux 上的 Kubernetes 需要关闭交换空间：

``` bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a
```

## <a name="flannel-only-enable-bridged-ipv4-traffic-to-iptables"></a>（仅限 Flannel）启用到 iptables 的桥接 IPv4 流量

如果选择 "Flannel" 作为网络解决方案，则建议启用到 iptables 链的桥接 IPv4 流量。 你应该[已为主节点完成此操作](network-topologies.md#flannel-in-host-gateway-mode)，现在需要为要加入的 Linux 节点重复此操作。 可以使用以下命令完成此操作：

``` bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

## <a name="copy-kubernetes-certificate"></a>复制 Kubernetes 证书

对于**常规（非根）用户**，请执行以下3个步骤。

1. 为 Linux 目录创建 Kubernetes：

```bash
mkdir -p $HOME/.kube
```

2. 从 master 复制 Kubernetes 证书文件 `$HOME/.kube/config` （ [from master](./creating-a-linux-master.md#collect-cluster-information) ），并 `$HOME/.kube/config` 在辅助角色上另存为。

> [!tip]
> 可以使用基于 scp 的工具（如[WinSCP](https://winscp.net/eng/download.php) ）在节点之间传输配置文件。

3. 按如下所示设置复制的配置文件的文件所有权：

``` bash
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## <a name="joining-node"></a>联接节点

最后，若要加入群集，请运行 `kubeadm join` [前面记下](./creating-a-linux-master.md#initialize-master)的命令**作为根**：

```bash
kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>
```

如果成功，应看到类似于下面的输出：

![text](./media/node-join.png)

## <a name="next-steps"></a>后续步骤

在本部分中，我们介绍了如何将 Linux 辅助角色加入到 Kubernetes 群集。 现在，你已准备好执行步骤6：
> [!div class="nextstepaction"]
> [部署 Kubernetes 资源](./deploying-resources.md)