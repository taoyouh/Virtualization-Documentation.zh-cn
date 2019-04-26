---
title: Windows 容器平台
description: 了解有关新容器构建基块可在 Windows 中的详细信息。
keywords: LCOW，linux 容器，docker，容器，containerd、 cri、 runhcs，runc
author: scooley
ms.date: 11/19/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: a0e62b32-0c4c-4dd4-9956-8056e9abd9e5
ms.openlocfilehash: 9f38775d56a95d96bef42b3a33c2571cc5fb2ca0
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/26/2019
ms.locfileid: "9578378"
---
# <a name="container-platform-tools-on-windows"></a>在 Windows 上的容器平台工具

Windows 容器平台正在展开 ！ Docker 的第一个片段的容器旅程，现在，我们正在创建其他容器平台工具。

* [containerd/cri](https://github.com/containerd/cri)的新在 Windows Server 2019/Windows 10 1809年。
* [runhcs](https://github.com/Microsoft/hcsshim/tree/master/cmd/runhcs) -runc 的 Windows 容器主机对应。
* [hcs](https://docs.microsoft.com/virtualization/api/)的主机计算服务 + 方便填充，以使其更易于使用。
  * [hcsshim](https://github.com/microsoft/hcsshim)
  * [dotnet computevirtualization](https://github.com/microsoft/dotnet-computevirtualization)

本文将讨论 Windows 和 Linux 容器平台，以及每个容器平台工具。

## <a name="windows-and-linux-container-platform"></a>Windows 和 Linux 容器平台

在 Linux 环境中，容器管理工具，例如 Docker 基于一组更细化的容器工具： [runc](https://github.com/opencontainers/runc)和[containerd](https://containerd.io/)。

![在 Linux 上的 docker 体系结构](media/docker-on-linux.png)

`runc` 是用于创建并运行根据[OCI 容器运行时规范](https://github.com/opencontainers/runtime-spec)的容器的 Linux 命令行工具。

`containerd` 是从下载并解压缩该容器映像分发到容器执行和监督管理容器生命周期守护程序。

在 Windows 上，我们所花的不同方法。  当我们开始使用 Docker 来支持 Windows 容器的工作时，我们基于直接 HCS （主机计算服务）。  [这篇博客文章](https://blogs.technet.microsoft.com/virtualization/2017/01/27/introducing-the-host-compute-service-hcs/)已满有关我们为何生成 HCS 和为什么我们采用这种方法对容器最初的信息。

![初始 Windows 上的 Docker 引擎体系结构](media/hcs.png)

到目前为止，Docker 仍直接在 HCS 调用。 接下来，但是，容器管理工具扩展，以包括 Windows 容器和 Windows 容器主机可以调用到 containerd 和 runhcs 它们 containerd 和 Linux 上的 runc 上调用的方法。

## <a name="runhcs"></a>runhcs

`runhcs` 是的分叉`runc`。  如`runc`，`runhcs`命令行客户端用于运行应用程序打包根据开放容器计划 (OCI) 格式，而是开放容器计划规范的兼容的实现。

功能 runc 和 runhcs 之间的差异包括：

* `runhcs` 在 Windows 上运行。  通信[HCS](containerd.md#hcs)创建和管理容器。
* `runhcs` 可以运行各种不同的容器类型。

  * Windows 和 Linux [HYPER-V 隔离](../manage-containers/hyperv-container.md)
  * Windows 处理容器 （容器映像必须匹配在容器主机）

**用法：**

``` cmd
runhcs run [ -b bundle ] <container-id>
```

`<container-id>` 容器实例会启动你名称。 名称必须是唯一容器主机上。

捆绑包目录 (使用`-b bundle`) 是可选的。  
与 runc，容器使用捆绑包进行配置。 容器的捆绑包为容器的 OCI 规范文件的目录"config.json"。  "Bundle"的默认值是当前目录。

OCI 规范文件、"config.json"，必须具有两个字段正常运行：

* 容器的暂存空间路径
* 容器的层目录的路径

Runhcs 中可用的容器命令包括：

* 用于创建和运行的容器工具
  * **运行**创建并运行容器
  * **创建**创建一个容器

* 若要管理在容器中运行的进程的工具：
  * **开始菜单**中创建的容器执行用户定义过程
  * **exec**运行容器内的一个新进程
  * **暂停**暂停暂停容器内的所有进程
  * **恢复**恢复已暂停的所有进程
  * **ps** ps 显示在容器内运行的进程

* 工具来管理容器的状态
  * **状态**输出容器的状态
  * **终止**发送指定的信号 (默认： SIGTERM) 到容器的初始化过程
  * **删除**删除持有经常与分离的容器使用的容器的任何资源

仅可以被视为多容器的命令是**列表**。  它列出正在运行或暂停容器启动 runhcs 与给定的根。

### <a name="hcs"></a>HCS

我们对 HCS 界面 GitHub 上具有两个包装器可用。 由于 HCS 是 C API，包装器轻松从较高级别语言调用 HCS。  

* [hcsshim](https://github.com/microsoft/hcsshim) -HCSShim 编写在定位，并且它是 runhcs 的基础。
抓取 AppVeyor 的最新版本或自行生成。
* [dotnet computevirtualization](https://github.com/microsoft/dotnet-computevirtualization) -dotnet computevirtualization 是 C# 的包装器 HCS。

如果你想要使用 HCS （直接或通过包装器），或者你想要使 HCS 的铁锈/Haskell/InsertYourLanguage 包装器，请留下评论。

如 HCS 更深入地了解，观看[John Stark DockerCon 演示文稿](https://www.youtube.com/watch?v=85nCF5S8Qok)。

## <a name="containerdcri"></a>containerd/cri

> [!IMPORTANT]
> CRI 支持仅可用于 Server 2019/Windows 10 1809年及更高版本。  我们还仍会主动适用于 Windows 开发 containerd。
> 开发人员/仅用于测试。

[CRI](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto) （容器运行时接口） 而 OCI 规范定义单个容器，作为共享沙盒中 workload(s) 描述容器环境称为广告荚。  Pod 可以包含一个或多个容器工作负荷。  Pod 让容器 orchestrator，如 Kubernetes 和服务的构造网格处理分组应在与一些共享资源，如内存和 vNETs 在同一个主机的工作负荷。

CRI 规范的链接：

* [RunPodSandbox](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L24) -Pod 规范
* [CreateContainer](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L47)的工作负荷规范

![Containerd 基于容器环境](media/containerd-platform.png)

虽然 runHCS 和 containerd 都可以管理在任何 Windows 系统 Server 2016 或更高版本，支持 Pod （容器的组） 所需在 Windows 容器工具重大更改。  CRI 支持可用于在 Windows Server 2019/Windows 10 1809年及更高版本。