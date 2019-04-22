---
title: Windows 容器网络
description: Windows 容器高级网络。
keywords: docker, 容器
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: 001f1abaeefaf34e12b0f7e3323bf32140080d05
ms.sourcegitcommit: a5ff22c205149dac4fc05325ef3232089826f1ef
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/22/2019
ms.locfileid: "9380001"
---
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

```
PS C:\> Get-NetAdapter
```

## <a name="specify-the-dns-suffix-andor-the-dns-servers-of-a-network"></a>指定网络的 DNS 后缀和/或 DNS 服务器

> 适用于所有网络驱动程序 

使用选项 `-o com.docker.network.windowsshim.dnssuffix=<DNS SUFFIX>` 指定网络的 DNS 后缀，使用选项 `-o com.docker.network.windowsshim.dnsservers=<DNS SERVER/S>` 指定网络的 DNS 服务器。 例如，你可以使用以下命令将网络的 DNS 后缀设置为“example.com”，将网络的 DNS 服务器设置为 4.4.4.4 和 8.8.8.8：

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.dnssuffix=abc.com -o com.docker.network.windowsshim.dnsservers=4.4.4.4,8.8.8.8 MyTransparentNetwork
```

## <a name="vfp"></a>VFP

有关更多信息，请参阅[此文章](https://www.microsoft.com/en-us/research/project/azure-virtual-filtering-platform/)。

## <a name="tips--insights"></a>提示和见解
下面是好用的提示和见解的列表，这些内容的灵感源于我们从社区中听到的关于 Windows 容器网络的常见问题…

#### <a name="hns-requires-that-ipv6-is-enabled-on-container-host-machines"></a>HNS 要求在容器主机上启用 IPv6 
在 [KB4015217](https://support.microsoft.com/en-us/help/4015217/windows-10-update-kb4015217) 中，HNS 要求在 Windows 容器主机上启用 IPv6。 如果你遇到错误，如以下错误，则可能是在主机上禁用了 IPv6。
```
docker: Error response from daemon: container e15d99c06e312302f4d23747f2dfda4b11b92d488e8c5b53ab5e4331fd80636d encountered an error during CreateContainer: failure in a Windows system call: Element not found.
```
我们正在努力更改平台以自动检测/避免此问题。 目前可使用以下解决方法来确保在你的主机上启用了 IPv6：

```
C:\> reg delete HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip6\Parameters  /v DisabledComponents  /f
```


#### <a name="linux-containers-on-windows"></a>Windows 上的 Linux 容器

**新建：** 我们正在努力实现在_无 Moby Linux 虚拟机_的情况下并行运行 Linux 和 Windows 容器。 有关详细信息，请参阅此[关于 Windows 上的 Linux 容器 (LCOW) 的博客文章](https://blog.docker.com/2017/11/docker-for-windows-17-11/)。 下面是如何[开始](https://docs.microsoft.com/en-us/virtualization/windowscontainers/quick-start/quick-start-windows-10-linux)。
> 注意：LCOW 将弃用 Moby Linux 虚拟机，并且它将使用默认的 HNS“nat”内部 vSwitch。

#### <a name="moby-linux-vms-use-dockernat-switch-with-docker-for-windows-a-product-of-docker-cehttpswwwdockercomcommunity-edition"></a>Moby Linux 虚拟机将 DockerNAT 交换机与适用于 Windows 的 Docker（[Docker CE](https://www.docker.com/community-edition) 的产品）配合使用

Windows 10 上适用于 Windows 的 Docker（Docker CE 引擎的 Windows 驱动程序）将使用名为“DockerNAT”的内部 vSwitch，以将 Moby Linux 虚拟机连接到容器主机。 在 Windows 上使用 Moby Linux 虚拟机的开发人员应该知道他们的主机使用的是 DockerNAT vSwitch，而不是由 HNS 服务创建的“nat”vSwitch（这是用于 Windows 容器的默认交换机）。



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