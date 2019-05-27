---
title: 关于 Docker
description: 了解 Docker。
keywords: docker, 容器
author: Heidilohr
ms.author: helohr
ms.date: 05/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: cdb7771a1293e7c3fd505103f0010bdfba47cc31
ms.sourcegitcommit: daf1d2b5879c382404fc4d59f1c35c88650e20f7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/23/2019
ms.locfileid: "9674874"
---
# <a name="about-docker"></a>关于 Docker

阅读有关容器的资料时，你将不可避免地看到有关 Docker 的信息。 Docker 引擎是一个容器管理工具集, 用于打包和传递容器映像。 生成的图像可以作为容器 (无论是在本地、在云中还是在个人计算机上) 在任意位置运行。

![](media/docker.png)

你可以像使用任何其他容器一样管理带有[Docker](https://www.docker.com)的 Windows Server 容器。

通过容器的命名空间隔离和资源管理的概念已有很长时间, 返回到 BSD Jails、Solaris 区域和基本的 UNIX 更改根 (chroot) 机制。 Docker 通过一个通用工具集、打包模型和部署机制来为开发奠定坚实的基础, 从而简化了应用程序的 containerization 和分发。 然后, 这些应用程序可以在任何 Linux 主机上和 Windows 中的任意位置运行。

广泛使用的打包模型和部署技术通过为任何主机提供相同的管理命令来简化管理, 从而为无缝的 DevOps 提供了一个独特的机会。 你还可以创建一个可在任何环境 (以秒为单位) 部署的 Docker 图像, 无论它是开发人员的桌面、测试计算机还是一组生产计算机。 这在 DockerHub 容器中创建了大量和不断增加的应用程序, 这些应用程序在 docker 容器中使用, 它是由 Docker 的公共容器的应用程序注册表。

现在, 让我们来讨论一下应用程序体系, 以及如何在 Docker 概念上构建一个适合你的需要的开发和部署工作流。

## <a name="get-started-with-docker"></a>快速开始使用 Docker

若要了解如何利用 Docker 构建容器, 请参阅[Windows 上的 Docker 引擎](../manage-docker/configure-docker-daemon.md)。 你还可以更深入地访问[docker 网站](https://www.docker.com), 了解如何使用 docker。