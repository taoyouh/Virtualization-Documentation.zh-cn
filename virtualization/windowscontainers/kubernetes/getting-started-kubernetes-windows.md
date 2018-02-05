---
title: "Windows 上的 Kubernetes"
author: gkudra-msft
ms.author: gekudray
ms.date: 11/16/2017
ms.topic: get-started-article
ms.prod: containers
description: "使用 v1.9 beta 版本将 Windows 节点加入到 Kubernetes 群集中。"
keywords: "kubernetes, 1.9, windows, 入门"
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: f1b832f8a21c034582e157342acf7826fb7b6ea3
ms.sourcegitcommit: b0e21468f880a902df63ea6bc589dfcff1530d6e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/17/2018
---
# <a name="kubernetes-on-windows"></a>Windows 上的 Kubernetes #
借助 Kubernetes 1.9 和 Windows Server [版本 1709](https://docs.microsoft.com/en-us/windows-server/get-started/whats-new-in-windows-server-1709#networking) 的最新版本，用户可以充分利用 Windows 网络中的最新功能：

  - **共享的 Pod 隔离舱**：基础结构和工作线程 Pod 现在共享网络隔离舱（类似于 Linux 命名空间）
  - **终结点优化**：借助隔离舱共享，容器服务需要跟踪的终结点数（至少）仅为之前的二分之一
  - **数据路径优化**：虚拟筛选平台和主机网络服务的改进实现了基于内核的负载平衡


此页面可用作开始将全新 Windows 节点加入到现有基于 Linux 的群集的指南。 若要彻底从头开始，请参阅[此页面](./creating-a-linux-master.md) &mdash; 可用于部署 Kubernetes 群集的多种资源的其中一项 &mdash;，以我们曾用的方式从头开始设置主机。


<a name="definitions"></a>以下是本指南中引用的一些术语的定义：

  - **外部网络**是节点通信所在的网络。
  - <a name="cluster-subnet-def"></a>**群集子网**是可路由的虚拟网络；节点分配有此网络中更小的子网，以供它们的 Pod 使用。
  - **服务子网**是 11.0/16 上不可路由的纯虚拟子网，无论网络拓扑如何，Pod 都使用该子网统一访问服务。 它通过节点上运行的 `kube-proxy` 与可路由的地址空间进行相互转换。


## <a name="what-we-will-accomplish"></a>我们将完成的任务 ##
本指南结束时，我们将完成以下任务：

> [!div class="checklist"]  
> * [网络拓扑](#network-topology)准备就绪。  
> * [Linux 主机](#preparing-the-linux-master)节点配置完毕。  
> * 将 [Windows 工作者节点](#preparing-a-windows-node)连接到上述节点。  
> * [示例 Windows 服务](#running-a-sample-service)部署完毕。  
> * 解决[常见问题和错误](./common-problems.md)。  


## <a name="network-topology"></a>网络拓扑 ##
有多种方法能够使虚拟[群集子网](#cluster-subnet-def)可路由。 你可以：

  - 配置[主机网关模式](./configuring-host-gateway-mode.md)，并设置节点之间的静态下一跃点路由以实现 Pod 间通信。
  - 配置智能柜顶式 (TOR) 交换机以路由子网。
  - 使用第三方覆盖插件，如 [Flannel](https://coreos.com/flannel/docs/latest/kubernetes.html)（Windows 对 Flannel 的支持为 beta 版本）。


## <a name="preparing-the-linux-master"></a>准备 Linux 主机 ##
无论是按照[我们的说明](./creating-a-linux-master.md)进行操作，还是已经具有现有群集，Linux 主机只需 Kubernetes 证书配置。 这可能在 `/etc/kubernetes/admin.conf`、`~/.kube/config` 或其他地方，具体取决于你的设置。


## <a name="preparing-a-windows-node"></a>准备 Windows 节点 ##
> [!Note]  
> Windows 部分中的所有代码段都将在_提升的_ PowerShell 中运行。

Kubernetes 使用 [Docker](https://www.docker.com/) 作为其容器 Orchestrator，因此我们需要安装它。 你可以按照[官方 MSDN 说明](virtualization/windowscontainers/manage-docker/configure-docker-daemon.md#install-docker)、[Docker 说明](https://store.docker.com/editions/enterprise/docker-ee-server-windows)进行操作，或者尝试以下步骤：

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
Install-Package -Name Docker -ProviderName DockerMsftProvider
Restart-Computer -Force
```

[此 Microsoft 存储库](https://github.com/Microsoft/SDN)上有一个脚本集合，这将有助于我们将此节点加入到群集中。 你可以直接在[此处](https://github.com/Microsoft/SDN/archive/master.zip)下载 ZIP 文件。 我们只需要 `Kubernetes/windows` 文件夹，其中的内容应该移到 `C:\k\` 中：

```powershell
wget https://github.com/Microsoft/SDN/archive/master.zip -o master.zip
Expand-Archive master.zip -DestinationPath master
mkdir C:/k/
mv master/SDN-master/Kubernetes/windows/* C:/k/
rm -recurse -force master,master.zip
```

将[之前已识别](#preparing-the-linux-master)的证书文件复制到此新 `C:\k` 目录中。


### <a name="creating-the-pause-image"></a>创建“暂停”映像 ###
现在，`docker` 已安装，我们需要准备“暂停”映像，以供 Kubernetes 准备基础结构 Pod。

```powershell
docker pull microsoft/windowsservercore:1709
docker tag microsoft/windowsservercore:1709 microsoft/windowsservercore:latest
cd C:/k/
docker build -t kubeletwin/pause .
```

> [!Note]  
> 我们将其标记为 `:latest`，因为它将决定稍后将部署的示例服务，尽管实际上这可能不_会_是可用的最新 Windows Server Core 映像。 请务必小心容器映像冲突；没有预期的标记可能会导致不兼容容器映像的 `docker pull`，从而导致[部署问题](./common-problems.md#when-deploying-docker-containers-keep-restarting)。 


### <a name="downloading-binaries"></a>下载二进制文件 ###
在 `pull` 发生的同时，下载 Kubernetes 中的以下客户端二进制文件：

  - `kubectl.exe`
  - `kubelet.exe`
  - `kube-proxy.exe`

你可以通过最新 1.9 版本的 `CHANGELOG.md` 文件中的链接下载这些文件。 截至编写本文时，最新版本为 [1.9.1](https://github.com/kubernetes/kubernetes/releases/tag/v1.9.1)，并且 Windows 二进制文件位于[此处](https://storage.googleapis.com/kubernetes-release/release/v1.9.1/kubernetes-node-windows-amd64.tar.gz)。 使用 [7-Zip](http://www.7-zip.org/) 等工具解压缩存档并将二进制文件放在 `C:\k\` 中。


### <a name="joining-the-cluster"></a>加入群集 ###
节点现在可以加入群集。 在两个*已提升*的单独 PowerShell 窗口中，（按此顺序）运行这些脚本。 第一个脚本中的 `-ClusterCidr` 参数是配置的[群集子网](#cluster-subnet-def)；这里的子网是 `192.168.0.0/16`。

```powershell
./start-kubelet.ps1 -ClusterCidr 192.168.0.0/16
./start-kubeproxy.ps1
```

Windows 节点将即刻显示在 Linux 主机中的 `kubectl get nodes` 下！


### <a name="validating-your-network-topology"></a>验证你的网络拓扑 ###
一些基本测试将验证正确的网络配置：

  - **节点间连接**：主节点与 Windows 工作线程节点之间的 ping 操作应该在两个方向上都成功。

  - **Pod 子网到节点的连接**：虚拟 Pod 接口与节点之间的 ping 操作。 分别在 Linux 和 Windows 上的 `route -n` 和 `ipconfig` 下面查找网关地址，并寻找 `cbr0` 接口。

如果这些基本测试都不起作用，请尝试使用[疑难解答页面](./common-problems.md#network-connectivity)来解决常见问题。


## <a name="running-a-sample-service"></a>运行示例服务 ##
我们将部署一个非常简单的[基于 PowerShell 的 Web 服务](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml)，以确保成功加入群集并正确配置网络。


在 Linux 主机上，下载并运行该服务：

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/WebServer.yaml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

这将创建一项部署和一项服务，然后无限期观察 Pod 以跟踪其状态；观察完后只需按下 `Ctrl+C` 退出 `watch` 命令。


如果一切顺利，你将能够确认：

  - 可以在 Windows 端的 `docker ps` 命令下面看到 4 个容器
  - `curl` （针对 Linux 主机中端口 80 上的 *Pod* IP）可以获取 Web 服务器响应；这表明网络中节点到 Pod 的通信正常。
  - `curl` （针对端口 4444 上的*节点* IP）可以获取 Web 服务器响应；这表明主机到容器的端口映射正确。
  - 可以通过 `docker exec` *在 Pod 之间*（如果拥有多个 Windows 节点，也可以跨主机）执行 ping 操作；这表明 Pod 之间的通信正常。
  - `curl` Linux 主机和个别 Pod 中的虚拟*服务 IP*（在 `kubectl get services` 下显示）。
  - `curl` 带 Kubernetes [默认 DNS 后缀](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services)的*服务名称*，并演示 DNS 功能。

> [!Warning]  
> Windows 节点将无法访问服务 IP。 这是一个[已知限制](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip)。
