---
title: "使用 Hyper-V 和 Windows PowerShell"
description: "使用 Hyper-V 和 Windows PowerShell"
keywords: windows 10, hyper-v
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 6d1ae036-0841-4ba5-b7e0-733aad31e9a7
translationtype: Human Translation
ms.sourcegitcommit: e14ede0a2b13de08cea0a955b37a21a150fb88cf
ms.openlocfilehash: a8e567b6447aa73f14825b7054d977d2b003a726

---

# 使用 Hyper-V 和 Windows PowerShell

现在你已基本了解如何部署 Hyper-V、创建虚拟机和管理这些虚拟机，让我们研究一下如何使用 PowerShell 来自动执行其中大部分活动。

### 返回 Hyper-V 命令列表

1.  单击 Windows“开始”按钮，键入“**PowerShell**”。
2.  运行以下命令以显示适用于 Hyper-V PowerShell 模块的 PowerShell 命令的可搜索列表。

 ```powershell
get-command -module hyper-v | out-gridview
```
  获取的内容如下所示：

  ![](media\command_grid.png)

3. 若要了解有关特定 PowerShell 命令的详细信息，请使用 `get-help`。 例如，运行以下命令将返回有关 `get-vm` Hyper-V 命令的信息。

  ```powershell
get-help get-vm
```
 该输出向你显示构建命令的方法、必需和可选参数定义以及可以使用的别名。

 ![](media\get_help.png)


### 返回虚拟机列表

使用 `get-vm` 命令返回虚拟机列表。

1. 在 PowerShell 中，运行以下命令：
 
 ```powershell
get-vm
```
 显示内容如下所示：

 ![](media\get_vm.png)

2. 若要仅返回已启动的虚拟机列表，请将筛选器添加到 `get-vm` 命令。 可通过使用 where-object 命令添加筛选器。 有关筛选的详细信息，请参阅[使用 Where-Object](https://technet.microsoft.com/en-us/library/ee177028.aspx) 文档。   

 ```powershell
 get-vm | where {$_.State -eq ‘Running’}
 ```
3.  若要列出所有处于关机状态的虚拟机，请运行以下命令。 此命令是步骤 2 中的命令的副本且筛选器从“正在运行”更改为“关闭”。

 ```powershell
 get-vm | where {$_.State -eq ‘Off’}
 ```

### 启动和关闭虚拟机

1. 若要启动特定虚拟机，请运行附带虚拟机名称的以下命令：

 ```powershell
 Start-vm -Name <virtual machine name>
 ```

2. 若要启动所有当前已关机的虚拟机，请获取这些虚拟机的列表并将该列表通过管道传递到“start-vm”命令：

  ```powershell
 get-vm | where {$_.State -eq ‘Off’} | start-vm
 ```
3. 若要关闭所有正在运行的虚拟机，请运行以下命令：
 
  ```powershell
 get-vm | where {$_.State -eq ‘Running’} | stop-vm
 ```

### 创建 VM 检查点

若要使用 PowerShell 创建检查点，请使用 `get-vm` 命令选择虚拟机，然后通过管道将该虚拟机传递到 `checkpoint-vm` 命令。 最后，使用 `-snapshotname` 为该检查点命名。 完整命令如下所示：

 ```powershell
 get-vm -Name <VM Name> | checkpoint-vm -snapshotname <name for snapshot>
 ```
### 创建新的虚拟机

以下示例演示如何在 PowerShell 集成脚本环境 (ISE) 中创建新的虚拟机。 这是一个简单示例，并可扩展为包含其他 PowerShell 功能以及更高级的 VM 部署。

1. 若要打开 PowerShell ISE，请单击“开始”，键入 **PowerShell ISE**。
2. 运行以下代码来创建虚拟机。 有关 New-VM 命令的详细信息，请参阅 [New-VM](https://technet.microsoft.com/en-us/library/hh848537.aspx) 文档。

  ```powershell
 $VMName = "VMNAME"

 $VM = @{
     Name = $VMName 
     MemoryStartupBytes = 2147483648
     Generation = 2
     NewVHDPath = "C:\Virtual Machines\$VMName\$VMName.vhdx"
     NewVHDSizeBytes = 53687091200
     BootDevice = "VHD"
     Path = "C:\Virtual Machines\$VMName "
     SwitchName = (get-vmswitch).Name[0]
 }

 New-VM @VM
  ```

## 总结和参考

本文档介绍了一些研究 Hyper-V PowerShell 模块的简单步骤以及一些示例方案。 有关 Hyper-V PowerShell 模块的详细信息，请参阅 [Windows PowerShell 中的 Hyper-V Cmdlet 参考](https://technet.microsoft.com/%5Clibrary/Hh848559.aspx)。  
 


<!--HONumber=Jun16_HO4-->


