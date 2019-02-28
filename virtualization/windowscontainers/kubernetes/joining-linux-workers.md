---
title: 加入 Linux 节点
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: 将 Linux 节点加入到 Kubernetes 群集与 v1.13。
keywords: kubernetes，1.13，windows，入门
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: c32cc300fd97eb53605e2f51e6a83e5889747561
ms.sourcegitcommit: 41318edba7459a9f9eeb182bf8519aac0996a7f1
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/28/2019
ms.locfileid: "9120465"
---
# <a name="joining-linux-nodes-to-a-cluster"></a>Linux 节点加入群集

[设置 Kubernetes 主节点](creating-a-linux-master.md)并[选择所需的网络解决方案](network-topologies.md)后，你就可以将 Linux 节点加入到群集。 这在加入之前需要某些[Linux 节点上的准备](joining-linux-workers.md#preparing-a-linux-node)。
> [!tip]
> 面向**Ubuntu 16.04**针对定制的 Linux 说明。 其他认证运行 Kubernetes 的 linux 还应提供等效命令可以替代。 它们将还兼容成功与 Windows。

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

若要能够使用容器，你需要一个容器引擎，如 Docker。 若要获取最新版本，你可以使用 Docker 安装[这些说明](https://docs.docker.com/install/linux/docker-ce/ubuntu/)。 你可以验证该 docker 是否已正确安装通过运行`hello-world`图像：

```bash
docker run hello-world
```

## <a name="install-kubeadm"></a>安装 kubeadm

下载`kubeadm`为 Linux 分发的二进制文件并初始化群集。

> [!Important]  
> 具体取决于你 Linux 分发，你可能需要替换`kubernetes-xenial`下面与正确[代号](https://wiki.ubuntu.com/Releases)。

``` bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl 
```

## <a name="disable-swap"></a>禁用交换

在 Linux 上的 Kubernetes 需要交换空间将其关闭：

``` bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a
```

## <a name="flannel-only-enable-bridged-ipv4-traffic-to-iptables"></a>(仅 flannel)启用 iptables 桥接的 IPv4 流量

如果你选择 Flannel 作为网络解决方案建议启用桥接 iptables 链 IPv4 流量。 你应具有[已完成此主机](network-topologies.md#flannel-in-host-gateway-mode)和现在需要重复想要加入的 Linux 节点。 它可以完成使用以下命令：

``` bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

## <a name="copy-kubernetes-certificate"></a>将 Kubernetes 证书复制

**作为常规，（非根） 用户**，请执行以下 3 个步骤。

1. 创建 Kubernetes 的 Linux 目录：

```bash
mkdir -p $HOME/.kube
```

2. 将 Kubernetes 证书文件复制 (`$HOME/.kube/config`)[从主服务器](./creating-a-linux-master.md#collect-cluster-information)并另存为`$HOME/.kube/config`上工作。

> [!tip]
> 如[WinSCP](https://winscp.net/eng/download.php) scp 基于工具可用于节点之间传输的配置文件。

3. 设置复制的配置文件的文件所有权，如下所示：

``` bash
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## <a name="joining-node"></a>加入节点

最后，若要加入群集，请运行`kubeadm join`[我们之前所述下](./creating-a-linux-master.md#initialize-master)**作为根**命令：

```bash
kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>
```

如果成功，你应该会看到与此类似输出：

![文本](./media/node-join.png)

## <a name="next-steps"></a>后续步骤

在此部分中，我们将介绍如何将 Linux 工作者加入到我们 Kubernetes 群集。 现在，你可以随时步骤 6:
> [!div class="nextstepaction"]
> [部署 Kubernetes 资源](./deploying-resources.md)