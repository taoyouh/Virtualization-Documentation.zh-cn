---
title: 已知问题
description: Windows Server 容器的已知问题
keywords: 元数据, 容器, 版本,
author: weijuans
ms. author: weijuans
manager: taylob
ms.date: 05/26/2020
ms.openlocfilehash: f53ff0c8c07e86b25358a3acba09622fdd3a6bbb
ms.sourcegitcommit: 82f45088d8b39e2c1f3f2a33bf359d18a88f975a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/18/2020
ms.locfileid: "84978329"
---
# <a name="known-issues"></a>已知问题

## <a name="know-issues-of-windows-server-version-2004"></a>Windows Server 版本 2004 的已知问题

### <a name="1-performance-issue-on-server-core-container"></a>1.服务器核心容器上的性能问题
在准备正式发布 Windows Server 版本 2004 的过程中，与我们在 2019 年 12 月的博客（参见[此处](https://techcommunity.microsoft.com/t5/containers/making-windows-server-core-containers-40-smaller/ba-p/1058874)）以及 .NET 团队博客（参见[此处](https://devblogs.microsoft.com/dotnet/we-made-windows-server-core-container-images-40-smaller/)）中记录的性能改进相比，我们和 .NET 团队共同发现 2020 年 5 月 27 日版本中当前服务器核心容器映像存在潜在的性能问题。 然后，我们重新在 Windows Server 版本 2004 预览体验计划预览发行版服务器核心容器映像上进行了性能分析。 

我们观察到的征兆包括：

如果你使用服务器核心容器映像来构建自己的映像并上传到远程容器注册表（如 Azure 容器注册表），然后你从注册表拉取该映像并运行它，你会发现容器的运行速度变慢了。 但是，如果在本地构建和运行映像，则不会观察到性能差异。

我们已找到该问题的根本原因，且正在努力解决它。 可查找以下链接，它们用于跟踪问题：[microsoft/hcsshim#830](https://github.com/microsoft/hcsshim/issues/830)；

[moby/moby#41066](https://github.com/moby/moby/issues/41066)；

[containerd/containerd#4301](https://github.com/containerd/containerd/issues/4301)。

我们还在合作伙伴 Mirantis 积极合作，力图在 Docker EE 中加入该修补程序。

## <a name="know-issues-of-windows-server-version-1909"></a>Windows Server 版本 1909 的已知问题

## <a name="know-issues-of-windows-server-version-1903"></a>Windows Server 版本 1903 的已知问题

## <a name="know-issues-of-windows-server-2019windows-server-version-1809"></a>Windows Server 2019/Windows Server 版本 1809 的已知问题

## <a name="know-issues-of-windows-server-2016"></a>Windows Server 2016 的已知问题
