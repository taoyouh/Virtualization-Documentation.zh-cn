---
title: Windows 容器中的 GPU 加速
description: Windows 容器中存在的 GPU 加速级别
keywords: docker，容器，设备硬件
author: cwilhit
ms.openlocfilehash: 281241e07e4bc184e73c4e74a117b44253a775be
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/26/2019
ms.locfileid: "9578650"
---
# <a name="gpu-acceleration-in-windows-containers"></a>Windows 容器中的 GPU 加速

对于许多容器化工作负载，CPU 计算资源提供足够的性能。 但是，对于特定类别的工作负荷，Gpu （图形处理单元） 提供的整体的并行计算能力可以加快操作的数量级，将关机成本和非常提高吞吐量。

Gpu 已很多受欢迎的工作负载，从传统呈现和模拟的机器学习培训和推断的常见工具。 Windows 容器支持用于 DirectX 和基于它的所有框架 GPU 加速。

> [!IMPORTANT]
> 此功能需要版本支持的 Docker 的`--device`Windows 容器的命令行选项。 已计划正式 Docker 支持即将推出的 Docker EE 引擎 19.03 版本。 在此之前，Docker[上游源](https://master.dockerproject.org/)包含必要的位。

## <a name="requirements"></a>要求

使用此功能，你的环境必须满足以下要求：

- 容器主机必须运行 Windows Server 2019 或 Windows 10 版本 1809年或更高版本。
- 容器基本映像必须[mcr.microsoft.com/windows:1809](https://hub.docker.com/_/microsoft-windowsfamily-windows)或更高版本。 目前不支持 Windows Server Core 和 Nano Server 容器映像。
- 容器主机必须 Docker 引擎运行 19.03 或更高版本。
- 容器主机必须具有 GPU 正在运行显示驱动程序版本 WDDM 2.5 或更高版本。

若要检查你显示的驱动程序的 WDDM 版本，容器主机上运行 DirectX 诊断工具 (dxdiag.exe)。 在该工具的"Display"选项卡上，查看"驱动程序"部分中，如下所示。

![Dxdiag](media/dxdiag.png)

## <a name="run-a-container-with-gpu-acceleration"></a>具有带 GPU 加速运行的容器

若要启动容器具有带 GPU 加速，运行以下命令：

```shell
docker run --isolation process --device class/5B45201D-F2F2-4F3B-85BB-30FF1F953599 mcr.microsoft.com/windows:1809
```

> [!IMPORTANT]
> DirectX （和基于它的所有框架） 现在是具有 GPU 可以加速的唯一 Api。 不支持第三方框架。

## <a name="hyper-v-isolated-windows-container-support"></a>超 V 隔离的 Windows 容器支持

超 V 隔离的 Windows 容器中的工作负载的 GPU 加速当前不受支持。

## <a name="hyper-v-isolated-linux-container-support"></a>超 V 隔离的 Linux 容器支持

超 V 隔离的 Linux 容器中的工作负载的 GPU 加速当前不受支持。
