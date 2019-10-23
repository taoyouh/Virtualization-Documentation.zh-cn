---
title: 容器与虚拟机
description: 本主题讨论容器和虚拟机之间的一些主要相似性和差异，以及你可能希望使用它们的情况。 容器和虚拟机每个容器和虚拟机都具有各自的用途-事实上，许多容器部署将虚拟机用作主机操作系统，而不是直接在硬件上运行，尤其是在云中运行容器时。
keywords: docker、容器、vm、虚拟机
author: jasongerend
ms.author: jgerend
ms.date: 10/21/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 63150dfde007ec942446387064ad59f05b0aaa43
ms.sourcegitcommit: d0411b05d1ef7328a770766b84fd0743f9d9c237
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2019
ms.locfileid: "10254384"
---
# <a name="containers-vs-virtual-machines"></a>容器与虚拟机

本主题讨论容器和虚拟机（Vm）之间的一些主要相似性和差异，以及你可能希望使用它们的情况。 每个容器和 Vm 都具有各自的用途-事实上，许多容器部署将 Vm 用作主机操作系统，而不是直接在硬件上运行，尤其是在云中运行容器时。

有关容器的概述，请参阅[窗口和容器](index.md)。

## <a name="container-architecture"></a>容器体系结构

容器是一个独立的轻型接收器，用于在主机操作系统上运行应用程序。 容器在主机操作系统的内核（可视为操作系统的掩蔽管道）的顶部生成，并且仅包含应用和某些轻量级操作系统 Api 和在用户模式下运行的服务，如此图所示。

![显示如何在内核顶部运行容器的体系结构图表](media/container-diagram.svg)

## <a name="virtual-machine-architecture"></a>虚拟机体系结构

相对于容器，Vm 运行完整的操作系统（包括其自己的内核），如下图所示。

![展示 Vm 在主机操作系统旁边运行完整操作系统的体系结构图](media/virtual-machine-diagram.svg)

## <a name="containers-vs-virtual-machines"></a>容器与虚拟机

下表显示了这些补充技术的一些相似性和差异。

|                 | 虚拟机  | 树枝  |
| --------------  | ---------------- | ---------- |
| 能力       | 提供来自主机操作系统和其他 Vm 的完全隔离。 这在强安全边界非常重要的情况下很有用，例如从同一服务器或群集上的竞争公司托管应用。 | 通常提供来自主机和其他容器的轻型隔离，但不提供作为 VM 的强安全边界。 （你可以通过使用[hyper-v 隔离模式](../manage-containers/hyperv-container.md)隔离轻型 VM 中的每个容器来提高安全性）。 |
| 操作系统 | 运行包括内核在内的完整操作系统，因此需要更多系统资源（CPU、内存和存储）。 | 运行操作系统的用户模式部分，并且可以通过使用较少的系统资源定制为仅包含应用所需的服务。 |
| 来宾兼容性 | 仅在虚拟机内部运行任何操作系统 | 在与[主机相同的操作系统版本](../deploy-containers/version-compatibility.md)上运行（hyper-v 隔离使你能够在轻型 VM 环境中运行早期版本的同一 OS）
| 部署     | 使用 Windows 管理中心或 Hyper-v 管理器部署单个 Vm;使用 PowerShell 或 System Center Virtual Machine Manager 部署多个虚拟机。 | 通过命令行使用 Docker 来部署单个容器;使用 orchestrator （如 Azure Kubernetes 服务）部署多个容器。 |
| 操作系统更新和升级 | 在每个 VM 上下载并安装操作系统更新。 安装新的操作系统版本需要升级，或者经常创建全新的 VM。 这可能非常耗时，尤其是当你有大量的 Vm 时 .。。 | 更新或升级容器内的操作系统文件是相同的： <br><ol><li>编辑容器图像的生成文件（称为 "Dockerfile"）以指向最新版本的 Windows 基础映像。 </li><li>通过此新的基本映像重建容器映像。</li><li>将容器映像推送到你的容器注册表。</li> <li>使用 orchestrator 进行重新部署。<br>Orchestrator 提供强大的自动化功能来以比例执行此操作。 有关详细信息，请参阅[教程：在 Azure Kubernetes 服务中更新应用程序](https://docs.microsoft.com/azure/aks/tutorial-kubernetes-app-update)。</li></ol> |
| 永久存储 | 为单个 VM 使用本地存储的虚拟硬盘（VHD），或为多个服务器共享的存储使用 SMB 文件共享 | 将 Azure 磁盘用于本地存储单个节点或 Azure 文件（SMB 共享），用于存储由多个节点或服务器共享的存储。 |
| 负载平衡 | 虚拟机负载平衡将运行中的 Vm 移动到故障转移群集中的其他服务器。 | 容器本身不移动;因此，orchestrator 可以自动启动或停止群集节点上的容器，以管理加载和可用性中的更改。 |
| 容错 | Vm 可以故障转移到群集中的另一台服务器，并在新服务器上重新启动 VM 的操作系统。  | 如果群集节点出现故障，则运行在它上面的任何容器都将由另一个群集节点上的 orchestrator 快速重新创建。 |
| 网络     | 使用虚拟网络适配器。 | 使用虚拟网络适配器的隔离视图，提供稍低的虚拟化-主机的防火墙与容器共享-同时使用较少的资源。 有关详细信息，请参阅[Windows 容器网络](../container-networking/architecture.md)。 |