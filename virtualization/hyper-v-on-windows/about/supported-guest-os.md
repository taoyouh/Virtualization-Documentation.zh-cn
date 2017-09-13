---
title: Supported Windows guests
description: Supported Windows guests.
keywords: windows 10, hyper-v
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: ae4a18ed-996b-4104-90c5-539c90798e4c
ms.openlocfilehash: ac1ec3740cfa75acd8c005df3f67531c693cd914
ms.sourcegitcommit: 63e890ad2caae3ce55503af81c6c5ff0d1e630f6
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/17/2017
---
# Supported Windows guests 

This article lists the operating system combinations supported in Hyper-V on Windows.  It also serves as an introduction to integration services and other factors in support.

## What does support mean? 
Support means Microsoft has tested these host/guest combinations.  Issues with these combinations may receive attention from Product Support Services.
 
Microsoft provides support for guest operating systems in the following manner:
* Issues found in Microsoft operating systems and in integration services are supported by Microsoft support.
* For issues found in other operating systems that have been certified by the operating system vendor to run on Hyper-V, support is provided by the vendor.
* For issues found in other operating systems, Microsoft submits the issue to the multi-vendor support community, [TSANet](http://www.tsanet.org/).

In order to be supported, both the Hyper-V host and guest must be updated with all critical updates available through Windows Update.

## Supported guest operating systems

In order to receive support, both the Windows guest operating systems and the host operating system must be current with all critical updates available through Windows Update.

| Guest operating system |  Maximum number of virtual processors | Notes | 
|:-----|:-----|:-----|
| Windows 10 | 32 | |
| Windows 8.1 | 32 | |
| Windows 8 | 32 |  |
| Windows 7 with Service Pack 1 (SP 1) | 4 | Ultimate, Enterprise, and Professional editions (32-bit and 64-bit). |
| Windows 7 | 4 | Ultimate, Enterprise, and Professional editions (32-bit and 64-bit). |
| Windows Vista with Service Pack 2 (SP2) | 2 | Business, Enterprise, and Ultimate, including N and KN editions. | 
| - | | |
| Windows Server 2016 | 64 | |
| Windows Server 2012 R2 | 64 | |
| Windows Server 2012 | 64 | |
| Windows Server 2008 R2 with Service Pack 1 (SP 1) | 64 | Datacenter, Enterprise, Standard and Web editions. |
| Windows Server 2008 with Service Pack 2 (SP 2) | 4 | Datacenter, Enterprise, Standard and Web editions (32-bit and 64-bit). |
| Windows Home Server 2011 | 4 | |
| Windows Small Business Server 2011 | Essentials edition - 2, Standard edition - 4 | |
  
 > Windows 10 can run as a guest operating system on Windows 8.1 and Windows Server 2012 R2 Hyper-V hosts.

## Supported Linux and Free BSD

| Guest operating system |  |
|:-----|:------|
| [CentOS and Red Hat Enterprise Linux ](https://technet.microsoft.com/library/dn531026.aspx) | |
| [Debian virtual machines on Hyper-V](https://technet.microsoft.com/library/dn614985.aspx) | |
| [SUSE](https://technet.microsoft.com/en-us/library/dn531027.aspx) | |
| [Oracle Linux](https://technet.microsoft.com/en-us/library/dn609828.aspx)  | |
| [Ubuntu](https://technet.microsoft.com/en-us/library/dn531029.aspx) | |
| [FreeBSD](https://technet.microsoft.com/library/dn848318.aspx) | |

For more information, including support information on past versions of Hyper-V, see [Linux and FreeBSD Virtual Machines on Hyper-V](https://technet.microsoft.com/library/dn531030.aspx).
