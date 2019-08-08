---
title: 在 Windows 容器中打印后台处理程序
description: 介绍 Windows 容器中打印后台处理程序服务的当前工作行为
keywords: docker、容器、打印机、后台处理程序
author: cwilhit
ms.openlocfilehash: e104a87046545b90d244783aafb62ad9d151e14b
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/07/2019
ms.locfileid: "9999094"
---
# <a name="print-spooler-in-windows-containers"></a>在 Windows 容器中打印后台处理程序

与打印服务有依赖关系的应用程序可以使用 Windows 容器成功地进行容器。 必须满足特殊要求才能成功启用打印机服务功能。 本指南介绍如何正确配置你的部署。

> [!IMPORTANT]
> 当容器中成功访问打印服务时, 功能在表单中受限制;某些与打印相关的操作可能不起作用。 例如, 由于不**支持从容器内部安装驱动**程序, 因此无法为将打印机驱动程序安装到主机的应用进行容器。 如果您找到了不受支持的打印功能, 而您希望在容器中支持该功能, 请打开下面的反馈。

## <a name="setup"></a>安装

* 主机应为 Windows Server 2019 或 Windows 10 专业版/企业版2018更新或更高版本。
* [Mcr.microsoft.com/windows](https://hub.docker.com/_/microsoft-windowsfamily-windows)图像应为目标的基本图像。 其他 Windows 容器基本映像 (如 Nano Server 和 Windows Server Core) 不会携带打印服务器角色。

### <a name="hyper-v-isolation"></a>Hyper-V 隔离

我们建议你通过 Hyper-v 隔离运行容器。 在此模式下运行时, 你可以使用对打印服务的访问权限运行所需的多个容器。 无需修改主机上的后台处理程序服务。

你可以通过以下 PowerShell 查询验证功能:

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

由于进程隔离容器的共享内核本质, 当前行为限制用户只能在整个主机及其所有容器子中运行打印机后台处理程序服务的**一个实例**。 如果主机正在运行打印机后台处理程序, 则必须先停止主机上的服务, 然后才能在来宾中 attemping 启动打印机服务。

> [!TIP]
> 如果在容器和主机中同时启动容器和同时查询后台处理程序服务, 二者都将报告其状态为 "正在运行"。 但不要 deceived-容器将无法查询可用打印机的列表。 主机的后台处理程序服务必须不运行。 

若要检查主机是否正在运行打印机服务, 请使用下面的 PowerShell 中的查询:

```PowerShell
PS C:\Users\Administrator> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler

PS C:\Users\Administrator>
```

若要停止主机上的后台处理程序服务, 请在下面的 PowerShell 中使用以下命令:

```PowerShell
Stop-Service spooler
Set-Service spooler -StartupType Disabled
```

启动容器并验证对打印机的访问权限。

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