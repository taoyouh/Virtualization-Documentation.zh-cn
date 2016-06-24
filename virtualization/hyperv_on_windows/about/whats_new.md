---
redirect_url: ../windows_welcome
---

## 网络适配器和内存的热添加和删除

现在，在运行虚拟机期间，无需停机即可添加或删除网路适配器。 这适用于运行 Windows 和 Linux 操作系统的第 2 代虚拟机。 

还可以在虚拟机运行期间调整向其分配的内存量，即使尚未启用“动态内存”。 这适用于第 1 代和第 2 代虚拟机。

## 生产检查点

借助生产检查点，可以轻松创建虚拟机的“时间点”映像，这些映像稍后可以以所有生产工作负荷完全支持的方法进行还原。 此操作通过使用来宾内的备份技术创建检查点（而而不是使用已保存状态技术）实现。 对于生产检查点，卷快照服务 (VSS) 可在 Windows 虚拟机内使用。 Linux 虚拟机会刷新其文件系统缓冲区来创建文件系统一致性检查点。 如果你想要使用已保存状态技术创建检查点，仍可以选择将标准检查点用于虚拟机。 


> **重要提示：**新虚拟机的默认行为是创建生产检查点，备用方案为创建标准检查点。 
 

## Hyper-V 管理器改进

- **备用凭据支持** – 现在，当连接到另一台 Windows 10 Technical Preview 远程主机时，可以在 Hyper-V 管理器中使用不同的凭据集。 还可以保存这些凭据，以便更易于以后登录。 

- **下层管理** - 现在可以使用 Hyper-V 管理器来管理多个版本的 Hyper-V。 借助 Windows 10 Technical Preview 中的 Hyper-V 管理器，你可以在 Windows Server 2012、Windows 8、Windows Server 2012 R2 和 Windows 8.1 上管理运行 Hyper-V 的计算机。

- **更新的管理协议** - Hyper-V 管理器已更新为使用 WS-MAN 协议（该协议允许 CredSSP、Kerberos 或 NTLM 身份验证）与远程 Hyper-V 主机进行通信。 当使用 CredSSP 连接到远程 Hyper-V 主机时，它允许你执行实时迁移，而无需首先启用 Active Directory 中的约束委派。 基于 WS-MAN 的基础结构同样简化了启用远程管理主机所需的配置。 WS-MAN 通过端口 80（该端口默认处于打开状态）进行连接。


## 连接待机兼容性 

在使用始终可用/始终连接 (AOAC) 电源模型的计算机上启用 Hyper-V 时，“连接待机”电源状态立即可用。

在 Windows 8 和 8.1 中，Hyper-V 导致使用始终可用/始终连接 (AOAC) 电源模型（也称为 InstantON）的计算机永远不会进入睡眠状态。 请参阅[知识库文章](
https://support.microsoft.com/en-us/kb/2973536) 了解完整说明。


## Linux 安全启动 

在第 2 代虚拟机上运行的多个 Linux 操作系统，现在可以使用支持的安全启动选项进行启动。  Ubuntu 14.04 及更高版本和 SUSE Linux Enterprise Server 12 支持在运行 Technical Preview 的主机上进行安全启动。 首次启动虚拟机之前，必须指定虚拟机应使用 Microsoft UEFI 证书颁发机构。  在提升的 Windows Powershell 提示符下，键入：

    Set-VMFirmware [-VMName] <VMName> [-SecureBootTemplate] <MicrosoftUEFICertificateAuthority>

有关在 Hyper-V 上运行 Linux 虚拟机的详细信息，请参阅 [Linux and FreeBSD Virtual Machines on Hyper-V（Hyper-V 上的 Linux 和 FreeBSD 虚拟机）](http://technet.microsoft.com/library/dn531030.aspx)。
 
 
## 虚拟机配置版本

将虚拟机从运行 Windows 8.1 的主机移动或导入到在 Windows 10 上运行 Hyper-V 的主机时，不会自动升级该虚拟机的配置文件。 这使得该虚拟机可以移回运行 Windows 8.1 的主机。 在手动更新虚拟机配置版本前，你无法将新的 Hyper-V 功能与虚拟机一起使用。 

虚拟机配置版本表示哪个版本的 Hyper-V 与虚拟机的配置、已保存状态和快照文件相兼容。 配置版本为 5 的虚拟机与 Windows 8.1 兼容，并且可以在 Windows 8.1 和 Windows 10 上运行。 配置版本为 6 的虚拟机与 Windows 10 兼容，不可以在 Windows 8.1 上运行。

### 查看配置版本

在提升的命令提示符下运行以下命令：

``` PowerShell
Get-VM * | Format-Table Name, Version
```

### 升级配置版本 

从提升的 Windows PowerShell 提示符，运行下列命令之一：

``` 
Update-VmConfigurationVersion <VMName>
```

或者

``` 
Update-VmConfigurationVersion <VMObject>
```

> **重要提示：**
>
- 升级虚拟机配置版本后，不能将该虚拟机移动到运行 Windows 8.1 的主机。
- 不能将虚拟机配置版本从版本 6 降级为版本 5。
- 必须先关闭虚拟机，才可以升级虚拟机配置。
- 升级完成后，虚拟机使用新的配置文件格式。 有关详细信息，请参阅 [Configuration file format（配置文件格式）](#configuration-file-format)。


## <a name="configuration-file-format"></a>配置文件格式

虚拟机现在具有新的配置文件格式，该文件格式旨在提高读取和写入虚拟机配置数据的效率。 它还旨在降低因存储故障而导致数据损坏的可能性。 新的配置文件将 .VMCX 扩展名用于虚拟机配置数据，而将 .VMRS 扩展名用于运行时状态数据。 

> **重要提示：**.VMCX 文件为二进制格式。 不支持直接编辑 .VMCX 或 .VMRS 文件。

<!--HONumber=Jun16_HO2-->


