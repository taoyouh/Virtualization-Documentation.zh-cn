---
title: "嵌套虚拟化"
description: "嵌套虚拟化"
keywords: windows 10, hyper-v
author: johncslack
ms.date: 12/18/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 68c65445-ce13-40c9-b516-57ded76c1b15
ms.openlocfilehash: d5b8e888b62495c98c0691dc0d62deecf7c1eb6e
ms.sourcegitcommit: 6eefb890f090a6464119630bfbdc2794e6c3a3df
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/18/2017
---
# <a name="run-hyper-v-in-a-virtual-machine-with-nested-virtualization"></a>借助嵌套虚拟化在虚拟机中运行 Hyper-V

嵌套虚拟化是一项功能，使你可以在 Hyper-V 虚拟机 (VM) 内运行 Hyper-V。 这适用于在虚拟机中运行 Visual Studio 手机仿真器或测试通常需要多台主机的配置。

![](./media/HyperVNesting.png)

## <a name="prerequisites"></a>必备条件

* Hyper-V 主机和来宾必须都是 Windows Server 2016/Windows 10 周年更新或更高版本。
* VM 配置版本 8.0 或更高版本。
* 采用 VT-x 和 EPT 技术的 Intel 处理器 - 嵌套目前**仅限于 Intel**。
* 与二级虚拟机的虚拟网络存在一些差异。 请参阅“嵌套的虚拟机网络”。


## <a name="configure-nested-virtualization"></a>配置嵌套虚拟化

1. 创建虚拟机。 请参阅以上针对必需 OS 和 VM 版本的先决条件。
2. 虚拟机处于关闭状态时，请在物理 Hyper-V 主机上运行以下命令。 这样可以实现虚拟机的嵌套虚拟化。

```
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $true
```
3. 启动虚拟机。
4. 在虚拟机中安装 Hyper-V，就像针对物理服务器一样。 有关安装 Hyper-V 的详细信息，请参阅[安装 Hyper-V](../quick-start/enable-hyper-v.md)。

## <a name="disable-nested-virtualization"></a>禁用嵌套虚拟化
可使用以下 PowerShell 命令禁用已停止虚拟机的嵌套虚拟化：
```
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $false
```

## <a name="dynamic-memory-and-runtime-memory-resize"></a>动态内存和运行时内存大小调整
当 Hyper-V 在虚拟机内运行时，必须关闭虚拟机以调整其内存。 这意味着，即使启用了动态内存，内存量将不会波动。 对于没有启用动态内存的虚拟机，在虚拟机开启情况下对调整内存量的任何尝试都将失败。 

请注意，只启用嵌套虚拟化不会影响动态内存或者运行时内存大小调整。 仅当 Hyper-V 在 VM 中运行时会出现不兼容。

## <a name="networking-options"></a>网络连接选项

以下两个选项适用于嵌套虚拟机网络： 

1. MAC 地址欺骗
2. NAT 网络

### <a name="mac-address-spoofing"></a>MAC 地址欺骗
为了通过两台虚拟交换机路由网络数据包，必须在第一级虚拟交换机上启用 MAC 地址欺骗。 此操作使用以下 PowerShell 命令完成。

``` PowerShell
Get-VMNetworkAdapter -VMName <VMName> | Set-VMNetworkAdapter -MacAddressSpoofing On
```

### <a name="network-address-translation-nat"></a>网络地址转换 (NAT)
第二个选项依赖于网络地址转换 (NAT)。 此方法非常适合于无法使用 MAC 地址欺骗的情况，例如在公有云环境中。

首先，必须在主机虚拟机（“中间”虚拟机）中创建一个虚拟 NAT 交换机。 请注意，IP 地址仅作举例之用，会因环境不同有所差异：

``` PowerShell
New-VMSwitch -Name VmNAT -SwitchType Internal
New-NetNat –Name LocalNAT –InternalIPInterfaceAddressPrefix “192.168.100.0/24”
```

接下来，将 IP 地址分配给网络适配器：

``` PowerShell
Get-NetAdapter "vEthernet (VmNat)" | New-NetIPAddress -IPAddress 192.168.100.1 -AddressFamily IPv4 -PrefixLength 24
```

每个嵌套的虚拟机必须分配有一个 IP 地址和网关。 请注意，网关 IP 必须指向上一步中的 NAT 适配器。 你也可能想要分配 DNS 服务器：

``` PowerShell
Get-NetAdapter "Ethernet" | New-NetIPAddress -IPAddress 192.168.100.2 -DefaultGateway 192.168.100.1 -AddressFamily IPv4 -PrefixLength 24
Netsh interface ip add dnsserver “Ethernet” address=<my DNS server>
```

## <a name="how-nested-virtualization-works"></a>嵌套虚拟化的工作方式

现代处理器包括可使虚拟化更快且更安全的硬件功能。 Hyper-V 依赖这些处理器扩展来运行虚拟机（例如 Intel VT-x 和 AMD-V）。 通常，Hyper-V 启动后，它会阻止其他软件使用这些处理器功能。  这会阻止来宾虚拟机运行 Hyper-V。

嵌套式虚拟化会向来宾虚拟机提供此硬件支持。

下图显示了没有嵌套的 Hyper-V。  Hyper-V 虚拟机监控程序完全控制硬件虚拟化功能（橙色箭头），并且不会向来宾操作系统公开它们。

![](./media/HVNoNesting.png)

相反，下图显示了已启用嵌套虚拟化的 Hyper-V。 在此情况下，Hyper-V 会向其虚拟机公开硬件虚拟化扩展。 启用嵌套后，来宾虚拟机可以安装其自己的虚拟机监控程序并运行其自己的来宾 VM。

## <a name="3rd-party-virtualization-apps"></a>第三方虚拟化应用

除 Hyper-V 外的虚拟化应用程序在 Hyper-V 虚拟机中不受支持，可能会失败。 这包括需要硬件虚拟化扩展的任何软件。
