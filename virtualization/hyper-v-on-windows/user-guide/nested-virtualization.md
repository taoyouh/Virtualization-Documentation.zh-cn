---
title: "嵌套虚拟化"
description: "嵌套虚拟化"
keywords: windows 10, hyper-v
author: theodthompson
ms.date: 06/20/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 68c65445-ce13-40c9-b516-57ded76c1b15
translationtype: Human Translation
ms.sourcegitcommit: ffdf89b0ae346197b9ae631ee5260e0565261c55
ms.openlocfilehash: b0903a3e00b92bc60e26282dde072b66030ada09

---

# 借助嵌套虚拟化在虚拟机中运行 Hyper-V

嵌套虚拟化是一项功能，使你可以在 Hyper-V 虚拟机内运行 Hyper-V。 换而言之，借助嵌套虚拟化，Hyper-V 主机本身可进行虚拟化。 嵌套虚拟化的一些用例包括在虚拟化容器主机中运行 Hyper-V 容器、在虚拟化环境中设置 Hyper-V 实验室或者无需单个硬件测试多台计算机方案。 本文档将详细介绍软件和硬件先决条件、配置步骤和限制。 

## 先决条件

- 运行 Windows Server 2016 或 Windows 10 周年更新的 Hyper-V 主机。
- 运行 Windows Server 2016 或 Windows 10 周年更新的 Hyper-V VM。
- 配置版本为 8.0 或更高的 Hyper-V VM。
- 采用 VT-x 和 EPT 技术的 Intel 处理器。

## 配置嵌套虚拟化

1. 创建虚拟机。 请参阅以上针对必需 OS 和 VM 版本的先决条件。
2. 虚拟机处于关闭状态时，请在物理 Hyper-V 主机上运行以下命令。 这样可以实现虚拟机的嵌套虚拟化。

```none
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $true
```
3. 启动虚拟机。
4. 在虚拟机中安装 Hyper-V，就像针对物理服务器一样。 有关安装 Hyper-V 的详细信息，请参阅[安装 Hyper-V]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_install)。

## 禁用嵌套虚拟化
可使用以下 PowerShell 命令禁用已停止虚拟机的嵌套虚拟化：
```none
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $false
```

## 动态内存和运行时内存大小调整
当 Hyper-V 在虚拟机内运行时，必须关闭虚拟机以调整其内存。 这意味着，即使启用了动态内存，内存量将不会波动。 对于没有启用动态内存的虚拟机，在虚拟机开启情况下对调整内存量的任何尝试都将失败。 

请注意，只启用嵌套虚拟化不会影响动态内存或者运行时内存大小调整。 仅当 Hyper-V 在 VM 中运行时会出现不兼容。

## 网络连接选项
有两个选项可用于与嵌套虚拟机连接：MAC 地址欺骗和 NAT 模式。

### MAC 地址欺骗
为了通过两台虚拟交换机路由网络数据包，必须在第一级虚拟交换机上启用 MAC 地址欺骗。 使用以下 PowerShell 命令完成此操作。

```none
Get-VMNetworkAdapter -VMName <VMName> | Set-VMNetworkAdapter -MacAddressSpoofing On
```
### 网络地址转换
第二个选项依赖于网络地址转换 (NAT)。 此方法非常适合于无法使用 MAC 地址欺骗的情况，例如在公有云环境中。

首先，必须在主机虚拟机（“中间”虚拟机）中创建一个虚拟 NAT 交换机。 请注意，IP 地址仅作举例之用，会因环境不同有所差异：
```none
new-vmswitch -name VmNAT -SwitchType Internal
New-NetNat –Name LocalNAT –InternalIPInterfaceAddressPrefix “192.168.100.0/24”
```
接下来，将 IP 地址分配给网络适配器：
```none
get-netadapter "vEthernet (VmNat)" | New-NetIPAddress -IPAddress 192.168.100.1 -AddressFamily IPv4 -PrefixLength 24
```
每个嵌套的虚拟机必须分配有一个 IP 地址和网关。 请注意，网关 IP 必须指向上一步中的 NAT 适配器。 您可能想要分配 DNS 服务器：
```none
get-netadapter "Ethernet" | New-NetIPAddress -IPAddress 192.168.100.2 -DefaultGateway 192.168.100.1 -AddressFamily IPv4 -PrefixLength 24
Netsh interface ip add dnsserver “Ethernet” address=<my DNS server>
```

## 第三方虚拟化应用
除 Hyper-V 外的虚拟化应用程序在 Hyper-V 虚拟机中不受支持，可能会失败。 这包括需要硬件虚拟化扩展的任何软件。



<!--HONumber=Oct16_HO4-->


