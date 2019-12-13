---
title: Windows 10 上的 windows 和 Linux 容器
description: 容器部署快速入门
keywords: docker，容器，LCOW
author: taylorb-microsoft
ms.date: 08/16/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: a52c18f13d0d6bd2102f045827285821a187579b
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909577"
---
# <a name="linux-containers-on-windows-10"></a>Windows 10 上的 Linux 容器

本练习将演练如何在 Windows 10 上创建和运行 Linux 容器。

在此快速入门中，你将完成以下操作：

1. 安装 Docker Desktop
2. 在 Windows 上使用 Linux 容器运行简单的 Linux 容器（LCOW）

本快速入门特定于 Windows 10。 可在此页面左侧的目录中找到其他快速入门文档。

## <a name="prerequisites"></a>必备条件

请确保满足以下要求：
- 一台运行 Windows 10 专业版、Windows 10 企业版或 Windows Server 2019 版本1809或更高版本的物理计算机系统
- 请确保已启用[hyper-v](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) 。

***Hyper-v 隔离：*** Windows 上的 Linux 容器需要 Windows 10 上的 Hyper-v 隔离，以便为开发人员提供适当的 Linux 内核来运行容器。 有关 Hyper-v 隔离的详细信息，请参阅[关于 Windows 容器](../about/index.md)页。

## <a name="install-docker-desktop"></a>安装 Docker Desktop

下载[Docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows)并运行安装程序（需要登录。 如果还没有帐户，请创建一个。 Docker 文档中提供了[详细的安装说明](https://docs.docker.com/docker-for-windows/install)。

> 如果已安装 Docker，请确保版本18.02 或更高版本支持 LCOW。 通过运行 `docker -v` 或检查*Docker*来进行检查。

> 若要运行 LCOW 容器，必须激活 Docker 设置中的 "试验性功能" 选项， *> Daemon* 。

## <a name="run-your-first-lcow-container"></a>运行你的第一个 LCOW 容器

在此示例中，将部署 BusyBox 容器。 首先，尝试运行 "Hello World" BusyBox 图像。

```console
docker run --rm busybox echo hello_world
```

请注意，当 Docker 尝试请求图像时，这将返回错误。 出现这种情况的原因是 Docker 需要通过 `--platform` 标志的指令来确认正确匹配映像和主机操作系统。 由于 Windows 容器模式下的默认平台为 Windows，因此添加一个 `--platform linux` 标志来请求和运行容器。

```console
docker run --rm --platform linux busybox echo hello_world
```

一旦用指定的平台拉取映像后，就不再需要 `--platform` 标志。 运行命令，不测试此命令。

```console
docker run --rm busybox echo hello_world
```

运行 `docker images` 以返回已安装的映像的列表。 在这种情况下，Windows 和 Linux 映像。

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

> [!TIP]
> 附赠：有关运行 LCOW，请参阅 Docker 的相应[博客文章](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/)。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [了解如何构建示例应用程序](./building-sample-app.md)
