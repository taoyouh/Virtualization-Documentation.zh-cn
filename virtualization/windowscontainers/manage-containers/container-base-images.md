---
title: Windows 容器基本图像
description: Windows 容器基本图像概述以及何时使用它们。
keywords: docker, 容器, 哈希
author: patricklang
ms.date: 09/25/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 88e6e080-cf8f-41d8-a301-035959dc5ce0
ms.openlocfilehash: f5dcaf4958828b1bcf31a96e5fb70eda0508eb96
ms.sourcegitcommit: e9dda81f1f68359ece9ef132a184a30880bcdb1b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/27/2019
ms.locfileid: "10161744"
---
# <a name="container-base-images"></a>容器基映像

Windows 提供了可供用户构建的四个容器基映像。 每个基本映像都是不同的 Windows 操作系统风格，具有不同的磁盘空间，并且携带不同数量的 Windows API 集。

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows-servercore" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Windows Server Core</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>支持传统的 .NET 框架应用程序。</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows-nanoserver" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Nano Server</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>针对 .NET Core 应用程序而构建。</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Windows</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>提供完整的 Windows API 集。</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows-iotcore" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Windows IoT 核心版</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>专为 IoT 应用程序而构建。</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="image-discovery"></a>图像发现

通过[Docker 集线器](https://hub.docker.com/_/microsoft-windows-base-os-images)可发现所有 Windows 容器基映像。 Windows 容器基本映像本身从[mcr.microsoft.com](https://azure.microsoft.com/en-us/services/container-registry/)（Microsoft 容器注册表（mcr））提供。 这是 Windows 容器基本图像的 pull 命令的外观如下所示：

```code
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

MCR 没有自己的目录体验，它旨在支持诸如 Docker 集线器之类的现有目录。 由于 Azure 全球通用空间和与 Azure CDN 结合使用，MCR 提供了一致且快速的图像提取体验。 Azure 客户在 Azure 中运行工作负荷，受益于网络性能增强，以及与 MCR （Microsoft 容器映像的源）、Azure Marketplace 和不断扩展的 Azure 中的服务数量的紧密集成作为部署包格式的容器。

## <a name="choosing-a-base-image"></a>选择基本图像

如何选择要在其上构建的正确的基本图像？ 对于大多数用户， `Windows Server Core` `Nanoserver`将是最适合使用的图像。

### <a name="guidelines"></a>指南

 虽然你可以随意定位你所需的任何图像，但以下是有助于指导你选择的指南：

- **你的应用程序是否需要完整的 .NET framework？** 如果此问题的答案为 "是"，则应`Windows Server Core`确定目标。
- **是否根据 .NET Core 构建 Windows 应用？** 如果此问题的答案为 "是"，则应`Nanoserver`确定目标。
- **您是否正在构建 IoT 应用程序？** 如果此问题的答案为 "是"，则应`IoT Core`确定目标。
- **Windows Server Core 容器映像是否缺少你的应用所需的依赖关系？** 如果此问题的答案是 "是"，则应尝试瞄准`Windows`。 此图像比其他基本映像更大，但它将携带许多核心 Windows 库（如 GDI 库）。

> [!TIP]
> 许多 Windows 用户希望 containerize 依赖于 .NET 的应用程序。 除了此处介绍的四个基本图像，Microsoft 还会发布多个 Windows 容器映像，这些图像预配置了常用 Microsoft 框架，例如[.net framework](https://hub.docker.com/_/microsoft-dotnet-framework)图像和[ASP .net](https://hub.docker.com/_/microsoft-dotnet-framework-aspnet/)图像。

### <a name="windows-server-core-vs-nanoserver"></a>Windows Server Core vs Nanoserver

`Windows Server Core` 最`Nanoserver`常见的基本图像是目标。 这些映像之间的主要区别在于 Nanoserver 具有明显较小的 API 图面。 Nanoserver 映像中缺少 PowerShell、WMI 和 Windows 服务堆栈。

Nanoserver 已构建为仅提供足够的 API 图面来运行依赖于 .NET core 或其他新式开放源代码框架的应用。 与较小的 APi 图面相比，Nanoserver 映像具有比其他 Windows 基本映像更小的磁盘空间。 请记住，你始终可以在自己认为合适的条件下在 Nano Server 上添加层。 有关此情况的示例，请查看 [.NET Core Nano Server Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile)。
