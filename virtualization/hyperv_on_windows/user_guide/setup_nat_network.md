---
title: 设置 NAT 网络
description: 设置 NAT 网络
keywords: windows 10, hyper-v
author: jmesser81
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1f8a691c-ca75-42da-8ad8-a35611ad70ec
---

# 设置 NAT 网络

Windows 10 Hyper-V 允许虚拟网络的本机网络地址转换 (NAT)。

本指南将引导你完成：
* 创建 NAT 网络
* 将现有虚拟机连接到新网络
* 确认虚拟机正确连接

要求：
* Windows 版本 14295 或更高版本
* 已启用 Hyper-V 角色（单击[此处](../quick_start/walkthrough_create_vm.md)查看相关说明）

> **注意：**当前，Hyper-V 仅允许你创建一个 NAT 网络。

## NAT 概述
NAT 使用主计算机的 IP 地址和端口向虚拟机授予对网络资源的访问权限。

网络地址转换 (NAT) 是一种网络模式，旨在通过将一个外部 IP 地址和端口映射到更大的内部 IP 地址集来转换 IP 地址。  基本上，NAT 开关使用 NAT 映射表将流量从一个 IP 地址和端口号路由到与网络上的设备（虚拟机、计算机和容器等）关联的正确内部 IP 地址

此外，NAT 允许多个虚拟机托管需要相同（内部）通信端口的应用程序，方法是将它们映射到唯一的外部端口。

出于所有这些原因，NAT 网络对于容器技术是很常见的（请参阅[容器网络](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/management/container_networking)）。


## 创建 NAT 虚拟网络
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

祝贺你！  你现在已拥有一个虚拟 NAT 网络！  若要向 NAT 网络添加虚拟机，请按照[以下说明](setup_nat_network.md#connect-a-virtual-machine)进行操作。

## 连接虚拟机

若要将虚拟机连接到新的 NAT 网络，请使用“VM 设置”菜单将你在 [NAT 网络设置](setup_nat_network.md#create-a-nat-virtual-network)部分的第一步中创建的内部交换机连接到虚拟机。


## 疑难解答

此工作流假设主机上没有其他 NAT。 但是，有时多个应用程序或服务将需要使用 NAT。 由于 Windows (WinNAT) 仅支持一个内部 NAT 子网前缀，因此尝试创建多个 NAT 将使系统处于未知状态。

### 疑难解答步骤
1. 请确保你只有一个 NAT

  ``` PowerShell
  Get-NetNat
  ```
2. 如果 NAT 已经存在，请将其删除

  ``` PowerShell
  Get-NetNat | Remove-NetNat
  ```

3. 请确保你只有一个用于 NAT 的“内部”vmSwitch。 记录 vSwitch 的名称，以便步骤 4 使用

  ``` PowerShell
  Get-VMSwitch
  ```

4. 检查是否仍存在从旧 NAT 向适配器分配的专用 IP 地址（例如 NAT 默认网关 IP 地址 - 通常为 *.1）

  ``` PowerShell
  Get-NetIPAddress -InterfaceAlias "vEthernet(<name of vSwitch>)"
  ```

5. 如果旧专用 IP 地址仍在使用中，请将其删除  
   ``` PowerShell
  Remove-NetIPAddress -InterfaceAlias "vEthernet(<name of vSwitch>)" -IPAddress <IPAddress>
  ```

## 多个应用程序使用相同的 NAT

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


## 参考
详细了解 [NAT 网络](https://en.wikipedia.org/wiki/Network_address_translation)


<!--HONumber=May16_HO5-->


