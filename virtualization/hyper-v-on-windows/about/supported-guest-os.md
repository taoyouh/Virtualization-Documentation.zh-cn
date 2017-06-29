---
title: "受支持的 Windows 来宾"
description: "受支持的 Windows 来宾。"
keywords: windows 10, hyper-v
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: ae4a18ed-996b-4104-90c5-539c90798e4c
ms.openlocfilehash: 8b764cb06b94465f516f9e5e8f06860ced8b39bb
ms.sourcegitcommit: d5f30aa1bdfb34dd9e1909d73b5bd9f4153d6b46
ms.translationtype: HT
ms.contentlocale: zh-CN
---
# <a name="supported-windows-guests"></a>受支持的 Windows 来宾 

本文列出了在 Windows 上的 Hyper-V 中受支持的操作系统组合。  它还介绍了集成服务和支持的其他因素。

## <a name="what-does-support-mean"></a>支持的含义是什么？ 
支持意味着 Microsoft 已测试这些主机/来宾组合。  这些组合的问题可能会受到产品支持服务的关注。
 
Microsoft 按如下方式为来宾操作系统提供支持：
* Microsoft 为在 Microsoft 操作系统和集成服务中找到的问题提供支持。
* 对于经操作系统供应商认证可以在 Hyper-V 上运行的其他操作系统中发现的问题，应由该供应商提供支持。
* 对于在其他操作系统中发现的问题，Microsoft 会将问题提交到多供应商支持社区 [TSANet](http://www.tsanet.org/)。

若要获取支持，Hyper-V 主机和来宾必须使用通过 Windows 更新提供的所有重要更新进行更新。

## <a name="supported-guest-operating-systems"></a>支持的来宾操作系统

若要收到支持，Windows 来宾操作系统和主机操作系统必须使用通过 Windows 更新提供的所有重要更新来保持最新状态。

| 来宾操作系统 |  虚拟处理器的最大数量 | 注释 | 
|:-----|:-----|:-----|
| Windows 10 | 32 | |
| Windows 8.1 | 32 | |
| Windows 8 | 32 |    |
| 带有 Service Pack 1 (SP 1) 的 Windows 7 | 4 | 旗舰版、企业版和专业版版本（32 位和 64 位）。 |
| Windows 7 | 4 | 旗舰版、企业版和专业版版本（32 位和 64 位）。 |
| Windows Vista Service Pack 2 (SP2) | 2 | Business、Enterprise 和 Ultimate，包括 N 和 KN 版本。 | 
| - | | |
| Windows Server 2012 R2 | 64 | |
| Windows Server 2012 | 64 | |
| 带有 Service Pack 1 (SP 1) 的 Windows Server 2008 R2 | 64 | Datacenter、Enterprise、Standard 和 Web 版本。 |
| 带有 Service Pack 2 (SP 2) 的 Windows Server 2008 | 4 | Datacenter、Enterprise、Standard 和 Web 版本（32 位和 64 位）。 |
| Windows Home Server 2011 | 4 | |
| Windows Small Business Server 2011 | Essentials 版本 - 2，Standard 版本 - 4 | |
  
 > Windows 10 可以作为来宾操作系统在 Windows 8.1 和 Windows Server 2012 R2 Hyper-V 主机上运行。

## <a name="supported-linux-and-free-bsd"></a>受支持的 Linux 和 FreeBSD

| 来宾操作系统 |  |
|:-----|:------|
| [CentOS 和 Red Hat Enterprise Linux ](https://technet.microsoft.com/library/dn531026.aspx) | |
| [Hyper-V 上的 Debian 虚拟机](https://technet.microsoft.com/library/dn614985.aspx) | |
| [SUSE](https://technet.microsoft.com/en-us/library/dn531027.aspx) | |
| [Oracle Linux](https://technet.microsoft.com/en-us/library/dn609828.aspx)  | |
| [Ubuntu](https://technet.microsoft.com/en-us/library/dn531029.aspx) | |
| [FreeBSD](https://technet.microsoft.com/library/dn848318.aspx) | |

有关详细信息（包括有关以前版本的 Hyper-V 的支持信息），请参阅 [Linux and FreeBSD Virtual Machines on Hyper-V](https://technet.microsoft.com/library/dn531030.aspx)（Hyper-V 上的 Linux 和 FreeBSD 虚拟机）。
