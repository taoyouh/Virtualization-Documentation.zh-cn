---
title: "嵌套虚拟化"
description: "嵌套虚拟化"
keywords: windows 10, hyper-v
author: theodthompson
manager: timlt
ms.date: 06/20/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 68c65445-ce13-40c9-b516-57ded76c1b15
ms.sourcegitcommit: ef18b63c454b3c12a7067d3604ba142d55403226
ms.openlocfilehash: 2d1679ffe4876ddd4eefe1b457098e8797899492

---

# 借助嵌套虚拟化在虚拟机中运行 Hyper-V

嵌套虚拟化是一项功能，使你可以在 Hyper-V 虚拟机内运行 Hyper-V。 换而言之，借助嵌套虚拟化，Hyper-V 主机本身可进行虚拟化。 嵌套虚拟化的一些用例包括在虚拟化容器主机中运行 Hyper-V 容器、在虚拟化环境中设置 Hyper-V 实验室或者无需单个硬件测试多台计算机方案。 本文档将详细介绍软件和硬件先决条件、配置步骤，并提供故障排除的详细信息。

## 先决条件

- 运行 Windows 预览体验成员版本（Windows Server 2016 或 Windows 10）或版本 10565 或更高版本的 Hyper-V 主机。
- 这两个虚拟机监控程序（父级与子级）都必须运行相同的 Windows 版本（10565 或更高版本）。
- 采用 Intel VT-x 和 EPT 技术的 Intel 处理器。

## 配置嵌套虚拟化

首先，创建一台虚拟机，**不要打开虚拟机**。 有关详细信息，请参阅[创建虚拟机](../quick_start/walkthrough_create_vm.md)。

创建虚拟机后，请在物理 HYPER-V 主机上运行以下命令。 这样就可以在虚拟机上实现嵌套虚拟化。

```none
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $true
```
运行嵌套 Hyper-V 主机时，必须在虚拟机上禁用动态内存。 这可以通过虚拟机的属性，或者通过使用以下 PowerShell 命令进行配置。
```none
Set-VMMemory -VMName <VMName> -DynamicMemoryEnabled $false
```

完成这些步骤后，可以启动虚拟机并安装 Hyper-V。 有关安装 Hyper-V 的详细信息，请参阅[安装 Hyper-V]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_install)。

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


## 已知问题

- 启用了 Device Guard 的主机无法向来宾公开虚拟化扩展。
- 已启用基于虚拟化的安全 (VBS) 的虚拟机不能同时启用嵌套。 若要使用嵌套虚拟化，必须先禁用 VBS。
- 在虚拟机中启用嵌套虚拟化后，以下功能将不再与该 VM 兼容。  
  * 运行时内存调整大小和动态内存
  * 检查点
  * 启用 Hyper-V 的虚拟机不能进行实时迁移。

## 常见问题和疑难解答

我的虚拟机无法启动，我应该怎么做？

1. 请确保动态内存已关闭。
2. 请确保你拥有支持的 Intel 处理器。
3. 在主机上从提升的提示符中运行[此 PowerShell 脚本](https://raw.githubusercontent.com/Microsoft/Virtualization-Documentation/master/hyperv-tools/Nested/Get-NestedVirtStatus.ps1)。

## 反馈

通过 Windows 反馈应用、[虚拟化论坛](https://social.technet.microsoft.com/Forums/windowsserver/En-us/home?forum=winserverhyperv)，或通过 [GitHub](https://github.com/Microsoft/Virtualization-Documentation) 报告其他问题。



<!--HONumber=Jun16_HO3-->


