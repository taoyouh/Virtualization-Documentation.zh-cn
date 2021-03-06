---
title: Windows 容器网络
description: Windows 容器的网络驱动程序和拓扑。
keywords: docker, 容器
author: jmesser81
ms.date: 03/27/2018
ms.topic: conceptual
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: c7be67f3c114e7f6b122dffff399b2c292c26369
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192165"
---
# <a name="windows-container-network-drivers"></a>Windows 容器网络驱动程序

除了利用 Docker 在 Windows 上创建的默认“nat”网络之外，用户还可以定义自定义容器网络。 用户定义的网络可以使用 Docker CLI [`docker network create -d <NETWORK DRIVER TYPE> <NAME>`](https://docs.docker.com/engine/reference/commandline/network_create/) 命令创建。 在 Windows 上，有以下网络驱动程序类型可用：

- **nat** – 连接到使用“nat”驱动程序创建的网络的容器将连接到*内部* Hyper-V 交换机，并从用户指定的 (``--subnet``) IP 前缀中接收 IP 地址。 支持从容器主机到容器终结点的端口转发/映射。

  >[!NOTE]
  > 重新启动后，不再保留在 Windows Server 2019 （或更高版本）上创建的 NAT 网络。

  > 如果已安装 Windows 10 创意者更新（或更高版本），则支持多个 NAT 网络。

- **transparent** – 连接到使用“transparent”驱动程序创建的网络的容器将通过*外部* Hyper-V 交换机连接到物理网络。 可使用外部 DHCP 服务器静态（需要用户指定的 ``--subnet`` 选项）或动态分配来自物理网络的 IP。

  >[!NOTE]
  >由于以下要求，Azure Vm 不支持通过透明网络连接容器主机。

  > 要求：在虚拟化方案（容器主机为 VM）中使用此模式时 _，需要 MAC 地址欺骗_。

- **overlay** - 当 Docker 引擎在[群模式](../manage-containers/swarm-mode.md)下运行时，连接到覆盖网络的容器可以与跨多个容器主机连接到相同网络的其他容器通信。 在群群集上创建的每个覆盖网络都使用自己的 IP 子网创建，该子网由专用 IP 前缀定义。 覆盖网络驱动程序使用 VXLAN 封装。 **当使用合适的网络控制平面（如 Flannel）时，可与 Kubernetes 配合使用。**
  > 要求：确保环境满足创建覆盖网络所需的[先决条件](https://docs.docker.com/network/overlay/#operations-for-all-overlay-networks)。

  > 要求：在 Windows Server 2019 上，这需要[KB4489899](https://support.microsoft.com/help/4489899)。

  > 要求：在 Windows Server 2016 上，这需要[KB4015217](https://support.microsoft.com/help/4015217/windows-10-update-kb4015217)。

  >[!NOTE]
  >在 Windows Server 2019 上，Docker Swarm 创建的覆盖网络利用 VFP NAT 规则进行出站连接。 这意味着，给定容器接收1个 IP 地址。 这还意味着， `ping` `Test-NetConnection` 在调试的情况下，应使用其 TCP/UDP 选项配置基于 ICMP 的工具（如或）。

- **l2bridge** -类似于 `transparent` 网络模式，附加到使用 "l2bridge" 驱动程序创建的网络的容器将通过*外部*hyper-v 交换机连接到物理网络。 L2bridge 的不同之处在于，容器终结点将与主机具有相同的 MAC 地址，因为入口和出口的第2层地址转换（MAC 重新写入）操作。 在群集方案中，这有助于缓解交换机的压力，使其不必了解有时短期的临时容器的 MAC 地址。 可以通过两种不同的方式配置 L2bridge 的网络：
  1. 已为 L2bridge 网络配置与容器主机相同的 IP 子网
  2. 使用新的自定义 IP 子网配置 L2bridge 网络

  在配置2中，用户将需要在充当网关并为指定的前缀配置路由功能的主机网络隔离舱上添加终结点。
  > 要求：需要 Windows Server 2016、Windows 10 创意者更新或更高版本。


- **l2bridge** -类似于 `transparent` 网络模式，附加到使用 "l2bridge" 驱动程序创建的网络的容器将通过*外部*hyper-v 交换机连接到物理网络。 L2bridge 的不同之处在于，容器终结点将与主机具有相同的 MAC 地址，因为入口和出口的第2层地址转换（MAC 重新写入）操作。 在群集方案中，这有助于缓解交换机的压力，使其不必了解有时短期的临时容器的 MAC 地址。 可以通过两种不同的方式配置 L2bridge 的网络：
  1. 已为 L2bridge 网络配置与容器主机相同的 IP 子网
  2. 使用新的自定义 IP 子网配置 L2bridge 网络

  在配置2中，用户将需要在充当网关并为指定的前缀配置路由功能的主机网络隔离舱上添加终结点。
  >[!TIP]
  >可在[此处](https://techcommunity.microsoft.com/t5/networking-blog/l2bridge-container-networking/ba-p/1180923)找到有关如何配置和安装 l2bridge 的更多详细信息。

- **l2tunnel** -类似于 l2bridge，但_此驱动程序只能在 Microsoft 云 Stack （Azure）中使用_。 来自容器的数据包会发送到应用了 SDN 策略的虚拟化主机。


## <a name="network-topologies-and-ipam"></a>网络拓扑和 IPAM

下表显示了如何为每个网络驱动程序的内部连接（容器间连接）和外部连接提供网络连接。

### <a name="networking-modesdocker-drivers"></a>网络模式/Docker 驱动程序

  | Docker Windows 网络驱动程序 | 典型用途 | 容器到容器（单节点） | 容器到外部（单节点 + 多节点） | 容器到容器（多节点） |
  |-------------------------------|:------------:|:------------------------------------:|:------------------------------------------------:|:-----------------------------------:|
  | **NAT（默认）** | 非常适用于开发人员 | <ul><li>相同子网：通过 Hyper-V 虚拟交换机进行桥接</li><li> 跨子网：不支持（仅有一个 NAT 内部前缀）</li></ul> | 通过管理 vNIC（已绑定到 WinNAT）路由 | 不直接支持：需要通过主机公开端口 |
  | **透明** | 非常适用于开发人员或小型部署 | <ul><li>相同子网：通过 Hyper-V 虚拟交换机进行桥接</li><li>跨子网：通过容器主机路由</li></ul> | 通过能够直接访问（物理）网络适配器的容器主机进行路由 | 通过能够直接访问（物理）网络适配器的容器主机进行路由 |
  | **覆盖** | 适用于多节点;对于 Docker Swarm 是必需的，在 Kubernetes 中提供 | <ul><li>相同子网：通过 Hyper-V 虚拟交换机进行桥接</li><li>跨子网：网络流量通过管理 vNIC 进行封装和路由</li></ul> | 不直接支持-要求在 windows server 2016 上附加到 NAT 网络的第二个容器终结点或 Windows Server 2019 上的 VFP NAT 规则。  | 相同/跨子网：网络流量使用 VXLAN 封装并通过管理 vNIC 进行路由 |
  | **L2Bridge** | 用于 Kubernetes 和 Microsoft SDN | <ul><li>相同子网：通过 Hyper-V 虚拟交换机进行桥接</li><li> 跨子网：在入口和出口重新写入容器 MAC 地址并路由</li></ul> | 在入口和出口重新写入容器 MAC 地址 | <ul><li>相同子网：桥接</li><li>跨子网：通过 WSv1809 和更高版本上的管理 vNIC 路由</li></ul> |
  | **L2Tunnel**| 仅限 Azure | 相同/跨子网：固定到策略所应用于的物理主机的 Hyper-V 虚拟交换机 | 流量必须经过 Azure 虚拟网络网关 | 相同/跨子网：固定到策略所应用于的物理主机的 Hyper-V 虚拟交换机 |

### <a name="ipam"></a>IPAM

系统以不同方式为每个网络驱动程序分配 IP 地址。 Windows 使用主机网络服务 (HNS) 为 nat 驱动程序提供 IPAM，并使用 Docker 群模式（内部 KVS）为 overlay 驱动程序提供 IPAM。 所有其他网络驱动程序都使用外部 IPAM。

| 网络模式/驱动程序 | IPAM |
| -------------------------|:----:|
| NAT | 主机网络服务（HNS）在内部 NAT 子网前缀中的动态 IP 分配和分配 |
| 透明 | 对容器主机网络前缀内的 IP 地址进行静态或动态（使用外部 DHCP 服务器）IP 分配和指定 |
| 覆盖 | 根据 Docker 引擎群模式管理的前缀进行动态 IP 分配并通过 HNS 进行指定 |
| L2Bridge | 容器主机网络前缀中的 IP 地址的静态 IP 分配和分配（也可以通过 HNS 分配） |
| L2Tunnel | 仅限 Azure - 通过插件进行动态 IP 分配和指定 |

### <a name="service-discovery"></a>服务发现

只有某些 Windows 网络驱动程序才支持服务发现。

|  | 本地服务发现  | 全局服务发现 |
| :---: | :---------------     |  :---                |
| nat | YES | 是（通过 Docker EE） |
| overlay | YES | 是，具有 Docker EE 或 kube |
| transparent | 是 | 是 |
| l2bridge | 是 | 是，包含 kube |
