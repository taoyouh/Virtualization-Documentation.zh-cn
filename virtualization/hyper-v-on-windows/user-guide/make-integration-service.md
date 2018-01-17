---
title: "创建你自己的集成服务"
description: "Windows 10 集成服务。"
keywords: windows 10, hyper-v, HVSocket, AF_HYPERV
author: scooley
ms.date: 04/07/2017
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: 1ef8f18c-3d76-4c06-87e4-11d8d4e31aea
ms.openlocfilehash: 01b9c2febb9f098c7981599894e488b946857900
ms.sourcegitcommit: 5fe5c30acfc4d5edb28633d30669f616fcc5d59a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/21/2017
---
# <a name="make-your-own-integration-services"></a>创建你自己的集成服务

从 Windows 10 周年更新开始，任何人都可以创建通过 Hyper-V 套接字在 Hyper-V 主机与其虚拟机之间进行通信的应用程序。Hyper-V 套接字是一种 Windows 套接字，使用面向虚拟器的新地址系列和专用终结点。  所有通信在 Hyper-V 套接字上运行时均无需使用网络，并且所有数据都保留在相同的物理内存中。   使用 Hyper-V 套接字的应用程序类似于 Hyper-V 集成服务。

本文档演示了如何创建在 Hyper-V 套接字上构建的简单程序。

**受支持的主机操作系统**
* Windows 10 和更高版本
* Windows Server 2016 和更高版本

**受支持的来宾操作系统**
* Windows 10 和更高版本
* Windows Server 2016 和更高版本
* 使用 Linux Integration Services 的 Linux 来宾（请参阅 [Windows 上的 Hyper-V 支持的 Linux 和 FreeBSD 虚拟机](https://technet.microsoft.com/library/dn531030.aspx)）

**功能和限制**
* 支持内核模式或用户模式操作
* 仅限数据流
* 无块内存（并非最适用于备份/视频）

--------------

## <a name="getting-started"></a>入门

要求：
* C/C++ 编译器。  如果没有该组件，请查看 [Visual Studio Community](https://aka.ms/vs)
* [Windows 10 SDK](https://developer.microsoft.com/windows/downloads/windows-10-sdk) - 已在 Visual Studio 2015 Update 3 以及更高版本上预安装。
* 使用至少一个虚拟机运行以上其中一个主机操作系统的计算机。 - 这用于测试应用程序。

> **注意：**用于 Hyper-V 套接字的 API 不久后将在 Windows 10 中公开发布。  使用 HVSocket 的应用程序将在任何 Widnows 10 主机和来宾上运行，但仅可使用高于 14290 版本的 Windows SDK 进行开发。

## <a name="register-a-new-application"></a>注册新应用程序
若要使用 Hyper-V 套接字，必须向 Hyper-V 主机的注册表注册应用程序。

通过在注册表中注册该服务，你将获得：
*  用于启用、禁用和列出可用服务的 WMI 管理
*  直接与虚拟机进行通信的权限

以下 PowerShell 将注册一个名为“HV Socket Demo”的新应用程序。  必须以管理员身份运行此应用程序。  手册注册说明如下。

``` PowerShell
$friendlyName = "HV Socket Demo"

# Create a new random GUID.  Add it to the services list
$service = New-Item -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices" -Name ((New-Guid).Guid)

# Set a friendly name
$service.SetValue("ElementName", $friendlyName)

# Copy GUID to clipboard for later use
$service.PSChildName | clip.exe
```


**注册表位置和信息：**
```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices\
```
在此注册表位置中，你将看到多个 GUID。  这些都是我们的内置服务。

每个服务的注册表中的信息：
* `Service GUID`
    * `ElementName (REG_SZ)` -- 这是服务的友好名称

若要注册你自己的服务，请使用你自己的 GUID 和友好名称来创建新的注册表项。

友好名称将与新的应用程序相关联。  它将出现在性能计数器和其他 GUID 不适合的位置中。

注册表条目将如下所示：
```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices\
    999E53D4-3D5C-4C3E-8779-BED06EC056E1\
        ElementName REG_SZ  VM Session Service
    YourGUID\
        ElementName REG_SZ  Your Service Friendly Name
```

> **提示：**若要在 PowerShell 中生成 GUID，并将其复制到剪贴板，请运行：
``` PowerShell
(New-Guid).Guid | clip.exe
```

## <a name="create-a-hyper-v-socket"></a>创建 Hyper-V 套接字

在最基本的情况下，定义套接字需要地址系列、连接类型和协议。

下面是一个简单的[套接字定义](
https://msdn.microsoft.com/en-us/library/windows/desktop/ms740506(v=vs.85).aspx
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
* 类型 - `SOCK_STREAM`
* 协议 - `HV_PROTOCOL_RAW`


下面是一个示例声明/实例化：
``` C
SOCKET sock = socket(AF_HYPERV, SOCK_STREAM, HV_PROTOCOL_RAW);
```


## <a name="bind-to-a-hyper-v-socket"></a>绑定到 Hyper-V 套接字

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
* VM ID - 这是每个 VM 分配的唯一 ID。  使用以下 PowerShell 代码段可以找到 VM 的 ID。
  ```PowerShell
  (Get-VM -Name $VMName).Id
  ```
* 服务 ID - [如上所述](#RegisterANewApplication)，在 Hyper-V 主机注册表中用于注册应用程序的 GUID。

还有一组 VMID 通配符可在未连接到特定虚拟机时使用。

### <a name="vmid-wildcards"></a>VMID 通配符

| 名称 | GUID | 说明 |
|:-----|:-----|:-----|
| HV_GUID_ZERO | 00000000-0000-0000-0000-000000000000 | 侦听器应绑定到此 VMID 以接受所有分区中的连接。 |
| HV_GUID_WILDCARD | 00000000-0000-0000-0000-000000000000 | 侦听器应绑定到此 VMID 以接受所有分区中的连接。 |
| HV_GUID_BROADCAST | FFFFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF | |
| HV_GUID_CHILDREN | 90db8b89-0d35-4f79-8ce9-49ea0ac8b7cd | 子级的通配符地址。 侦听器应绑定到此 VMID 以接受其子级中的连接。 |
| HV_GUID_LOOPBACK | e0e16197-dd56-4a10-9195-5ee7a155a838 | 环回地址。 使用此 VMID 连接到与该连接器相同的分区。 |
| HV_GUID_PARENT | a42e7cda-d03f-480c-9cc2-a4de20abb878 | 父地址。 使用此 VMID 连接到该连接器的父分区。* |


\* `HV_GUID_PARENT` 虚拟机的父级是其主机。  容器的父级是容器的主机。
从在虚拟机中运行的容器进行连接将连接到托管该容器的 VM。
侦听此 VmId 可接受以下来源的连接：（内部容器）：容器主机。
（内部 VM：容器主机/非容器）：VM 主机。
（非内部 VM：容器主机/非容器）：不受支持。

## <a name="supported-socket-commands"></a>受支持的套接字命令

Socket() Bind() Connect() Send() Listen() Accept()

## <a name="useful-links"></a>有用链接
[完整 WinSock API](https://msdn.microsoft.com/en-us/library/windows/desktop/ms741394.aspx)

[Hyper-V 集成服务参考](../reference/integration-services.md)