---
title: Windows 10 上的 windows 和 Linux 容器
description: 容器部署快速入门
keywords: docker、容器、LCOW
author: taylorb-microsoft
ms.date: 11/8/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 91031f9394cb3fcb1af6c4813f8805ad6f79bf8c
ms.sourcegitcommit: a7f9ab96be359afb37783bbff873713770b93758
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/28/2019
ms.locfileid: "9681097"
---
# <a name="linux-containers-on-windows-10"></a>Windows 10 上的 Linux 容器

> [!div class="op_single_selector"]
> - [Windows 上的 Linux 容器](quick-start-windows-10-linux.md)
> - [Windows 上的 windows 容器](quick-start-windows-10.md)

该练习将指导你在 Windows 10 上创建和运行 Linux 容器。

在此快速入门中, 你将完成以下操作:

1. 安装 Docker 桌面
2. 使用 Windows 上的 Linux 容器 (LCOW) 运行简单的 Linux 容器

本快速入门特定于 Windows 10。 可在此页面左侧的目录中找到其他快速入门文档。

## <a name="prerequisites"></a>系统必备

请确保满足以下要求: <<<<<<< 头
- 通过秋季式创意者更新 (版本 1709) 或更高版本运行 Windows 10 专业版或企业版的一台物理计算机系统
- 请确保已启用[hyper-v](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) 。
=======
- 运行 Windows 10 专业版、Windows 10 企业版或 Windows Server 2019 版本1809或更高版本的一台物理计算机系统
- 请确保已启用[hyper-v](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) 。
>>>>>>> 原创/母版

***Hyper-v 隔离:*** Windows 上的 Linux 容器需要 Windows 10 上的 Hyper-v 隔离, 以便为开发人员提供合适的 Linux 内核来运行容器。 有关 Hyper-v 隔离的详细信息可以在 "[关于 Windows 容器](../about/index.md)" 页面上找到。

## <a name="install-docker-desktop"></a>安装 Docker 桌面

下载[Docker 桌面](https://store.docker.com/editions/community/docker-ce-desktop-windows)并运行安装程序 (您将需要登录。 如果尚未有帐户, 请创建一个帐户。 Docker 文档中提供了[详细的安装说明](https://docs.docker.com/docker-for-windows/install)。

> 如果你已安装了 Docker, 请确保你有版本18.02 或更高版本, 以支持 LCOW。 通过运行`docker -v`或检查*Docker*来进行检查。

> 必须激活*Docker 设置 _GT_ 守护*程序中的 "实验功能" 选项才能运行 LCOW 容器。

## <a name="run-your-first-lcow-container"></a>运行第一个 LCOW 容器

对于此示例, 将部署一个 BusyBox 容器。 首先, 尝试运行 "Hello World" BusyBox 图像。

```console
docker run --rm busybox echo hello_world
```

请注意, 当 Docker 试图拉入图像时, 这会返回错误。 出现这种情况是因为 Dockers 需要通过`--platform`标志的指令来确认正确匹配的图像和主机操作系统。 由于 Windows 容器模式中的默认平台是 Windows, 因此添加`--platform linux`一个标志以拉入并运行容器。

```console
docker run --rm --platform linux busybox echo hello_world
```

通过指示的平台提取图像后, 不再需要`--platform`标志。 运行命令而无需测试此命令。

```console
docker run --rm busybox echo hello_world
```

运行`docker images`以返回已安装的映像的列表。 在这种情况下, Windows 和 Linux 映像都是。

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

> [!TIP]
> 附加: 在运行 LCOW 时查看 Docker 的相应[博客文章](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/)。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [了解如何构建示例应用](./building-sample-app.md)
