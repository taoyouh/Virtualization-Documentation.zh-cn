---
title: 容器中的持久性存储
description: Windows 容器如何持久存储
keywords: 容器, 卷, 存储, 装载, 绑定挂载
author: cwilhit
ms.openlocfilehash: 945a78d4ecb9c96da4de8f7246f84b6b444dd5b5
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909667"
---
# <a name="persistent-storage-in-containers"></a>容器中的持久性存储

<!-- Great diagram would be great! -->

在某些情况下，你可能需要应用能够将数据持久保存在容器中，或者需要将生成容器时未包括的文件显示到容器中。 可以通过多种方式向容器提供永久性存储：

- 绑定挂载
- 命名卷

Docker 很好地概述了如何[使用卷](https://docs.docker.com/engine/admin/volumes/volumes/)，所以最好先阅读此概述。 本页面的其余部分重点介绍 Linux 和 Windows 之间的差异，并提供 Windows 相关示例。

## <a name="bind-mounts"></a>绑定挂载

[绑定挂载](https://docs.docker.com/engine/admin/volumes/bind-mounts/)允许容器与主机共享目录。 当希望将重启容器后在本地计算机上可用的文件存储在某个位置，或者当希望与多个容器共享此位置时，可以借助此功能。 如果你希望在可以访问相同文件的多台计算机上运行容器，则应该改用命名卷或 SMB 装载。

### <a name="permissions"></a>权限

用于绑定挂载的权限模型因容器隔离级别而异。

使用 **Hyper-V 隔离**的容器使用简单的只读或读写权限模型。 可以使用 `LocalSystem` 帐户在主机上访问文件。 如果你在容器中被拒绝访问，请确保 `LocalSystem` 可以在主机上访问该目录。 使用只读标记时，对容器内的卷进行的更改将不可见或将不保留到主机上的目录中。

使用**进程隔离**的 Windows 容器略微不同，因为它们使用容器中的进程标识来访问数据，即遵循文件 ACL 的规定。 在容器中运行的进程的标识（默认情况下为 Windows Server Core 上的“ContainerAdministrator”和 Nano Server 容器上的“ContainerUser”）将用于访问装入卷（而非 `LocalSystem`）中的文件和目录，并将需要获得访问权限以使用数据。

由于这些标识只存在于容器上下文中，而不存在于存储文件所在的主机上，因此在将 ACL 配置为授予容器访问权限时，应使用 `Authenticated Users` 等众所周知的安全组。

> [!WARNING]
> 请不要将 `C:\` 等敏感目录绑定挂载到不受信任的容器中。 这会允许更改主机上通常无法访问的文件，并且可能会产生安全漏洞。

示例用法：

- `docker run -v c:\ContainerData:c:\data:RO` 用于只读访问
- `docker run -v c:\ContainerData:c:\data:RW` 用于读写访问
- `docker run -v c:\ContainerData:c:\data` 用于读写访问（默认）

### <a name="symlinks"></a>符号链接

符号链接在容器中解析。 如果将主机路径绑定挂载到属于符号链接或者包含符号链接的容器 - 容器将无法访问它们。

### <a name="smb-mounts"></a>SMB 装载

在 Windows Server 版本 1709 及更高版本中，利用称为“SMB 全局映射”的功能可以在主机上装载 SMB 共享，然后将该共享上的目录传递到容器中。 不需要为容器配置特定的服务器、共享、用户名或密码 - 这些都在主机上进行处理。 该容器将如同具有本地存储一样工作。

#### <a name="configuration-steps"></a>配置步骤

1. 在容器主机上，全局性地映射远程 SMB 共享：
    ```
    $creds = Get-Credential
    New-SmbGlobalMapping -RemotePath \\contosofileserver\share1 -Credential $creds -LocalPath G:
    ```
    此命令将使用凭据向远程 SMB 服务器进行身份验证。 然后，将远程共享路径映射到 G: 驱动器号（可以是任何其他可用的驱动器号）。 在此容器主机上创建的容器现在可以将其数据卷映射到 G: 驱动器上的路径。

    > [!NOTE]
    > 对容器使用 SMB 全局映射时，容器主机上的所有用户都可以访问远程共享。 在容器主机上运行的任何应用程序也将有权访问映射的远程共享。

2. 创建容器，并将数据卷映射到全局装载的 SMB 共享  docker run -it --name demo -v g:\ContainerData:G:\AppData1 microsoft/windowsservercore:1709 cmd.exe

    在容器内部，G:\AppData1 将映射到远程共享的“ContainerData”目录。 存储在全局映射的远程共享上的任何数据都将能够供容器内的应用程序使用。 多个容器可以通过相同的命令获得对此共享数据的读/写访问权限。

此 SMB 全局映射支持是 SMB 客户端功能，可以在任何兼容的 SMB 服务器上工作，包括：

- 存储空间直通 (S2D) 或传统 SAN 上的横向扩展文件服务器
- Azure 文件（SMB 共享）
- 传统文件服务器
- SMB 协议的第三方实现（例如：NAS 设备）

> [!NOTE]
> SMB 全局映射不支持 Windows Server 版本 1709 中的 DFS、DFSN 和 DFSR 共享。

## <a name="named-volumes"></a>命名卷

命名卷允许你按名称创建卷、将卷分配给容器，以及稍后以相同名称重新使用该卷。 你无需跟踪卷的创建位置的实际路径，只需跟踪名称。 Windows 上的 Docker 引擎有一个内置的命名卷插件，可以在本地计算机上创建卷。 如果要在多台计算机上使用命名卷，则需要使用其他插件。

示例步骤：

1. `docker volume create unwound` - 创建名为“unwound”的卷
2. `docker run -v unwound:c:\data microsoft/windowsservercore` - 启动容器并将卷映射到 c:\data
3. 将某些文件写入容器中的 c:\data，然后停止容器
4. `docker run -v unwound:c:\data microsoft/windowsservercore` - 启动新容器
5. 在新容器中运行 `dir c:\data` - 文件仍在原位置

> [!NOTE]
> Windows Server 会将目标路径名（容器内的路径）转换为小写；也就是说，如果 Linux 容器中的路径为 `-v unwound:c:\MyData` 或 `-v unwound:/app/MyData`，系统会在 Linux 容器中映射容器中目录 `c:\mydata` 或 `/app/mydata`（如果该目录不存在，则还会创建它）。
