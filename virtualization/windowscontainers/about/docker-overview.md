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
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910787"
---
# <a name="about-docker"></a>关于 Docker

阅读有关容器的资料时，你将不可避免地看到有关 Docker 的信息。 Docker 引擎是一个容器管理工具集，用于打包和传递容器映像。 生成的映像可在任何位置作为容器运行，无论它是在本地、云中还是个人计算机上。

![](media/docker.png)

可以使用 [Docker](https://www.docker.com) 来管理 Windows Server 容器，就像管理任何其他容器一样。

通过容器进行命名空间隔离和资源管理的概念已存在很长时间，可以追溯到 BSD Jails、Solaris Zones 和基本的 UNIX 更改根 (chroot) 机制。 Docker 通过通用工具集、封装模型和部署机制为开发打下了坚实的基础，简化了应用程序的容器化和分发。 然后，这些应用程序就可以在任何 Linux 主机和 Windows 中的任意位置运行。

这是一种普遍存在的包装模型和部署技术，可以提供针对任何主机的相同管理命令，通过这种方式来简化管理，为无缝 DevOps 创造独特的机会。 还可以创建以相同方式在几秒内在任何环境中部署的 Docker 映像，不管是开发人员的台式机、测试计算机还是一组生产计算机。 这创造出了封装在 Docker 容器中的巨大且持续增长的应用程序生态系统，其中 DockerHub 是 Docker 所维护的公共容器化应用程序注册表。

现在，让我们谈一谈应用程序的生态系统以及你可以如何以 Docker 概念为基础创建适合你的需求的开发和部署工作流。

## <a name="get-started-with-docker"></a>Docker 入门

若要了解如何通过 Docker 生成容器，请参阅 [Windows上的 Docker 引擎](../manage-docker/configure-docker-daemon.md)。 还可以访问 [Docker 网站](https://www.docker.com)，更深入地了解如何使用 Docker。