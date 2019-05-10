---
title: Windows 和 Windows 10 上的 Linux 容器
description: 容器部署快速入门
keywords: docker，容器 LCOW
author: taylorb-microsoft
ms.date: 11/8/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 357fc101b2b0e4d6ccdf53a948ab8d91d19a1522
ms.sourcegitcommit: 34d8b2ca5eebcbdb6958560b1f4250763bee5b48
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "9621565"
---
# <a name="linux-containers-on-windows-10"></a>Windows 10 上的 Linux 容器

> [!div class="op_single_selector"]
> - [Windows 上的 Linux 容器](quick-start-windows-10-linux.md)
> - [在 Windows 上的 Windows 容器](quick-start-windows-10.md)

本练习将演练创建和 Windows 10 上运行 Linux 容器。

在本快速入门中，你将完成：

1. 适用于 Windows 的已安装的 Docker
2. 运行使用 Linux 容器上 Windows (LCOW) 的简单 Linux 容器

本快速入门特定于 Windows 10。 可以在此页面左侧的目录中找到其他快速入门文档。

## <a name="prerequisites"></a>系统必备

请确保你满足以下要求：
- 一个运行 Windows 10 专业版或企业版 Fall Creators update （版本 1709年） 或更高版本的物理计算机系统
- 请确保启用[HYPER-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) 。

***HYPER-V 隔离：*** Windows 上的 Linux 容器要求在 Windows 10 上的 HYPER-V 隔离才能为开发人员提供相应的 Linux 内核，若要运行该容器。 更多有关 HYPER-V 隔离可以找到[有关 Windows 容器](../about/index.md)页面上。

## <a name="install-docker-for-windows"></a>安装适用于 Windows 的 Docker

[Docker for Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows)下载并运行安装程序 （你将需要登录。 创建一个帐户如果你还没有）。 Docker 文档中提供了[详细的安装说明](https://docs.docker.com/docker-for-windows/install)。

> 如果已安装 Docker，请确保您有 18.02 或更高版本来支持 LCOW 版本。 通过运行检查`docker -v`或检查*有关 Docker*。

> 若要运行 LCOW 容器，必须激活*Docker 设置 > 守护程序*中的实验性功能选项。

## <a name="run-your-first-lcow-container"></a>运行你的第一个 LCOW 容器

对于此示例中，将部署 BusyBox 容器。 首先，尝试运行 Hello World BusyBox 图像。

```console
docker run --rm busybox echo hello_world
```

请注意，这可以返回错误，当 Docker 尝试拉取映像。 发生这种情况是因为 Dockers 需要通过指令`--platform`标志来确认相应地匹配图像和主机操作系统。 由于 Windows 容器模式中的默认平台是 Windows，添加`--platform linux`标志来拉取和运行该容器。

```console
docker run --rm --platform linux busybox echo hello_world
```

具有与平台，已拉取映像后`--platform`标志不再需要。 运行而无需它的命令对此进行测试。

```console
docker run --rm busybox echo hello_world
```

运行`docker images`返回已安装的映像的列表。 在此情况下，Windows 和 Linux 映像。

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

> [!TIP]
> 奖励： 请参阅上运行 LCOW Docker 的相应[博客文章](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/)。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [了解如何生成示例应用](./building-sample-app.md)