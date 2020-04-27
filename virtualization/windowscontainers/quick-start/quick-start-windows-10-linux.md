---
title: Windows 10 上的 Windows 和 Linux 容器
description: 容器部署快速入门
keywords: docker, 容器, LCOW
author: taylorb-microsoft
ms.date: 08/16/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: a52c18f13d0d6bd2102f045827285821a187579b
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/24/2020
ms.locfileid: "74909577"
---
# <a name="linux-containers-on-windows-10"></a>Windows 10 上的 Linux 容器

本练习将演练如何在 Windows 10 上创建并运行 Linux 容器。

在本快速入门中，我们将完成以下操作：

1. 安装 Docker Desktop
2. 使用 Windows 上的 Linux 容器 (LCOW) 运行简单的 Linux 容器

本快速入门特定于 Windows 10。 此页面左侧的目录中提供其他快速入门文档。

## <a name="prerequisites"></a>必备条件

请确保满足以下要求：
- 一个运行 Windows 10 专业版、Windows 10 企业版或 Windows Server 2019 版本 1809 或更高版本的物理计算机系统
- 确保已启用 [Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements)。

Hyper-V 隔离：Windows 上的 Linux 容器需要 Windows 10 上的 Hyper-V 隔离，以便为开发人员提供适当的 Linux 内核来运行容器。 有关 Hyper-V 隔离的详细信息，可参阅[关于 Windows 容器](../about/index.md)页。

## <a name="install-docker-desktop"></a>安装 Docker Desktop

下载 [Docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows) 并运行安装程序（需要登录。 如果没有帐户，请创建一个）。 Docker 文档中提供了[详细的安装说明](https://docs.docker.com/docker-for-windows/install)。

> 如果已安装 Docker，请确保你有支持 LCOW 的 18.02 或更高版本。 通过运行 `docker -v` 进行查看，或者查看*关于 Docker*。

> 必须激活“Docker 设置”>“守护程序”中的“试验性功能”选项，才能运行 LCOW 容器。 

## <a name="run-your-first-lcow-container"></a>运行你的第一个 LCOW 容器

在此示例中，我们将部署 BusyBox 容器。 首先，尝试运行“Hello World”BusyBox 映像。

```console
docker run --rm busybox echo hello_world
```

请注意，当 Docker 尝试拉取映像时，这样做会返回错误。 出现这种情况的原因是，Docker 需要一个使用 `--platform` 标志的指令来确认映像和主机操作系统是否已正确匹配。 由于 Windows 容器模式下的默认平台为 Windows，因此添加 `--platform linux` 标志即可拉取并运行容器。

```console
docker run --rm --platform linux busybox echo hello_world
```

一旦用指定的平台拉取了映像，就不再需要 `--platform` 标志。 运行不带该标志的命令，对此进行测试。

```console
docker run --rm busybox echo hello_world
```

运行 `docker images` 会返回已安装映像的列表。 在本示例中，会返回 Windows 和 Linux 映像。

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

> [!TIP]
> 其他参考文档：请参阅 Docker 的相应[博客文章](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/)，了解如何运行 LCOW。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [了解如何生成示例应用](./building-sample-app.md)
