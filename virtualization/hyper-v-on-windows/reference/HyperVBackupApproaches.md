# <a name="hyper-v-backup-approaches"></a>Hyper-V 备份方法
可以通过 Hyper-V 从主机操作系统备份虚拟机，无需在虚拟机内运行自定义备份软件。  开发人员可以根据自己的需求使用多种方法。
## <a name="hyper-v-vss-writer"></a>Hyper-V VSS 编写器
Hyper-V 在支持 Hyper-V 的所有 Windows Server 版本上实现 VSS 编写器。  此 VSS 编写器允许开发人员利用现有的 VSS 基础结构来备份虚拟机。  但是，其设计目的是在进行小规模备份操作时，同时备份服务器上的所有虚拟机。

若要更好地了解此体系结构，请参阅此演示： https://channel9.msdn.com/Events/TechEd/NorthAmerica/2010/VIR322
## <a name="hyper-v-wmi-based-backup"></a>基于 Hyper-V WMI 的备份
从 Windows Server 2016 起，Hyper-V 开始通过 Hyper-V WMI API 支持备份。  此方法仍会利用虚拟机内部的 VSS 进行备份，但不再使用主机操作系统中的 VSS，  而是使用参考点和复原更改跟踪 (RCT) 的组合，使开发人员能够以高效方式访问有关已备份虚拟机的信息。  此方法比在主机中使用 VSS 更具可伸缩性，但仅在 Windows Server 2016 及更高版本上使用。

若要更好地了解此体系结构，请参阅此演示： https://channel9.msdn.com/Events/TechEd/Europe/2014/CDP-B318 

下面还提供了有关如何使用这些 API 的示例： https://www.powershellgallery.com/packages/xHyper-VBackup
## <a name="methods-for-reading-backups-from-wmi-based-backup"></a>从基于 WMI 的备份读取备份的方法
使用 Hyper-V WMI 创建虚拟机备份时，可以通过三种方法从备份中读取实际数据。  每种方法都有不同的优缺点。
### <a name="wmi-export"></a>WMI 导出
开发人员可以通过 Hyper-V WMI 接口导出备份数据（如上一示例中所用的那样）。  Hyper-V 会将更改编译到虚拟硬盘中，并将文件复制到请求的位置。  此方法易于使用，适用于所有方案且可远程操作。  但是，生成的虚拟硬盘驱动器通常会创建大量可通过网络进行传输的数据。
### <a name="win32-apis"></a>Win32 API
开发人员可以使用虚拟硬盘 Win32 API 集中的 SetVirtualDiskInformation API、GetVirtualDiskInformation API 和 QueryChangesVirtualDisk API，如下所述： https://docs.microsoft.com/windows/desktop/api/_vhd/ 请注意，若要使用这些 API，仍需使用 Hyper-V WMI 在关联的虚拟机上创建参考点。  然后，可以使用这些 Win32 API 对已备份虚拟机的数据进行高效访问。  Win32 API 有几项限制：
* 只能在本地访问
* 不支持从共享的虚拟硬盘文件中读取数据
* 它们返回相对于虚拟硬盘的内部结构的数据地址

### <a name="remote-shared-virtual-disk-protocol"></a>远程共享虚拟磁盘协议
最后，如果开发人员需要高效访问共享虚拟硬盘文件中的备份数据信息，他们需要使用远程共享虚拟磁盘协议。  此协议记录在[此处](https://docs.microsoft.com/openspecs/windows_protocols/ms-rsvd/c865c326-47d6-4a91-a62d-0e8f26007d15)。
