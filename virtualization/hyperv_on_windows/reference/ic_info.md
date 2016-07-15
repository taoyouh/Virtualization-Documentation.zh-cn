---
title: "Hyper-V 集成服务"
description: "Hyper-V 集成服务参考"
keywords: windows 10, hyper-v, integration services, integration components
author: scooley
manager: timlt
ms.date: 05/25/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 18930864-476a-40db-aa21-b03dfb4fda98
translationtype: Human Translation
ms.sourcegitcommit: eae8f504039a1b3732ecfb31b8068667aab104f9
ms.openlocfilehash: 633a6b2f3fe737a6e7515fd3f923335e39cf9d17

---

# Hyper-V 集成服务

集成服务（通常称为集成组件）是允许虚拟机与 Hyper-V 主机通信的服务。 其中许多服务都很便利，但其他服务对虚拟机能够正常工作却至关重要。

本文是 Windows 中提供的每个集成服务的参考。  它也将作为特定集成服务或其历史记录的相关信息的起点。


## 快速参考

| Name | Windows 服务名称 | Linux 守护程序名称 |  描述 | 禁用后对 VM 的影响 |
|:---------|:---------|:---------|:---------|:---------|
| [Hyper-V 检测信号服务](#hyper-v-heartbeat-service) |  vmicheartbeat | hv_utils | 报告虚拟机运行正常。 | 变化不定 |
| [Hyper-V 来宾关闭服务](#hyper-v-guest-shutdown-service) | vmicshutdown | hv_utils |  允许主机触发虚拟机关闭。 | **高** |
| [Hyper-V 时间同步服务](#hyper-v-time-synchronization-service) | vmictimesync | hv_utils | 使虚拟机的时钟与主计算机的时钟同步。 | **高** |
| [Hyper-V 数据交换服务 (KVP)](#hyper-v-data-exchange-service-kvp) | vmickvpexchange | hv_kvp_daemon | 提供交换虚拟机和主机之间的基本元数据的方法。 | 中型 |
| [Hyper-V 卷影复制请求程序](#hyper-v-volume-shadow-copy-requestor) | vmicvss | hv_vss_daemon | 允许卷影复制服务在不关闭虚拟机的情况下对其进行备份。 | 变化不定 |
| [Hyper-V 来宾服务接口](#hyper-v-powershell-direct-service) | vmicguestinterface | hv_fcopy_daemon | 提供 Hyper-V 主机将文件复制到虚拟机或从虚拟机复制文件的界面。 | 低 |
| [Hyper-V PowerShell Direct 服务](#hyper-v-powershell-direct-service) | vmicvmsession | 不可用 | 提供在没有网络连接的情况下，使用 PowerShell 管理虚拟机的方法。 | 低 |  


## Hyper-V 检测信号服务

**Windows 服务名称：**vmicheartbeat  
**Linux 守护程序名称：**hv_utils  
**说明：**告知 Hyper-V 主机，虚拟机已安装操作系统并且正常启动。  
**添加：**Windows Server 2012、Windows 8  
**影响：**禁用后，虚拟机无法报告其内部的操作系统是否正常运行。  这可能会影响某些类型的监视和主机端诊断。  

检测信号服务使其可以回答如“虚拟机启动了吗？”等类似的基本问题。  

当 Hyper-V 报告虚拟机状态为“正在运行”（请参阅以下示例）时，表示 Hyper-V 在为虚拟机预留资源，而不是已安装或正在运行操作系统。  这是检测信号十分有用的地方。  检测信号服务告知 Hyper-V，虚拟机内的操作系统已启动。  

### 使用 PowerShell 检查检测信号

以管理员身份运行 [Get-VM](https://technet.microsoft.com/en-us/library/hh848479.aspx) 以查看虚拟机的检测信号：
``` PowerShell
Get-VM -VMName $VMName | select Name, State, Status
```

你的输出应类似下面的形式：
```
Name    State    Status
----    -----    ------
DemoVM  Running  Operating normally
```

`Status` 字段由检测信号服务确定。



## Hyper-V 来宾关闭服务

**Windows 服务名称：**vmicshutdown  
**Linux 守护程序名称：**hv_utils  
**说明：**允许 Hyper-V 主机请求关闭虚拟机。  主机始终可以强制关闭虚拟机，但这样做类似于切换电源开关而不是选择关闭。
**添加：**Windows Server 2012、Windows 8  
**影响：****重大影响** 禁用后，主机无法触发虚拟机中的友好关闭。  所有关闭都将为硬关机，可能导致数据丢失或数据损坏。


## Hyper-V 时间同步服务

**Windows 服务名称：**vmictimesync  
**Linux 守护程序名称：**hv_utils  
**说明：**使虚拟机的系统时钟与物理计算机的系统时钟同步。  
**添加：**Windows Server 2012、Windows 8  
**影响：****重大影响** 禁用后，虚拟机的时钟将出现不确定的偏移。  


## Hyper-V 数据交换服务 (KVP)

**Windows 服务名称：**vmickvpexchange  
**Linux 守护程序名称：**hv_kvp_daemon  
**说明：**提供交换虚拟机和主机之间的基本元数据的机制。  
**添加：**Windows Server 2012、Windows 8  
**影响：**禁用后，运行 Windows 8 或 Windows Server 2012 或更早版本的虚拟机将不会收到 Hyper-V 集成服务的更新。  禁用数据交换可能还会影响某些类型的监视和主机端诊断。

数据交换服务（有时称为 KVP）使用 Windows 注册表中的键值对 (KVP)，在虚拟机和 Hyper-V 主机之间共享少量计算机信息。  相同的机制还可用于在虚拟机和主机之间共享自定义数据。

键值对由“键”和“值”组成。 键和值都是字符串，不支持任何其他数据类型。 当创建或更改键值对时，来宾和主机都可以看到。 键值对信息通过 Hyper-V VMbus 传输，不需要来宾操作系统和 Hyper-V 主机之间的任何类型的网络连接。 

数据交换服务是保留有关虚拟机信息的重要工具，对于交互式数据共享或数据传输，请使用 [PowerShell Direct](#hyper-v-powershell-direct-service)。 


**用户指南：**  
* [Using key-value pairs to share information between the host and guest on Hyper-V](https://technet.microsoft.com/en-us/library/dn798287.aspx)（使用键值对在 Hyper-V 上的主机和来宾之间共享信息）。


## Hyper-V 卷影复制请求程序

**Windows 服务名称：**vmicvss  
**Linux 守护程序名称：**hv_vss_daemon  
**说明：**允许卷影复制服务备份虚拟机上的应用程序和数据。  
**添加：**Windows Server 2012、Windows 8  
**影响：**禁用后，虚拟机无法在运行（使用 VSS）的情况下被备份。  

卷影复制服务 ([VSS](https://msdn.microsoft.com/en-us/library/aa384589.aspx)) 需要卷影复制请求程序集成服务。  卷影复制服务 (VSS) 捕获并复制运行系统（特别是服务器）上的映像以进行备份，但不会过度降低其提供的服务的性能和稳定性。  此集成服务通过使用主机的备份过程，协调虚拟机的工作负荷来实现它。

有关卷影复制的详细信息，请阅读[此处](https://msdn.microsoft.com/en-us/library/dd405549.aspx)的内容。


## Hyper-V 来宾服务接口

**Windows 服务名称：**vmicguestinterface  
**Linux 守护程序名称：**hv_fcopy_daemon  
**说明：**提供 Hyper-V 主机双向复制文件到虚拟机或从虚拟机双向复制文件的界面。  
**添加：**Windows Server 2012 R2、Windows 8.1  
**影响：**禁用后，主机无法使用 `Copy-VMFile` 将文件复制到来宾和从来宾复制文件。  阅读更多有关 [Copy-VMFile cmdlet](https://technet.microsoft.com/library/dn464282.aspx) 的内容。  

**注意：**  
Д¬ИПЗйїцПВЅыУГ。  请参阅 [PowerShell Direct - 使用 Copy-Item](../user_guide/vmsession.md#copy-files-with-new-pssession-and-copy-item)。 


## Hyper-V PowerShell Direct 服务

**Windows 服务名称：**vmicvmsession  
**Linux 守护程序名称：**n/a  
**说明：**提供在没有虚拟网络的情况下，通过 VM 会话使用 PowerShell 管理虚拟机的机制。    
**添加：**Windows Server TP3、Windows 10  
**影响：**禁用此服务后，主机将无法使用 PowerShell Direct 连接到虚拟机。  

**注意：**  
服务名称最初为 Hyper-V VM 会话服务。  
PowerShell Direct 还在继续开发中，仅在 Windows 10/Windows Server 技术预览版 3 或更高版本的主机/来宾上可用。  


不管 Hyper-V 主机或虚拟机上的网络配置或远程管理设置如何，PowerShell Direct 都允许在 Hyper-V 主机上的虚拟机中管理 PowerShell。 这使得 Hyper-V 管理员能够更简单地自动化管理和配置任务，并为其编写脚本。

[阅读更多有关 PowerShell Direct 的内容](../user_guide/vmsession.md)。  

**用户指南：**  
* [在虚拟机中运行脚本](../user_guide/vmsession.md#run-a-script-or-command-with-invoke-command)
* [将文件复制到虚拟机和从虚拟机复制文件](../user_guide/vmsession.md#copy-files-with-new-pssession-and-copy-item)


## 其他资源

**用户指南：**  
* [启用/禁用 Hyper-V 主机上的集成服务](../user_guide/managing_ics.md#enable-or-disable-integration-services-using-powershell)
* 启用/禁用虚拟机上的集成服务。
** [Windows](../user_guide/managing_ics.md#manage-integration-services-from-guest-os-windows)
** [Linux](../user_guide/managing_ics.md#manage-integration-services-from-guest-os-linux)
* [集成服务的更新与维护](../user_guide/managing_ics.md#integration-service-maintenance)


<!--HONumber=Jul16_HO1-->


