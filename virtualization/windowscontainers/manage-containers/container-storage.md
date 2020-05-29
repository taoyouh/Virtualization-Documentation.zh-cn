---
title: 容器存储概述
description: Windows Server 容器如何使用主机和其他存储类型
keywords: 容器, 卷, 存储, 装载, 绑定挂载
author: cwilhit
ms.openlocfilehash: f758877f1131813fe4637a01c03b49d7a18a83c4
ms.sourcegitcommit: db085db8a54664184a2f7cfa01d00598a1c66992
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/04/2020
ms.locfileid: "78288669"
---
# <a name="container-storage-overview"></a>容器存储概述

<!-- Great diagram would be great! -->

本主题概述了容器在 Windows 上使用存储的各种方式。 在使用存储方面，容器的行为与虚拟机不同。 本质上，容器旨在阻止在其中运行的应用在主机的文件系统中写入状态。 默认情况下，容器使用“暂存”空间，但 Windows 也提供持久存储方法。

## <a name="scratch-space"></a>暂存空间

默认情况下，Windows 容器使用临时存储。 所有容器 I/O 都在“暂存空间”中进行，每个容器都有其自己的暂存空间。 文件创建和文件写入都被捕获到暂存空间中，不会逃逸到主机中。 当停止容器实例时，暂存空间中进行的所有更改都会被丢弃。 启动新的容器实例时，将为该实例提供一个新的暂存空间。

## <a name="layer-storage"></a>层存储

如[容器概述](../about/index.md)中所述，容器映像是以一系列层表示的文件的捆绑包。 层存储是内置于容器中的所有文件。 每次对该容器执行 `docker pull` 然后执行 `docker run` 时，它们都是一样的。

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

## <a name="persistent-storage"></a>持久性存储

Windows 容器支持通过绑定装载和卷提供持久性存储的各种机制。 若要了解详细信息，请参阅[容器中的持久性存储](./persistent-storage.md)。

## <a name="storage-limits"></a>存储限制

Windows 应用程序的常见模式是先查询可用磁盘空间量，然后再安装或创建新文件，或者触发临时文件清理。  Windows 容器中的 C: 驱动器表示一个 20GB 的虚拟可用大小，这样设置的目的是最大限度地提高应用程序兼容性。

某些用户可能想要覆盖此默认设置，将可用空间配置为更小或更大的值。 这可以通过“storage-opt”配置中的“size”选项来实现。

### <a name="examples"></a>示例

命令行：`docker run --storage-opt "size=50GB" mcr.microsoft.com/windows/servercore:ltsc2019 cmd`

也可直接更改 docker 配置文件：

```Docker Configuration File
"storage-opt": [
    "size=50GB"
  ]
```

> [!TIP]
> 此方法也适用于 docker build。 请参阅[配置 docker](https://docs.microsoft.com/virtualization/windowscontainers/manage-docker/configure-docker-daemon#configure-docker-with-configuration-file) 文档，以了解关于修改 docker 配置文件的更多详细信息。
