# 生成新的管理服务

从 Windows 10 开始，Hyper-V 允许在 Hyper-V 来宾和主机之间进行已注册的套接字连接，而不依赖于网络连接。 使用 Hyper-V 套接字，服务可以独立于网络堆栈运行，并且所有数据都保留在相同的物理内存中。

本文档演练了如何创建一个在 Hyper-V 套接字上生成的简单应用程序以及如何开始使用它们。

[PowerShell Direct](../user_guide/vmsession.md) 是使用 Hyper-V 套接字进行通信的应用程序（在这种情况下是内置 Windows 服务）的一个示例。

**受支持的主机操作系统**
* Windows 10
* Windows Server Technical Preview 3
* 未来版本 (Server 2016 +)

**受支持的来宾操作系统**
* Windows 10
* Windows Server Technical Preview 3
* 未来版本 (Server 2016 +)

**功能和限制**
* 支持内核模式或用户模式操作
* 仅限数据流
* 无块内存（并非最适用于备份/视频）

--------------


## 开始使用

现在，本机代码 (C/C++) 中提供了 Hyper-V 套接字。

若要编写简单的应用程序，你将需要：
* C 编译器。 如果没有该功能，请查看 [Visual Studio 代码](https://aka.ms/vs)
* 一台运行 Hyper-V 和虚拟机的计算机。
* 主机和来宾 (VM) 操作系统必须是 Windows 10 、Windows Server Technical Preview 3 或更高版本。
* Windows SDK - 下面是包含 `hvsocket.h` 的 [Win10 SDK](https://dev.windows.com/en-us/downloads/windows-10-sdk) 的链接。

## 注册新应用程序

若要使用 Hyper-V 套接字，必须向 Hyper-V 主机的注册表注册应用程序。

通过在注册表中注册该服务，你将获得：
*  用于启用、禁用和列出可用服务的 WMI 管理
*  直接与虚拟机进行通信的权限

以下 PowerShell 将注册一个名为“HV Socket Demo”的新应用程序。 必须以管理员身份运行此应用程序。 手册说明如下。

``` PowerShell
$friendlyName = "HV Socket Demo"

# Create a new random GUID and add it to the services list then add the name as a value

$service = New-Item -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices" -Name ([System.Guid]::NewGuid().ToString())

$service.SetValue("ElementName", $friendlyName)

# Copy GUID to clipboard for later use
$service.PSChildName | clip.exe
```

** 注册表位置和信息 **

``` 
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices\
```
在此注册表位置中，你将看到多个 GUID。 这些都是我们的内置服务。

每个服务的注册表中的信息：
* `服务 GUID`
* `ElementName (REG_SZ)` - 这是服务的友好名称

若要注册你自己的服务，请使用你自己的 GUID 和友好名称来创建新的注册表项。

友好名称将与新的应用程序相关联。 它将出现在性能计数器和其他 GUID 不适合的位置中。

注册表条目将如下所示：
```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices\
    999E53D4-3D5C-4C3E-8779-BED06EC056E1\
        ElementName REG_SZ  VM Session Service
    YourGUID\
        ElementName REG_SZ  Your Service Friendly Name
```

>** 提示：**若要在 PowerShell 中生成 GUID，并将其复制到剪贴板，请运行：
``` PowerShell
[System.Guid]::NewGuid().ToString() | clip.exe
```

## 创建 Hyper-V 套接字

在最基本的情况下，定义套接字需要地址系列、连接类型和协议。

下面是一个简单的 [套接字定义](
https://msdn.microsoft.com/zh-cn/library/windows/desktop/ms740506 (v=vs.85).aspx
)

``` C
SOCKET WSAAPI socket(
  _In_ int af,
  _In_ int type,
  _In_ int protocol
);
```

对于 Hyper-V 套接字：
* 地址系列 - `AF_HYPERV`
* 类型 - `SOCK_STREAM`、`SOCK_DGRAM` 或 `SOCK_RAW`
* 协议 - `HV_PROTOCOL_RAW`


下面是一个示例声明/实例化：
``` C
SOCKET sock = socket(AF_HYPERV, SOCK_STREAM, HV_PROTOCOL_RAW);
```


## 绑定到 Hyper-V 的套接字

绑定将套接字与连接信息相关联。

为了方便起见，会在下面复制函数定义，请在[此处](https://msdn.microsoft.com/en-us/library/windows/desktop/ms737550.aspx)阅读有关绑定的详细信息。

``` C
int bind(
  _In_ SOCKET                s,
  _In_ const struct sockaddr *name,
  _In_ int                   namelen
);
```

与用于标准 Internet 协议地址系列 (`AF_INET`)（由该主机上的主机 IP 地址和端口号组成）的套接字地址 (sockaddr) 相比，用于 `AF_HYPERV` 的套接字地址使用上面定义的虚拟机 ID 和应用程序 ID 来建立连接。

由于 Hyper-V 套接字不依赖于网络堆栈、TCP/IP、DNS 等，因此套接字终结点需要的是非 IP，而不是仍可明确描述连接的主机名和格式。

下面是 Hyper-V 套接字的套接字地址的定义：

``` C
struct SOCKADDR_HV
{
     ADDRESS_FAMILY Family;
     USHORT Reserved;
     GUID VmId;
     GUID ServiceId;
};
```

作为 IP 或主机名的替代，AF_HYPERV 终结点非常依赖于以下两个 GUID：
* VM ID - 这是每个 VM 分配的唯一 ID。 使用以下 PowerShell 代码段可以找到 VM 的 ID。
  ```PowerShell
  (Get-VM -Name $VMName).Id
  ```
* 服务 ID - [如上所述](#RegisterANewApplication)，在 Hyper-V 主机注册表中用于注册应用程序的 GUID。

还有一组 VMID 通配符可在未连接到特定虚拟机时使用。

### VMID 通配符

| 名称| GUID| 描述|
|:-----|:-----|:-----|
| HV_GUID_ZERO| 00000000-0000-0000-0000-000000000000| 侦听器应绑定到此 VMID 以接受所有分区中的连接。|
| HV_GUID_WILDCARD| 00000000-0000-0000-0000-000000000000| 侦听器应绑定到此 VMID 以接受所有分区中的连接。|
| HV_GUID_BROADCAST| FFFFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF| |
| HV_GUID_CHILDREN| 90db8b89-0d35-4f79-8ce9-49ea0ac8b7cd| 子级的通配符地址。侦听器应绑定到此 VMID 以接受其子级中的连接。|
| HV_GUID_LOOPBACK| e0e16197-dd56-4a10-9195-5ee7a155a838| 环回地址。使用此 VMID 连接到与该连接器相同的分区。|
| HV_GUID_PARENT| a42e7cda-d03f-480c-9cc2-a4de20abb878| 父地址。使用此 VMID 连接到该连接器的父分区。*|


***HV_GUID_PARENT**
虚拟机的父级是其主机。 容器的父级是容器的主机。
从在虚拟机中运行的容器进行连接将连接到托管该容器的 VM。
侦听此 VMID 可接受以下来源的连接：
（内部容器）：容器主机。
（内部 VM：容器主机/非容器）：VM 主机。
（非内部 VM：容器主机/非容器）：不受支持。

## 受支持的套接字命令

Socket()
Bind()
Connect()
Send()
Listen()
Accept()

[完整的 WinSock API](https://msdn.microsoft.com/en-us/library/windows/desktop/ms741394.aspx)

## 正在进行的工作

正常断开连接
选择



<!--HONumber=Dec15_HO1-->
