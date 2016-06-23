---
title: 容器数据卷
description: 使用 Windows 容器创建和管理数据卷。
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: f5998534-917b-453c-b873-2953e58535b1
---

# 容器数据卷

**这是初步内容，可能还会更改。** 

创建容器时，你可能需要创建新的数据目录，或将现有的目录添加到容器中。 这可通过添加数据卷来实现。 数据卷对容器和容器主机均可见，并且可以在它们之间共享数据。 在同一容器主机上也可以在多个容器之间共享数据卷。 本文档将详细介绍如何创建、检查和删除数据卷。

## 数据卷

### 创建新的数据卷

使用 `docker run` 命令的 `-v` 参数创建新的数据卷。 默认情况下，新的数据卷存储于主机上的“c:\ProgramData\Docker\volumes”下。

此示例将创建一个名为“new-data-volume”的数据卷。 可以在“c:\new-data-volume”处正在运行的容器中访问此数据卷。

```none
docker run -it -v c:\new-data-volume windowsservercore cmd
```

有关创建卷的详细信息，请参阅 [Manage data in containers on docker.com](https://docs.docker.com/engine/userguide/containers/dockervolumes/#data-volumes)（在 docker.com 上管理容器中的数据）。

### 装载现有目录

除了创建新的数据卷之外，你可能希望将现有目录从主机传递到容器中。 这也可以通过使用 `docker run` 命令的 `-v` 参数来实现。 主机目录中所有的文件也可在容器中使用。 任何在已装入卷中由容器创建的文件在主机上都可用。 同一目录可装载到多个容器上。 在此配置中，可以在容器间共享数据。

在此示例中，将源目录“c:\source”作为“c:\destination”装载到容器内。

```none
docker run -it -v c:\source:c:\destination windowsservercore cmd
```

有关装载主机目录的详细信息，请参阅 [Manage data in containers on docker.com](https://docs.docker.com/engine/userguide/containers/dockervolumes/#mount-a-host-directory-as-a-data-volume)（在 docker.com 上管理容器中的数据）。

### 装载单个文件

通过显式指定文件名，可以将单个文件装载到容器中。 在此示例中，将要共享的目录包含许多文件，但只有“config.ini”文件在容器内部可用。 

```none
docker run -it -v c:\container-share\config.ini windowsservercore cmd
```

在正在运行的容器内部，仅 config.ini 文件可见。

```none
c:\container-share>dir
 Volume in drive C has no label.
 Volume Serial Number is 7CD5-AC14

 Directory of c:\container-share

04/04/2016  12:53 PM    <DIR>          .
04/04/2016  12:53 PM    <DIR>          ..
04/04/2016  12:53 PM    <SYMLINKD>     config.ini
               0 File(s)              0 bytes
               3 Dir(s)  21,184,208,896 bytes free
```

有关装载单个文件的详细信息，请参阅 [Manage data in containers on docker.com](https://docs.docker.com/engine/userguide/containers/dockervolumes/#mount-a-host-directory-as-a-data-volume)（在 docker.com 上管理容器中的数据）。

### 数据卷容器

可以使用 `docker run` 命令的 `--volumes-from` 参数，从其他正在运行的容器中继承数据卷。 使用此继承，可以创建明确用于为容器化应用程序承载数据卷的容器。 

此示例将数据卷从容器“cocky_bell”装载到新的容器中。 新的容器启动后，在此卷中找到的数据可用于在该容器中运行的应用程序。  

```none
docker run -it --volumes-from cocky_bell windowsservercore cmd
```

有关数据容器的详细信息，请参阅 [Manage data in containers on docker.com](https://docs.docker.com/engine/userguide/containers/dockervolumes/#mount-a-host-file-as-a-data-volume)（在 docker.com 上管理容器中的数据）。

### 检查共享的数据卷

可以使用 `docker inspect` 命令查看已装入的卷。

```none
docker inspect backstabbing_kowalevski
```

这将返回有关容器的信息，包括一个名为“Mounts”的部分，其中包含了有关已装入卷的数据，如源和目标目录。

```none
"Mounts": [
    {
        "Source": "c:\\container-share",
        "Destination": "c:\\data",
        "Mode": "",
        "RW": true,
        "Propagation": ""
}
```

有关检查卷的详细信息，请参阅 [Manage data in containers on docker.com](https://docs.docker.com/engine/userguide/containers/dockervolumes/#locating-a-volume)（在 docker.com 上管理容器中的数据）。



<!--HONumber=May16_HO4-->


