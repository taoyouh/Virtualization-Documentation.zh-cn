---
redirect_url: http://aka.ms/upgradevmconfig
translationtype: Human Translation
ms.sourcegitcommit: e14ede0a2b13de08cea0a955b37a21a150fb88cf
ms.openlocfilehash: 54db04f9096dc1ef9572594b321f400fbbfeada0

---

# 迁移和升级虚拟机 

如果将虚拟机移动到最初通过 Windows 8.1 或更早版本中的 Hyper-V 创建的 Windows 10 主机，则在手动更新虚拟机配置版本前，你将无法使用新的虚拟机功能。 

若要升级配置版本，请关闭虚拟机，然后在 Hyper-V 管理器中选择升级虚拟机配置。  你还可以打开提升的 Windows PowerShell 命令提示符，并键入： 

 ```PowerShell
Update-VmVersion <vmname> | <vmobject>
```

## 如何检查在 Hyper-V 上运行的虚拟机的配置版本？ 

若要查找配置版本，请打开提升的 Windows PowerShell 命令提示符，并运行以下命令：

**Get-VM * | Format-Table Name, Version**

PowerShell 命令将生成以下示例输出：

```
Name        State       CPUUsage(%) MemoryAssigned(M)   Uptime              Status                  Version
    
Atlantis    Running         0       1024                00:04:20.5910000    Operating normally      5.0
    
SGC VM      Running         0       538                 00:02:44.8350000    Operating normally      6.2
```


## 如果我不升级配置版本，会发生什么情况？

如果你有使用较早版本的 Hyper-V 创建的虚拟机，则在更新 VM 版本前，某些功能可能无法用于这些虚拟机。

新的 Hyper-V 功能的最低 VM 配置版本：

| **功能名称**                       | **最小虚拟机版本** ||
| ：------------------------------------- | -----------------： || | 热添加/删除内存                  |                6.0 || | 热添加/删除网络适配器        |                5.0 || | 对于 Linux 虚拟机的安全引导              |                6.0 || | 生产检查点                 |                6.0 || | PowerShell Direct                      |                6.2 || | 虚拟受信任的平台模块 (vTPM) |               6.2 | ||虚拟机分组 |               6.2 | |



## 虚拟机配置版本 ##

将虚拟机从运行 Windows 8.1 的主机移动或导入到运行 Windows 10 上的 Hyper-V 的主机时，不会自动升级该虚拟机的配置文件。 这使得该虚拟机可以移回运行 Windows 8.1 的主机。 在手动更新虚拟机配置版本前，你无法访问新的虚拟机功能。 

虚拟机配置版本表示哪个版本的 Hyper-V 与虚拟机的配置、已保存状态和快照文件兼容。 配置版本为 5 的虚拟机与 Windows 8.1 兼容，并且可以在 Windows 8.1 和 Windows 10 上运行。 配置版本为 6 的虚拟机与 Windows 10 兼容，不可以在 Windows 8.1 上运行。

无需同时升级所有虚拟机。 你可以在需要时选择升级特定虚拟机。 但是，在手动更新每台虚拟机的配置版本前，你无法访问新的虚拟机功能。  


----------------
**重要 **

• 升级虚拟机配置版本后，不能将该虚拟机移动到运行 Windows 8.1 的主机。

• 不能将虚拟机配置版本从版本 6 降级到版本 5。

• 必须先关闭虚拟机，才可以升级虚拟机配置。

• 升级完成后，虚拟机使用新的配置文件格式。 有关详细信息，请参阅新的虚拟机配置文件格式。

--------





## 升级虚拟机版本时会发生什么情况？
在将虚拟机的配置版本手动升级到版本 6.x 时，你将更改用于存储虚拟机配置和检查点文件的文件结构。 

升级后的虚拟机使用新的配置文件格式，该文件格式旨在提高读取和写入虚拟机配置数据的效率。 升级还减少了存储失败时数据损坏的可能性。 

下表列出了升级后的虚拟机的二进制文件位置和扩展信息。  

|**虚拟机配置文件和检查点文件（版本 6.x）**|**说明**||
|：---------------|：----------------|| |**虚拟机配置** | 配置信息以使用 .vmcx 扩展名的二进制文件格式进行存储。 || |**虚拟机运行时状态** | 运行时状态数据以使用 .vmrs 扩展名的二进制文件格式的形式进行存储。  || |**虚拟机虚拟硬盘**|虚拟机的虚拟硬盘文件。 它们使用 .vhd 或 .vhdx 文件扩展名。   || |**自动虚拟硬盘文件**| 用于虚拟机检查点的差异磁盘文件。 它们使用 .avhdx 文件扩展名。 || |**检查点文件** |检查点存储在多个检查点文件中。 每个检查点都会创建一个配置文件和运行时状态文件。 检查点文件使用 .vmrs 和 .vmcx 文件扩展名。 这些新文件格式还可用于生产检查点和标准检查点。 ||

在将虚拟机配置版本升级到版本 6.x 后，无法从版本 6.x 降级到版本 5。 

必须关闭虚拟机，才可以升级虚拟机配置。

下表列出了新虚拟机或升级后的虚拟机的默认文件位置。

|   **虚拟机文件（版本 6.x）** | **说明** ||
|：-----|：-----|| |**虚拟机配置文件**|C:\ProgramData\Microsoft\Windows\Hyper-V\Virtual Machines || |**虚拟机运行时状态文件**|C:\ProgramData\Microsoft\Windows\Hyper-V\Virtual Machines || |**检查点文件（.vmrs、.vmcx）**|C:\ProgramData\Microsoft\Windows\Snapshots || |**虚拟硬盘文件 (.vhd/.vhdx)**|C:\Users\Public\Documents\Hyper-V\Virtual Hard Disks || |**自动虚拟硬盘文件 (.avhdx)**|C:\Users\Public\Documents\Hyper-V\Virtual Hard Disks ||







<!--HONumber=Jun16_HO4-->


