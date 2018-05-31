---
title: Windows 容器网络
description: Windows 容器的网络驱动程序和拓扑。
keywords: docker, 容器
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: 3fb36008d9304e6fbdd318c55e012ff3f096d58b
ms.sourcegitcommit: ec186664e76d413d3bf75f2056d5acb556f4205d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/11/2018
ms.locfileid: "1876020"
---
# <a name="windows-container-network-drivers"></a>Windows 容器网络驱动程序  

除了利用 Docker 在 Windows 上创建的默认“nat”网络之外，用户还可以定义自定义容器网络。 可以使用 Docker CLI [`docker network create -d <NETWORK DRIVER TYPE> <NAME>`](https://docs.docker.com/engine/reference/commandline/network_create/) 命令创建用户定义的网络。 在 Windows 上，有以下网络驱动程序类型可用：

- **nat** – 连接到使用“nat”驱动程序创建的网络的容器将连接到*内部* Hyper-V 交换机，并从用户指定的 (``--subnet``) IP 前缀中接收 IP 地址。 支持从容器主机到容器终结点的端口转移/映射。
  > 如果你安装了 Windows 10 创意者更新，则支持多个 NAT 网络！

- **transparent** – 连接到使用“transparent”驱动程序创建的网络的容器将通过*外部* Hyper-V 交换机连接到物理网络。 可使用外部 DHCP 服务器静态（需要用户指定的 ``--subnet`` 选项）或动态分配来自物理网络的 IP。 

- **overlay** - 当 Docker 引擎在[群模式](../manage-containers/swarm-mode.md)下运行时，连接到覆盖网络的容器可以与跨多个容器主机连接到相同网络的其他容器通信。 在群群集上创建的每个覆盖网络都使用自己的 IP 子网创建，该子网由专用 IP 前缀定义。 覆盖网络驱动程序使用 VXLAN 封装。 **在使用合适的网络控制层面（Flannel 或 OVN）时可与 Kubernetes 配合使用。**
  > 注意：要创建覆盖网络，请确保你的环境满足这些*必需的*[先决条件](https://docs.docker.com/network/overlay/#operations-for-all-overlay-networks)。

  > 注意：需要 Windows Server 2016 与 [KB4015217](https://support.microsoft.com/en-us/help/4015217/windows-10-update-kb4015217)、Windows 10 创意者更新或更高版本。

- **l2bridge** - 当容器连接到使用“l2bridge”驱动程序创建的网络时，其将与容器主机位于同一 IP 子网中，并通过*外部* Hyper-V 交换机连接到物理网络。 必须根据与容器主机相同的前缀静态分配 IP 地址。 由于对入口和出口执行了第 2 层地址转换（MAC 重写）操作，因此主机上的所有容器终结点都将具有与主机相同的 MAC 地址。
  > 注意：在虚拟化方案中使用此模式（容器主机是虚拟机）时，_必须进行 MAC 地址欺骗_。
  
  > 注意：需要 Windows Server 2016、Windows 10 创意者更新或更高版本。

- **l2tunnel** - 与 l2bridge 类似，但是_此驱动程序应仅用于 Microsoft 云堆栈_。 来自容器的数据包会发送到应用了 SDN 策略的虚拟化主机。

> 若要了解如何通过 Microsoft SDN 堆栈将容器终结点连接到现有租户虚拟网络，请参阅[将容器附加到虚拟网络](https://technet.microsoft.com/en-us/windows-server-docs/networking/sdn/manage/connect-container-endpoints-to-a-tenant-virtual-network)主题。

> Windows 10 创意者更新（或更高版本）引入了平台支持，以将新容器终结点添加到正在运行的容器中（即“热添加”）！


## <a name="network-topologies-and-ipam"></a>网络拓扑和 IPAM
下表显示了如何为每个网络驱动程序的内部连接（容器间连接）和外部连接提供网络连接。

### <a name="networking-modes--docker-drivers"></a>网络模式/Docker 驱动程序

  | Docker Windows 网络驱动程序 | 典型用法 | 容器到容器（单个节点） | 容器到外部（单个节点 + 多个节点） | 容器到容器（多个节点） |
  |-------------------------------|:------------:|:------------------------------------:|:------------------------------------------------:|:-----------------------------------:|
  | **NAT（默认）** | 非常适用于开发人员 | <ul><li>相同子网：通过 Hyper-V 虚拟交换机进行桥接</li><li> 跨子网：在 WS2016 中不受支持（只有一个 NAT 内部前缀）</li></ul> | 通过管理 vNIC（已绑定到 WinNAT）路由 | 不直接支持：需要通过主机公开端口 |
  | **Transparent** | 非常适用于开发人员或小型部署 | <ul><li>相同子网：通过 Hyper-V 虚拟交换机进行桥接</li><li>跨子网：通过容器主机路由</li></ul> | 通过能够直接访问（物理）网络适配器的容器主机进行路由 | 通过能够直接访问（物理）网络适配器的容器主机进行路由 |
  | **Overlay** | Docker 群要求，多节点 | <ul><li>相同子网：通过 Hyper-V 虚拟交换机进行桥接</li><li>跨子网：网络流量通过管理 vNIC 进行封装和路由</li></ul> | 不直接支持 - 需要连接到 NAT 网络的第二个容器终结点 | 相同/跨子网：网络流量使用 VXLAN 封装并通过管理 vNIC 进行路由 |
  | **L2Bridge** | 用于 Kubernetes 和 Microsoft SDN | <ul><li>相同子网：通过 Hyper-V 虚拟交换机进行桥接</li><li> 跨子网：在入口和出口重新写入容器 MAC 地址并路由</li></ul> | 在入口和出口重新写入容器 MAC 地址 | <ul><li>相同子网：桥接</li><li>跨子网：在 WS2016 中不受支持。</li></ul> |
  | **L2Tunnel**| 仅限 Azure | 相同/跨子网：固定到策略所应用于的物理主机的 Hyper-V 虚拟交换机 | 流量必须经过 Azure 虚拟网络网关 | 相同/跨子网：固定到策略所应用于的物理主机的 Hyper-V 虚拟交换机 |

### <a name="ipam"></a>IPAM 
系统以不同方式为每个网络驱动程序分配 IP 地址。 Windows 使用主机网络服务 (HNS) 为 nat 驱动程序提供 IPAM，并使用 Docker 群模式（内部 KVS）为 overlay 驱动程序提供 IPAM。 所有其他网络驱动程序都使用外部 IPAM。

| 网络模式/驱动程序 | IPAM |
| -------------------------|:----:|
| NAT | 根据内部 NAT 子网前缀由主机网络服务 (HNS) 进行动态 IP 分配和指定 |
| Transparent | 对容器主机网络前缀内的 IP 地址进行静态或动态（使用外部 DHCP 服务器）IP 分配和指定 |
| Overlay | 根据 Docker 引擎群模式管理的前缀进行动态 IP 分配并通过 HNS 进行指定 |
| L2Bridge | 对容器主机网络前缀内的 IP 地址进行静态 IP 分配和指定（也可以通过 HNS 插件来分配） |
| L2Tunnel | 仅限 Azure - 通过插件进行动态 IP 分配和指定 |

### <a name="service-discovery"></a>服务发现
只有某些 Windows 网络驱动程序才支持服务发现。

|  | 本地服务发现  | 全局服务发现 |
| :---: | :---------------     |  :---                |
| nat | 是 | NA |  
| overlay | 是 | 是（通过 Docker EE） |
| transparent | 否 | 否 |
| l2bridge | 否 | 是（通过 Kube-DNS） |
