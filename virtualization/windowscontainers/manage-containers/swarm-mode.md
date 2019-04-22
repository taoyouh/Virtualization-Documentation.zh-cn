---
title: 群模式入门
description: 初始化群群集，创建一个覆盖网络，以及将服务连接到网络。
keywords: docker, 容器, 群, 编排
author: kallie-b
ms.date: 02/9/2017
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 5ceb9626-7c48-4d42-81f8-9c936595ad85
ms.openlocfilehash: d3543d9e6f9e28278ab9f64fb1f4fa19d1507b08
ms.sourcegitcommit: a5ff22c205149dac4fc05325ef3232089826f1ef
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/22/2019
ms.locfileid: "9380441"
---
# <a name="getting-started-with-swarm-mode"></a>群模式入门 

## <a name="what-is-swarm-mode"></a>什么是“群模式”？
群模式是一项 Docker 功能，可提供内置容器编排能力（包括 Docker 主机的本机群集和容器工作负载的调度）。 当一组 Docker 主机的 Docker 引擎共同在“群模式”下运行时，这组 Docker 主机就形成了一个“群”群集。 有关群模式的其他背景信息，请参阅 [Docker 的主要文档站点](https://docs.docker.com/engine/swarm/)。

## <a name="manager-nodes-and-worker-nodes"></a>管理器节点和工作者节点
一个群由两种容器主机类型构成：*管理器节点*和*工作者节点*。 每一个群均通过管理器节点进行初始化，且用于控制和监视群的所有 Docker CLI 命令均必须从其中一个管理器节点执行。 管理器节点可以视为群状态的“维持者”，它们共同形成一个共识组，可以持续感知在群上运行的服务的状态；它们的任务则是确保群的实际状态始终符合开发人员或管理员定义的预期状态。 

>[!NOTE]
>任何给定的群均可以有多个管理器节点，但始终必须有*至少一个*。 

工作者节点由 Docker 群通过管理器节点进行编排。 若要加入群，工作者节点必须使用“加入令牌”，该令牌在初始化群时由管理器节点生成。 工作者节点仅接收和执行来自管理器节点的任务，因此不要求（和拥有）对群状态的感知。

## <a name="swarm-mode-system-requirements"></a>群模式系统要求

（若要使用群的完整功能至少两个节点建议） 运行**Windows 10 创意者更新**或**Windows Server 2016** *的最新 updates\ * 所有*，设置为至少一个物理或虚拟计算机系统容器主机 （请参阅主题中， [Windows 10 上的 Windows 容器](https://docs.microsoft.com/en-us/virtualization/windowscontainers/quick-start/quick-start-windows-10)或[Windows Server 上的 Windows 容器](https://docs.microsoft.com/en-us/virtualization/windowscontainers/quick-start/quick-start-windows-server)的更多有关如何开始使用 Windows 10 上的 Docker 容器的详细信息）。

\***注意**：Windows Server 2016 上的 Docker 群需要 [KB4015217](https://support.microsoft.com/en-us/help/4015217/windows-10-update-kb4015217)

**Docker 引擎 v1.13.0 或更高版本**

打开端口：以下端口必须在每台主机上可用。 在某些系统上，这些端口默认为打开。
- TCP 端口 2377 用于群集管理通信
- TCP 和 UDP 端口 7946 用于节点间通信
- UDP 端口 4789 用于覆盖网络通信

## <a name="initializing-a-swarm-cluster"></a>初始化群群集

要初始化群，只需从你的其中一个容器主机（将 \<HOSTIPADDRESS\> 替换为主机的本地 IPv4 地址）运行以下命令：

```
# Initialize a swarm 
C:\> docker swarm init --advertise-addr=<HOSTIPADDRESS> --listen-addr <HOSTIPADDRESS>:2377
```
从给定的容器主机运行此命令时，该主机上的 Docker 引擎开始作为管理器节点在群模式下运行。

## <a name="adding-nodes-to-a-swarm"></a>将节点添加到群

多个节点都*不*需要利用群模式和覆盖网络模式功能。 使用一个主机在群模式下运行，便可使用所有群/覆盖功能（即，一个管理器节点，使用 `docker swarm init` 命令置于群模式下）。

### <a name="adding-workers-to-a-swarm"></a>将工作者添加到群

从管理器节点初始化群后，可以使用另外一个简单的命令将其他主机作为工作者添加到群：

```
C:\> docker swarm join --token <WORKERJOINTOKEN> <MANAGERIPADDRESS>
```

此时，\<MANAGERIPADDRESS\> 是群管理器节点的本地 IP 地址，\<WORKERJOINTOKEN\> 则是由从管理器节点运行 `docker swarm init` 命令而获得的工作者加入令牌。 要获得加入令牌，也可以在初始化群后，从管理器节点运行以下其中一个命令：

```
# Get the full command required to join a worker node to the swarm
C:\> docker swarm join-token worker

# Get only the join-token needed to join a worker node to the swarm
C:\> docker swarm join-token worker -q
```

### <a name="adding-managers-to-a-swarm"></a>将管理器添加到群
使用以下命令可以将其他管理器节点添加到群群集：

```
C:\> docker swarm join --token <MANAGERJOINTOKEN> <MANAGERIPADDRESS>
```

同样，\<MANAGERIPADDRESS\> 也是群管理器节点的本地 IP 地址。 加入令牌 \<MANAGERJOINTOKEN\> 是用于群的*管理器*加入令牌，可以通过从现有管理器节点运行以下其中一个命令的方式获得：

```
# Get the full command required to join a **manager** node to the swarm
C:\> docker swarm join-token manager

# Get only the join-token needed to join a **manager** node to the swarm
C:\> docker swarm join-token manager -q
```

## <a name="creating-an-overlay-network"></a>创建覆盖网络

群群集配置完成后，可以在群上创建覆盖网络。 从群管理器节点运行以下命令可以创建覆盖网络：

```
# Create an overlay network 
C:\> docker network create --driver=overlay <NETWORKNAME>
```

此时，\<NETWORKNAME\> 是你要提供给网络的名称。

## <a name="deploying-services-to-a-swarm"></a>将服务部署到群
创建覆盖网络后，可以创建服务并将服务连接到网络。 使用以下语法创建服务：

```
# Deploy a service to the swarm
C:\> docker service create --name=<SERVICENAME> --endpoint-mode dnsrr --network=<NETWORKNAME> <CONTAINERIMAGE> [COMMAND] [ARGS…]
```

此时，\<SERVICENAME\> 是你要提供给服务的名称，你将使用该名称通过服务发现（使用 Docker 的本机 DNS 服务器）引用服务。 \<NETWORKNAME\> 是你想要将该服务连接到的网络的名称（例如，“myOverlayNet”）。 \<CONTAINERIMAGE\> 是将对服务进行定义的容器映像的名称。

>[!NOTE]
>此命令的第二个参数`--endpoint-mode dnsrr`，需要为 Docker 引擎指定 DNS 轮循将用于网络流量均衡服务容器终结点。 目前，DNS 轮循是在 Windows 上受支持的唯一一个负载平衡策略。用于 Windows docker 主机的[路由网](https://docs.docker.com/engine/swarm/ingress/) 暂时还不受支持，但将很快发布。 寻找替代性负载平衡策略的用户现在可以设置一个外部负载平衡器（例如，NGINX），并使用群的[发布端口模式](https://docs.docker.com/engine/reference/commandline/service_create/#/publish-service-ports-externally-to-the-swarm--p---publish) 公开要进行负载平衡的容器主机端口。

## <a name="scaling-a-service"></a>缩放服务
将服务部署到群群集后，组成该服务的容器实例会部署到整个群集中。 默认情况下，一项服务由一个容器实例（服务的“副本”或“任务”）支持。 但是，通过对 `docker service create` 命令使用 `--replicas` 选项或在创建服务后缩放服务，也可以创建具有多个任务的服务。

服务可伸缩性是 Docker 群的一个重要优势，这一优势也可以通过一个 Docker 命令加以利用：

```
C:\> docker service scale <SERVICENAME>=<REPLICAS>
```

此时，\<SERVICENAME\> 是被缩放的服务的名称，\<REPLICAS\> 是服务要缩放到的任务或容器实例数量。


## <a name="viewing-the-swarm-state"></a>查看群状态

可运用几个命令查看群和在群上运行的服务的状态。

### <a name="list-swarm-nodes"></a>列出群节点
使用以下命令查看当前加入群的节点的列表，包括每个节点的状态信息。 必须从**管理器节点**运行此命令。

```
C:\> docker node ls
```

在此命令的输出中，你将注意到其中一个节点带有星号 (*) 标记；星号仅指示当前节点，即从中运行 `docker node ls` 命令的节点。

### <a name="list-networks"></a>列出网络
使用以下命令查看给定节点上存在的网络的列表。 要查看覆盖网络，必须从在群模式下运行的**管理器节点**运行此命令。

```
C:\> docker network ls
```

### <a name="list-services"></a>列出服务
使用以下命令查看当前在群上运行的服务的列表，包括其状态信息。

```
C:\> docker service ls
```

### <a name="list-the-container-instances-that-define-a-service"></a>列出定义服务的容器实例
使用以下命令查看有关为给定服务运行的容器实例的详细信息。 此命令的输出包括各容器在其上运行的 ID 和节点，以及容器的状态信息。  

```
C:\> docker service ps <SERVICENAME>
```
## <a name="linuxwindows-mixed-os-clusters"></a>Linux+Windows 混合操作系统群集

最近，我们团队的一位成员发布了一个简短的三部分演示，介绍如何使用 Docker 群设置 Windows+Linux 混合操作系统应用程序。 如果你不熟悉 Docker 群，或者不熟悉如何使用它来运行混合操作系统应用程序，那么非常适合从这里入门。 赶快去看看：
- [使用 Docker 群运行 Windows+Linux 容器化应用程序（第 1 部分/共 3 部分）](https://www.youtube.com/watch?v=ZfMV5JmkWCY&t=170s)
- [使用 Docker 群运行 Windows+Linux 容器化应用程序（第 2 部分/共 3 部分）](https://www.youtube.com/watch?v=VbzwKbcC_Mg&t=406s)
- [使用 Docker 群运行 Windows+Linux 容器化应用程序（第 3 部分/共 3 部分）](https://www.youtube.com/watch?v=I9oDD78E_1E&t=354s)

### <a name="initializing-a-linuxwindows-mixed-os-cluster"></a>初始化 Linux+Windows 混合操作系统群集
初始化混合操作系统群群集非常简单 -- 只要防火墙规则配置正确，并且你的主机有权访问另一个主机，那么只需运行标准的 `docker swarm join` 命令，即可将 Linux 主机添加到群：
```
C:\> docker swarm join --token <JOINTOKEN> <MANAGERIPADDRESS>
```
你也可以使用初始化 Windows 主机中的群时运行的相同命令来初始化 Linux 主机中的群：
```
# Initialize a swarm 
C:\> docker swarm init --advertise-addr=<HOSTIPADDRESS> --listen-addr <HOSTIPADDRESS>:2377
```

### <a name="adding-labels-to-swarm-nodes"></a>将标签添加到群节点
为了对混合操作系统群群集启动 Docker 服务，必须采用某种方法来区分哪些群节点运行的操作系统是该服务所面向的操作系统，哪些不是。 [Docker 对象标签](https://docs.docker.com/engine/userguide/labels-custom-metadata/)提供了一种有用的方法来标记节点，以便可以创建服务并将其配置为仅在与其操作系统匹配的节点上运行。 

>[!NOTE]
>[Docker 对象标签](https://docs.docker.com/engine/userguide/labels-custom-metadata/)可用于将元数据应用于各种 Docker 对象 （包括容器映像、 容器、 卷和网络），用于各种用途 （例如标签可用于单独的前端和后端组件应用程序，通过允许前端微服务，以在 secheduled 仅在前端标记节点和后端 mircoservices 以仅在后端标记的节点上计划）。 在本例中，我们使用节点上的标签以区分 Windows 操作系统节点和 Linux 操作系统节点。

若要标记现有的群节点，请使用以下语法：

```
C:\> docker node update --label-add <LABELNAME>=<LABELVALUE> <NODENAME>
```

此时，`<LABELNAME>` 是你正在创建的标签的名称 - 例如，在本例中，我们按节点操作系统来区分节点，因此标签的逻辑名称可能是“os”。 `<LABELVALUE>` 是标签的值 - 在本例中，你可以选择使用值“windows”和“linux”。 （当然，你可以为标签和标签值选择任何名称，前提是保持一致）。 `<NODENAME>` 是你要标记的节点的名称；你可以通过运行 `docker node ls` 使自己想起节点的名称。 

**例如**，如果你的群集中有四个群节点，包括两个 Windows 节点和两个 Linux 节点，那么标签更新命令可能如下所示：

```
# Example -- labeling 2 Windows nodes and 2 Linux nodes in a cluster...
C:\> docker node update --label-add os=windows Windows-SwarmMaster
C:\> docker node update --label-add os=windows Windows-SwarmWorker1
C:\> docker node update --label-add os=linux Linux-SwarmNode1
C:\> docker node update --label-add os=linux Linux-SwarmNode2
```

### <a name="deploying-services-to-a-mixed-os-swarm"></a>将服务部署到混合操作系统群
使用群节点标签将服务部署到群集很简单；只需对 [`docker service create`](https://docs.docker.com/engine/reference/commandline/service_create/) 命令使用 `--constraint` 选项：

```
# Deploy a service with swarm node constraint
C:\> docker service create --name=<SERVICENAME> --endpoint-mode dnsrr --network=<NETWORKNAME> --constraint node.labels.<LABELNAME>=<LABELVALUE> <CONTAINERIMAGE> [COMMAND] [ARGS…]
```

例如，使用以上示例中的标签和标签值命名法时，一组服务创建命令（一个用于创建基于 Windows 的服务，一个用于创建基于 Linux 的服务）可能如下所示：

```
# Example -- using the 'os' label and 'windows'/'linux' label values, service creation commands might look like these...

# A Windows service
C:\> docker service create --name=win_s1 --endpoint-mode dnsrr --network testoverlay --constraint 'node.labels.os==windows' microsoft/nanoserver:latest powershell -command { sleep 3600 }

# A Linux service
C:\> docker service create --name=linux_s1 --endpoint-mode dnsrr --network testoverlay --constraint 'node.labels.os==linux' redis
```

## <a name="limitations"></a>限制
当前，Windows 上的群模式有以下限制：
- 不支持数据平面加密（即，使用 `--opt encrypted` 选项的容器间通信）
- 暂时不支持用于 Windows docker 主机的[路由网](https://docs.docker.com/engine/swarm/ingress/)，但即将发布。 寻找替代性负载平衡策略的用户现在可以设置一个外部负载平衡器（例如，NGINX），并使用群的[发布端口模式](https://docs.docker.com/engine/reference/commandline/service_create/#/publish-service-ports-externally-to-the-swarm--p---publish) 公开要进行负载平衡的容器主机端口。 下面是有关这方面的更多详细信息。

## <a name="publish-ports-for-service-endpoints"></a>发布服务终结点的端口
Windows 尚不支持 Docker 群的[路由网](https://docs.docker.com/engine/swarm/ingress/)功能，但是寻求发布服务终结点端口的用户现在可以使用发布端口模式来执行此操作。 

若要为定义服务的每个任务/容器终结点发布主机端口，请对 `docker service create` 命令使用 `--publish mode=host,target=<CONTAINERPORT>` 参数：

```
# Create a service for which tasks are exposed via host port
C:\ > docker service create --name=<SERVICENAME> --publish mode=host,target=<CONTAINERPORT> --endpoint-mode dnsrr --network=<NETWORKNAME> <CONTAINERIMAGE> [COMMAND] [ARGS…]
```

例如，以下命令将创建服务“s1”，并且将通过容器端口 80 和随机选择的主机端口对此服务公开每个任务。

```
C:\ > docker service create --name=s1 --publish mode=host,target=80 --endpoint-mode dnsrr web_1 powershell -command {echo sleep; sleep 360000;}
```

使用发布端口模式创建服务后，可以查询该服务以查看每个服务任务的端口映射：

```
C:\ > docker service ps <SERVICENAME>
```
上述命令将返回有关（跨所有群主机）为服务运行的每个容器实例的详细信息。 一列输出（“端口”列）将包括 \<HOSTPORT\>->\<CONTAINERPORT\>/tcp 格式的每个主机的端口信息。 每个容器实例的 \<HOSTPORT\> 值都将不同，因为每个容器都是在其自己的主机端口上发布的。


## <a name="tips--insights"></a>提示和见解 

#### *<a name="existing-transparent-network-can-block-swarm-initializationoverlay-network-creation"></a>现有的透明网络可以阻止群初始化/创建覆盖网络* 
在 Windows 上，覆盖和透明网络驱动程序都需要一个要绑定到（虚拟）主机网络适配器的外部 vSwitch。 创建覆盖网络时，即会创建一个新的交换机，然后将其连接到开放的网络适配器。 透明网络模式也使用主机网络适配器。 同时，任何给定的网络适配器一次只能绑定到一个交换机 - 如果主机只有一个网络适配器，那么它一次只能连接到一个外部 vSwitch，无论该 vSwitch 是用于覆盖网络还是用于透明网络都不例外。 

因此，如果容器主机只有一个网络适配器，则可能会遇到透明网络阻止创建覆盖网络（或覆盖网络阻止创建透明网络）的问题，因为透明网络当前占用了主机的唯一虚拟网络接口。

以下两种方法可以解决此问题：
- *选项 1 - 删除现有透明网络：* 在初始化群之前，确保容器主机上没有现有透明网络。 删除透明网络，以确保主机上有要用于创建覆盖网络的可用虚拟网络适配器。
- *选项 2 - 在主机上创建其他（虚拟）网络适配器：* 你可以在主机上创建要用于创建覆盖网络的其他网络适配器，而不是删除主机上的任何透明网络。 若要执行此操作，只需创建新的外部网络适配器（使用 PowerShell 或 Hyper-V 管理器）；部署新接口后，初始化群时，主机网络服务 (HNS) 将在你的主机上自动识别它，并使用它来绑定外部 vSwitch 以创建覆盖网络。



