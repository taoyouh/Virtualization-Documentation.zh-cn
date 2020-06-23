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
ms.openlocfilehash: 17186d868c0934c4af670e522b26f9205dd16f76
ms.sourcegitcommit: 6a5c237bff2c953fec2ce1e09424375a7c615010
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/09/2020
ms.locfileid: "84632978"
---
# <a name="linux-containers-on-windows-10"></a>Windows 10 上的 Linux 容器

本练习将演练如何在 Windows 10 上创建并运行 Linux 容器。

在本快速入门中，我们将完成以下操作：

1. 安装 Docker Desktop
2. 运行简单的 Linux 容器

本快速入门特定于 Windows 10。 此页面左侧的目录中提供其他快速入门文档。

## <a name="prerequisites"></a>必备条件

请确保满足以下要求：
- 一个运行 Windows 10 专业版、Windows 10 企业版或 Windows Server 2019 版本 1809 或更高版本的物理计算机系统
- 确保已启用 [Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements)。 

## <a name="install-docker-desktop"></a>安装 Docker Desktop

下载 [Docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows) 并运行安装程序（需要登录。 如果没有帐户，请创建一个）。 Docker 文档中提供了[详细的安装说明](https://docs.docker.com/docker-for-windows/install)。

## <a name="run-your-first-linux-container"></a>运行你的第一个 Linux 容器

要运行 Linux 容器，需要确保 Docker 面向的是正确的守护程序。 在系统托盘中单击 Docker 鲸鱼图标时，可从操作菜单中选择 `Switch to Linux Containers` 来切换此选项。 如果看到 `Switch to Windows Containers`，则表示目标已经是 Linux 守护程序。

![Docker 系统任务栏菜单，显示“切换到 Windows 容器”命令。](./media/switchDaemon.png)

确认目标是正确的守护程序后，使用以下命令运行容器：

```console
docker run --rm busybox echo hello_world
```

容器应会运行、打印出“hello_world”，然后退出。 

查询 `docker images` 时，应会看到你刚才拉取运行的 Linux 容器映像：

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [了解如何生成示例应用](./building-sample-app.md)
