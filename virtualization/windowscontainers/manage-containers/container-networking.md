---
title: "Windows 容器网络"
description: "为 Windows 容器配置网络。"
keywords: "docker, 容器"
author: jmesser81
ms.date: 08/22/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: 70e662e73693d9fef9d36635289ec9affb8d0331
ms.sourcegitcommit: f542e8c95b5bb31b05b7c88f598f00f76779b519
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/01/2018
---
# <a name="windows-container-networking"></a>Windows 容器网络
> ***请参考 [Docker 容器网络](https://docs.docker.com/engine/userguide/networking/)，以了解一般 docker 网络命令、选项和语法。*** 除了本文档中所描述的任何情况之外，所有 Docker 网络命令在 Windows 上都受支持，并且所用语法与 Linux 上的语法相同。 但请注意，Windows 和 Linux 网络堆栈不同，因此你会发现某些 Linux 网络命令（如 ifconfig）在 Windows 上不受支持。

## <a name="basic-networking-architecture"></a>基本网络体系结构
本主题概述 Docker 如何在 Windows 上创建和管理网络。 对于网络，Windows 容器的作用类似于虚拟机。 每个容器都有一个连接到 Hyper-V 虚拟交换机 (vSwitch) 的虚拟网络适配器 (vNIC)。 Windows 支持可通过 Docker 创建的以下五个不同的网络驱动程序或模式：*nat*、*overlay*、*transparent*、*l2bridge* 和 *l2tunnel*。 根据物理网络基础结构和单 VS 多主机网络要求，应该选择最符合你需要的网络驱动程序。

<figure>
  <img src="media/windowsnetworkstack-simple.png">
</figure>  

首次运行 docker 引擎时，它将创建默认的 NAT 网络“nat”，该网络使用内部 vSwitch 和一个名为 `WinNAT` 的 Windows 组件。 如果主机上有通过 PowerShell 或 Hyper-V 管理器创建的任何预先存在的外部 vSwitch，则也可以使用 *transparent* 网络驱动程序将它们提供给 Docker，并且在你运行 ``docker network ls`` 命令时可以看到它们  

<figure>
  <img src="media/docker-network-ls.png">
</figure>

> - ***内部*** vSwitch 是未直接连接到容器主机上的网络适配器的 vSwitch 

> - ***外部*** vSwitch 是_已_直接连接到容器主机上的网络适配器的 vSwitch  

<figure>
  <img src="media/get-vmswitch.png">
</figure>

“Nat”网络是在 Windows 上运行的容器的默认网络。 任何在 Windows 上运行但未使用任何标志或参数来实现特定网络配置的容器都将连接到默认的“nat”网络，并且将被自动分配一个“nat”网络内部前缀 IP 范围内的 IP 地址。 用于“nat”的默认内部 IP 前缀是 172.16.0.0/16。 


## <a name="windows-container-network-drivers"></a>Windows 容器网络驱动程序  

除了利用 Docker 在 Windows 上创建的默认“nat”网络之外，用户还可以定义自定义容器网络。 可以使用 Docker CLI [`docker network create -d <NETWORK DRIVER TYPE> <NAME>`](https://docs.docker.com/engine/reference/commandline/network_create/) 命令创建用户定义的网络。 在 Windows 上，有以下网络驱动程序类型可用：

- **nat** – 连接到使用“nat”驱动程序创建的网络的容器将从用户指定的 (``--subnet``) IP 前缀中接收 IP 地址。 支持从容器主机到容器终结点的端口转发/映射。
> 注意：Windows 10 创意者更新现在支持多个 NAT 网络！ 

- **transparent** – 连接到使用“transparent”驱动程序创建的网络的容器将直接连接到物理网络。 可使用外部 DHCP 服务器静态（需要用户指定的 ``--subnet`` 选项）或动态分配来自物理网络的 IP。 

- **overlay** - __新增！__  当 docker 引擎在[群模式](./swarm-mode.md)下运行时，连接到覆盖网络的容器可以与跨多个容器主机连接到相同网络的其他容器通信。 在群群集上创建的每个覆盖网络都使用自己的 IP 子网创建，该子网由专用 IP 前缀定义。 覆盖网络驱动程序使用 VXLAN 封装。
> 需要 Windows Server 2016 与 [KB4015217](https://support.microsoft.com/en-us/help/4015217/windows-10-update-kb4015217) 或 Windows 10 创意者更新 

- **l2bridge** - 连接到使用“l2bridge”驱动程序创建的网络的容器将与容器主机在同一 IP 子网中。 必须从与容器主机相同的前缀中静态分配 IP 地址。 由于对入口和出口执行了第 2 层地址转换（MAC 重写）操作，因此主机上的所有容器终结点都将具有相同的 MAC 地址。
> 需要 Windows Server 2016 或 Windows 10 创意者更新

- **l2tunnel** - _此驱动程序应仅用于 Microsoft 云堆栈_

> 若要了解如何通过 Microsoft SDN 堆栈将容器终结点连接到覆盖虚拟网络，请参阅 [Attaching Containers to a Virtual Network](https://technet.microsoft.com/en-us/windows-server-docs/networking/sdn/manage/connect-container-endpoints-to-a-tenant-virtual-network)（将容器附加到虚拟网络）主题。

> Windows 10 创意者更新引入了平台支持，以将新容器终结点添加到正在运行的容器中（即“热添加”）。 这将启用端到端操作，并暂停[未完成的 Docker 拉取请求](https://github.com/docker/libnetwork/pull/1661)

## <a name="network-topologies-and-ipam"></a>网络拓扑和 IPAM
下表显示了如何为每个网络驱动程序的内部连接（容器间连接）和外部连接提供网络连接。

<figure>
  <img src="media/network-modes-table.png">
</figure>

### <a name="ipam"></a>IPAM 
系统以不同方式为每个网络驱动程序分配 IP 地址。 Windows 使用主机网络服务 (HNS) 为 nat 驱动程序提供 IPAM，并使用 Docker 群模式（内部 KVS）为 overlay 驱动程序提供 IPAM。 所有其他网络驱动程序都使用外部 IPAM。

<figure>
  <img src="media/ipam.png">
</figure>

# <a name="details-on-windows-container-networking"></a>关于 Windows 容器网络的详细信息

## <a name="isolation-namespace-with-network-compartments"></a>隔离（命名空间）与网络隔离舱
每个容器终结点都放在其自己的__网络隔离舱__中，网络隔离舱类似于 Linux 中的网络命名空间。 管理主机 vNIC 和主机网络堆栈都位于默认的网络隔离舱中。 为了对同一主机上的各容器强制进行网络隔离，将为每个 Windows Server 和 Hyper-V 容器创建网络隔离舱，并将该容器的网络适配器安装在其中。 Windows Server 容器使用主机 vNIC 连接到虚拟交换机。 Hyper-V 容器使用合成 VM NIC（不公开到实用工具 VM）连接到虚拟交换机。 

<figure>
  <img src="media/network-compartment-visual.png">
</figure>

```powershell 
Get-NetCompartment
```

## <a name="windows-firewall-security"></a>Windows 防火墙安全性

Windows 防火墙用于通过端口 ACL 来增强网络安全性。

> 注意：默认情况下，为连接到覆盖网络的所有容器终结点都创建了“全部允许”规则   

<figure>
  <img src="media/windows-firewall-containers.png">
</figure>

## <a name="container-network-management-with-host-network-service"></a>容器网络管理与主机网络服务

下图显示了主机网络服务 (HNS) 和主机计算服务 (HCS) 如何共同创建容器并将终结点连接到网络。 

<figure>
  <img src="media/HNS-Management-Stack.png">
</figure>

# <a name="advanced-network-options-in-windows"></a>Windows 中的高级网络选项
系统支持多个网络驱动程序选项，以便充分利用特定于 Windows 的功能和特性。 

## <a name="switch-embedded-teaming-with-docker-networks"></a>交换机嵌入式组合与 Docker 网络

> 适用于所有网络驱动程序 

通过使用 `-o com.docker.network.windowsshim.interface` 选项指定多个网络适配器（用逗号分隔）来创建供 Docker 使用的容器主机网络时，你可以充分利用[交换机嵌入式组合](https://technet.microsoft.com/en-us/windows-server-docs/networking/technologies/hyper-v-virtual-switch/rdma-and-switch-embedded-teaming#a-namebkmksswitchembeddedaswitch-embedded-teaming-set)。 

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2", "Ethernet 3" TeamedNet
```

## <a name="set-the-vlan-id-for-a-network"></a>设置网络 VLAN ID

> 适用于 transparent 和 l2bridge 网络驱动程序 

若要设置网络 VLAN ID，请将选项 `-o com.docker.network.windowsshim.vlanid=<VLAN ID>` 用于 `docker network create` 命令。 例如，你可以使用以下命令创建 VLAN ID 为 11 的透明网络：

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.vlanid=11 MyTransparentNetwork
```
当你为网络设置 VLAN ID 时，就是在为连接到该网络的任何容器终结点设置 VLAN 隔离。

> 确保你的主机网络适配器（物理）处于 trunk 模式，以允许处于访问模式下的 vSwitch 在正确的 VLAN 上通过 vNIC（容器终结点）端口处理所有带标记的流量。

## <a name="specify-the-name-of-a-network-to-the-hns-service"></a>为 HNS 服务指定网络名称

> 适用于所有网络驱动程序 

一般情况下，当你使用 `docker network create` 创建容器网络时，你提供的网络名称将由 Docker 服务使用，而不是由 HNS 服务使用。 如果你要创建网络，你可以使用选项 `-o com.docker.network.windowsshim.networkname=<network name>` 为 `docker network create` 命令指定由 HNS 服务提供的名称。 例如，可以采用以下命令，用为 HNS 服务指定的名称创建透明网络：

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.networkname=MyTransparentNetwork MyTransparentNetwork
```

## <a name="bind-a-network-to-a-specific-network-interface"></a>将网络绑定到特定网络接口

> 适用于除“nat”之外的所有网络驱动程序  

要将网络（通过 Hyper-V 虚拟交换机连接）绑定到特定网络接口，请对 `docker network create` 命令使用选项 `-o com.docker.network.windowsshim.interface=<Interface>`。 例如，可以使用以下命令创建连接到“以太网 2”网络接口的透明网络：

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" TransparentNet2
```

> 注意：*com.docker.network.windowsshim.interface* 的值是网络适配器的*名称*，可通过以下方式找到：

>```
PS C:\> Get-NetAdapter
```
## Specify the DNS Suffix and/or the DNS Servers of a Network

> Applies to all network drivers 

Use the option, `-o com.docker.network.windowsshim.dnssuffix=<DNS SUFFIX>` to specify the DNS suffix of a network, and the option, `-o com.docker.network.windowsshim.dnsservers=<DNS SERVER/S>` to specify the DNS servers of a network. For example, you might use the following command to set the DNS suffix of a network to "example.com" and the DNS servers of a network to 4.4.4.4 and 8.8.8.8:

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.dnssuffix=abc.com -o com.docker.network.windowsshim.dnsservers=4.4.4.4,8.8.8.8 MyTransparentNetwork
```

## VFP

The Virtual Filtering Platform (VFP) extension is a Hyper-V virtual switch, forwarding extension used to enforce network policy and manipulate packets. For instance, VFP is used by the 'overlay' network driver to perform VXLAN encapsulation and by the 'l2bridge' driver to perform MAC re-write on ingresss and egress. The VFP extension is only present on Windows Server 2016 and Windows 10 Creators Update. To check and see if this is running correctly a user run two commands:

```powershell
Get-Service vfpext

# This should indicate the extension is Running: True 
Get-VMSwitchExtension  -VMSwitchName <vSwitch Name> -Name "Microsoft Azure VFP Switch Extension"
```

## <a name="tips--insights"></a>提示和见解
下面是好用的提示和见解的列表，这些内容的灵感源于我们从社区中听到的关于 Windows 容器网络的常见问题…

#### <a name="hns-requires-that-ipv6-is-enabled-on-container-host-machines"></a>HNS 要求在容器主机上启用 IPv6 
在 [KB4015217](https://support.microsoft.com/en-us/help/4015217/windows-10-update-kb4015217) 中，HNS 要求在 Windows 容器主机上启用 IPv6。 如果你遇到错误，如以下错误，则可能是在主机上禁用了 IPv6。
```
docker: Error response from daemon: container e15d99c06e312302f4d23747f2dfda4b11b92d488e8c5b53ab5e4331fd80636d encountered an error during CreateContainer: failure in a Windows system call: Element not found.
```
我们正在努力更改平台以自动检测/避免此问题。 目前可使用以下解决方法来确保在你的主机上启用了 IPv6：

```
C:\> reg delete HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip6\Parameters  /v DisabledComponents  /f
```

#### <a name="moby-linux-vms-use-dockernat-switch-with-docker-for-windows-a-product-of-docker-cehttpswwwdockercomcommunity-edition-instead-of-hns-internal-vswitch"></a>Moby Linux VM 将 DockerNAT 交换机与适用于 Windows 的 Docker（[Docker CE](https://www.docker.com/community-edition) 的产品）配合使用，而不是使用 HNS 内部 vSwitch 
Windows 10 上适用于 Windows 的 Docker（Docker CE 引擎的 Windows 驱动程序）将使用名为“DockerNAT”的内部 vSwitch，以将 Moby Linux VM 连接到容器主机。 在 Windows 上使用 Moby Linux VM 的开发人员应该知道他们的主机使用的是 DockerNAT vSwitch，而不是由 HNS 服务创建的 vSwitch（这是用于 Windows 容器的默认交换机）。 

#### <a name="to-use-dhcp-for-ip-assignment-on-a-virtual-container-host-enable-macaddressspoofing"></a>若要使用 DHCP 在虚拟容器主机上分配 IP，请启用 MACAddressSpoofing 
如果容器主机被虚拟化，而你想要使用 DHCP 进行 IP 分配，则必须在虚拟机网络适配器上启用 MACAddressSpoofing。 否则，Hyper-V 主机将会阻止来自容器（位于具有多个 MAC 地址的 VM 中）的网络流量。 你可以使用以下 PowerShell 命令启用 MACAddressSpoofing：
```
PS C:\> Get-VMNetworkAdapter -VMName ContainerHostVM | Set-VMNetworkAdapter -MacAddressSpoofing On
```
如果你要将 VMware 作为虚拟机监控程序来运行，则将需要启用混杂模式以使其正常工作。 可在[此处](https://kb.vmware.com/s/article/1004099)找到详细信息
#### <a name="creating-multiple-transparent-networks-on-a-single-container-host"></a>在单个容器主机上创建多个透明网络
如果希望创建多个透明网络，则必须指定外部 Hyper-V 虚拟交换机应绑定到的（虚拟）网络适配器。 若要指定网络接口，请使用以下语法：
```
# General syntax:
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface=<INTERFACE NAME> <NETWORK NAME> 

# Example:
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" myTransparent2
```

#### <a name="remember-to-specify---subnet-and---gateway-when-using-static-ip-assignment"></a>记住在使用静态 IP 分配时要指定 *--subnet* 和 *--gateway*
使用静态 IP 分配时，必须首先确保已在创建网络时指定 *--subnet* 和 *--gateway* 参数。 子网和网关 IP 地址应该与容器主机的网络设置（即物理网络）相同。 例如，下面介绍了如何创建透明网络并使用静态 IP 分配在该网络上运行终结点：

```
# Example: Create a transparent network using static IP assignment
# A network create command for a transparent container network corresponding to the physical network with IP prefix 10.123.174.0/23
C:\> docker network create -d transparent --subnet=10.123.174.0/23 --gateway=10.123.174.1 MyTransparentNet
# Run a container attached to MyTransparentNet
C:\> docker run -it --network=MyTransparentNet --ip=10.123.174.105 windowsservercore cmd
```

#### <a name="dhcp-ip-assignment-not-supported-with-l2bridge-networks"></a>L2Bridge 网络不支持 DHCP IP 分配
使用 l2bridge 驱动程序创建的容器网络仅支持静态 IP 分配。 如上所述，请记住使用 *--subnet* 和 *--gateway* 参数来创建为静态 IP 分配配置的网络。

#### <a name="networks-that-leverage-external-vswitch-must-each-have-their-own-network-adapter"></a>利用外部 vSwitch 的网络均须具有自己的网络适配器
请注意，如果在同一容器主机上创建了使用外部 vSwitch 进行连接的多个网络（例如，透明、L2 桥接、L2 透明网络），则每个网络都需要具有自己的网络适配器。 

#### <a name="ip-assignment-on-stopped-vs-running-containers"></a>已停止与正在运行的容器上的 IP 分配
静态 IP 分配直接在容器的网络适配器上执行，并且必须仅当容器处于已停止状态时执行。 容器运行时，（Windows Server 2016 中）不支持“热添加”容器网络适配器或更改网络堆栈。
> 注意：此行为在 Windows 10 创意者更新上发生了变化，因为平台现在支持“热添加”。 在合并此[未完成的 Docker 拉取请求](https://github.com/docker/libnetwork/pull/1661)后，此功能将启用 E2E 功能

#### <a name="existing-vswitch-not-visible-to-docker-can-block-transparent-network-creation"></a>现有 vSwitch（对 Docker 不可见）可以阻止创建透明网络
如果创建透明网络出错，可能是未经 Docker 自动发现的系统上存在外部 vSwitch，因而使得透明网络无法绑定到容器主机的外部网络适配器。 

创建透明网络时，Docker 首先创建网络的外部 vSwitch，然后尝试将交换机绑定到（外部）网络适配器 - 该适配器可以是 VM 网络适配器或物理网络适配器。 如果已在容器主机上创建 vSwitch，*且对 Docker 可见*，Windows Docker 引擎将使用该交换机，而不是重新创建一个。 但是，如果该 vSwitch 创建于带外（即在容器主机上使用 HYper-V 管理器或 PowerShell 创建）并且对 Docker 不可见，Windows Docker 引擎将尝试创建一个新的 vSwitch，从而无法将新交换机连接到容器主机外部网络适配器（因为网络适配器已经连接到创建于带外的交换机）。

例如，如果在 Docker 服务运行期间首先在主机上创建新的 vSwitch，然后尝试创建透明网络，则会出现此问题。 在此情况下，Docker 无法识别创建的交换机，并会为透明网络创建新的 vSwitch。

解决此问题的方法有三种：

* 当然，可以删除带外创建的 vSwitch，这将允许 Docker 创建新的 vSwitch 并将其顺利连接到主机网络适配器。 选择此方法前，请确保其他服务（如 Hyper-V）未使用带外 vSwitch。
* 或者，如果决定使用创建于带外的外部 vSwitch，请重启 Docker 和 HNS 服务以*使交换机对 Docker 可见。*
```
PS C:\> restart-service hns
PS C:\> restart-service docker
```
* 另一种方法是使用“-o com.docker.network.windowsshim.interface”选项以将透明网络的外部 vSwitch 绑定到未在容器主机上使用的特定网络适配器（即，除带外创建的 vSwitch 使用的其他网络适配器）。 有关“-o”选项的详细信息，请参阅本文档上方的[透明网络](https://msdn.microsoft.com/virtualization/windowscontainers/management/container_networking#transparent-network)部分。


## <a name="unsupported-features-and-network-options"></a>不受支持的功能和网络选项 

以下网络选项在 Windows 中不受支持，并且不能传递给 ``docker run``：
 * 容器链接（例如 ``--link``）- _替代方法是依赖服务发现_
 * IPv6 地址（例如 ``--ip6``）
 * DNS 选项（例如 ``--dns-option``）
 * 多个 DNS 搜索域（例如 ``--dns-search``）
 
以下网络选项和功能在 Windows 中不受支持，并且不能传递给 ``docker network create``：
 * --aux-address
 * --internal
 * --ip-range
 * --ipam-driver
 * --ipam-opt
 * --ipv6 

以下网络选项在 Docker 服务中不受支持
* 数据平面加密（例如 ``--opt encrypted``） 


## <a name="windows-server-2016-work-arounds"></a>Windows Server 2016 解决方法 

虽然我们会继续添加新功能并促进开发，但是其中一些功能将不会向后移植到旧版平台。 而最佳行动计划是获取 Windows 10 和 Windows Server 的最新更新。  以下部分列出了适用于 Windows Server 2016 和旧版本的 Windows 10（即 1704 创意者更新之前的版本）的一些解决方法和注意事项

### <a name="multiple-nat-networks-on-ws2016-container-host"></a>WS2016 容器主机上的多个 NAT 网络

必须在较大的内部 NAT 网络前缀下创建任何新 NAT 网络的分区。 通过在 PowerShell 中运行以下命令并引用“InternalIPInterfaceAddressPrefix”字段可以找到该前缀。

```
PS C:\> Get-NetNAT
```

例如，主机的 NAT 网络内部前缀可能是 172.16.0.0/16。 在本例中，可以使用 Docker 来创建其他 NAT 网络，*前提是它们是 172.16.0.0/16 前缀的子集。* 例如，可以使用 IP 前缀 172.16.1.0/24（网关，172.16.1.1）和 172.16.2.0/24（网关，172.16.2.1）创建两个 NAT 网络。

```
C:\> docker network create -d nat --subnet=172.16.1.0/24 --gateway=172.16.1.1 CustomNat1
C:\> docker network create -d nat --subnet=172.16.2.0/24 --gateway=172.16.1.1 CustomNat2
```

使用以下选项可以列出新建的网络：
```
C:\> docker network ls
```

### <a name="docker-compose"></a>Docker Compose

[Docker Compose](https://docs.docker.com/compose/overview/) 可用于同时定义和配置容器网络以及将使用这些网络的容器/服务。 定义容器将连接的网络的过程中，Compose“网络”键可用作顶级键。 例如，以下语法将 Docker 创建的预先存在的 NAT 网络定义为在给定 Compose 文件中定义的所有容器/服务的“默认”网络。

```
networks:
 default:
  external:
   name: "nat"
```

同样，以下语法可用于定义自定义 NAT 网络。

> 注意：下例中定义的“自定义 NAT 网络”是指容器主机预先存在的 NAT 内部前缀的分区。 有关更多上下文，请参阅上方“多个 NAT 网络”部分。

```
networks:
  default:
    driver: nat
    ipam:
      driver: default
      config:
      - subnet: 172.16.3.0/24
```

有关使用 Docker Compose 定义/配置容器网络的详细信息，请参阅 [Compose File reference](https://docs.docker.com/compose/compose-file/)（Compose 文件引用）。

### <a name="service-discovery"></a>服务发现
只有某些 Windows 网络驱动程序才支持服务发现。

|  | 本地服务发现  | 全局服务发现 |
| :---: | :---------------     |  :---                |
| nat | 是 | NA |  
| overlay | 是 | 是 |
| transparent | 否 | 否 |
| l2bridge | 否 | 否 |


