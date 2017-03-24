---
title: "群模式入门"
description: "初始化群群集，创建一个覆盖网络，以及将服务连接到网络。"
keywords: "docker, 容器, 群, 编排"
author: kallie-b
ms.date: 02/9/2017
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 5ceb9626-7c48-4d42-81f8-9c936595ad85
translationtype: Human Translation
ms.sourcegitcommit: f615c6dd268932a2ff99ac12c4e9ffdcf2cc217e
ms.openlocfilehash: ee6053003b31f226d2cfba8566f274ccc19d97ec
ms.lasthandoff: 03/02/2017

---

# 群模式入门 

**重要说明：***目前，群模式和覆盖网络支持仅作为即将发布的 Windows 10 创意者更新的一部分对 [Windows 预览体验成员](https://insider.windows.com/) 可用。针对更多 Windows 平台的支持功能即将发布。*

## 什么是“群模式”？
群模式是一项 Docker 功能，可提供内置容器编排能力（包括 Docker 主机的本机群集和容器工作负载的调度）。 当一组 Docker 主机的 Docker 引擎共同在“群模式”下运行时，这组 Docker 主机就形成了一个“群”群集。 有关群模式的其他背景信息，请参阅 [Docker 的主要文档站点](https://docs.docker.com/engine/swarm/)。

## 管理器节点和工作者节点
一个群由两种容器主机类型构成：*管理器节点*和*工作者节点*。 每一个群均通过管理器节点进行初始化，且用于控制和监视群的所有 Docker CLI 命令均必须从其中一个管理器节点执行。 管理器节点可以视为群状态的“维持者”，它们共同形成一个共识组，可以持续感知在群上运行的服务的状态；它们的任务则是确保群的实际状态始终符合开发人员或管理员定义的预期状态。 

>    **注意：**任何给定的群均可以有多个管理器节点，但始终必须有*至少一个*节点。 

工作者节点由 Docker 群通过管理器节点进行编排。 若要加入群，工作者节点必须使用“加入令牌”，该令牌在初始化群时由管理器节点生成。 工作者节点仅接收和执行来自管理器节点的任务，因此不要求（和拥有）对群状态的感知。

## 群模式系统要求

至少有一个运行 **Windows 10 创意者更新**（对 [Windows 预览体验成员](https://insider.windows.com/) 计划的成员可用）的物理或虚拟计算机系统（要使用群的完整功能，建议至少两个节点）设置为容器主机（请参阅主题 [Windows 10 上的 Windows 容器](https://docs.microsoft.com/en-us/virtualization/windowscontainers/quick-start/quick-start-windows-10) 获取有关如何开始使用 Windows 10 上的 Docker 容器的更多详细信息）

**Docker 引擎 v1.13.0 或更高版本**

打开端口：以下端口必须在每台主机上可用。 在某些系统上，这些端口默认为打开。
- TCP 端口 2377 用于群集管理通信
- TCP 和 UDP 端口 7946 用于节点间通信
- TCP 和 UDP 端口 4789 用于覆盖网络通信

## 初始化群群集
要初始化群，只需从你的其中一个容器主机（将 \<HOSTIPADDRESS\> 替换为主机的本地 IPv4 地址）运行以下命令：

```none
# Initialize a swarm 
C:\> docker swarm init --advertise-addr=<HOSTIPADDRESS> --listen-addr <HOSTIPADDRESS>:2377
```
从给定的容器主机运行此命令时，该主机上的 Docker 引擎开始作为管理器节点在群模式下运行。

## 将节点添加到群

> **注意：**使用群模式和覆盖网络模式功能*不*需要多个节点。 使用一个主机在群模式下运行，便可使用所有群/覆盖功能（即，一个管理器节点，使用 `docker swarm init` 命令置于群模式下）。

### 将工作者添加到群
从管理器节点初始化群后，可以使用另外一个简单的命令将其他主机作为工作者添加到群：

```none
C:\> docker swarm join --token <WORKERJOINTOKEN> <MANAGERIPADDRESS>
```

此时，\<MANAGERIPADDRESS\> 是群管理器节点的本地 IP 地址，\<WORKERJOINTOKEN\> 则是由从管理器节点运行 `docker swarm init` 命令而获得的工作者加入令牌。 要获得加入令牌，也可以在初始化群后，从管理器节点运行以下其中一个命令：

```none
# Get the full command required to join a worker node to the swarm
C:\> docker swarm join-token worker

# Get only the join-token needed to join a worker node to the swarm
C:\> docker swarm join-token worker -q
```

### 将管理器添加到群
使用以下命令可以将其他管理器节点添加到群群集：

```none
C:\> docker swarm join --token <MANAGERJOINTOKEN> <MANAGERIPADDRESS>
```

同样，\<MANAGERIPADDRESS\> 也是群管理器节点的本地 IP 地址。 加入令牌 \<MANAGERJOINTOKEN\> 是用于群的*管理器*加入令牌，可以通过从现有管理器节点运行以下其中一个命令的方式获得：

```none
# Get the full command required to join a **manager** node to the swarm
C:\> docker swarm join-token manager

# Get only the join-token needed to join a **manager** node to the swarm
C:\> docker swarm join-token manager -q
```

## 创建覆盖网络

群群集配置完成后，可以在群上创建覆盖网络。 从群管理器节点运行以下命令可以创建覆盖网络：

```none
# Create an overlay network 
C:\> docker network create --driver=overlay <NETWORKNAME>
```

此时，\<NETWORKNAME\> 是你要提供给网络的名称。

## 将服务部署到群
创建覆盖网络后，可以创建服务并将服务连接到网络。 使用以下语法创建网络：

```none
# Deploy a service to the swarm
C:\> docker service create --name=<SERVICENAME> --endpoint-mode dnsrr --network=<NETWORKNAME> <CONTAINERIMAGE> [COMMAND] [ARGS…]
```

此时，\<SERVICENAME\> 是你要提供给服务的名称，你将使用该名称通过服务发现（使用 Docker 的本机 DNS 服务器）引用服务。 \<NETWORKNAME\> 是你想要将该服务连接到的网络的名称（例如，“myOverlayNet”）。 \<CONTAINERIMAGE\> 是将对服务进行定义的容器映像的名称。

> **注意：**要指定 DNS 轮循机制将网络流量均衡分配到服务容器终结点时要使用的 Docker 引擎，需要使用此命令的第二个参数，`--endpoint-mode dnsrr`。 目前，DNS 轮循是在 Windows 上受支持的唯一一个负载平衡策略。用于 Windows docker 主机的[路由网](https://docs.docker.com/engine/swarm/ingress/) 暂时还不受支持，但将很快发布。 寻找替代性负载平衡策略的用户现在可以设置一个外部负载平衡器（例如，NGINX），并使用群的[发布端口模式](https://docs.docker.com/engine/reference/commandline/service_create/#/publish-service-ports-externally-to-the-swarm--p---publish) 公开要进行负载平衡的容器主机端口。

## 缩放服务
将服务部署到群群集后，组成该服务的容器实例会部署到整个群集中。 默认情况下，一项服务由一个容器实例（服务的“副本”或“任务”）支持。 但是，通过对 `docker service create` 命令使用 `--replicas` 选项或在创建服务后缩放服务，也可以创建具有多个任务的服务。

服务可伸缩性是 Docker 群的一个重要优势，这一优势也可以通过一个 Docker 命令加以利用：

```none
C:\> docker service scale <SERVICENAME>=<REPLICAS>
```

此时，\<SERVICENAME\> 是被缩放的服务的名称，\<REPLICAS\> 是服务要缩放到的任务或容器实例数量。

## 查看群状态

可运用几个命令查看群和在群上运行的服务的状态。

### 列出群节点
使用以下命令查看当前加入群的节点的列表，包括每个节点的状态信息。 必须从**管理器节点**运行此命令。

```none
C:\ docker node ls
```

在此命令的输出中，你将注意到其中一个节点带有星号 (*) 标记；星号仅指示当前节点，即从中运行 `docker node ls` 命令的节点。

### 列出网络
使用以下命令查看给定节点上存在的网络的列表。 要查看覆盖网络，必须从在群模式下运行的**管理器节点**运行此命令。

```none
C:\ docker network ls
```

### 列出服务
使用以下命令查看当前在群上运行的服务的列表，包括其状态信息。

```none
C:\ docker service ls
```

### 列出定义服务的容器实例
使用以下命令查看有关为给定服务运行的容器实例的详细信息。 此命令的输出包括各容器在其上运行的 ID 和节点，以及容器的状态信息。  

```none
C:\ docker service ps <SERVICENAME>
```

## 限制
当前，Windows 上的群模式有以下限制：
- 已知缺陷：覆盖和群模式目前仅在连接以太网的容器主机上受支持；**它们无法在连接 WLAN 的主机上工作。** 我们正在修复此问题。
- 不支持数据平面加密（即，使用 `--opt encrypted` 选项的容器间通信）
- 暂时不支持用于 Windows docker 主机的[路由网](https://docs.docker.com/engine/swarm/ingress/)，但即将发布。 寻找替代性负载平衡策略的用户现在可以设置一个外部负载平衡器（例如，NGINX），并使用群的[发布端口模式](https://docs.docker.com/engine/reference/commandline/service_create/#/publish-service-ports-externally-to-the-swarm--p---publish) 公开要进行负载平衡的容器主机端口。  



