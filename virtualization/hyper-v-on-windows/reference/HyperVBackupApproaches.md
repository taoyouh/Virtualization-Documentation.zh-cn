# <a name="hyper-v-backup-approaches"></a>Hyper-v 备份方法
Hyper-v 允许你从主机操作系统备份虚拟机, 而无需在虚拟机内运行自定义备份软件。  开发人员可使用多种方法, 具体取决于他们的需求。
## <a name="hyper-v-vss-writer"></a>Hyper-v VSS 编写器
Hyper-v 在支持 Hyper-v 的所有 Windows Server 版本上实现 VSS 编写器。  此 VSS 书写器使开发人员能够利用现有 VSS 基础结构来备份虚拟机。  但是, 它设计用于对服务器上的所有虚拟机同时进行备份的小型备份操作。

若要更好地理解此体系结构-请参阅本演示文稿:https://channel9.msdn.com/Events/TechEd/NorthAmerica/2010/VIR322
## <a name="hyper-v-wmi-based-backup"></a>基于 hyper-v WMI 的备份
从 Windows Server 2016 开始, Hyper-v 通过 Hyper-v WMI API 开始支持备份。  此方法仍利用虚拟机内部的 VSS 进行备份, 但不再在主机操作系统中使用 VSS。  相反, 使用引用点和弹性更改跟踪 (.RCT) 的组合, 使开发人员能够以高效的方式访问有关备份的虚拟机的信息。  此方法比在主机中使用 VSS 更具伸缩性, 但它仅在 Windows Server 2016 和更高版本上可用。

若要更好地理解此体系结构-请参阅本演示文稿:https://channel9.msdn.com/Events/TechEd/Europe/2014/CDP-B318 

下面也提供了有关如何在此处使用这些 Api 的示例:https://www.powershellgallery.com/packages/xHyper-VBackup
## <a name="methods-for-reading-backups-from-wmi-based-backup"></a>从基于 WMI 的备份中读取备份的方法
使用 Hyper-v WMI 创建虚拟机备份时, 有三种方法可读取备份中的实际数据。  每种都有独特的优势和缺点。
### <a name="wmi-export"></a>WMI 导出
开发人员可以通过 Hyper-v WMI 接口导出备份数据 (如上示例中所用)。  Hyper-v 会将更改编译到虚拟硬盘中, 并将文件复制到请求的位置。  此方法易于使用, 适用于所有方案且可远程使用。  但是, 生成的虚拟硬盘通常会创建大量数据以通过网络传输。
### <a name="win32-apis"></a>Win32 Api
开发人员可以在虚拟硬盘 Win32 API 上使用 SetVirtualDiskInformation、GetVirtualDiskInformation 和 QueryChangesVirtualDisk Api, 如下所述: https://docs.microsoft.com/windows/desktop/api/_vhd/请注意, 要使用这些 api, 仍需要使用 hyper-v WMI 创建引用关联的虚拟机上的点。  然后, 这些 Win32 Api 允许高效访问备份虚拟机的数据。  Win32 Api 有以下几个限制:
*   它们仅可在本地访问
*   不支持从共享的虚拟硬盘文件中读取数据
*   它们返回相对于虚拟硬盘内部结构的数据地址

### <a name="remote-shared-virtual-disk-protocol"></a>远程共享虚拟磁盘协议
最后, 如果开发人员需要高效地访问共享虚拟硬盘文件中的备份数据信息, 他们将需要使用远程共享虚拟磁盘协议。  [此处](https://docs.microsoft.com/openspecs/windows_protocols/ms-rsvd/c865c326-47d6-4a91-a62d-0e8f26007d15)介绍了此协议。
