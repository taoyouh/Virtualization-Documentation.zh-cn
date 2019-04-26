---
title: 将 Hyper-V WMIv1 移植到 WMIv2
description: 了解如何将 Hyper-V WMIv1 移植到 WMIv2
keywords: windows 10, hyper-v, WMIv1, WMIv2, WMI, Msvm_VirtualSystemGlobalSettingData, root\virtualization
author: scooley
ms.date: 04/13/2017
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: b13a3594-d168-448b-b0a1-7d77153759a8
ms.openlocfilehash: e2d6faabe77346199a5d292fcfd92cdfd63909b8
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/26/2019
ms.locfileid: "9575148"
---
# <a name="move-from-hyper-v-wmi-v1-to-wmi-v2"></a>从 Hyper-V WMI v1 移到 WMI v2

Windows Management Instrumentation (WMI) 是 Hyper-V 管理器和 Hyper-V 的 PowerShell cmdlet 下层的管理接口。  虽然大多数用户会使用我们的 PowerShell cmdlet 或 Hyper-V 管理器，但有时开发人员需要直接使用 WMI。  

已经存在两个 Hyper-V WMI 命名空间（或 Hyper-V WMI API 版本）。
* 在 Windows Server 2008 中引入且最终可用于 Windows Server 2012 的 WMI v1 命名空间 (root\virtualization)
* 在 Windows Server 2012 中引入的 WMI v2 命名空间 (root\virtualization\v2)

本文档包含用于将与旧 WMI 命名空间通信的代码转换为新代码的资源参考。  最初，本文将用作 API 信息和示例代码/脚本的存储库，可用于帮助将任何使用 Hyper-V WMI API 的程序或脚本从 v1 命名空间移植到 v2 命名空间。

## <a name="msdn-samples"></a>MSDN 示例

[Hyper-V 虚拟机迁移示例](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-virtual-machine-aef356ee)  
[Hyper-V 虚拟光纤通道示例](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-virtual-Fiber-35d27dcd)  
[Hyper-V 计划的虚拟机示例](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-planned-virtual-8c7b7499)  
[Hyper-V 应用程序运行状况监视示例](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-application-health-dc0294f2)  
[虚拟硬盘管理示例](http://code.msdn.microsoft.com/windowsdesktop/Virtual-hard-disk-03108ed3)  
[Hyper-V 复制示例](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-replication-sample-d2558867)  
[Hyper-V 指标示例](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-metrics-sample-2dab2cb1)  
[Hyper-V 动态内存示例](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-dynamic-memory-9b0b1d05)  
[Hyper-V 可扩展交换机扩展筛选器驱动程序](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-Extensible-Virtual-e4b31fbb)  
[Hyper-V 网络示例](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-networking-sample-7c47e6f5)  
[Hyper-V 资源池管理示例](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-resource-pool-df906d95)  
[Hyper-V 恢复快照示例](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-recovery-snapshot-ea72320c)  

## <a name="samples-from-blogs"></a>博客中的示例

[使用 Hyper-V WMI V2 命名空间将网络适配器添加到 VM](http://blogs.msdn.com/b/taylorb/archive/2013/07/15/adding-a-network-adapter-to-a-vm-using-the-hyper-v-wmi-v2-namespace.aspx)  
[使用 Hyper-V WMI V2 命名空间将 VM 网络适配器连接到交换机](http://blogs.msdn.com/b/taylorb/archive/2013/07/15/connecting-a-vm-network-adapter-to-a-switch-using-the-hyper-v-wmi-v2-namespace.aspx)  
[使用 Hyper-V WMI V2 命名空间更改 NIC 的 MAC 地址](http://blogs.msdn.com/b/taylorb/archive/2013/08/12/changing-the-mac-address-of-nic-using-the-hyper-v-wmi-v2-namespace.aspx)  
[使用 Hyper-V WMI V2 命名空间从 VM 中删除网络适配器](http://blogs.msdn.com/b/taylorb/archive/2013/08/12/removing-a-network-adapter-to-a-vm-using-the-hyper-v-wmi-v2-namespace.aspx)  
[使用 Hyper-V WMI V2 命名空间将 VHD 连接到 VM](http://blogs.msdn.com/b/taylorb/archive/2013/08/12/attaching-a-vhd-to-a-vm-using-the-hyper-v-wmi-v2-namespace.aspx)  
[使用 Hyper-V WMI V2 命名空间从 VM 中删除 VHD](http://blogs.msdn.com/b/taylorb/archive/2013/08/12/removing-a-vhd-from-a-vm-using-the-hyper-v-wmi-v2-namespace.aspx)  
[使用 Hyper-V WMI V2 命名空间创建 VM](http://blogs.msdn.com/b/virtual_pc_guy/archive/2013/06/20/creating-a-virtual-machine-with-wmi-v2.aspx)

