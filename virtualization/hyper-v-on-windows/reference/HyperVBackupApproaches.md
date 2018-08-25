# <a name="hyper-v-backup-approaches"></a>HYPER-V 备份方法
HYPER-V 允许您向备份虚拟机，主机操作系统，无需运行虚拟机中的自定义备份软件。  有若干种方法可供开发人员可以利用具体取决于他们的需求。
## <a name="hyper-v-vss-writer"></a>HYPER-V VSS 编写器
HYPER-V 实现 VSS 编写器上的 Windows Server HYPER-V 支持其中的所有版本。  此 VSS 编写器使开发人员可以利用备份虚拟机的现有 VSS 基础架构。  但是，它专为其中的服务器上的所有虚拟机进行都备份同时小型备份操作。

若要更好地理解此体系结构 – 引用此演示文稿：https://channel9.msdn.com/Events/TechEd/NorthAmerica/2010/VIR322
## <a name="hyper-v-wmi-based-backup"></a>基于 HYPER-V WMI 的备份
从 Windows Server 2016，HYPER-V 启动支持通过 HYPER-V WMI API 的备份。  此方法仍利用 VSS 虚拟机中进行备份，但不再主机操作系统中使用 VSS。  而是组合的参考点和弹性修订 (RCT) 用于允许开发人员访问有关备份更有效地虚拟机。  此方法是比在主机使用 VSS 更具可伸缩性，但是仅可在 Windows Server 2016 及更高版本。

若要更好地理解此体系结构 – 引用此演示文稿：https://channel9.msdn.com/Events/TechEd/Europe/2014/CDP-B318 

有关如何使用这些 Api 可在此处还有一个示例：https://msconfiggallery.cloudapp.net/packages/xHyper-VBackup
## <a name="methods-for-reading-backups-from-wmi-based-backup"></a>用于从 WMI 基于备份读取备份方法
在创建虚拟机备份使用 HYPER-V WMI 时，有三种用于从备份中读取的实际数据的方法。  每个具有独特的优点和缺点。
### <a name="wmi-export"></a>WMI 导出
开发人员可以将导出的 HYPER-V WMI 接口 （如上面的示例中使用） 通过备份数据。  HYPER-V 将编译到虚拟硬盘驱动器上的更改，并将文件复制到请求的位置。  此方法易于使用和适用于所有方案是远程。  但是，通常生成的虚拟硬盘创建大量数据可通过网络传输。
### <a name="win32-apis"></a>Win32 Api
开发人员可以使用 SetVirtualDiskInformation，GetVirtualDiskInformation 并 QueryChangesVirtualDisk Api 虚拟硬盘 Win32 API 在此处将记录的设置： https://docs.microsoft.com/en-us/windows/desktop/api/_vhd/ ，若要使用这些 Api，HYPER-V WMI 仍需要使用创建引用的注释在关联的虚拟机上的点。  高效的备份的虚拟机的数据访问然后允许这些 Win32 Api。  Win32 Api 有一些限制：
*   他们只能访问本地
*   执行不支持读取数据中的共享虚拟硬盘文件
*   它们返回是相对于虚拟硬盘的内部结构的数据地址

### <a name="remote-shared-virtual-disk-protocol"></a>远程共享的虚拟磁盘协议
最后，如果开发人员需要高效地从共享的虚拟硬盘文件 – 访问备份数据信息他们将需要使用远程共享虚拟磁盘协议。  下面介绍了此协议：https://msdn.microsoft.com/en-us/library/dn393384.aspx
