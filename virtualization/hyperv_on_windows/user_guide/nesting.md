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
ms.sourcegitcommit: 26a8adb426a7cf859e1a9813da2033e145ead965
ms.openlocfilehash: d17413fc572e59ec21ff513ef5de994c6716aa08

---

# 借助嵌套虚拟化在虚拟机中运行 Hyper-V

嵌套虚拟化是一项功能，使你可以在 Hyper-V 虚拟机内运行 Hyper-V。 换而言之，借助嵌套虚拟化，Hyper-V 主机本身可进行虚拟化。 嵌套虚拟化的一些用例包括在虚拟化容器主机中运行 Hyper-V 容器、在虚拟化环境中设置 Hyper-V 实验室或者无需单个硬件测试多台计算机方案。 本文档将详细介绍软件和硬件先决条件、配置步骤，并提供故障排除的详细信息。 如果在 Windows 预览体验成员预览版本 14361 或更高版本上运行 Hyper-V，请参阅 [Nested Virtualization Preview for Windows Insiders: Builds 14361+](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting#nested-virtualization-preview-for-windows-insiders-builds-14361-)（Windows 预览体验成员的嵌套虚拟化预览版本：版本 14361 以上）。

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

##Windows 预览体验成员的嵌套虚拟化预览版本：版本 14361 以上
几个月前，我们宣布了 Hyper-V 嵌套虚拟化的早期预览版本：版本 10565。 我们都很高兴看到这项功能已生成，并且非常荣幸与 Windows 预览体验成员共享更新。

###嵌套虚拟化需要新虚拟机版本
从版本 14361 开始，启用嵌套虚拟化的虚拟机均需要版本 8.0。 对于较旧主机上创建的启用嵌套的虚拟机，将需要版本更新。 

####更新虚拟机版本
若要继续使用嵌套虚拟化技术，需要将虚拟机版本更新为 8.0。 即必须删除已保存的状态且需要关闭虚拟机。 以下 PowerShell cmdlet 将更新虚拟机版本：
```none
Update-VMVersion -Name <VMName>
```
####禁用嵌套虚拟化
如果不希望更新虚拟机，则可以禁用嵌套虚拟化，以便可以启动虚拟机：
```none
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $false
```

###虚拟机 8.0 版的新行为 
在预览版中对已启用的虚拟机的工作进行了几处更改：
-   创建和应用检查点现在适用于启用了嵌套虚拟化的虚拟机。
-   现在可以保存并启动已启用嵌套的虚拟机。
-   现在，已启用嵌套虚拟化的虚拟机可以在启用了基于虚拟化的安全的主机上运行（包括设备保护和凭据保护）。
-   我们改进了现有限制的错误消息。

###功能限制
-   嵌套虚拟化旨在 Hyper-V 虚拟机中运行 Hyper-V。 第三方虚拟化应用程序均不受支持，且可能在 Hyper-V 虚拟机中出现故障。
-   动态内存与嵌套虚拟化不兼容。 一旦 Hyper-V 在虚拟机内部运行，虚拟机不能在运行时更改它的内存。 
-   运行时内存调整大小与嵌套虚拟化不兼容。 在内部运行 Hyper-V 的同时调整虚拟机的内存大小将会失败。 
-   仅 Intel 系统支持嵌套虚拟化。

###已知问题
版本 14361 中存在一个已知问题，其中第 2 代虚拟机无法启动并出现以下错误：
```none
“Cannot modify property without enabling VirtualizationBasedSecurityOptOut”
```
此问题可通过禁用嵌套虚拟化得以暂时解决，或选择不使用基于虚拟化的安全：
```none
Set-VMSecurity -VMName <vmname> -VirtualizationBasedSecurityOptOut $true
```

###我们正在倾听
与往常一样，请继续使用 Windows 反馈应用发送反馈。 如果有任何疑问，请在我们的文档 [GitHub](https://github.com/Microsoft/Virtualization-Documentation) 页上记录相应的问题。 



<!--HONumber=Jun16_HO4-->


