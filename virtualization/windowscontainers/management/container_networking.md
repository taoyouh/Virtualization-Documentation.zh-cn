---
title: "Windows 容器网络"
description: "为 Windows 容器配置网络。"
keywords: "docker, 容器"
author: jmesser81
manager: timlt
ms.date: 08/22/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
translationtype: Human Translation
ms.sourcegitcommit: 7b5cf299109a967b7e6aac839476d95c625479cd
ms.openlocfilehash: 2e26177f3e653e9102dc91070b987e28ef713bed

---

# 容器网络

关于网络，Windows 容器的作用类似于虚拟机。 每个容器都有一个连接到虚拟交换机的虚拟网络适配器，通过该适配器可转发入站和出站流量。 为了强制隔离同一主机上的各容器，将为每个 Windows Server 和 Hyper-V 容器创建网络隔离舱，将该容器的网络适配器安装在其中。 Windows Server 容器使用主机 vNIC 连接到虚拟交换机。 Hyper-V 容器使用合成 VM NIC（不公开到实用工具 VM）连接到虚拟交换机。

Windows 容器支持四种不同的网络驱动程序或模式：*nat**transparent**l2bridge*和 *l2tunnel*。 根据物理网络基础结构和单 VS 多主机网络要求，应该选择最符合你需要的网络模式。 

首次运行 dockerd 服务时，Docker 引擎将在默认情况下创建一个 nat 网络。 创建的默认内部 IP 前缀为 172.16.0.0/12。 

> 注意：如果容器主机 IP 也具有此相同的前缀，则需要按如下所述更改 NAT 内部 IP 前缀

容器终结点将附加到此默认网络，并将获得一个由内部前缀分配的 IP 地址。 Windows 当前仅支持一个 NAT 网络（尽管挂起的[拉取请求](https://github.com/docker/docker/pull/25097)有助于解决此限制）。 

可以在同一容器主机上创建使用不同驱动程序（例如 transparent、l2bridge）的其他网络。 下表显示了如何为每种模式的内部连接（容器间连接）和外部连接提供网络连接。

- **网络地址转换** - 每个容器都将收到来自内部专用 IP 前缀（例如 172.16.0.0/12）的一个 IP 地址。 支持从容器主机到容器终结点的端口转发/映射

- **透明** - 每个容器终结点直接连接到物理网络。 可使用外部 DHCP 服务器静态或动态分配来自物理网络的 IP

- **L2 桥接 **每个容器终结点都将与容器主机位于同一 IP 子网。 必须从与容器主机相同的前缀中静态分配 IP 地址。 由于第 2 层地址转换，主机上的所有容器终结点都将具有相同的 MAC 地址。

- **L2 隧道模式 ** - _这种模式应仅用于 Microsoft 云堆栈_

> 若要了解如何通过 Microsoft SDN 堆栈将容器终结点连接到覆盖虚拟网络，请参阅 [Attaching Containers to a Virtual Network](https://technet.microsoft.com/en-us/windows-server-docs/networking/sdn/manage/connect-container-endpoints-to-a-tenant-virtual-network)（将容器附加到虚拟网络）主题。

## 单节点

|  | 容器 - 容器 | 容器 - 外部 |
| :---: | :---------------     |  :---                |
| nat | 通过 Hyper-V 虚拟交换机进行桥接 | 通过应用了地址转换的 WinNAT 进行路由 | 
| transparent | 通过 Hyper-V 虚拟交换机进行桥接 | 直接访问物理网络 | 
| l2bridge | 通过 Hyper-V 虚拟交换机进行桥接|  通过 MAC 地址转换访问物理网络|  



## 多节点

|  | 容器 - 容器 | 容器 - 外部 |
| :---: | :----       | :---------- |
| nat | 必须引用外部容器主机 IP 和端口；通过应用了地址转换的 WinNAT 进行路由 | 必须引用外部主机；通过应用了地址转换的 WinNAT 进行路由 | 
| transparent | 必须直接引用容器 IP 终结点 | 直接访问物理网络 | 
| l2bridge | 必须直接引用容器 IP 终结点| 通过 MAC 地址转换访问物理网络| 


## 网络的创建 

### （默认）NAT 网络

Windows Docker 引擎使用 IP 前缀 172.16.0.0/12 创建默认“nat”网络。 当前 Windows 容器主机上仅允许存在一个 nat 网络。 如果用户想要创建一个具有特定 IP 前缀的 nat 网络，则可通过更改 docker 的配置文件 daemon.json（位于 C:\ProgramData\Docker\config\daemon.json（如果不存在，创建一个））中的选项，执行两个操作中的任意一个。
 1. 使用 _"fixed-cidr": "< IP Prefix > / Mask"_ 选项将创建默认 nat 网络，其中已指定 IP 前缀和匹配项
 2. 使用 _"bridge": "none"_ 将不会创建默认网络；用户可以通过 *docker network create -d <driver>* 命令使用任意驱动程序来创建一个用户定义网络

在执行这些配置选项中的任意一个选项之前，必须首先停止 Docker 服务，并删除任何预先存在的 nat 网络。

```none
PS C:\> Stop-Service docker
PS C:\> Get-ContainerNetwork | Remove-ContainerNetwork

...Edit the daemon.json file...

PS C:\> Start-Service docker
```

如果用户已将 "fixed-cidr" 选项添加到了 daemon.json 文件，docker 引擎将在此时创建一个用户定义 nat 网络，并指定自定义 IP 前缀和掩码。 如果用户添加了 "bridge:none" 选项，则需要手动创建一个网络。

```none
# Create a user-defined nat network
C:\> docker network create -d nat --subnet=192.168.1.0/24 --gateway=192.168.1.1 MyNatNetwork
```

默认情况下，容器终结点将连接到默认 nat 网络。 如果未创建默认 nat 网络（因为已在 daemon.json 中指定 "bridge:none"），或如果要求对不同的、用户定义的网络具有访问权限，则用户可指定具有 docker 运行命令的 *--network*参数。

```none
# Connect new container to the MyNatNetwork
C:\> docker run -it --network=MyNatNetwork <image> <cmd>
```

#### 端口映射

若要访问在连接到 NAT 网络的容器内运行的应用程序，则需要在容器主机和容器终结点之间创建端口映射。 必须在容器处于创建时间或停止状态时指定这些映射。

```none
# Creates a static mapping between port TCP:80 of the container host and TCP:80 of the container
C:\> docker run -it -p 80:80 <image> <cmd>

# Create a static mapping between port 8082 of the container host and port 80 of the container.
C:\> docker run -it -p 8082:80 windowsservercore cmd
```

此外，通过使用 -p 参数或在具有 -p 参数的 Dockerfile 中使用 EXPOSE 命令，还支持动态端口映射。 将在容器主机上选择一个随机选择的临时端口，并在运行 Docker ps 时进行检查。

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
 1. 每个容器主机仅支持一个 NAT 网络（一个内部 IP 前缀）
 2. 容器终结点仅可从使用内部 IP 和端口的容器主机处访问

可以使用不同的驱动程序创建其他网络。 

> Docker 网络驱动程序为全部小写。

### 透明网络

若要使用透明网络模式，使用驱动程序名称“透明”创建容器网络。 

```none
C:\> docker network create -d transparent MyTransparentNetwork
```

如果容器主机被虚拟化，而你想要使用 DHCP 进行 IP 分配，则必须在虚拟机网络适配器上启用 MACAddressSpoofing。 否则，Hyper-V 主机将会阻止来自容器（位于具有多个 MAC 地址的 VM 中）的网络流量。

```none
PS C:\> Get-VMNetworkAdapter -VMName ContainerHostVM | Set-VMNetworkAdapter -MacAddressSpoofing On
```

> 如果希望创建多个透明网络，则必须指定（自动创建的）外部 Hyper-V 虚拟交换机应绑定到的（虚拟）网络适配器。

若要将网络（通过 Hyper-V 虚拟交换机连接）绑定到特定网络接口，请使用选项 *-o com.docker.network.windowsshim.interface=<Interface>*

```none
# Create a transparent network which is attached to the "Ethernet 2" network interface
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" TransparentNet2
```

*com.docker.network.windowsshim.interface* 的值为适配器的名称，来自： 
```none
Get-NetAdapter
```

连接到透明网络的容器终结点的 IP 地址可以通过一个外部 DHCP 服务器静态或动态进行分配。

使用静态 IP 分配时，必须首先确保已在创建网络时指定 *--subnet* 和 *--gateway* 参数。 子网和网关 IP 地址应该与容器主机的网络设置（即物理网络）相同。

```none
# Create a transparent network corresponding to the physical network with IP prefix 10.123.174.0/23
C:\> docker network create -d transparent --subnet=10.123.174.0/23 --gateway=10.123.174.1 TransparentNet3
```
请使用 *--ip* 选项指定 IP 地址，以便 docker 运行命令

```none
C:\> docker run -it --network=TransparentNet3 --ip 10.123.174.105 <image> <cmd>
```

> 请确保此 IP 地址未分配到任何物理网络上的其他网络设备

由于容器终结点拥有对物理网络的直接访问权限，因此无需指定端口映射

### L2 桥接 

若要使用 L2 桥接网络模式，使用驱动程序名称“l2bridge”创建容器网络。 再次强调，必须指定对应于物理网络的子网和网关。

```none
C:\> docker network create -d l2bridge --subnet=192.168.1.0/24 --gateway=192.168.1.1 MyBridgeNetwork
```

L2 桥接网络仅支持静态 IP 分配。 

> 当在 SDN 结构上使用 L2 桥接网络时，仅支持动态 IP 分配。 请参阅 [Attaching Containers to a Virtual Network](https://technet.microsoft.com/en-us/windows-server-docs/networking/sdn/manage/connect-container-endpoints-to-a-tenant-virtual-network)（将容器附加到虚拟网络）主题，以了解详细信息。

## 其他操作和配置

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
C:\> docker network rm "<network name>"
```

这将清除容器网络所用的所有 Hyper-V 虚拟交换机，以及所创建的任何网络地址转换（WinNAT - NetNat 实例）。

### 网络检查 

若要查看哪些容器连接到特定网络和与这些容器终结点相关联的 IP，可以运行以下命令。

```none
C:\> docker network inspect <network name>
```

### 多个容器网络

多个容器网络可在单个容器主机上进行创建，但注意以下内容：
* 每个容器主机只可创建一个 NAT 网络。
* 每个使用外部 vSwitch 进行连接的多个网络（例如透明、L2 桥接、L2 透明）必须使用其自身的网络适配器。

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


## 注意事项和陷阱

### 防火墙

容器主机需要创建特定防火墙规则用于启用 ICMP (Ping) 和 DHCP。 Windows Server 容器需要 ICMP 和 DHCP 以在同一主机上两个容器之间进行 ping 操作，以及接收通过 DHCP 动态分配的 IP 地址。 在 TP5 中，这些规则将通过 Install-ContainerHost.ps1 脚本进行创建。 TP5 之后，这些规则将自动创建。 NAT 端口转发规则所对应的所有防火墙规则将自动创建，并在容器停止后清除。

### 不支持的功能

以下网络功能现无法通过 Docker CLI 受到支持
 * 容器链接（例如 --link）
 * 容器的基于名称/服务的 IP 解析。 _服务更新很快就会提供此支持_
 * 默认覆盖网络驱动程序

以下网络选项在 Windows Docker 上现不受支持：
 * --add-host
 * --dns
 * --dns-opt
 * --dns-search
 * -h，--hostname
 * --net-alias
 * --aux-address
 * --internal
 * --ip-range

 > 在 Windows Server 2016 Technical Preview 5 和最新的 Windows Insider Preview (WIP)“外部测试版”版本中存在一个已知的 bug，即升级到新版本会导致生成一个重复（即“泄漏”）的容器网络和 vSwitch。 为解决此问题，请运行以下脚本。
```none
$KeyPath = "HKLM:\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\SwitchList"
$keys = get-childitem $KeyPath
foreach($key in $keys)
{
   if ($key.GetValue("FriendlyName") -eq 'nat')
   {
      $newKeyPath = $KeyPath+"\"+$key.PSChildName
      Remove-Item -Path $newKeyPath -Recurse
   }
}
remove-netnat -Confirm:$false
Get-ContainerNetwork | Remove-ContainerNetwork
Get-VmSwitch -Name nat | Remove-VmSwitch # Note: failure is expected
Stop-Service docker
Set-Service docker -StartupType Disabled
```
> 重启主机，然后运行剩余的步骤：
```none
Get-NetNat | Remove-NetNat -Confirm $false
Set-Service docker -StartupType automatic
Start-Service docker 
```



<!--HONumber=Aug16_HO4-->


