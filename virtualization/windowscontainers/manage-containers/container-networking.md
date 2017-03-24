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
translationtype: Human Translation
ms.sourcegitcommit: 23d4b665da627f35cf5fce49c3c9974d0ef287dd
ms.openlocfilehash: e56a5b984cc1c42e27628d00a5cd532788aef11c
ms.lasthandoff: 02/10/2017

---

# 容器网络

关于网络，Windows 容器的作用类似于虚拟机。 每个容器都有一个连接到虚拟交换机 (vSwitch) 的虚拟网络适配器 (vNIC)，通过该适配器可转发入站和出站流量。 为了强制隔离同一主机上的各容器，将为每个 Windows Server 和 Hyper-V 容器创建网络隔离舱，将该容器的网络适配器安装在其中。 Windows Server 容器使用主机 vNIC 连接到虚拟交换机。 Hyper-V 容器使用合成 VM NIC（不公开到实用工具 VM）连接到虚拟交换机。

Windows 容器支持四种不同的网络驱动程序或模式：*nat**transparent**l2bridge*和 *l2tunnel*。 根据物理网络基础结构和单 VS 多主机网络要求，应该选择最符合你需要的网络模式。

首次运行 dockerd 服务时，Docker 引擎将在默认情况下创建一个 NAT 网络。 创建的默认内部 IP 前缀为 172.16.0.0/12。 容器终结点会自动附加到此默认网络，并获得由内部前缀分配的 IP 地址。

> 注意：如果容器主机 IP 具有此相同前缀，则需要按如下所述更改 NAT 内部 IP 前缀。

可以在同一容器主机上创建使用不同驱动程序（例如 transparent、l2bridge）的其他网络。 下表显示了如何为每种模式的内部连接（容器间连接）和外部连接提供网络连接。

- **网络地址转换 (NAT)** - 每个容器都将收到来自内部专用 IP 前缀（例如 172.16.0.0/12）的一个 IP 地址。 支持从容器主机到容器终结点的端口转发/映射

- **透明** - 每个容器终结点直接连接到物理网络。 可使用外部 DHCP 服务器静态或动态分配来自物理网络的 IP

- **[新！]覆盖** - docker 引擎在[群模式](./swarm-mode.md) 中运行时，可使用基于 VXLAN 技术的覆盖网络连接多个容器主机中的容器终结点。 在群群集上创建的每个覆盖网络都使用自己的 IP 子网创建，该子网由专用 IP 前缀定义。

- **L2 桥接** - 每个容器终结点都将与容器主机位于同一 IP 子网内。 必须从与容器主机相同的前缀中静态分配 IP 地址。 由于第 2 层地址转换，主机上的所有容器终结点都将具有相同的 MAC 地址。

- **L2 隧道模式 ** - _这种模式应仅用于 Microsoft 云堆栈_

> 若要了解如何通过 Microsoft SDN 堆栈将容器终结点连接到覆盖虚拟网络，请参阅 [Attaching Containers to a Virtual Network](https://technet.microsoft.com/en-us/windows-server-docs/networking/sdn/manage/connect-container-endpoints-to-a-tenant-virtual-network)（将容器附加到虚拟网络）主题。

## 单节点

|  | 容器 - 容器 | 容器 - 外部 |
| :---: | :---------------     |  :---                |
| nat | 通过 Hyper-V 虚拟交换机进行桥接 | 通过应用了地址转换的 WinNAT 进行路由 |
| transparent | 通过 Hyper-V 虚拟交换机进行桥接 | 直接访问物理网络 |
| 覆盖 | VXLAN 封装在 Hyper-V 虚拟交换机中的 VFP 转发扩展内进行；*主机内部*通信通过由 Hyper-V 虚拟交换机进行的桥接来实现 | 通过应用了地址转换的 WinNAT 进行路由
| l2bridge | 通过 Hyper-V 虚拟交换机进行桥接|  通过 MAC 地址转换访问物理网络|  



## 多节点

|  | 容器 - 容器 | 容器 - 外部 |
| :---: | :----       | :---------- |
| nat | 必须引用外部容器主机 IP 和端口；通过应用了地址转换的 WinNAT 进行路由 | 必须引用外部主机；通过应用了地址转换的 WinNAT 进行路由 |
| transparent | 必须直接引用容器 IP 终结点 | 直接访问物理网络 |
| 覆盖 | VXLAN 封装在 Hyper-V 虚拟交换机中的 VFP 转发扩展内进行；*主机内部*通信直接引用 IP 终结点 | 通过应用了地址转换的 WinNAT 进行路由| 
| l2bridge | 必须直接引用容器 IP 终结点| 通过 MAC 地址转换访问物理网络|


## 网络的创建

### （默认）NAT 网络

Windows Docker 引擎使用 IP 前缀 172.16.0.0/12 创建默认的 NAT 网络（Docker 中称为“nat”）。 如果用户想要创建一个具有特定 IP 前缀的 NAT 网络，则可通过更改 Docker 的配置文件 daemon.json（位于 C:\ProgramData\Docker\config\daemon.json（如果不存在，创建一个））中的选项，执行两个操作中的任意一个。
 1. 使用 _"fixed-cidr": "< IP Prefix > / Mask"_ 选项将创建默认 NAT 网络，其中已指定 IP 前缀和匹配项
 2. 使用 _"bridge": "none"_ 将不会创建默认网络；用户可以通过 *docker network create -d <driver>* 命令使用任意驱动程序来创建一个用户定义网络

在执行这些配置选项中的任一选项之前，必须首先停止 Docker 服务，并删除任何预先存在的 NAT 网络。

```none
PS C:\> Stop-Service docker
PS C:\> Get-ContainerNetwork | Remove-ContainerNetwork

...Edit the daemon.json file...

PS C:\> Start-Service docker
```

如果已将 "fixed-cidr" 选项添加到 daemon.json 文件，Docker 引擎将创建用户定义的 NAT 网络，并指定自定义 IP 前缀和掩码。 相反，如果已添加 "bridge:none" 选项，则需要手动创建网络。

```none
# Create a user-defined NAT network
C:\> docker network create -d nat --subnet=192.168.1.0/24 --gateway=192.168.1.1 MyNatNetwork
```

默认情况下，容器终结点将连接到默认“nat”网络。 如果未创建“nat”网络（因为已在 daemon.json 中指定 "bridge:none"），或如果要求对不同的、用户定义的网络具有访问权限，用户可指定具有 docker 运行命令的 *--network* 参数。

```none
# Connect new container to the MyNatNetwork
C:\> docker run -it --network=MyNatNetwork <image> <cmd>
```

#### 端口映射

若要访问在连接到 NAT 网络的容器内运行的应用程序，则需要在容器主机和容器终结点之间创建端口映射。 必须在容器处于创建时间或停止状态时指定这些映射。

```none
# Creates a static mapping between port TCP:80 of the container host and TCP:80 of the container
C:\> docker run -it -p 80:80 <image> <cmd>

# Creates a static mapping between port 8082 of the container host and port 80 of the container.
C:\> docker run -it -p 8082:80 windowsservercore cmd
```

此外，通过使用 -p 参数或在具有 -p 参数的 Dockerfile 中使用 EXPOSE 命令，还支持动态端口映射。 如果未指定，将在容器主机上选择一个随机选择的临时端口，并通过运行“docker ps”进行检查。

```none
C:\> docker run -itd -p 80 windowsservercore cmd

# Network services running on port TCP:80 in this container can be accessed externally on port TCP:14824
C:\> docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                   NAMES
bbf72109b1fc        windowsservercore   "cmd"               6 seconds ago       Up 2 seconds        *0.0.0.0:14824->80/tcp*   drunk_stonebraker

# Container image specified EXPOSE 80 in Dockerfile - publish this port mapping
C:\> docker network
```
> 从 WS2016 TP5 和高于 14300 版的 Windows 会员版本起，将为所有 NAT 端口映射自动创建防火墙规则。 此防火墙规则将适用于容器主机，而非特定于特定容器终结点或网络适配器。

Windows NAT (WinNAT) 实现具有一些功能限制，具体请参阅此博客文章 [WinNAT capabilities and limitations](https://blogs.technet.microsoft.com/virtualization/2016/05/25/windows-nat-winnat-capabilities-and-limitations/)（WinNAT 的功能和限制）
 1. 每个容器主机仅支持一个 NAT 内部 IP 前缀，因此必须通过分割前缀来定义“多个” NAT 网络（请参阅本文档的“多个 NAT 网络”部分）。
 2. 仅可以使用容器内部 IP 和端口（使用“docker 网络检查<CONTAINER ID>”查找此信息）从容器主机访问容器终结点。

可以使用不同的驱动程序创建其他网络。

> Docker 网络驱动程序为全部小写。

### 透明网络

若要使用透明网络模式，使用驱动程序名称“透明”创建容器网络。

```none
C:\> docker network create -d transparent MyTransparentNetwork
```
> 注意：如果创建透明网络出错，可能是未经 Docker 自动发现的系统上存在外部 vSwitch，因而使得透明网络无法绑定到容器主机的外部网络适配器。 有关详细信息，请参考下方“注意事项和陷阱”下的“阻止透明网络创建的现有 vSwitch”部分。

如果容器主机被虚拟化，而你想要使用 DHCP 进行 IP 分配，则必须在虚拟机网络适配器上启用 MACAddressSpoofing。 否则，Hyper-V 主机将会阻止来自容器（位于具有多个 MAC 地址的 VM 中）的网络流量。

```none
PS C:\> Get-VMNetworkAdapter -VMName ContainerHostVM | Set-VMNetworkAdapter -MacAddressSpoofing On
```

> 如果希望创建多个透明网络，则必须指定（自动创建的）外部 Hyper-V 虚拟交换机应绑定到的（虚拟）网络适配器。

连接到透明网络的容器终结点的 IP 地址可以通过一个外部 DHCP 服务器静态或动态进行分配。

使用静态 IP 分配时，必须首先确保已在创建网络时指定 *--subnet* 和 *--gateway* 参数。 子网和网关 IP 地址应该与容器主机的网络设置（即物理网络）相同。

```none
# Create a transparent network corresponding to the physical network with IP prefix 10.123.174.0/23
C:\> docker network create -d transparent --subnet=10.123.174.0/23 --gateway=10.123.174.1 TransparentNet3
```
请使用 *--ip* 选项为 `docker run` 命令指定 IP 地址：

```none
C:\> docker run -it --network=TransparentNet3 --ip 10.123.174.105 <image> <cmd>
```

> 请确保此 IP 地址未分配到任何物理网络上的其他网络设备

由于容器终结点可以直接访问物理网络，因此无需指定端口映射。

### 覆盖网络

*若要使用覆盖网络模式，必须使用作为管理器节点在群模式中运行的 Docker 主机。* 要了解有关群模式以及如何初始化群管理器的详细信息，请参阅主题[群模式入门](./swarm-mode.md)。

若要创建覆盖网络，请通过**群管理器节点**运行以下命令：

```none
# Create an overlay network from a swarm manager node, called "myOverlayNet"
C:\> docker network create --driver=overlay myOverlayNet
```

### L2 桥接

若要使用 L2 桥接网络模式，请使用驱动程序名称“l2bridge”创建容器网络。 再次强调，必须指定对应于物理网络的子网和网关。

```none
C:\> docker network create -d l2bridge --subnet=192.168.1.0/24 --gateway=192.168.1.1 MyBridgeNetwork
```

L2 桥接网络仅支持静态 IP 分配。

> 当在 SDN 结构上使用 L2 桥接网络时，仅支持动态 IP 分配。 请参阅[将容器附加到虚拟网络](https://technet.microsoft.com/en-us/windows-server-docs/networking/sdn/manage/connect-container-endpoints-to-a-tenant-virtual-network)主题，以了解详细信息。

## 其他操作和配置

> 我们会不断改进 Windows 上的 Docker。 为了确保能够访问所有最新功能，请确保使用最新版本的 Docker 引擎。 你可以使用 `docker -v` 查看你的 docker 版本。 请参考 [Windows 上的 Docker 引擎](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-docker/configure-docker-daemon)主题，获取 Docker 配置指南。

### 列出可用网络

```none
# list container networks
C:\> docker network ls

NETWORK ID          NAME                DRIVER              SCOPE
0a297065f06a        nat                 nat                 local
d42516aa0250        none                null                local
```

### 删除网络

使用 `docker network rm` 删除容器网络。

```none
C:\> docker network rm <network name>
```

这将清除容器网络所用的所有 Hyper-V 虚拟交换机，以及所创建的任何网络地址转换（WinNAT - NetNat 实例）。

### 网络检查

若要查看连接到特定网络的容器和与这些容器终结点相关联的 IP，可以运行以下命令。

```none
C:\> docker network inspect <network name>
```

### 为 HNS 服务指定网络名称

**一般情况下，当你使用 `docker network create` 创建容器网络时，你提供的网络名称将由 Docker 服务使用，而不是由 HNS 服务使用。**

如果你要创建网络，你可以使用选项 `-o com.docker.network.windowsshim.networkname=<network name>` 为 `docker network create` 命令指定由 HNS 服务提供的名称。 例如，可以使用以下命令，使用为 HNS 服务指定的名称创建透明网络：

```none
C:\> docker network create -d transparent -o com.docker.network.windowsshim.networkname=MyTransparentNetwork MyTransparentNetwork
```

#### 示例：默认 HNS 命名行为

若要将此命名行为选项放入上下文中，请参阅以下屏幕截图，其演示了*不*使用此命名选项时，HNS 服务如何对网络命名。 在此示例中，网络名称“MyTransparentNetwork”对 Docker 可见，正如通过 `docker network ls` 命令所示。 但该网络名称对 HNS 服务不可见，正如通过 `Get-ContainerNetwork` Windows PowerShell 命令所示；相反，HNS 已经对网络自动生成大型字母数字名称。

><figure>
  <img src="media/SpecifyName_Capture.PNG">
  <figcaption>示例：<i>不</i>为 HNS 服务指定网络名称。 </figcaption>
</figure>

#### 示例：为 HNS 服务指定网络名称

另一方面，*确实*使用 `-o com.docker.network.windowsshim.networkname=<network name>` 时，HNS 服务会使用指定的名称，而不使用生成的名称。 下方屏幕截图展示了此行为。

><figure>
  <img src="media/SpecifyName_Capture_2.PNG">
  <figcaption>示例：使用“-o com.docker.network.windowsshim.networkname=<network name>”选项为 HNS 服务指定网络名称。</figcaption>
</figure>


### 将网络绑定到特定网络接口

要将网络（通过 Hyper-V 虚拟交换机连接）绑定到特定网络接口，请对 `docker network create` 命令使用选项 `-o com.docker.network.windowsshim.interface=<Interface>`。 例如，可以使用以下命令创建连接到“以太网 2”网络接口的透明网络：

```none
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" TransparentNet2
```

> 注意：*com.docker.network.windowsshim.interface* 的值是网络适配器的*名称*，可通过以下方式找到：

>```none
PS C:\> Get-NetAdapter
```

### Set the VLAN ID for a Network

To set a VLAN ID for a network, use the option, `-o com.docker.network.windowsshim.vlanid=<VLAN ID>` to the `docker network create` command. For instance, you might use the following command to create a transparent network with a VLAN ID of 11:

```none
C:\> docker network create -d transparent -o com.docker.network.windowsshim.vlanid=11 MyTransparentNetwork
```
当你为网络设置 VLAN ID 时，就是在为连接到该网络的任何容器终结点设置 VLAN 隔离。

**注意：**确保你的主机网络适配器（物理）处于 trunk 模式，以允许处于访问模式下的 vSwitch 在正确的 VLAN 上通过 vNIC（容器终结点）端口处理所有带标记的流量。


### 指定网络的 DNS 后缀和/或 DNS 服务器

使用选项 `-o com.docker.network.windowsshim.dnssuffix=<DNS SUFFIX>` 指定网络的 DNS 后缀，使用选项 `-o com.docker.network.windowsshim.dnsservers=<DNS SERVER/S>` 指定网络的 DNS 服务器。 例如，你可以使用以下命令将网络的 DNS 后缀设置为“example.com”，将网络的 DNS 服务器设置为 4.4.4.4 和 8.8.8.8：

```none
C:\> docker network create -d transparent -o com.docker.network.windowsshim.dnssuffix=abc.com -o com.docker.network.windowsshim.dnsservers=4.4.4.4,8.8.8.8 MyTransparentNetwork
```

### 多个容器网络
可在单个容器主机上创建多个容器网络，但应注意以下事项：

* 每个使用外部 vSwitch 进行连接的多个网络（例如透明、L2 桥接、L2 透明）必须使用其自身的网络适配器。
* 目前，在单个容器主机上创建多个 NAT 网络的解决方案是对现有 NAT 网络的内部前缀进行分区。 有关此方面的进一步指导，请参考下方“多个 NAT 网络”部分。

### 多个 NAT 网络
通过对主机的 NAT 网络内部前缀进行分区可以在单个容器主机上定义多个 NAT 网络。

必须在较大的内部 NAT 网络前缀下创建任何新 NAT 网络的分区。 通过在 PowerShell 中运行以下命令并引用“InternalIPInterfaceAddressPrefix”字段可以找到该前缀。

```none
PS C:\> Get-NetNAT
```

例如，主机的 NAT 网络内部前缀可能是 172.16.0.0/12。 在此情况下，可以使用 Docker 来创建其他 NAT 网络，*只要它们属于 172.16.0.0/12 前缀。* 例如，可以使用 IP 前缀 172.16.0.0/16 创建两个 NAT 网络（网关，172.16.0.1）和 172.17.0.0/16（网关，172.17.0.1）。

```none
C:\> docker network create -d nat --subnet=172.16.0.0/16 --gateway=172.16.0.1 CustomNat1
C:\> docker network create -d nat --subnet=172.17.0.0/16 --gateway=172.17.0.1 CustomNat2
```

使用以下选项可以列出新建的网络：
```none
C:\> docker network ls
```


### 网络选择

创建 Windows 容器时，可以指定容器网络适配器将连接到的网络。 如果不指定任何网络，则将使用默认的 NAT 网络。

若要将容器连接到非默认的 NAT 网络，请使用 --network 选项与 Docker 运行命令。

```none
C:\> docker run -it --network=MyTransparentNet windowsservercore cmd
```

### 静态 IP 地址

```none
C:\> docker run -it --network=MyTransparentNet --ip=10.80.123.32 windowsservercore cmd
```

静态 IP 分配直接在容器的网络适配器上执行，并且必须仅当容器处于已停止状态时执行。 容器运行时，不支持“热添加”容器网络适配器或更改网络堆栈。

## Docker Compose 和服务发现

> 有关如何使用 Docker Compose 和服务发现定义多服务、横向扩展应用程序的实例，请访问 [Virtualization Blog](https://blogs.technet.microsoft.com/virtualization/)（虚拟化博客）上的[此文章](https://blogs.technet.microsoft.com/virtualization/2016/10/18/use-docker-compose-and-service-discovery-on-windows-to-scale-out-your-multi-service-container-application/)。

### Docker Compose

[Docker Compose](https://docs.docker.com/compose/overview/) 可用于同时定义和配置容器网络以及将使用这些网络的容器/服务。 定义容器将连接的网络的过程中，Compose“网络”键可用作顶级键。 例如，以下语法将 Docker 创建的预先存在的 NAT 网络定义为在给定 Compose 文件中定义的所有容器/服务的“默认”网络。

```none
networks:
 default:
  external:
   name: "nat"
```

同样，以下语法可用于定义自定义 NAT 网络。

> 注意：下例中定义的“自定义 NAT 网络”是指容器主机预先存在的 NAT 内部前缀的分区。 有关更多上下文，请参阅上方“多个 NAT 网络”部分。

```none
networks:
  default:
    driver: nat
    ipam:
      driver: default
      config:
      - subnet: 172.17.0.0/16
```

有关使用 Docker Compose 定义/配置容器网络的详细信息，请参阅 [Compose File reference](https://docs.docker.com/compose/compose-file/)（Compose 文件引用）。

### 服务发现
内置于 Docker 的服务发现可为容器和服务处理服务注册和 IP (DNS) 映射的名称；通过服务发现，所有容器终结点都可以根据名称（容器名或服务名）发现对方。 这在使用多个容器终结点来定义单个服务的横向扩展方案中尤为有用。 在这种情况下，无论服务后台运行了多少个容器，都可以通过服务发现将服务视作单个实体。 对于多容器服务，采用循环法管理传入网络流量，通过该方法，可使用 DNS 负载平衡将流量统一分配到实现给定服务的所有容器实例。

## 覆盖网络和 Docker 群模式（多节点容器网络）
本机覆盖网络驱动程序和 Docker 群模式共同为 Windows 上的多节点（群集）方案提供支持。 要了解有关覆盖和群模式的详细信息，请访问我们在发布面向 Windows 10 预览体验成员的覆盖/群时随附的[博客文章](https://blogs.technet.microsoft.com/virtualization/2017/02/09/overlay-network-driver-with-support-for-docker-swarm-mode-now-available-to-windows-insiders-on-windows-10/)，或者参阅主题[群模式入门](./swarm-mode.md)。

## 注意事项和陷阱

### 阻止透明网络创建的现有 vSwitch

创建透明网络时，Docker 首先创建网络的外部 vSwitch，然后尝试将交换机绑定到（外部）网络适配器 - 该适配器可以是 VM 网络适配器或物理网络适配器。 如果已在容器主机上创建 vSwitch，*且对 Docker 可见*，Windows Docker 引擎将使用该交换机，而不是重新创建一个。 但是，如果该 vSwitch 创建于带外（即在容器主机上使用 HYper-V 管理器或 PowerShell 创建）并且对 Docker 不可见，Windows Docker 引擎将尝试创建一个新的 vSwitch，从而无法将新交换机连接到容器主机外部网络适配器（因为网络适配器已经连接到创建于带外的交换机）。

例如，如果在 Docker 服务运行期间首先在主机上创建新的 vSwitch，然后尝试创建透明网络，则会出现此问题。 在此情况下，Docker 无法识别创建的交换机，并会为透明网络创建新的 vSwitch。

解决此问题的方法有三种：

* 当然，可以删除带外创建的 vSwitch，这将允许 Docker 创建新的 vSwitch 并将其顺利连接到主机网络适配器。 选择此方法前，请确保其他服务（如 Hyper-V）未使用带外 vSwitch。
* 或者，如果决定使用创建于带外的外部 vSwitch，请重启 Docker 和 HNS 服务以*使交换机对 Docker 可见。*
```none
PS C:\> restart-service hns
PS C:\> restart-service docker
```
* 另一种方法是使用“-o com.docker.network.windowsshim.interface”选项以将透明网络的外部 vSwitch 绑定到未在容器主机上使用的特定网络适配器（即，除带外创建的 vSwitch 使用的其他网络适配器）。 有关“-o”选项的详细信息，请参阅本文档上方的[透明网络](https://msdn.microsoft.com/virtualization/windowscontainers/management/container_networking#transparent-network)部分。

### 不支持的功能

以下网络功能现无法通过 Docker CLI 受到支持
 * 容器链接（例如 --link）

以下网络选项目前在 Windows Docker 上不受支持：
 * --add-host
 * --dns-opt
 * --dns-search
 * --aux-address
 * --internal
 * --ip-range

