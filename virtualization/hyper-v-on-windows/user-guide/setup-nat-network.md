---
title: "设置 NAT 网络"
description: "设置 NAT 网络"
keywords: windows 10, hyper-v
author: jmesser81
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1f8a691c-ca75-42da-8ad8-a35611ad70ec
ms.openlocfilehash: ca951a356fbf11ed784b78f9742fd43da005e58c
ms.sourcegitcommit: bb171f4a858fefe33dd0748b500a018fd0382ea6
ms.translationtype: HT
ms.contentlocale: zh-CN
---
# <a name="set-up-a-nat-network"></a>设置 NAT 网络

Windows 10 Hyper-V 允许虚拟网络的本机网络地址转换 (NAT)。

本指南将引导你完成：
* 创建 NAT 网络
* 将现有虚拟机连接到新网络
* 确认虚拟机正确连接

要求：
* Windows 10 周年更新或更高版本
* 已启用 Hyper-V（单击[此处](../quick-start/enable-hyper-v.md) 查看相关说明）

> **注意：**目前，每台主机可以创建一个 NAT 网络。 有关 Windows NAT (WinNAT) 实现、功能和限制的更多详细信息，请参考 [WinNAT 功能和限制博客](https://blogs.technet.microsoft.com/virtualization/2016/05/25/windows-nat-winnat-capabilities-and-limitations/)

## <a name="nat-overview"></a>NAT 概述
NAT 使用主计算机的 IP 地址和端口通过内部 Hyper-V 虚拟开关向虚拟机授予对网络资源的访问权限。

网络地址转换 (NAT) 是一种网络模式，旨在通过将一个外部 IP 地址和端口映射到更大的内部 IP 地址集来转换 IP 地址。  基本上，NAT 使用流量表将流量从一个外部（主机）IP 地址和端口号路由到与网络上的终结点（虚拟机、计算机和容器等）关联的正确内部 IP 地址

此外，NAT 允许多个虚拟机托管需要相同（内部）通信端口的应用程序，方法是将它们映射到唯一的外部端口。

出于所有这些原因，NAT 网络对于容器技术是很常见的（请参阅[容器网络](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/management/container_networking)）。


## <a name="create-a-nat-virtual-network"></a>创建 NAT 虚拟网络
让我们演练新 NAT 网络的设置。

1.  以管理员身份打开 PowerShell 控制台。  

2. 创建内部开关  

  ``` PowerShell
  New-VMSwitch -SwitchName "SwitchName" -SwitchType Internal
  ```

3. 使用 [New-NetIPAddress](https://technet.microsoft.com/en-us/library/hh826150.aspx) 配置 NAT 网关。  

  下面是常规命令：
  ``` PowerShell
  New-NetIPAddress -IPAddress <NAT Gateway IP> -PrefixLength <NAT Subnet Prefix Length> -InterfaceIndex <ifIndex>
  ```

  若要配置网关，你将需要一些有关你的网络的信息：  
  * **IPAddress** - NAT 网关 IP 指定要用作 NAT 网关 IP 的 IPv4 或 IPv6 地址。  
    常规形式将为 a.b.c.1（例如 172.16.0.1）。  尽管最后一个位置不一定必须是.1，但通常是（基于前缀长度）

    通用网关 IP 为 192.168.0.1  

  * **PrefixLength** - NAT 子网前缀长度定义的 NAT 本地子网大小（子网掩码）。
    子网前缀长度将为 0 到 32 之间的整数值。

    0 将映射整个 Internet，32 将只允许一个映射的 IP。  常用值的范围为 24 到 12，具体取决于需要附加到 NAT 的 IP 数。

    常用 PrefixLength 为 24 -- 这是子网掩码 255.255.255.0

  * **InterfaceIndex** - ifIndex 是上述步骤中创建的虚拟交换机的接口索引。

    你可以通过运行以下命令查找接口索引 `Get-NetAdapter`

    你的输出应类似下面的形式：

    ```
    PS C:\> Get-NetAdapter

    Name                  InterfaceDescription               ifIndex Status       MacAddress           LinkSpeed
    ----                  --------------------               ------- ------       ----------           ---------
    vEthernet (intSwitch) Hyper-V Virtual Ethernet Adapter        24 Up           00-15-5D-00-6A-01      10 Gbps
    Wi-Fi                 Marvell AVASTAR Wireless-AC Net...      18 Up           98-5F-D3-34-0C-D3     300 Mbps
    Bluetooth Network ... Bluetooth Device (Personal Area...      21 Disconnected 98-5F-D3-34-0C-D4       3 Mbps

    ```

    内部交换机的名称将类似于 `vEthernet (SwitchName)`，接口描述将为 `Hyper-V Virtual Ethernet Adapter`。

  运行以下内容来创建 NAT 网关：

  ``` PowerShell
  New-NetIPAddress -IPAddress 192.168.0.1 -PrefixLength 24 -InterfaceIndex 24
  ```

4. 使用 [New-NetNat](https://technet.microsoft.com/en-us/library/dn283361(v=wps.630).aspx) 配置 NAT 网络。  

  下面是常规命令：

  ``` PowerShell
  New-NetNat -Name <NATOutsideName> -InternalIPInterfaceAddressPrefix <NAT subnet prefix>
  ```

  若要配置网关，你将需要提供一些有关网络和 NAT 网关的信息：  
  * **Name** - NATOutsideName 描述 NAT 网络的名称。  将使用此参数删除 NAT 网络。

  * **InternalIPInterfaceAddressPrefix** - NAT 子网前缀同时描述上述 NAT 网关 IP 前缀和上述 NAT 子网前缀长度。

    常规形式将为 a.b.c.0/NAT 子网前缀长度

    综上所述，对于本示例，我们将使用 192.168.0.0/24

  对于我们的示例，运行以下命令以设置 NAT 网络：

  ``` PowerShell
  New-NetNat -Name MyNATnetwork -InternalIPInterfaceAddressPrefix 192.168.0.0/24
  ```

祝贺你！  你现在已拥有一个虚拟 NAT 网络！  若要向 NAT 网络添加虚拟机，请按照[以下说明](#connect-a-virtual-machine)进行操作。

## <a name="connect-a-virtual-machine"></a>连接虚拟机

若要将虚拟机连接到新的 NAT 网络，请使用“VM 设置”菜单将你在 [NAT 网络设置](#create-a-nat-virtual-network)部分的第一步中创建的内部交换机连接到虚拟机。

由于 WinNAT 本身不会将 IP 地址分配给某个终结点（例如，VM），因此，你将需要从 VM 内手动完成此操作，即设置 NAT 内部前缀范围内的 IP 地址、设置默认网关 IP 地址，以及设置 DNS 服务器信息。 唯一需要注意的是终结点何时连接到容器。 在这种情况下，主机网络服务 (HNS) 分配并使用主机计算服务 (HCS) 直接向容器分配 IP 地址、网关 IP 和 DNS 信息。


## <a name="configuration-example-attaching-vms-and-containers-to-a-nat-network"></a>配置示例：将 VM 和容器连接到 NAT 网络

_如果你需要将多个 VM 和容器连接到单个 NAT，则需要确保 NAT 内部子网前缀大小足以包含由不同的应用程序或服务（例如用于 Windows 的 Docker 和 Windows 容器 – HNS）分配的 IP 范围。 这将需要应用程序级别的 IP 分配和网络配置，或者必须由管理员完成手动配置，从而保证不会在同一主机上重用现有 IP 分配。_

### <a name="docker-for-windows-linux-vm-and-windows-containers"></a>用于 Windows 的 Docker（Linux VM）和 Windows 容器
下面的解决方案将允许用于 Windows 的 Docker（运行 Linux 容器的 Linux VM）和 Windows 容器使用单独的内部 vSwitch 共享同一个 WinNAT 实例。 Linux 和 Windows 容器之间的连接将起作用。

用户已通过名为“VMNAT”的内部 vSwitch 将 VM 连接到 NAT 网络，现在想要使用 Docker 引擎安装 Windows 容器功能
```none
PS C:\> Get-NetNat “VMNAT”| Remove-NetNat (this will remove the NAT but keep the internal vSwitch).
Install Windows Container Feature
DO NOT START Docker Service (daemon)
Edit the arguments passed to the docker daemon (dockerd) by adding –fixed-cidr=<container prefix> parameter. This tells docker to create a default nat network with the IP subnet <container prefix> (e.g. 192.168.1.0/24) so that HNS can allocate IPs from this prefix.
PS C:\> Start-Service Docker; Stop-Service Docker
PS C:\> Get-NetNat | Remove-NetNAT (again, this will remove the NAT but keep the internal vSwitch)
PS C:\> New-NetNat -Name SharedNAT -InternalIPInterfaceAddressPrefix <shared prefix>
PS C:\> Start-Service docker
```
Docker/HNS 将 IP 分配给 <container prefix> 的 Windows 容器，管理员将 IP 分配给 <shared prefix> 差异集的 VM 以及 <container prefix>

用户已通过运行 docker 引擎安装了 Windows 容器功能，现在想要将 VM 连接到 NAT 网络
```none
PS C:\> Stop-Service docker
PS C:\> Get-ContainerNetwork | Remove-ContainerNetwork -force
PS C:\> Get-NetNat | Remove-NetNat (this will remove the NAT but keep the internal vSwitch)
Edit the arguments passed to the docker daemon (dockerd) by adding -b “none” option to the end of docker daemon (dockerd) command to tell docker not to create a default NAT network.
PS C:\> New-ContainerNetwork –name nat –Mode NAT –subnetprefix <container prefix> (create a new NAT and internal vSwitch – HNS will allocate IPs to container endpoints attached to this network from the <container prefix>)
PS C:\> Get-Netnat | Remove-NetNAT (again, this will remove the NAT but keep the internal vSwitch)
PS C:\> New-NetNat -Name SharedNAT -InternalIPInterfaceAddressPrefix <shared prefix>
PS C:\> New-VirtualSwitch -Type internal (attach VMs to this new vSwitch)
PS C:\> Start-Service docker
```
Docker/HNS 将 IP 分配给 <container prefix> 的 Windows 容器，管理员将 IP 分配给 <shared prefix> 差异集的 VM 以及 <container prefix>

最终，你应该有两个内部 VM 开关以及在这两个开关之间共享的一个 NetNat。

## <a name="multiple-applications-using-the-same-nat"></a>多个应用程序使用相同的 NAT

某些情况下需要多个应用程序或服务使用同一个 NAT。 在这种情况下，必须遵循以下工作流，以便多个应用程序/服务可以使用更大的 NAT 内部子网前缀

**_我们将以与 Windows 容器功能共存于同一主机上的 Docker 4 Windows - Docker Beta - Linux VM 作为示例，对其进行详细介绍。 此工作流可能会随时发生变化_**

1. C:\> net stop docker
2. 停止 Docker4Windows MobyLinux VM
3. PS C:\> Get-ContainerNetwork | Remove-ContainerNetwork -force
4. PS C:\> Get-NetNat | Remove-NetNat  
   *删除任何先前存在的容器网络（即删除 vSwitch、删除 NetNat、清理）*  

5. New-ContainerNetwork -Name nat -Mode NAT –subnetprefix 10.0.76.0/24（此子网将用于 Windows 容器功能）*创建名为 nat 的内部 vSwitch*  
   *创建名为“nat”、IP 前缀为 10.0.76.0/24 的 NAT 网络*  

6. Remove-NetNAT  
   *删除 DockerNAT 和 nat NAT 网络（保留内部 vSwitch）*  

7. New-NetNat -Name DockerNAT -InternalIPInterfaceAddressPrefix 10.0.0.0/17（这将创建较大的 NAT 网络，以供 D4W 和容器共享）  
   *创建名为“DockerNAT”的 NAT 网络，且该网络采用较大的前缀 10.0.0.0/17*  

8. 运行 Docker4Windows (MobyLinux.ps1)  
   *创建内部 vSwitch DockerNAT*  
   *创建名为“DockerNAT”、IP 前缀为 10.0.75.0/24 的 NAT 网络*  

9. Net start docker  
   *Docker 将使用用户定义的 NAT 网络作为默认网络连接到 Windows 容器*  

最终，你将具有两个内部 vSwitch – 一个名为 DockerNAT，另一个名为 nat。 你仅将拥有一个 NAT 网络 (10.0.0.0/17)，可通过运行 Get-NetNat 确认。 Windows 容器的 IP 地址将由 Windows 主机网络服务 (HNS) 从 10.0.76.0/24 子网分配。 基于现有 MobyLinux.ps1 脚本，将从 10.0.75.0/24 子网分配 Docker 4 Windows 的 IP 地址。


## <a name="troubleshooting"></a>疑难解答

### <a name="multiple-nat-networks-are-not-supported"></a>不支持多个 NAT 网络  
本指南假设主机上没有其他 NAT。 但是，应用程序或服务将要求使用一个 NAT，并且有可能创建一个 NAT，使之作为安装的一部分。 由于 Windows (WinNAT) 仅支持一个内部 NAT 子网前缀，因此尝试创建多个 NAT 将使系统处于未知状态。

若想知道这是否是个问题，请确保你只有一个 NAT：
``` PowerShell
Get-NetNat
```

如果已存在一个 NAT，请将其删除
``` PowerShell
Get-NetNat | Remove-NetNat
```
请确保应用程序或功能（例如 Windows 容器）仅有一个“内部”vmSwitch。 记录 vSwitch 的名称
``` PowerShell
Get-VMSwitch
```

检查是否仍存在从旧 NAT 向适配器分配的专用 IP 地址（例如 NAT 默认网关 IP 地址 – 通常为 *.1）
``` PowerShell
Get-NetIPAddress -InterfaceAlias "vEthernet(<name of vSwitch>)"
```

如果旧专用 IP 地址仍在使用中，请将其删除
``` PowerShell
Remove-NetIPAddress -InterfaceAlias "vEthernet(<name of vSwitch>)" -IPAddress <IPAddress>
```

**删除多个 NAT**  
我们看到无意中创建了多个 NAT 网络的报告。 这是由于最近的版本（包括 Windows Server 2016 Technical Preview 5 和 Windows 10 Insider Preview 版本）存在一个 bug。 如果你看到多个 NAT 网络，在运行 docker network ls 或 Get-ContainerNetwork 之后，请在提升的 PowerShell 中执行以下操作：

```none
PS> $KeyPath = "HKLM:\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\SwitchList"
PS> $keys = get-childitem $KeyPath
PS> foreach($key in $keys)
PS> {
PS>    if ($key.GetValue("FriendlyName") -eq 'nat')
PS>    {
PS>       $newKeyPath = $KeyPath+"\"+$key.PSChildName
PS>       Remove-Item -Path $newKeyPath -Recurse
PS>    }
PS> }
PS> remove-netnat -Confirm:$false
PS> Get-ContainerNetwork | Remove-ContainerNetwork
PS>    Get-VmSwitch -Name nat | Remove-VmSwitch (_failure is expected_)
PS>    Stop-Service docker
PS> Set-Service docker -StartupType Disabled
Reboot Host
PS> Get-NetNat | Remove-NetNat
PS> Set-Service docker -StartupType automaticac
PS> Start-Service docker 
```

如有必要，请参阅 [setup guide for multiple applications using the same NAT](#multiple-applications-using-the-same-nat)（使用同一个 NAT 的多个应用程序的安装指南）以重新生成你的 NAT 环境。 

## <a name="references"></a>参考
详细了解 [NAT 网络](https://en.wikipedia.org/wiki/Network_address_translation)
