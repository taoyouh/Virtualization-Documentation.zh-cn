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
ms.sourcegitcommit: 16744984ede5ec94cd265b6bff20aee2f782ca88
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/18/2020
ms.locfileid: "77439274"
---
# <a name="container-platform-tools-on-windows"></a>Windows 上的容器平台工具

Windows 容器平台正在扩展！ Docker 是容器旅程的第一个部分，现在我们正在构建其他容器平台工具。

* [containerd/cri](https://github.com/containerd/cri) -在 Windows Server 2019/windows 10 1809 中新增。
* [runhcs](https://github.com/Microsoft/hcsshim/tree/master/cmd/runhcs) -与 runc 对应的 Windows 容器主机。
* [hcs](https://docs.microsoft.com/virtualization/api/) -主机计算服务和方便的填充程序，使其更易于使用。
  * [hcsshim](https://github.com/microsoft/hcsshim)
  * [dotnet-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization)

本文将讨论 Windows 和 Linux 容器平台以及每个容器平台工具。

## <a name="windows-and-linux-container-platform"></a>Windows 和 Linux 容器平台

在 Linux 环境中，容器管理工具（例如 Docker）是基于一组更精细的容器工具构建的： [runc](https://github.com/opencontainers/runc)和[containerd](https://containerd.io/)。

![Linux 上的 Docker 体系结构](media/docker-on-linux.png)

`runc` 是一种 Linux 命令行工具，用于根据[OCI 容器运行时规范](https://github.com/opencontainers/runtime-spec)创建和运行容器。

`containerd` 是一个守护程序，它管理容器生命周期，从下载容器到容器映像并将其解压缩到容器执行和监督。

在 Windows 上，我们采取了不同的方法。  开始使用 Docker 来支持 Windows 容器时，我们直接在 HCS （宿主计算服务）上生成。  [此博客文章](https://techcommunity.microsoft.com/t5/Containers/Introducing-the-Host-Compute-Service-HCS/ba-p/382332)详细介绍了为什么要构建 HCS，以及为什么我们最初采用这种方法。

![Windows 上的初始 Docker 引擎体系结构](media/hcs.png)

此时，Docker 仍直接调用 HCS。 不过，今后，容器管理工具将扩展为包含 Windows 容器，Windows 容器主机可以调入 containerd，并 runhcs 其在 Linux 上 containerd 和 runc 上的调用方式。

## <a name="runhcs"></a>runhcs

`runhcs` 是 `runc`的分叉。  与 `runc`一样，`runhcs` 是用于根据开放容器计划（OCI）格式来运行应用程序的命令行客户端，它是开放容器计划规范的兼容实现。

Runc 和 runhcs 之间的功能差异包括：

* `runhcs` 在 Windows 上运行。  它与[HCS](containerd.md#hcs)通信，以创建和管理容器。
* `runhcs` 可以运行各种不同的容器类型。

  * Windows 和 Linux [hyper-v 隔离](../manage-containers/hyperv-container.md)
  * Windows 进程容器（容器映像必须与容器主机匹配）

**使用情况**

``` cmd
runhcs run [ -b bundle ] <container-id>
```

`<container-id>` 是要启动的容器实例的名称。 名称在容器主机上必须唯一。

包目录（使用 `-b bundle`）是可选的。  
与 runc 一样，使用束配置容器。 容器的绑定是容器的 OCI 规范文件 "config" 的目录。  "捆绑" 的默认值为当前目录。

OCI 规范文件 "config.xml" 必须有两个字段才能正确运行：

* 容器暂存空间的路径
* 容器层目录的路径

Runhcs 中可用的容器命令包括：

* 用于创建和运行容器的工具
  * **运行**创建并运行容器
  * **创建**容器

* 用于管理在容器中运行的进程的工具：
  * **开始**在创建的容器中执行用户定义的进程
  * **exec**在容器内运行新进程
  * **暂停**暂停挂起容器中的所有进程
  * **resume**恢复先前暂停的所有进程
  * **ps** ps 显示在容器中运行的进程

* 用于管理容器状态的工具
  * **状态**输出容器的状态
  * **kill**将指定的信号（默认值： SIGTERM）发送到容器的初始化进程
  * **delete**删除容器所持有的所有资源，这些资源通常用于分离的容器

可视为多容器的唯一命令是**list**。  它列出了使用给定的根通过 runhcs 启动的运行或已暂停的容器。

### <a name="hcs"></a>HCS

GitHub 上提供了两个包装器，可与 HCS 交互。 由于 HCS 是一个 C API，因此可以使用包装来轻松地从更高级别的语言调用 HCS。  

* [hcsshim](https://github.com/microsoft/hcsshim) -hcsshim 是在中编写的，这是 runhcs 的基础。
从 AppVeyor 获取最新版本或自行生成。
* [dotnet-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization) -dotnet-COMPUTEVIRTUALIZATION 是 HCS C#的包装器。

如果要使用 HCS （直接或通过包装），或想要围绕 HCS 创建 Rust/Haskell/InsertYourLanguage 包装，请留下评论。

若要深入了解 HCS，请观看[John Stark 的 DockerCon 演示](https://www.youtube.com/watch?v=85nCF5S8Qok)。

## <a name="containerdcri"></a>containerd/cri

> [!IMPORTANT]
> CRI 支持仅在服务器 2019/Windows 10 1809 及更高版本中可用。  我们还会积极地开发 containerd for Windows。
> 仅限开发/测试。

尽管 OCI 规范定义了一个容器，但[CRI](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto) （容器运行时接口）在名为 pod 的共享沙箱环境中将容器描述为工作负荷。  Pod 可以包含一个或多个容器工作负荷。  Pod 使容器协调器（如 Kubernetes 和 Service Fabric 网格）处理应与某些共享资源（如内存和 Vnet）位于同一主机上的已分组工作负荷。

containerd/cri 为 pod 启用以下兼容性矩阵：

| 主机操作系统 | 容器操作系统 | 隔离 | Pod 支持？ |
|:-------------------------------------------------------------------------|:-----------------------------------------------------------------------------|:---------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------|
| <ul><li>Windows Server 2019/1809</ul></li><ul><li>Windows 10 1809</ul></li> | Linux | `hyperv` | 是—支持真正的多容器箱。 |
|  | Windows Server 2019/1809 | `process`* 或 `hyperv` | 是—如果每个工作负荷容器操作系统与实用工具 VM 操作系统匹配，则支持 true 多容器。 |
|  | Windows Server 2016、</br>Windows Server 1709、</br>Windows Server 1803 | `hyperv` | 部分-支持 pod 沙盒，如果容器操作系统与实用工具 VM 操作系统匹配，则可以支持每个实用工具 VM 的单个进程隔离容器。 |

\*Windows 10 主机仅支持 Hyper-v 隔离

CRI 规范的链接：

* [RunPodSandbox](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L24)规范
* [CreateContainer](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L47) -工作负荷规范

![基于 Containerd 的容器环境](media/containerd-platform.png)

尽管 runHCS 和 containerd 都可以在任何 Windows 系统服务器2016或更高版本上进行管理，但支持 pod （容器组）需要对 Windows 中的容器工具进行重大更改。  CRI 支持在 Windows Server 2019/Windows 10 1809 及更高版本上可用。
