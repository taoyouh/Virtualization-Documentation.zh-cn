---
title: 使用 PowerShell Direct 管理 Windows 虚拟机
description: 使用 PowerShell Direct 管理 Windows 虚拟机
keywords: windows 10, hyper-v, powershell, 集成服务, 集成组件, 自动化, powershell direct
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: fb228e06-e284-45c0-b6e6-e7b0217c3a49
ms.openlocfilehash: ea6b71200d3115ba3d156b2c133e1be2fa495261
ms.sourcegitcommit: 34d8b2ca5eebcbdb6958560b1f4250763bee5b48
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "9620915"
---
# <a name="virtual-machine-automation-and-management-using-powershell"></a>使用 PowerShell 实现虚拟机自动化和管理虚拟机

无论采用何种网络配置或远程管理设置，均可以在 Hyper-V 主机上的 Windows 10 或 Windows Server 2016 虚拟机中使用 PowerShell Direct 运行任意 PowerShell。

下面是一些你可以运行 PowerShell Direct 的方法：

* [作为交互式会话使用 Enter-pssession cmdlet](#create-and-exit-an-interactive-powershell-session)
* [作为用以执行单个命令或脚本的单用途部分使用 Invoke-command cmdlet](#run-a-script-or-command-with-invoke-command)
* [作为持久性会话 （版本 14280 及更高版本） 使用 New-pssession，项副本，并删除 PSSession cmdlet](#copy-files-with-new-pssession-and-copy-item)

## <a name="requirements"></a>要求
**操作系统要求：**
* 主机：运行 Hyper-V 的 Windows 10、Windows Server 2016 或更高版本。
* 来宾/虚拟机：Windows 10、Windows Server 2016 或更高版本。

如果要管理较旧的虚拟机，请使用虚拟机连接 (VMConnect) 或[为虚拟机配置虚拟网络](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc816585(v=ws.10))。 

**配置要求：**    
* 虚拟机必须在主机上本地运行。
* 虚拟机必须开启，且运行时需至少具有一个配置的用户配置文件。
* 必须以 Hyper-V 管理员身份登录主机计算机。
* 必须为虚拟机提供有效用户凭据。

-------------

## <a name="create-and-exit-an-interactive-powershell-session"></a>创建并退出交互式 PowerShell 会话

在虚拟机上运行 PowerShell 命令的最简单方法是启动交互会话。

会话启动时，所键入的命令会在虚拟机上运行，其效果就像直接在虚拟机上将其键入到 PowerShell 会话中那样。

**启动交互会话：**

1. 在 HYPER-V 主机上以管理员身份打开 PowerShell。

2. 运行以下命令之一以使用虚拟机名称或 GUID 创建交互会话：  
  
  ``` PowerShell
  Enter-PSSession -VMName <VMName>
  Enter-PSSession -VMId <VMId>
  ```
  
  出现提示时，提供虚拟机的凭据。

3. 在虚拟机上运行命令。
  
  你应该会看到作为 PowerShell 提示符前缀的 VMName 显示如下：
  
  ``` 
  [VMName]: PS C:\>
  ```
  
  所有运行的命令将会在虚拟机上进行。 若要测试，可运行 `ipconfig` 或 `hostname` 以确保这些命令正在虚拟机中运行。
  
4. 完成后，运行以下命令来关闭会话：  
  
   ``` PowerShell
   Exit-PSSession 
   ``` 

> 请注意：如果你的会话未连接，请参阅[疑难解答](#troubleshooting)了解可能的原因。 

若要了解有关这些 cmdlet 的详细信息，请参阅 [Enter-PSSession](https://docs.microsoft.com/powershell/module/Microsoft.PowerShell.Core/Enter-PSSession?view=powershell-5.1) 和 [Exit-PSSession](https://docs.microsoft.com/powershell/module/Microsoft.PowerShell.Core/Exit-PSSession?view=powershell-5.1)。 

-------------

## <a name="run-a-script-or-command-with-invoke-command"></a>使用 Invoke-Command 运行脚本或命令

配合使用 PowerShell Direct 和 Invoke-Command 非常适合需要在虚拟机上运行一个命令或一个脚本但在这一点之外无需继续与虚拟机进行交互的情况。

**运行单个命令：**

1. 在 HYPER-V 主机上以管理员身份打开 PowerShell。

2. 通过使用虚拟机名称或 GUID 运行以下命令之一来创建会话：  
   
   ``` PowerShell
   Invoke-Command -VMName <VMName> -ScriptBlock { command } 
   Invoke-Command -VMId <VMId> -ScriptBlock { command }
   ```
   
   出现提示时，提供虚拟机的凭据。
   
   该命令将在虚拟机上执行，如果存在到控制台的输出，会把此输出打印到控制台。  命令一运行将会自动关闭连接。
   
   
**运行脚本：**

1. 在 HYPER-V 主机上以管理员身份打开 PowerShell。

2. 通过使用虚拟机名称或 GUID 运行以下命令之一来创建会话：  
   
   ``` PowerShell
   Invoke-Command -VMName <VMName> -FilePath C:\host\script_path\script.ps1 
   Invoke-Command -VMId <VMId> -FilePath C:\host\script_path\script.ps1 
   ```
   
   出现提示时，提供虚拟机的凭据。
   
   该脚本将在虚拟机上执行。  命令一运行将会自动关闭连接。

若要了解有关此 cmdlet 的详细信息，请参阅 [Invoke-Command](https://docs.microsoft.com/powershell/module/Microsoft.PowerShell.Core/Invoke-Command?view=powershell-5.1)。 

-------------

## <a name="copy-files-with-new-pssession-and-copy-item"></a>使用 New-PSSession 和 Copy-Item 复制文件

> **注意：** PowerShell Direct 仅支持 Windows 版本 14280 及更高版本中的持久性会话

在编写用于跨一个或多个远程计算机协调操作的脚本时，持久性 PowerShell 会话会非常有用。  一经创建后，持久性会话会一直存在于后台，直到你决定将其删除。  这意味着你可以使用 `Invoke-Command` 或 `Enter-PSSession` 反复引用同一个会话而无需传递凭据。

通过使用相同的令牌，会话将保持原有状态。  由于持久性会话具有持久性，在会话中创建的或传递给会话的任何变量将跨多个调用被保留。 有多种工具可用于持久性会话。  在此示例中，我们将使用 [New-PSSession](https://docs.microsoft.com/powershell/module/Microsoft.PowerShell.Core/New-PSSession?view=powershell-5.1) 和 [Copy-Item](https://docs.microsoft.com/powershell/module/Microsoft.PowerShell.Management/Copy-Item?view=powershell-5.1) 在主机和虚拟机之间移动数据。

**创建会话，然后复制文件：**  

1. 在 HYPER-V 主机上以管理员身份打开 PowerShell。

2. 运行以下命令之一使用 `New-PSSession` 将持久性 PowerShell 会话创建到虚拟机。
  
  ``` PowerShell
  $s = New-PSSession -VMName <VMName> -Credential (Get-Credential)
  $s = New-PSSession -VMId <VMId> -Credential (Get-Credential)
  ```
  
  出现提示时，提供虚拟机的凭据。
  
  > **警告：**  
   14500 之前的版本中存在一个 Bug。  如果不使用 `-Credential` 标志显式指定凭据，来宾操作系统中的服务将崩溃，并且将需要重新启动。  如果你遇到此问题，可在[此处](#error-a-remote-session-might-have-ended)获取解决方法说明。
  
3. 将文件复制到虚拟机内。
  
  要将 `C:\host_path\data.txt` 从主机复制到虚拟机内，运行：
  
  ``` PowerShell
  Copy-Item -ToSession $s -Path C:\host_path\data.txt -Destination C:\guest_path\
  ```
  
4.  从虚拟机复制文件（到主机）。 
   
   要将 `C:\guest_path\data.txt` 从虚拟机复制到主机，运行：
  
  ``` PowerShell
  Copy-Item -FromSession $s -Path C:\guest_path\data.txt -Destination C:\host_path\
  ```

5. 使用 `Remove-PSSession` 停止持久性会话。
  
  ``` PowerShell 
  Remove-PSSession $s
  ```
  
-------------

## <a name="troubleshooting"></a>疑难解答

PowerShell Direct 显示了一小部分的常见错误消息。  以下是最常见的错误消息、一些原因和诊断问题的工具。

### <a name="-vmname-or--vmid-parameters-dont-exist"></a>-VMName 或 -VMID 参数不存在
**问题：**  
`Enter-PSSession``Invoke-Command` 或 `New-PSSession` 不具有 `-VMName` 或 `-VMId` 参数。

**可能的原因：**  
最可能的问题是你的主机操作系统不支持 PowerShell Direct。

可以运行以下命令检查你的 Windows 版本：

``` PowerShell
[System.Environment]::OSVersion.Version
```

如果你运行的是支持的版本，则有可能你的 PowerShell 版本不运行 PowerShell Direct。  对于 PowerShell Direct 和 JEA，主版本必须为 5 或更高版本。

可以运行以下命令检查你的 PowerShell 版本：

``` PowerShell
$PSVersionTable.PSVersion
```


### <a name="error-a-remote-session-might-have-ended"></a>错误：远程会话可能已结束
> **注意：**  
对于主机版本在 10240 与 12400 之间的 Enter-PSSession，下面的所有错误都报告为“一个远程会话可能已结束”。

**错误消息：**
```
Enter-PSSession : An error has occurred which Windows PowerShell cannot handle. A remote session might have ended.
```

**可能的原因：**
* 虚拟机存在但未运行。
* 来宾操作系统不支持 PowerShell Direct（请参阅[要求](#requirements)）
* PowerShell 尚不可用于来宾
  * 操作系统没有完成启动
  * 操作系统无法正常启动
  * 某些启动时事件需要用户输入

可使用 [Get-VM](https://docs.microsoft.com/powershell/module/hyper-v/get-vm?view=win10-ps) cmdlet 进行检查以查看主机上正在运行哪些虚拟机。

**错误消息：**  
```
New-PSSession : An error has occurred which Windows PowerShell cannot handle. A remote session might have ended.
```

**可能的原因：**
* 上面列出的原因之一 - 它们都同等适用于 `New-PSSession`  
* 当前版本中的一个 Bug，在这些版本中，必须使用 `-Credential` 显式传递凭据。  发生这种情况时，整个服务将在来宾操作系统中挂起，并需要重新启动。  可以检查是否仍可通过 Enter-PSSession 使用会话。

若要解决凭据问题，使用 VMConnect 登录到虚拟机，打开 PowerShell，并使用以下 PowerShell 重新启动 vmicvmsession 服务：

``` PowerShell
Restart-Service -Name vmicvmsession
```

### <a name="error-parameter-set-cannot-be-resolved"></a>错误：无法解析参数集
**错误消息：**  
``` 
Enter-PSSession : Parameter set cannot be resolved using the specified named parameters.
```

**可能的原因：**  
* `-RunAsAdministrator` 在连接到虚拟机时，不受支持。
     
  连接到 Windows 容器时，`-RunAsAdministrator` 标志将允许管理员连接，而无需显式凭据。  由于虚拟机未授予主机默示的管理员访问权限，因此你需要显式输入凭据。

使用 `-Credential` 参数或通过在系统提示时手动输入，可将管理员凭据传递给虚拟机。


### <a name="error-the-credential-is-invalid"></a>错误：凭据无效。

**错误消息：**  
```
Enter-PSSession : The credential is invalid.
```

**可能的原因：** 
* 无法验证来宾凭据
  * 提供的凭据不正确。
  * 来宾操作系统中没有任何用户帐户（操作系统以前未启动）
  * 如果以管理员身份进行连接：管理员还未设置为活动用户。  在[此处](<https://docs.microsoft.com/previous-versions/windows/it-pro/windows-8.1-and-8/hh825104(v=win.10)>)了解详细信息。
  
### <a name="error-the-input-vmname-parameter-does-not-resolve-to-any-virtual-machine"></a>错误：输入的 VMName 参数未解析为任何虚拟机。

**错误消息：**  
```
Enter-PSSession : The input VMName parameter does not resolve to any virtual machine.
```

**可能的原因：**  
* 你不是 HYPER-V 管理员。  
* 虚拟机不存在。

你可以使用 [Get-VM](https://docs.microsoft.com/powershell/module/hyper-v/get-vm?view=win10-ps) cmdlet 检查使用中的凭据是否具有 Hyper-V 管理员角色并查看哪些 VM 在主机上本地运行并已启动。


-------------

## <a name="samples-and-user-guides"></a>示例和用户指南

PowerShell Direct 支持 JEA（只需提供足够的管理）。  查看此用户指南以试用。

查看 [GitHub](https://github.com/Microsoft/Virtualization-Documentation/search?l=powershell&q=-VMName+OR+-VMGuid&type=Code&utf8=%E2%9C%93) 上的示例。
