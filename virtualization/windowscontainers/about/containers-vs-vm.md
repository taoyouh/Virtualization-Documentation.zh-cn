---
title: 容器与虚拟机
description: 本主题介绍容器和虚拟机之间的一些主要相似性和差异，以及何时可能需要使用其中的每一项。 容器和虚拟机都有自己的用途 - 事实上，许多容器部署使用虚拟机作为主机操作系统，而不是直接运行在设备上，在云中运行容器时尤其如此。
keywords: docker, 容器, vm, 虚拟机
author: jasongerend
ms.author: jgerend
ms.date: 10/21/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 63150dfde007ec942446387064ad59f05b0aaa43
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910817"
---
# <a name="containers-vs-virtual-machines"></a>容器与虚拟机

本主题介绍容器和虚拟机 (VM) 之间的一些主要相似性和差异，以及何时可能需要使用其中的每一项。 容器和 VM 都有自己的用途 - 事实上，许多容器部署使用 VM 作为主机操作系统，而不是直接运行在硬件上，在云中运行容器时尤其如此。

有关容器的概述，请参阅 [Windows 和容器](index.md)。

## <a name="container-architecture"></a>容器体系结构

容器是一个隔离的轻型接收器，用于在主机操作系统上运行应用程序。 容器在主机操作系统的内核（可以将其视为操作系统的隐藏管道）上构建，只包含应用和一些轻型操作系统 API 以及在用户模式下运行的服务，如下图所示。

![体系结构图，显示容器如何在内核上运行](media/container-diagram.svg)

## <a name="virtual-machine-architecture"></a>虚拟机体系结构

与容器不同，VM 运行的是完整的操作系统（包括其自己的内核），如下图所示。

![体系结构图，显示 VM 如何运行完整的操作系统（独立于主机操作系统）](media/virtual-machine-diagram.svg)

## <a name="containers-vs-virtual-machines"></a>容器与虚拟机

下表显示了这些互补技术的一些异同。

|                 | 虚拟机  | 容器  |
| --------------  | ---------------- | ---------- |
| 隔离       | 提供与主机操作系统和其他 VM 的完全隔离。 当强安全边界很关键时（例如，在同一台服务器或群集上托管来自竞争性公司的应用时），这很有用。 | 通常提供与主机和其他容器的轻度隔离，但不提供与 VM 一样强的安全边界。 （可以使用 [Hyper-V 隔离模式](../manage-containers/hyperv-container.md)隔离轻型 VM 中的每个容器，从而提高安全性。） |
| 操作系统 | 运行包含内核的完整操作系统，因此需要更多的系统资源（CPU、内存和存储）。 | 运行操作系统的用户模式部分，可以对其进行定制，使之只包含应用所需的服务，减少所使用的系统资源。 |
| 来宾兼容性 | 运行虚拟机内的几乎任何操作系统 | 在[与主机相同的操作系统版本](../deploy-containers/version-compatibility.md)上运行（Hyper-V 隔离使你能够在轻型 VM 环境中运行同一 OS 的早期版本）
| 部署     | 使用 Windows Admin Center 或 Hyper-V 管理器部署单个 VM；使用 PowerShell 或 System Center Virtual Machine Manager 部署多个 VM。 | 通过命令行使用 Docker 部署单个容器；使用 Azure Kubernetes 服务等业务流程协调程序部署多个容器。 |
| 操作系统更新和升级 | 在每个 VM 上下载并安装操作系统更新。 安装新的操作系统版本需要升级；通常情况下，直接创建全新 VM。 这样可能很耗时，尤其是在有大量 VM 的情况下... | 在容器中更新或升级操作系统文件的操作是相同的： <br><ol><li>编辑容器映像的生成文件（称为 Dockerfile），使之指向最新版 Windows 基础映像。 </li><li>用这个新的基础映像重新生成容器映像。</li><li>将容器映像推送到容器注册表。</li> <li>使用业务流程协调程序重新进行部署。<br>业务流程协调程序提供的强大的自动化功能允许大规模这样做。 有关详细信息，请参阅[教程：在 Azure Kubernetes 服务中更新应用程序](https://docs.microsoft.com/azure/aks/tutorial-kubernetes-app-update)。</li></ol> |
| 持久性存储 | 对单个 VM 使用进行本地存储的虚拟硬盘 (VHD)，或对多个服务器共享的存储使用 SMB 文件共享 | 使用 Azure 磁盘作为单个节点的本地存储，或将 Azure 文件存储（SMB 共享）用于由多个节点或服务器共享的存储。 |
| 负载平衡 | 虚拟机负载均衡将运行中的 VM 移动到故障转移群集中的其他服务器。 | 容器本身不移动，而是由业务流程协调程序在群集节点上自动启动或停止容器，以管理负载和可用性方面的更改。 |
| 容错 | VM 可以故障转移到群集中的另一台服务器，并在新服务器上重启 VM 的操作系统。  | 如果某个群集节点发生故障，则在该节点上运行的所有容器都将在另一个群集节点上由业务流程协调程序快速重新创建。 |
| 网络     | 使用虚拟网络适配器。 | 使用虚拟网络适配器的隔离视图，在减少使用资源的同时，稍微减少提供的虚拟化 – 主机的防火墙与容器共享。 有关详细信息，请参阅 [Windows 容器网络](../container-networking/architecture.md)。 |