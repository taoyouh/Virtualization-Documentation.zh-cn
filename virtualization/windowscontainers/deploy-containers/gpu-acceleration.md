---
title: Windows 容器中的 GPU 加速
description: Windows 容器中存在何种级别的 GPU 加速
keywords: docker、容器、设备、硬件
author: cwilhit
ms.openlocfilehash: 6e5010efee10f9b488cbeb57b14bc86f30c1e766
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/31/2019
ms.locfileid: "9883270"
---
# <a name="gpu-acceleration-in-windows-containers"></a>Windows 容器中的 GPU 加速

对于许多容器的工作负荷, CPU 计算资源可提供充足的性能。 但是, 对于特定的工作负荷类别, Gpu (图形处理单元) 所提供的整体并行计算能力可以按数量级的顺序加速操作, 从而降低成本并提高吞吐量。

Gpu 已经是许多常用工作负载的常见工具, 从传统的呈现和模拟到机器学习培训和推断。 Windows 容器支持 DirectX 的 GPU 加速以及在其上构建的所有框架。

## <a name="requirements"></a>要求

若要使用此功能, 你的环境必须满足以下要求:

- 容器主机必须运行 Windows Server 2019 或 Windows 10 版本1809或更高版本。
- 容器基本图像必须是[mcr.microsoft.com/windows:1809](https://hub.docker.com/_/microsoft-windowsfamily-windows)或更高版本。 当前不支持 Windows Server Core 和 Nano Server 容器图像。
- 容器主机必须运行 Docker 引擎19.03 或更高版本。
- 容器主机必须有一个 GPU 运行的显示驱动程序版本 WDDM 2.5 或更高版本。

若要检查你的显示驱动程序的 WDDM 版本, 请在你的容器主机上运行 DirectX 诊断工具 (dxdiag)。 在工具的 "显示" 选项卡中, 按如下所示查找 "驱动程序" 部分。

![Dxdiag](media/dxdiag.png)

## <a name="run-a-container-with-gpu-acceleration"></a>使用 GPU 加速运行容器

若要启动具有 GPU 加速的容器, 请运行以下命令:

```shell
docker run --isolation process --device class/5B45201D-F2F2-4F3B-85BB-30FF1F953599 mcr.microsoft.com/windows:1809
```

> [!IMPORTANT]
> DirectX (以及在其上构建的所有框架) 是目前可以使用 GPU 快速加速的 Api。 不支持第三方框架。

## <a name="hyper-v-isolated-windows-container-support"></a>Hyper-v-隔离的 Windows 容器支持

今天不支持 Hyper-v 隔离的 Windows 容器中的工作负荷的 GPU 加速。

## <a name="hyper-v-isolated-linux-container-support"></a>Hyper-v-隔离的 Linux 容器支持

目前不支持 Hyper-v 中的工作负荷的 GPU 加速-隔离的 Linux 容器。