---
title: Windows 容器快速入门
description: Windows 容器快速入门。
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-contianers
ms.service: windows-containers
ms.assetid: 4878f5d2-014f-4f3c-9933-97f03348a147
---

# Windows 容器快速入门

**这是初步内容，可能还会更改。** 

Windows 容器快速入门介绍了产品和容器技术、分步骤介绍了简单的容器部署示例，并且还提供了更高级主题的参考。 如果你是第一次使用容器或 Windows 容器，完成本快速入门中的每个步骤会为你带来技术上的实际动手体验。

## 1.什么是容器

它们是隔离、资源控制且可移植的操作环境。

基本上，容器是一个隔离的位置，应用程序可在其中运行，而不会影响系统的其他部分，并且系统也不会影响该应用程序。 容器是虚拟化的下一个演化。

如果你在容器内，看起来会很像你在一个新安装的物理计算机或虚拟机内。 并且，对 [Docker](https://www.docker.com/) 来说，可以使用与管理任何其他容器相同的方式来管理 Windows 容器。

## 2.Windows 容器类型

Windows 容器包括两个不同的容器类型或运行时。

**Windows Server 容器** - 通过进程和命名空间隔离技术提供应用程序隔离。 Windows Server 容器与容器主机和该主机上运行的所有容器共享内核。

**Hyper-V 容器** - 通过在高度优化的虚拟机中运行每个容器，在由 Windows Server 容器提供的隔离上扩展。 在此配置中，容器主机的内核不与 Hyper-V 容器共享。

## 3.容器基础知识

当你开始使用容器时，你会注意到容器和虚拟机之间的许多相似之处。 容器在操作系统上运行、具有文件系统，并且可以通过网络访问，就像它是物理或虚拟计算机系统一样。 话虽如此，但容器背后的技术和概念与虚拟机有很大不同。 在你开始创建和使用 Windows 容器时，以下关键概念将会很有用。 

**容器主机：**- 使用 Windows 容器功能配置的物理或虚拟计算机系统。

**容器操作系统映像：**- 从映像部署容器。 容器操作系统映像是可能组成容器的许多映像层中的第一层。 此映像提供操作系统环境。

**容器映像：**- 一个容器映像包含快速部署容器所需的基本操作系统映像、应用程序和所有应用程序依赖关系。 

**容器注册表：** -容器映像存储在容器注册表中，并且可以按需下载。 

**Dockerfile：** -Dockerfile 用于自动创建容器映像。

## 下一步：

[Windows Server 容器快速入门](./quick_start_windows_server.md)  

[Windows 10 容器快速入门](./quick_start_windows_10.md)



<!--HONumber=May16_HO4-->


