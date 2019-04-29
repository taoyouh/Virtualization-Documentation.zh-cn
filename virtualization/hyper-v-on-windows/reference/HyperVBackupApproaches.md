# <a name="hyper-v-backup-approaches"></a>HYPER-V 备份方法
从主机操作系统，而无需运行的虚拟机中的自定义备份软件中，HYPER-V 允许你对备份虚拟机。  有多种方法可用于开发人员利用具体取决于他们的需求。
## <a name="hyper-v-vss-writer"></a>HYPER-V VSS 编写器
HYPER-V 上的所有版本的 Windows Server 在支持 HYPER-V 实现 VSS 编写器。  此 VSS 编写器允许开发人员利用备份虚拟机的现有 VSS 基础结构。  但是，它专为小范围备份操作在服务器上的所有虚拟机会都备份同时。

若要更好地理解此体系结构 – 请参阅此演示文稿：https://channel9.msdn.com/Events/TechEd/NorthAmerica/2010/VIR322
## <a name="hyper-v-wmi-based-backup"></a>HYPER-V WMI 基于备份
从 Windows Server 2016 开始，HYPER-V 启动支持通过 HYPER-V WMI API 的备份。  此方法仍然利用 VSS 虚拟机中进行备份，但不会再在主机操作系统中使用 VSS。  相反，参考点和复原更改跟踪 (RCT) 的组合用于允许开发人员能够访问有关的信息以高效方式备份虚拟机。  这种方法是比在主机中，使用 VSS 的可扩展性更高，但仅适用于 Windows Server 2016 及更高版本。

若要更好地理解此体系结构 – 请参阅此演示文稿：https://channel9.msdn.com/Events/TechEd/Europe/2014/CDP-B318 

有关如何使用这些 Api 可此处还有一个示例：https://msconfiggallery.cloudapp.net/packages/xHyper-VBackup
## <a name="methods-for-reading-backups-from-wmi-based-backup"></a>从 WMI 基于备份读取备份方法
创建时使用 HYPER-V WMI 的虚拟机备份，有三种方法从备份读取的实际数据。  每个有唯一的优点和缺点。
### <a name="wmi-export"></a>WMI 导出
开发人员可以导出通过 HYPER-V WMI 接口 （如上述示例中使用） 的备份数据。  HYPER-V 将编译到虚拟硬盘驱动器的更改，并将文件复制到请求的位置。  此方法是易于使用，适用于所有方案并且可远程处理。  但是，通常生成虚拟硬盘创建大量要通过网络传输的数据。
### <a name="win32-apis"></a>Win32 Api
开发人员可以使用 SetVirtualDiskInformation，GetVirtualDiskInformation 和 QueryChangesVirtualDisk Api 虚拟硬盘 Win32 API 在此处设置如此处所述：https://docs.microsoft.com/en-us/windows/desktop/api/_vhd/请注意，若要使用这些 Api，HYPER-V WMI 仍需要在用于创建引用相关联的虚拟机上的点。  这些 Win32 Api 然后允许高效的备份虚拟机的数据访问。  Win32 Api 有一些限制：
*   仅可以本地访问
*   不要支持读取数据从共享虚拟硬盘文件
*   它们将返回数据都与虚拟硬盘的内部结构的地址

### <a name="remote-shared-virtual-disk-protocol"></a>远程共享的虚拟磁盘协议
最后，如果开发人员需要高效地从共享虚拟硬盘文件-访问备份数据信息它们需要使用远程共享虚拟磁盘协议。  此处记录了此协议：https://msdn.microsoft.com/en-us/library/dn393384.aspx
