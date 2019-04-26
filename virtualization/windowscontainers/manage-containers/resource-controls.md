---
title: 实施资源控制
description: 有关 Windows 容器资源控制的详细信息
keywords: Docker, 容器, CPU, 内存, 磁盘, 资源
author: taylorb-microsoft
ms.date: 11/21/2017
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8ccd4192-4a58-42a5-8f74-2574d10de98e
ms.openlocfilehash: e004bd4ca97960499826eeefdfb8e58b08e494b2
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/26/2019
ms.locfileid: "9574768"
---
# <a name="implementing-resource-controls-for-windows-containers"></a>实施 Windows 容器资源控制
某些资源控制可以按容器和按资源实施。  默认情况下，容器运行受典型 Windows 资源管理（总体上以公平分配为基础）影响，但通过实施以上控制，开发人员或管理员可以限制或影响资源使用情况。  可以控制的资源包括：CPU/处理器、内存/RAM、磁盘/存储和网络/吞吐量。

Windows 容器利用[作业对象](https://msdn.microsoft.com/en-us/library/windows/desktop/ms684161(v=vs.85).aspx)进行分组并跟踪与每个容器关联的进程。  资源控制在与容器关联的父作业对象上实施。 

在 [Hyper-V 隔离](https://docs.microsoft.com/en-us/virtualization/windowscontainers/about/index#windows-container-types)的情况下，系统同时对虚拟机和在虚拟机中运行的容器的作业对象进行自动资源控制，这样即使在容器中运行的进程绕过了或逃脱了作业对象控制，虚拟机也会确保其无法越过定义的资源控制。

## <a name="resources"></a>资源
此部分提供了每个资源与 Docker 命令行界面之间的映射，作为如何将资源控制用于（可能由 Orchestrator 或其他工具配置）相应的 Windows 主机计算服务 (HCS) API 以及 Windows 通常如何实施资源控制（请注意此描述为高级描述，基础实施可能发生变化）的示例。

|  | |
| ----- | ------|
| *内存* ||
| Docker 界面 | [--内存](https://docs.docker.com/engine/admin/resource_constraints/#memory) |
| HCS 界面 | [MemoryMaximumInMB]( https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| 共享的内核 | [JOB_OBJECT_LIMIT_JOB_MEMORY](https://msdn.microsoft.com/en-us/library/windows/desktop/ms684147(v=vs.85).aspx) |
| Hyper-V 隔离 | 虚拟机内存 |
| _有关 Windows Server 2016 中 Hyper-V 的提示：在使用内存容量时你会发现，容器一开始会分配内存量上限，随后又开始将其返回至容器主机。  在更高版本（1709 或更高版）中这种情形已经得到优化。_ |
| ||
| *CPU（计数）* ||
| Docker 界面 | [--CPU](https://docs.docker.com/engine/admin/resource_constraints/#cpu) |
| HCS 界面 | [ProcessorCount]( https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| 共享的内核 | 使用 [JOB_OBJECT_CPU_RATE_CONTROL_HARD_CAP](https://msdn.microsoft.com/en-us/library/windows/desktop/hh448384(v=vs.85).aspx)* 模拟 |
| Hyper-V 隔离 | 公开的虚拟处理器数量 |
| ||
| *CPU（百分比）* ||
| Docker 界面 | [--CPU 百分比](https://docs.docker.com/engine/admin/resource_constraints/#cpu) |
| HCS 界面 | [ProcessorMaximum](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| 共享的内核 | [JOB_OBJECT_CPU_RATE_CONTROL_HARD_CAP](https://msdn.microsoft.com/en-us/library/windows/desktop/hh448384(v=vs.85).aspx) |
| Hyper-V 隔离 | 虚拟机监控程序对虚拟处理器的限制 |
| ||
| *CPU（共享）* ||
| Docker 界面 | [--CPU 共享](https://docs.docker.com/engine/admin/resource_constraints/#cpu) |
| HCS 界面 | [ProcessorWeight](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| 共享的内核 | [JOB_OBJECT_CPU_RATE_CONTROL_WEIGHT_BASED](https://msdn.microsoft.com/en-us/library/windows/desktop/hh448384(v=vs.85).aspx) |
| Hyper-V 隔离 | 虚拟机监控程序虚拟处理器权重 |
| ||
| *存储（图像）* ||
| Docker 界面 | [--io-maxbandwidth/--io-maxiops]( https://docs.docker.com/edge/engine/reference/commandline/run/#usage) |
| HCS 界面 | [StorageIOPSMaximum 和 StorageBandwidthMaximum](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| 共享的内核 | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](https://msdn.microsoft.com/en-us/library/windows/desktop/mt280122(v=vs.85).aspx) |
| Hyper-V 隔离 | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](https://msdn.microsoft.com/en-us/library/windows/desktop/mt280122(v=vs.85).aspx) |
| ||
| *存储（卷）* ||
| Docker 界面 | [--storage-opt size=]( https://docs.docker.com/edge/engine/reference/commandline/run/#set-storage-driver-options-per-container) |
| HCS 界面 | [StorageSandboxSize](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| 共享的内核 | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](https://msdn.microsoft.com/en-us/library/windows/desktop/mt280122(v=vs.85).aspx) |
| Hyper-V 隔离 | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](https://msdn.microsoft.com/en-us/library/windows/desktop/mt280122(v=vs.85).aspx) |

## <a name="additional-notes-or-details"></a>其他说明或详细信息

### <a name="memory"></a>内存

Windows 容器在每个容器中运行某个系统进程，通常这些容器都提供按容器的功能，例如用户管理、网络等； 而这些进程所需的大部分内存均在容器之间共享，因此内存容量必须足够大，才能完成这些进程。  [系统要求](https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/system-requirements#memory-requirments)文档中提供了表格，列出了每种具备或不具备 Hyper-V 隔离的基本映像类型。

### <a name="cpu-shares-without-hyper-v-isolation"></a>CPU 共享（具备 Hyper-V 隔离）

使用 CPU 共享基础实施（不使用 Hyper-V 隔离时）配置 [JOBOBJECT_CPU_RATE_CONTROL_INFORMATION](https://msdn.microsoft.com/en-us/library/windows/desktop/hh448384(v=vs.85).aspx) 时，请专门将控制标记设置为 JOB_OBJECT_CPU_RATE_CONTROL_WEIGHT_BASED 并提供合适的权重。  作业对象的有效的权重范围是 1 - 9，默认权重是 5，其精确度低于 1 -10000 的主机计算服务值。  例如，7500 的共享权重对应的权重为 7，或 2500 的共享权重对应的值为 2。
