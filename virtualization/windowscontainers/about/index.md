---
title: About Windows Containers
description: Learn about Windows containers.
keywords: docker, containers
author: taylorb-microsoft
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8e273856-3620-4e58-9d1a-d1e06550448
ms.openlocfilehash: 2be7a06c7b7b154e392c30981cdf954d2d1b796e
ms.sourcegitcommit: 8e193d8c274a549aef497f16dcdb00d7855e9fa7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/02/2017
---
# Windows 容器

## 什么是容器

容器是一种将应用程序包装到其自身隔离空间内的方法。 位于容器中的应用不了解该容器外存在的所有其他应用程序或进程。 应用程序成功运行所需的所有依赖项也存在于此容器内。  无论容器移动到何处，应用程序都将始终得到满足，因为应用程序与其运行所需的一切都已绑定在一起。

这就好像是一个厨房。 我们打包所有的电器和家具、锅碗瓢盆、洗洁精和毛巾。 这便是我们的容器

<center style="margin: 25px">![](media/box1.png)</center>

我们现在可以带着这个容器，将它放在任何喜欢的入住公寓中，厨房还会是这个厨房。 我们所需做的全部工作就是为它接通水电，然后我们便可以立即开始烹饪（因为我们拥有所有需要的器具！）

<center style="margin: 25px">![](media/apartment.png)</center>

容器在很大程度上就像是这个厨房。 可以有不同类型的房间以及许多相同类型的房间。 重要的一点是容器与其所需的一切内容打包在一起。

在此处观看简短概述：[基于 Windows 的容器：使用企业级控制的现代应用开发](https://youtu.be/Ryx3o0rD5lY)。

## 容器基础知识

容器是独立的、资源受控制的和可移植的运行时环境，在主机或虚拟机上运行。 在容器中运行的应用程序或进程与所有需要的依赖项和配置文件打包在一起；在它看来，容器之外似乎没有任何其他进程在运行。

容器的主机为容器预配一组资源，且容器只会使用这些资源。 在容器看来，除了已经为其提供的资源之外，不存在其他资源，因此它不能接触到可能已为相邻容器预配的资源。

在你开始创建和使用 Windows 容器时，以下关键概念将会很有用。

**Container Host:** Physical or Virtual computer system configured with the Windows Container feature. 容器主机将运行一个或多个 Windows 容器。

**容器映像：**在对容器文件系统或注册表进行修改时（如软件安装），将在沙盒中捕获这些修改。 在许多情况下，你可能希望捕获此状态，以便可以创建继承这些更改的新容器。 That’s what an image is – once the container has stopped you can either discard that sandbox or you can convert it into a new container image. For example, let’s imagine that you have deployed a container from the Windows Server Core OS image. You then install MySQL into this container. Creating a new image from this container would act as a deployable version of the container. This image would only contain the changes made (MySQL), however would work as a layer on top of the Container OS Image.

**Sandbox:** Once a container has been started, all write actions such as file system modifications, registry modifications or software installations are captured in this ‘sandbox’ layer.

**Container OS Image:** Containers are deployed from images. The container OS image is the first layer in potentially many image layers that make up a container. 此映像提供操作系统环境。 容器操作系统映像是不可变的。 也就是说，不能对其进行修改。

**容器存储库：**每次创建容器映像时，容器映像及其依赖项都会存储在本地存储库中。 这些映像可以在容器主机上重复使用多次。 容器映像还可以存储在公共或私有注册表（如 DockerHub）中，以便可以在许多不同的容器主机上使用它们。

<center>![](media/containerfund.png)</center>

对于熟悉虚拟机的人员而言，容器可能具有令人难以置信的相似性。 A container runs an operating system, has a file system and can be accessed over a network just as if it was a physical or virtual computer system. 话虽如此，但容器背后的技术和概念与虚拟机有很大不同。

Microsoft Azure 专家 Mark Russinovich 有[一篇精彩的博客文章](https://azure.microsoft.com/en-us/blog/containers-docker-windows-and-trends/)详述了这些差异。

## Windows 容器类型

Windows Containers include two different container types, or runtimes.

**Windows Server Containers** – provide application isolation through process and namespace isolation technology. A Windows Server Container shares a kernel with the container host and all containers running on the host. 这些容器不提供敌对安全边界，不应该用于隔离不受信任的代码。 由于共享内核空间，这些容器要求具有相同的内核版本和配置。

**Hyper-V 隔离** - 通过在高度优化的虚拟机中运行每个容器，在由 Windows Server 容器提供的隔离上扩展。 In this configuration, the kernel of the container host is not shared with other containers on the same host. 这些容器旨在托管敌对多租户，并且具有与虚拟机相同的安全保证。 由于这些容器与主机或主机上的其他容器不共享内核，它们可运行（与受支持的版本）采用不同版本和配置的内核 - 例如 Windows 10 上的所有 Windows 容器都使用 Hyper-V 隔离以充分利用 Windows Server 内核版本和配置。

在 Windows 上运行容器时是否使用 Hyper-V 隔离将在运行时决定。 你可以在最初选择创建具有 Hyper-V 隔离的容器，而稍后在运行时选择将其作为 Windows Server 容器运行。

## 什么是 Docker？

阅读有关容器的资料时，你将不可避免地看到有关 Docker 的信息。 Docker 是对容器映像进行打包和传送的容器。 此自动化过程会生成稍后可能在任何地方（在本地、在云中或在个人计算机上）作为容器运行的映像（相当于模板）。

<center>![](media/docker.png)</center>

同任何其他容器一样，可以通过 [Docker](https://www.docker.com) 管理 Windows Server 容器。

## 面向开发人员的容器 ##

From a developer’s desktop to a testing machine to a set of production machines, a Docker image can be created that will deploy identically across any environment in seconds. This story has created a massive and growing ecosystem of applications packaged in Docker containers, with DockerHub, the public containerized-application registry that Docker maintains, currently publishing more than 180,000 applications in the public community repository.

When you containerize an app, only the app and the components needed to run the app are combined into an "image". Containers are then created from this image as you need them. You can also use an image as a baseline to create another image, making image creation even faster. Multiple containers can share the same image, which means containers start very quickly and use fewer resources. For example, you can use containers to spin up light-weight and portable app components – or ‘micro-services’ – for distributed apps and quickly scale each service separately.

Because the container has everything it needs to run your application, they are very portable and can run on any machine that is running Windows Server 2016. You can create and test containers locally, then deploy that same container image to your company's private cloud, public cloud or service provider. The natural agility of Containers supports modern app development patterns in large scale, virtualized and cloud environments.

With containers, developers can build an app in any language. These apps are completely portable and can run anywhere - laptop, desktop, server, private cloud, public cloud or service provider - without any code changes.  

Containers helps developers build and ship higher-quality applications, faster.

## Containers for IT Professionals ##

IT Professionals can use containers to provide standardized environments for their development, QA, and production teams. They no longer have to worry about complex installation and configuration steps. By using containers, systems administrators abstract away differences in OS installations and underlying infrastructure.

Containers help admins create an infrastructure that is simpler to update and maintain.

## Video Overview

<iframe src="https://channel9.msdn.com/Blogs/containers/Containers-101-with-Microsoft-and-Docker/player" width="800" height="450" allowFullScreen="true" frameBorder="0" scrolling="no"></iframe>

## 试用 Windows Server 容器

已准备好开始利用容器的强大功能？ 请点击下方的链接亲自开始部署你的第一个容器： <br/>
对于 Windows Server 上的用户，请转到此处 - [Windows Server 快速入门简介](../quick-start/quick-start-windows-server.md) <br/>
对于 Windows 10 上的用户，请转到此处 - [Windows 10 快速入门简介](../quick-start/quick-start-windows-10.md)

