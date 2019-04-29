---
title: Windows Server 容器存储
description: Windows Server 容器如何使用主机和其他存储类型
keywords: 容器, 卷, 存储, 装载, 绑定挂载
author: patricklang
ms.openlocfilehash: 7d22a149da21a3367b82f2920c189ae9a4b1c173
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/26/2019
ms.locfileid: "9574868"
---
# <a name="overview"></a>概述

<!-- Great diagram would be great! -->


## <a name="layer-storage"></a>层存储

这是内置于容器中的所有文件。 每次对该容器执行 `docker pull` 然后执行 `docker run` 时，它们都是一样的。


### <a name="where-layers-are-stored-and-how-to-change-it"></a>层的存储位置及其更改方式

在默认安装中，层存储在 `C:\ProgramData\docker` 中，并且分布在“image”和“windowsfilter”目录中。 你可以使用 `docker-root` 配置来更改层的存储位置，如 [Windows 上的 Docker 引擎](../manage-docker/configure-docker-daemon.md)文档中所示。

> [!NOTE]
> 层存储仅支持 NTFS。 不支持 ReFS。

不应修改层目录中的任何文件 - 这些文件应通过以下命令进行谨慎管理：

- [docker images](https://docs.docker.com/engine/reference/commandline/images/)
- [docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/)
- [docker pull](https://docs.docker.com/engine/reference/commandline/pull/)
- [docker load](https://docs.docker.com/engine/reference/commandline/load/)
- [docker save](https://docs.docker.com/engine/reference/commandline/save/)

### <a name="supported-operations-in-layer-storage"></a>层存储中支持的操作

运行中的容器可以使用大多数 NTFS 操作，但事务除外。 这包括设置 ACL，所有 ACL 均在容器内进行检查。 如果你想在容器内以多个用户身份运行进程，则可以在 `Dockerfile` 中使用 `RUN net user /create ...` 创建用户，设置文件 ACL，然后使用 [Dockerfile USER 指令](https://docs.docker.com/engine/reference/builder/#user)将进程配置为以该用户身份运行。


##  <a name="image-size"></a>映像大小
Windows 应用程序的常见模式是先查询可用磁盘空间量，然后再安装或创建新文件，或者触发临时文件清理。  为了最大限度地提高应用程序兼容性，Windows 容器中的 C: 驱动器将表示 20GB 的虚拟可用大小。  某些用户可能想要覆盖此默认值并将可用空间配置为较小值或较大值，此时可以通过“storage-opt”配置中的“size”选项来达到此目的。

### <a name="examples"></a>示例
命令行： `docker run --storage-opt "size=50GB" microsoft/windowsservercore:1709 cmd`

Docker 配置文件
```
"storage-opts": [
    "size=50GB"
  ]
```
> 请注意，此方法适用于 docker build。
请参阅[配置 docker](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-docker/configure-docker-daemon#configure-docker-with-configuration-file) 文档，以了解关于修改 docker 配置文件的更多详细信息。


## <a name="persistent-volumes"></a>永久卷

可以通过多种方式向容器提供永久性存储：

- 绑定挂载
- 命名卷

Docker 很好地概述了如何[使用卷](https://docs.docker.com/engine/admin/volumes/volumes/)，所以最好先阅读此概述。 本页面的其余部分重点介绍 Linux 和 Windows 之间的差异，并提供 Windows 相关示例。


### <a name="bind-mounts"></a>绑定挂载

[绑定挂载](https://docs.docker.com/engine/admin/volumes/bind-mounts/)允许容器与主机共享目录。 当希望将重启容器后在本地计算机上可用的文件存储在某个位置，或者当希望与多个容器共享此位置时，可以借助此功能。 如果你希望在可以访问相同文件的多台计算机上运行容器，则应该改用命名卷或 SMB 装载。

#### <a name="permissions"></a>权限

用于绑定挂载的权限模型因容器隔离级别而异。

使用 **Hyper-V 隔离**的容器（包括 Windows Server 版本 1709 上的 Linux 容器）使用简单的只读或读写权限模型。
可以使用 `LocalSystem` 帐户在主机上访问文件。 如果你在容器中被拒绝访问，请确保 `LocalSystem` 可以在主机上访问该目录。
使用只读标记时，对容器内的卷进行的更改将不可见或将不保留到主机上的目录中。

使用**进程隔离**的 Windows Server 容器略微不同，因为它们使用容器中的进程标识来访问数据，即遵循文件 ACL 的规定。
在容器中运行的进程的标识（默认情况下为 Windows Server Core 上的“ContainerAdministrator”和 Nano Server 容器上的“ContainerUser”）将用于访问装入卷（而非 `LocalSystem`）中的文件和目录，并将需要获得访问权限以使用数据。
由于这些标识只存在于容器上下文中，而不存在于存储文件所在的主机上，因此在将 ACL 配置为授予容器访问权限时，应使用 `Authenticated Users` 等众所周知的安全组。

> [!WARNING]
> 请不要将 `C:\` 等敏感目录绑定挂载到不受信任的容器中。 这会允许更改主机上通常无法访问的文件，并且可能会产生安全漏洞。

示例用法： 

- `docker run -v c:\ContainerData:c:\data:RO` 适用于只读访问
- `docker run -v c:\ContainerData:c:\data:RW` 适用于读写访问
- `docker run -v c:\ContainerData:c:\data` 适用于读写访问（默认）

#### <a name="symlinks"></a>符号链接

符号链接在容器中解析。 如果将主机路径绑定挂载到属于符号链接或者包含符号链接的容器 - 容器将无法访问它们。

#### <a name="smb-mounts"></a>SMB 装载

在 Windows Server 版本 1709 上，利用称为“SMB 全局映射”的新功能可以在主机上装载 SMB 共享，然后将该共享上的目录传递到容器中。 不需要为容器配置特定的服务器、共享、用户名或密码 - 这些都在主机上进行处理。 该容器将如同具有本地存储一样工作。

##### <a name="configuration-steps"></a>配置步骤

1. 在容器主机上，全局映射远程 SMB 共享：
    ```
    $creds = Get-Credential
    New-SmbGlobalMapping -RemotePath \\contosofileserver\share1 -Credential $creds -LocalPath G:
    ```
    此命令将使用的凭据远程 SMB 服务器进行身份验证。 然后，将远程共享路径映射到 G: 驱动器号（可以是任何其他可用的驱动器号）。 在此容器主机上创建的容器现在可以将其数据卷映射到 G: 驱动器上的路径。

    > 注意：对容器使用 SMB 全局映射时，容器主机上的所有用户都可以访问远程共享。 在容器主机上运行的任何应用程序也将有权访问映射的远程共享。

2. 创建容器，并将数据卷映射到全局装载的 SMB 共享  docker run -it --name demo -v g:\ContainerData:G:\AppData1 microsoft/windowsservercore:1709 cmd.exe

    在容器内部，G:\AppData1 将映射到远程共享的“ContainerData”目录。 存储在全局映射的远程共享上的任何数据都将能够供容器内的应用程序使用。 多个容器可以通过相同的命令获得对此共享数据的读/写访问权限。

此 SMB 全局映射支持是 SMB 客户端功能，可以在任何兼容的 SMB 服务器上工作，包括：

- 存储空间直通 (S2D) 或传统 SAN 上的横向扩展文件服务器
- Azure 文件（SMB 共享）
- 传统文件服务器
- SMB 协议的第三方实现（例如：NAS 设备）

> [!NOTE]
> SMB 全局映射不支持 Windows Server 版本 1709 中的 DFS、DFSN 和 DFSR 共享。

### <a name="named-volumes"></a>命名卷

命名卷允许你按名称创建卷、将卷分配给容器，以及稍后以相同名称重新使用该卷。 你无需跟踪卷的创建位置的实际路径，只需跟踪名称。 Windows 上的 Docker 引擎有一个内置的命名卷插件，可以在本地计算机上创建卷。 如果要在多台计算机上使用命名卷，则需要使用其他插件。

示例步骤：

1. `docker volume create unwound` - 创建名为“unwound”的卷
2. `docker run -v unwound:c:\data microsoft/windowsservercore` - 启动容器并将卷映射到 c:\data
3. 将某些文件写入容器中的 c:\data，然后停止容器
4. `docker run -v unwound:c:\data microsoft/windowsservercore` - 启动新容器
5. 在新容器中运行 `dir c:\data` - 文件仍在原位置
