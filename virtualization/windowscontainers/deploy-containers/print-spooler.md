---
title: Windows 容器中的打印后台处理程序
description: 介绍了 Windows 容器中的打印后台处理程序服务的当前工作行为
keywords: docker，容器，打印机，后台处理程序
author: cwilhit
ms.openlocfilehash: 45176e651ee2ef9b6daea9919004601734084083
ms.sourcegitcommit: 04c372c87c832f73a1aa120b0ff6c2c2b9c8c1b1
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/25/2019
ms.locfileid: "9257976"
---
# <a name="print-spooler-in-windows-containers"></a>Windows 容器中的打印后台处理程序

打印服务上的依赖项的应用程序可以在容器化成功与 Windows 容器。 不能容器化应用程序已安装打印机驱动程序到主机上的依赖项;从容器中的驱动程序安装是不受支持，因为它会泄漏到主机的容器状态。 有一些必须满足的成功启用打印机服务功能的特殊要求。 本指南介绍了如何正确配置你的部署。

> [!IMPORTANT]
> 获取访问的打印服务成功在容器中的工作原理，尽管形式; 有限功能一些与打印相关的操作可能不起作用。 请如果出现这种情况，打开了下面的反馈。

## <a name="setup"></a>安装

* 在主机应为 Windows Server 2019 或 Windows 10 专业版/企业版 2018 年 10 月更新或更高版本。
* [Mcr.microsoft.com/windows](https://hub.docker.com/_/microsoft-windowsfamily-windows)图像应定向的基本映像。 （如 Nano Server 和 Windows Server Core） 其他 Windows 容器基本映像不带有打印服务器角色。

### <a name="hyper-v-isolation"></a>Hyper-V 隔离

我们建议使用 HYPER-V 隔离运行你的容器。 在此模式下运行时，你可以根据需要运行具有访问权限的打印服务的多个容器。 不需要修改在主机上的后台打印服务。

你可以验证与下面的 PowerShell 查询的功能：

```PowerShell
PS C:\Users\Administrator> docker run -it --isolation hyperv mcr.microsoft.com/windows:1809 powershell.exe
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler


PS C:\> Get-Printer

Name                           ComputerName    Type         DriverName                PortName        Shared   Published
----                           ------------    ----         ----------                --------        ------   --------
Microsoft XPS Document Writer                  Local        Microsoft XPS Document... PORTPROMPT:     False    False
Microsoft Print to PDF                         Local        Microsoft Print To PDF    PORTPROMPT:     False    False
Fax                                            Local        Microsoft Shared Fax D... SHRFAX:         False    False


PS C:\>
```

### <a name="process-isolation"></a>进程隔离

由于的进程隔离的容器共享的内核性质，当前行为会限制用户跨主机及其所有容器子级运行只有**一个实例**的打印机后台处理程序服务。 如果主机运行的后台打印机，你必须在主机上之前 attemping 用于启动，来宾操作系统中的打印机服务停止服务。

> [!TIP]
> 如果你启动容器并同时查询的容器和主机中的后台处理程序服务，同时将报告其状态为正在运行。 但不要 deceived-容器将不能查询可用的打印机的列表。 主机的后台处理程序服务必须无法运行。 

若要查看主机是否正在运行的打印机服务，请在下面的 PowerShell 中使用查询：

```PowerShell
PS C:\Users\Administrator> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler

PS C:\Users\Administrator>
```

若要停止在主机上的后台打印服务，请在下面的 PowerShell 中使用以下命令：

```PowerShell
Stop-Service spooler
Set-Service spooler -StartupType Disabled
```

启动容器并验证可以访问打印机。

```PowerShell
PS C:\Users\Administrator> docker run -it --isolation process mcr.microsoft.com/windows:1809 powershell.exe
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.


PS C:\> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler


PS C:\> Get-Printer

Name                           ComputerName    Type         DriverName                PortName        Shared   Published
----                           ------------    ----         ----------                --------        ------   --------
Microsoft XPS Document Writer                  Local        Microsoft XPS Document... PORTPROMPT:     False    False
Microsoft Print to PDF                         Local        Microsoft Print To PDF    PORTPROMPT:     False    False
Fax                                            Local        Microsoft Shared Fax D... SHRFAX:         False    False


PS C:\>
```