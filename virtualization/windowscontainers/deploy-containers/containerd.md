---
title: Windows 容器平台
description: 了解有关 Windows 中可用的新容器构建基块的详细信息。
keywords: LCOW、linux 容器、docker、容器、containerd、cri、runhcs、runc
author: scooley
ms.date: 11/19/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: a0e62b32-0c4c-4dd4-9956-8056e9abd9e5
ms.openlocfilehash: 3107eb48dc9c75224b0c9dd9b436af6f0f451871
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998414"
---
# <a name="container-platform-tools-on-windows"></a>Windows 上的容器平台工具

Windows 容器平台正在扩展! Docker 是容器旅程的第一片, 现在我们正在构建其他容器平台工具。

* [containerd/cri](https://github.com/containerd/cri) -Windows Server 2019/windows 10 1809 中的新增项。
* [runhcs](https://github.com/Microsoft/hcsshim/tree/master/cmd/runhcs) -runc 的对应 Windows 容器主机。
* [hcs](https://docs.microsoft.com/virtualization/api/) -主机计算服务 + 便利的填充程序, 使其更易于使用。
  * [hcsshim](https://github.com/microsoft/hcsshim)
  * [新-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization)

本文将讨论 Windows 和 Linux 容器平台以及每个容器平台工具。

## <a name="windows-and-linux-container-platform"></a>Windows 和 Linux 容器平台

在 Linux 环境中, 容器管理工具 (如 Docker) 是基于一组更为精细的容器工具 ( [runc](https://github.com/opencontainers/runc)和[containerd](https://containerd.io/)) 构建的。

![Linux 上的 Docker 体系结构](media/docker-on-linux.png)

`runc` 是基于[OCI 容器运行规范](https://github.com/opencontainers/runtime-spec)创建和运行容器的 Linux 命令行工具。

`containerd` 是一个守护进程, 用于管理容器生命周期, 从下载容器到容器并将容器映像解压缩到容器执行和监督。

在 Windows 上, 我们采取了不同的方法。  开始使用 Docker 支持 Windows 容器时, 我们直接在 HCS (主机计算服务) 上生成。  [此博客文章](https://techcommunity.microsoft.com/t5/Containers/Introducing-the-Host-Compute-Service-HCS/ba-p/382332)详细介绍了为什么构建 HCS 以及我们最初为容器采取此方法的原因。

![Windows 上的初始 Docker 引擎体系结构](media/hcs.png)

此时, Docker 仍直接在 HCS 中调用。 但是, 接下来, 容器管理工具将展开以包括 Windows 容器, Windows 容器主机可以调入 containerd 和 runhcs 在 Linux 上 containerd 和 runc 上调用的方式。

## <a name="runhcs"></a>runhcs

`runhcs` 是的`runc`分叉。  Like `runc`) `runhcs`是基于开放容器倡议 (OCI) 格式来运行应用程序的命令行客户端, 它是开放容器计划规范的符合性实现。

Runc 和 runhcs 之间的功能差异包括:

* `runhcs` 在 Windows 上运行。  它与[HCS](containerd.md#hcs)进行通信以创建和管理容器。
* `runhcs` 可以运行各种不同的容器类型。

  * Windows 和 Linux [hyper-v 隔离](../manage-containers/hyperv-container.md)
  * Windows 进程容器 (容器映像必须与容器主机匹配)

**用法：**

``` cmd
runhcs run [ -b bundle ] <container-id>
```

`<container-id>` 是你正在启动的容器实例的名称。 该名称在你的容器主机上必须是唯一的。

捆绑包目录 (使用`-b bundle`) 是可选的。  
与 runc 一样, 使用绑定配置容器。 容器的捆绑包是容器的 OCI 规范文件 "config json" 的目录。  "捆绑包" 的默认值是当前目录。

OCI 规范文件 "config json" 必须具有两个字段才能正确运行:

* 容器暂存空间的路径
* 容器的图层目录的路径

Runhcs 中可用的容器命令包括:

* 用于创建和运行容器的工具
  * **运行**创建并运行容器
  * **创建**容器

* 用于管理容器中运行的进程的工具:
  * **开始**在创建的容器中执行用户定义的进程
  * **exec**在容器内运行新进程
  * **暂停**暂停暂停容器内的所有进程
  * **resume**恢复之前已暂停的所有进程
  * **ps** ps 显示容器内运行的进程

* 用于管理容器状态的工具
  * **状态**输出容器的状态
  * **kill**将指定的信号 (默认: SIGTERM) 发送到容器的 init 进程
  * **delete**删除由被分离的容器经常使用的容器所占用的任何资源

可以视为多容器的唯一命令为**list**。  它列出由 runhcs 使用给定的根启动的运行或暂停的容器。

### <a name="hcs"></a>HCS

GitHub 上提供了两个包装器, 可与 HCS 进行交互。 由于 HCS 是一个 C API, 因此包装使您可以轻松地从更高级别的语言调用 HCS。  

* [hcsshim](https://github.com/microsoft/hcsshim) -hcsshim 是在 Go 中编写的, 它是 runhcs 的基础。
从 AppVeyor 获取最新版本或自行构建。
* [新-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization) -新-COMPUTEVIRTUALIZATION 是 HCS 的 c # 包装器。

如果要使用 HCS (直接或通过包装), 或者希望在 HCS 周围创建 Rust/Haskell/InsertYourLanguage 包装器, 请留下评论。

若要深入了解 HCS, 请观看[约翰 Stark 的 DockerCon 演示文稿](https://www.youtube.com/watch?v=85nCF5S8Qok)。

## <a name="containerdcri"></a>containerd/cri

> [!IMPORTANT]
> CRI 支持仅在服务器 2019/Windows 10 1809 及更高版本中可用。  我们还在积极地为 Windows 开发 containerd。
> 仅适用于开发人员/测试。

虽然 OCI 规范定义单个容器, 但[CRI](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto) (容器运行时接口) 在名为 pod 的共享沙盒环境中将容器描述为工作负荷。  箱可以包含一个或多个容器工作负荷。  箱让容器 orchestrators (如 Kubernetes 和服务结构网格) 处理与某些共享资源 (如内存和 vNETs) 位于同一主机上的分组工作负荷。

containerd/cri 支持盒式以下兼容性矩阵:

| 主机操作系统 | 容器操作系统 | 能力 | Pod 支持？ |
|:-------------------------------------------------------------------------|:-----------------------------------------------------------------------------|:---------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------|
| <ul><li>Windows Server 2019/1809</ul></li><ul><li>Windows 10 1809</ul></li> | Linux | `hyperv` | 是-支持真正的多容器箱。 |
|  | Windows Server 2019/1809 | `process`* 或 `hyperv` | Yes-如果每个工作负荷容器操作系统都与实用工具 VM 操作系统匹配, 则支持 true 多容器箱。 |
|  | Windows Server 2016,</br>Windows Server 1709,</br>Windows Server 1803 | `hyperv` | 部分-如果容器操作系统与实用工具 VM 操作系统匹配, 则支持每个实用工具 VM 支持单个进程隔离容器的 pod 沙盒。 |

\ * Windows 10 主机仅支持 Hyper-v 隔离

指向 CRI 规范的链接:

* [RunPodSandbox](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L24) -Pod 规格
* [CreateContainer](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L47) -工作负荷规范

![基于 Containerd 的容器环境](media/containerd-platform.png)

尽管 runHCS 和 containerd 都可以在任何 Windows 系统服务器2016或更高版本上进行管理, 但支持盒式容器 (容器组) 对 Windows 中的容器工具所做的更改需要中断。  CRI 支持可在 Windows Server 2019/Windows 10 1809 和更高版本上使用。
