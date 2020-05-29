---
title: Windows 容器中的打印后台处理程序
description: 介绍了 Windows 容器中打印后台处理程序服务的当前工作行为
keywords: docker, 容器, 打印机, 后台处理程序
author: cwilhit
ms.openlocfilehash: e104a87046545b90d244783aafb62ad9d151e14b
ms.sourcegitcommit: 16744984ede5ec94cd265b6bff20aee2f782ca88
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/18/2020
ms.locfileid: "77439534"
---
# <a name="print-spooler-in-windows-containers"></a>Windows 容器中的打印后台处理程序

依赖于打印服务的应用程序可以通过 Windows 容器成功地进行容器化。 为了成功启用打印机服务功能，必须满足一些特殊要求。 本指南介绍了如何正确配置你的部署。

> [!IMPORTANT]
> 虽然可以在容器中成功访问打印服务，但该功能在形式上受限；与打印相关的某些操作无法完成。 例如，要求将打印机驱动程序安装到主机中的应用无法容器化，因为**不支持从容器中安装驱动程序**。 如果你发现某个打印功能不受支持，但你希望该功能在容器中受支持，请在下面提交反馈。

## <a name="setup"></a>安装

* 主机应当使用 Windows Server 2019 或 Windows 10 专业版/企业版的 2018 年 10 月更新版或更高版本。
* 应当使用 [mcr.microsoft.com/windows](https://hub.docker.com/_/microsoft-windowsfamily-windows) 映像作为基础映像。 其他 Windows 容器基础映像（例如 Nano Server 和 Windows Server Core）未携带打印服务器角色。

### <a name="hyper-v-isolation"></a>Hyper-V 隔离

建议运行采用 Hyper-V 隔离的容器。 在此模式下运行时，可以运行能够访问打印服务的任意数量的容器。 你不需要在主机上修改后台处理程序服务。

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

由于进程隔离容器的共享内核特性，当前行为会限制用户，使其在主机及其所有容器子代上只能运行打印机后台处理程序服务的**一个实例**。 如果主机正在运行打印机后台处理程序，当你尝试在来宾中启动打印机服务之前，必须先在主机上停止该服务。

> [!TIP]
> 如果启动某个容器并同时在该容器和主机中查询后台处理程序服务，则两者都会报告其状态为“正在运行”。 但不要被欺骗了，容器将无法查询可用打印机的列表。 主机的后台处理程序服务不得运行。 

若要检查主机是否正在运行打印机服务，请使用下面 PowerShell 中的查询：

```PowerShell
PS C:\Users\Administrator> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler

PS C:\Users\Administrator>
```

若要在主机上停止后台处理程序服务，请在下面的 PowerShell 中使用以下命令：

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