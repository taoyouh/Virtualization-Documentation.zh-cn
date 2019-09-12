---
title: Windows 容器基本映像历史记录
description: 与 SHA256 层哈希一起发布的 Windows 容器映像列表
keywords: docker, 容器, 哈希
author: patricklang
ms.date: 01/12/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 88e6e080-cf8f-41d8-a301-035959dc5ce0
ms.openlocfilehash: b2f2d6418fdda2ad0aa0b81c05efad6b99f74375
ms.sourcegitcommit: 73134bf279f3ed18235d24ae63cdc2e34a20e7b7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/12/2019
ms.locfileid: "10107901"
---
# <a name="container-base-images"></a>容器基映像

## <a name="supported-base-images"></a>支持的基本图像

Windows 容器提供四个容器基映像： Windows Server Core、Nano Server、Windows 和 IoT Core。 并非所有配置都支持这两个操作系统映像。 下表详细介绍所支持的配置。

|主机操作系统|Windows 容器|Hyper-V 隔离|
|---------------------|-----------------|-----------------|
|Windows Server 2016 或 Windows Server 2019 （标准版或数据中心）|服务器核心版、Nano Server、Windows|服务器核心版、Nano Server、Windows|
|Nano Server|Nano Server|服务器核心版、Nano Server、Windows|
|Windows 10 专业版或 Windows 10 企业版|不可用|服务器核心版、Nano Server、Windows|
|IoT Core|IoT Core|不可用|

> [!WARNING]  
> 从 Windows Server 版本1709开始，Nano Server 将不再作为容器主机提供。

## <a name="base-image-differences"></a>基本图像差异

如何选择要在其上构建的正确的基本图像？ 尽管你可以随意构建任何内容，但以下是每个图像的一般指南：

- [Windows Server Core](https://hub.docker.com/_/microsoft-windows-servercore)：如果你的应用程序需要完整的 .net framework，这是要使用的最佳图像。
- [Nano server](https://hub.docker.com/_/microsoft-windows-nanoserver)：对于仅需要 .net Core 的应用程序，Nano server 将提供大量精简图像。
- [Windows](https://hub.docker.com/_/microsoft-windowsfamily-windows)：你可能会发现你的应用程序依赖于服务器 Core 或 Nano server 映像（如 GDI 库）中缺少的某个组件或 .dll。 此映像携带 Windows 的完整依赖集。
- [IoT 核心](https://hub.docker.com/_/microsoft-windows-iotcore)：此图像专为[IoT 应用程序](https://developer.microsoft.com/windows/iot)而构建。 在面向 IoT 核心主机时，应使用此容器图像。

对于大多数用户，Windows Server Core 或 Nano Server 将是最适合使用的映像。 当你考虑在 Nano Server 上生成时，请注意以下事项：

- 已删除服务堆栈
- 未包含 .NET Core（但你可以使用 [.NET Core Nano Server 映像](https://hub.docker.com/r/microsoft/dotnet/)）
- 已删除 PowerShell
- 已删除 WMI
- 自 Windows Server 版本 1709 起，在用户上下文环境下运行应用程序时，需要管理员权限的命令将会失败。 你可以通过--用户标志（如 docker 运行-用户 ContainerAdministrator）指定容器管理员帐户，以后我们打算从 NanoServer 中完全删除管理员帐户。

以上是最显著的差异，其他差异并未一一列出。 还有一些缺失的其他组件并未在此列出。 请记住，你始终可以在自己认为合适的条件下在 Nano Server 上添加层。 有关此情况的示例，请查看 [.NET Core Nano Server Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile)。
