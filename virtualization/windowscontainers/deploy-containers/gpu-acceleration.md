---
title: Windows 容器中的 GPU 加速
description: Windows 容器中存在的 GPU 加速级别
keywords: docker, 容器, 设备, 硬件
author: cwilhit
ms.openlocfilehash: 8f63c74d7839385e21206188263b9e5d08e7eb60
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909907"
---
# <a name="gpu-acceleration-in-windows-containers"></a>Windows 容器中的 GPU 加速

对于许多容器化的工作负荷，CPU 计算资源可提供足够的性能。 但是，对于某一类工作负荷，GPU（图形处理单元）提供的大规模并行计算能力能够将操作速度提高几个数量级，从而大大降低成本并提高吞吐量。

GPU 已经成为许多常用工作负荷（从传统的渲染和模拟到机器学习训练和推理）的通用工具。 Windows 容器支持对 DirectX 和基于它构建的所有框架进行 GPU 加速。

> [!NOTE]
> 此功能在 Docker Desktop 版本 2.1 和 Docker 引擎 - 企业版 19.03 或更高版本中可用。

## <a name="requirements"></a>要求

要使此功能生效，你的环境必须满足以下要求：

- 容器主机必须运行 Windows Server 2019 或 Windows 10 版本 1809 或更高版本。
- 容器基础映像必须是 [mcr.microsoft.com/windows:1809](https://hub.docker.com/_/microsoft-windows) 或更高版本。 当前不支持 Windows Server Core 和 Nano Server 容器映像。
- 容器主机必须运行 Docker 引擎 19.03 或更高版本。
- 容器主机上必须有一个运行显示驱动程序版本 WDDM 2.5 或更高版本的 GPU。

若要检查显示驱动程序的 WDDM 版本，请在容器主机上运行 DirectX 诊断工具 (dxdiag.exe)。 在工具的“显示”选项卡中查看“驱动程序”部分，如下所示。

![Dxdiag](media/dxdiag.png)

## <a name="run-a-container-with-gpu-acceleration"></a>运行支持 GPU 加速的容器

若要启动支持 GPU 加速的容器，请运行以下命令：

```shell
docker run --isolation process --device class/5B45201D-F2F2-4F3B-85BB-30FF1F953599 mcr.microsoft.com/windows:1809
```

> [!IMPORTANT]
> DirectX（和基于它构建的所有框架）是目前仅有的可以通过 GPU 进行加速的 API。 不支持第三方框架。

## <a name="hyper-v-isolated-windows-container-support"></a>Hyper-V 隔离 Windows 容器支持

目前不支持对 Hyper-V 隔离 Windows 容器中的工作负荷进行 GPU 加速。

## <a name="hyper-v-isolated-linux-container-support"></a>Hyper-V 隔离 Linux 容器支持

目前不支持对 Hyper-V 隔离 Linux 容器中的工作负荷进行 GPU 加速。

## <a name="more-information"></a>详细信息

有关利用 GPU 加速的容器化 DirectX 应用的完整示例，请参阅 [DirectX 容器示例](https://github.com/MicrosoftDocs/Virtualization-Documentation/tree/master/windows-container-samples/directx)。
