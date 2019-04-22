---
title: 容器生态系统
description: 生成容器生态系统。
keywords: 元数据, 容器
author: PatrickLang
ms.date: 04/20/2016
ms.topic: about-article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 29fbe13a-228a-4eaa-9d4d-90ae60da5965
ms.openlocfilehash: 19340f2ca3ca11e9b75a223bf2b58e943328a0c5
ms.sourcegitcommit: a5ff22c205149dac4fc05325ef3232089826f1ef
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/22/2019
ms.locfileid: "9380151"
---
# <a name="building-a-container-ecosystem"></a>生成容器生态系统

若要了解为何生成容器生态系统如此重要，让我们先介绍一下 Docker。

## <a name="docker"></a>Docker

容器的概念（命名空间隔离和资源管理）已存在很长时间，可以追溯到 BSD Jails、Solaris Zones 和基本的 UNIX chroot（更改根）机制。   Docker 的用途是提供通用工具集、封装模型和部署机制。  通过执行此操作，Docker 简化了此和应用程序的分发。  然后可以在任何 Linux 主机上的任何位置运行这些应用程序，此功能也在 Windows 上提供。

普遍封装模型和部署技术通过提供对任何主机相同的管理命令来简化管理，并创建为无缝 DevOps 一个独特的机会。

从开发人员的桌面到测试计算机再到一组生产计算机，可以创建以相同方式在几秒内在任何环境中部署的 Docker 映像。 由此创造出了封装在 Docker 容器中的巨大且持续增长的应用程序生态系统，其中 DockerHub 是 Docker 所维护的公共容器化应用程序注册表。

Docker 为开发提供了绝佳的基础。

现在让我们谈一谈应用程序的生态系统以及你可以如何以 Docker 概念为基础创建适合你的需求的开发和部署工作流。

## <a name="components-in-a-container-ecosystem"></a>容器生态系统中的组件

Windows 容器是一个大型容器生态系统的关键组件。 我们正致力于跨行业在解决方案堆栈的每一层上为开发人员提供选项。

容器生态系统提供管理容器、共享容器和开发在容器中运行的应用的方法。

![](media/containerEcosystem.png)

Microsoft 希望在开发人员生成这些下一代的应用时为其提供选项和工作效率。  我们的目标是为开发人员提高效率，这意味着支持应用程序面向任何 Microsoft 云，而无需修改、重新编写或重新配置代码。

Microsoft 致力于以友好方式实现开放和生态系统。  我们积极支持融合多个相关的开发人员生态系统（如 Windows 和 Linux）来推动创新。

在随后的几个月内，我们将提供有关此开发生态系统中的其他合作伙伴的详细信息。
