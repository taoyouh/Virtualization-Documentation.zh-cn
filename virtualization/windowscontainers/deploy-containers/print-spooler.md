---
title: Windows 容器中的打印后台处理程序
description: 说明 Windows 容器中打印后台处理程序服务的当前工作行为
keywords: docker，容器，打印机，后台处理程序
author: cwilhit
ms.openlocfilehash: 48130bc6a826a45dfa49d0a3b4600d227f34704e
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910527"
---
# <a name="print-spooler-in-windows-containers"></a>Windows 容器中的打印后台处理程序

对打印服务具有依赖关系的应用程序可以通过 Windows 容器成功地进行容器化。 为了成功启用打印机服务功能，必须满足特殊要求。 本指南介绍如何正确配置你的部署。

> [!IMPORTANT]
> 当在容器中成功获取打印服务的访问权限时，功能受限于窗体;某些与打印相关的操作可能不起作用。 例如，由于不**支持在容器内安装驱动程序**，因此不能容器化依赖于将打印机驱动程序安装到主机上的应用程序。 如果你在容器中找到了一个不受支持的打印功能，请打开以下反馈。

## <a name="setup"></a>安装

* 主机应为 Windows Server 2019 或 Windows 10 专业版/企业版2018更新版或更高版本。
* [Mcr.microsoft.com/windows](https://hub.docker.com/_/microsoft-windowsfamily-windows)映像应为目标基准映像。 其他 Windows 容器基本映像（如 Nano Server 和 Windows Server Core）不携带打印服务器角色。

### <a name="hyper-v-isolation"></a>Hyper-V 隔离

建议运行包含 Hyper-v 隔离的容器。 在此模式下运行时，可以拥有所需的任意数量的容器，以便能够访问打印服务。 不需要在主机上修改后台处理程序服务。

可以通过以下 PowerShell 查询来验证功能：

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

由于进程隔离容器的共享内核本质，当前行为限制用户仅在主机上运行打印机后台处理程序服务的**一个实例**及其所有容器子。 如果主机运行的是打印机后台处理程序，则必须先停止主机上的服务，然后尝试才能在来宾中启动打印机服务。

> [!TIP]
> 如果同时启动容器并同时在容器和主机中查询后台处理程序服务，则这两种方法都会将其状态报告为 "正在运行"。 但不能 deceived--容器将无法查询可用打印机的列表。 主机的后台处理程序服务不能运行。 

若要检查主机是否正在运行打印机服务，请使用下面 PowerShell 中的查询：

```PowerShell
PS C:\Users\Administrator> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler

PS C:\Users\Administrator>
```

若要停止主机上的后台处理程序服务，请在下面的 PowerShell 中使用以下命令：

```PowerShell
Stop-Service spooler
Set-Service spooler -StartupType Disabled
```

启动容器并验证对打印机的访问。

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