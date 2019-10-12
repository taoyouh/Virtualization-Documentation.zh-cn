---
title: 容器存储概述
description: Windows Server 容器如何使用主机和其他存储类型
keywords: 容器, 卷, 存储, 装载, 绑定挂载
author: cwilhit
ms.openlocfilehash: fba08de884d59cc1b656895ec2b7078ba3975269
ms.sourcegitcommit: 22dcc1400dff44fb85591adf0fc443360ea92856
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/12/2019
ms.locfileid: "10209747"
---
# <a name="container-storage-overview"></a>容器存储概述

<!-- Great diagram would be great! -->

本主题概述了容器在 Windows 上使用存储的不同方式。 在存储时，容器的行为与虚拟机不同。 根据本质，生成容器以防止在其内部运行的应用通过主机的文件系统上的 "写入" 状态。 默认情况下，容器使用 "暂存" 空间，但 Windows 还提供持久保留存储的方法。

## <a name="scratch-space"></a>暂存空间

默认情况下，Windows 容器使用暂时存储。 所有容器 i/o 都将出现在 "暂存空间" 中，并且每个容器都将获取它们自己的暂存。 文件创建和文件写入在暂存空间中捕获，并且不会向主机进行转义。 当容器实例停止时，会丢弃在暂存空间中发生的所有更改。 启动新的容器实例时，将为该实例提供新的暂存空间。

## <a name="layer-storage"></a>层存储

如[容器概述](../about/index.md)中所述，容器图像是表示为一系列图层的文件包。 图层存储是已内置到容器中的所有文件。 每次对该容器执行 `docker pull` 然后执行 `docker run` 时，它们都是一样的。

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

## <a name="persistent-storage"></a>永久存储

Windows 容器支持通过 bind 装载和卷提供持久存储的机制。 若要了解详细信息，请参阅[容器中的永久存储](./persistent-storage.md)。

## <a name="storage-limits"></a>存储限制

Windows 应用程序的常见模式是先查询可用磁盘空间量，然后再安装或创建新文件，或者触发临时文件清理。  利用最大化应用程序兼容性的目标，Windows 容器中的 C：驱动器表示20的虚拟空间。

某些用户可能希望替代此默认设置，并将可用空间配置为更小或更大的值。 这可以通过 "存储选择" 配置中的 "大小" 选项实现。

### <a name="examples"></a>示例

命令行： `docker run --storage-opt "size=50GB" mcr.microsoft.com/windows/servercore:ltsc2019 cmd`

或者，你可以直接更改 docker 配置文件：

```Docker Configuration File
"storage-opts": [
    "size=50GB"
  ]
```

> [!TIP]
> 此方法也适用于 docker 生成。 请参阅[配置 docker](https://docs.microsoft.com/virtualization/windowscontainers/manage-docker/configure-docker-daemon#configure-docker-with-configuration-file) 文档，以了解关于修改 docker 配置文件的更多详细信息。
