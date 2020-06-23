---
title: Windows 10 上的 Linux 容器
description: 了解使用 Hyper-V 在 Windows 10 上像运行原生容器那样运行 Linux 容器的各种方法。
keywords: linux 容器, docker, 容器, windows 10
author: scooley
ms.date: 09/17/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: edfd11c8-ee99-42d8-9878-efc126fe1826
ms.openlocfilehash: 6b737129692ec8e56bebf290ad8064f010f78ea3
ms.sourcegitcommit: 6a5c237bff2c953fec2ce1e09424375a7c615010
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/09/2020
ms.locfileid: "84632967"
---
# <a name="linux-containers-on-windows-10"></a>Windows 10 上的 Linux 容器

Linux 容器在整个容器生态系统中占很大一部分，对开发人员体验和生产环境都至关重要。  但是，由于容器与容器主机共享一个内核，因此不能直接在 Windows 上运行 Linux 容器。 因此，虚拟化进入了我们的视野中。

## <a name="linux-containers-in-a-moby-vm"></a>Moby VM 中的 Linux 容器

若要在 Linux VM 中运行 Linux 容器，请按照 [Docker 入门指南](https://docs.docker.com/docker-for-windows/)中的说明进行操作。

首次于 2016 年发布以来（在 Windows 上的 Hyper-V 隔离或 Linux 容器功能发布之前），Docker 就一直能够使用在 Hyper-V 上运行的基于 [LinuxKit](https://github.com/linuxkit/linuxkit) 的虚拟机在 Windows 桌面上运行 Linux 容器。

在此模型中，Docker 客户端在 Windows 桌面上运行，但调用 Linux VM 上的 Docker 守护程序。

![Moby VM 作为容器主机](media/MobyVM.png)

在此模型中，所有 Linux 容器共享一个基于 Linux 的容器主机，并且所有 Linux 容器具有以下特点：

* 彼此共享内核和 Moby VM，但不与 Windows 主机共享。
* 与在 Linux 上运行的 Linux 容器具有一致的存储和网络属性（因为它们在 Linux VM 上运行）。

这也意味着 Linux 容器主机 (Moby VM) 需要运行 Docker 守护程序和所有 Docker 守护程序依赖项。

若要查看是否正在使用 Moby VM 运行，请使用 Hyper-V 管理器 UI 来检查用于 Moby VM 的 Hyper-V 管理器，或者通过在提升的 PowerShell 窗口中运行 `Get-VM` 来这样做。
