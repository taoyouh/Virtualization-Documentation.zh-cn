---
title: Windows 容器基本映像
description: 概述 Windows 容器基本映像以及何时使用它们。
keywords: docker, 容器, 哈希
author: patricklang
ms.date: 09/25/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 88e6e080-cf8f-41d8-a301-035959dc5ce0
ms.openlocfilehash: 9884cc0ae2d2f398d2dc2fb1997a70493a6de6c0
ms.sourcegitcommit: 16744984ede5ec94cd265b6bff20aee2f782ca88
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/28/2020
ms.locfileid: "76764182"
---
# <a name="container-base-images"></a>容器基本映像

Windows 提供了四个容器基本映像，用户可以从中进行构建。 每个基本映像都是不同的 Windows OS 风格，具有不同的磁盘空间，并携带不同数量的 Windows API。

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows-servercore" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>Windows Server 核心 
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none"></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>支持传统的 .NET framework 应用程序。</p>
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
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Nano Server 
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none"></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>为 .NET Core 应用程序生成的。</p>
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
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Windows 
                    <div class="has-padding-bottom-none">
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small"></h3>
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
                    </div>Windows IoT Core 
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none"></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>用于 IoT 应用程序的用途。</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="image-discovery"></a>映像发现

所有 Windows 容器基本映像都可以通过[Docker Hub](https://hub.docker.com/_/microsoft-windows-base-os-images)发现。 Windows 容器基本映像本身是从[mcr.microsoft.com](https://azure.microsoft.com/en-us/services/container-registry/)（Microsoft 容器注册表（mcr））提供的。 这就是 Windows 容器基本映像的拉取命令如下所示的原因：

```code
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

MCR 没有其自己的目录体验，旨在支持现有目录，例如 Docker 中心。 由于 Azure 的全球覆盖和与 Azure CDN 结合使用，MCR 提供了一个一致且快速的图像提取体验。 Azure 客户在 Azure 中运行工作负荷，受益于网络中的性能增强功能，并与 MCR （Microsoft 容器映像的源）、Azure Marketplace 以及扩展的 Azure 中的服务数量进行紧密集成。容器作为部署包格式。

## <a name="choosing-a-base-image"></a>选择基本映像

如何选择要生成的正确基本映像？ 对于大多数用户而言，`Windows Server Core` 和 `Nanoserver` 将是最适合使用的图像。

### <a name="guidelines"></a>指南

 虽然你可以自由定位所需的任何图像，但以下是一些指导原则，可帮助你选择：

- **您的应用程序是否需要完整的 .NET framework？** 如果此问题的答案是 "是"，则应以 `Windows Server Core`为目标。
- **是否基于 .NET Core 生成 Windows 应用？** 如果此问题的答案是 "是"，则应以 `Nanoserver`为目标。
- **你是否正在构建 IoT 应用程序？** 如果此问题的答案是 "是"，则应以 `IoT Core`为目标。
- **Windows Server Core 容器映像缺少应用所需的依赖项吗？** 如果此问题的答案是 "是"，则应该尝试以 `Windows`为目标。 此映像比其他基本映像要大得多，但它携带许多核心 Windows 库（如 GDI 库）。
- **你是 Windows 预览体验成员吗？** 如果是，则应考虑使用预览体验版映像。 请参阅下面的 "Windows 预览体验的基本映像"。

> [!TIP]
> 许多 Windows 用户希望容器化依赖于 .NET 的应用程序。 除了此处所述的四个基本映像，Microsoft 还发布了几个 Windows 容器映像，这些映像预配置了常用的 Microsoft 框架，如[.net framework](https://hub.docker.com/_/microsoft-dotnet-framework)映像和[ASP .net](https://hub.docker.com/_/microsoft-dotnet-framework-aspnet/)映像。

### <a name="base-images-for-windows-insiders"></a>Windows 预览体验的基本映像

Microsoft 提供每个容器基本映像的 "内部版本" 版本。 这些有问必答容器映像在容器映像中提供最新且最强大的功能开发。 当你运行的是 Windows 有问必答版（Windows 有问必答或 Windows Server 有问必答）的主机时，最好使用这些映像。 可在 Docker 中心获取 insider 映像：

- [mcr.microsoft.com/windows/servercore/insider](https://hub.docker.com/_/microsoft-windows-servercore-insider)
- [mcr.microsoft.com/windows/nanoserver/insider](https://hub.docker.com/_/microsoft-windows-nanoserver-insider)
- [mcr.microsoft.com/windows/iotcore/insider](https://hub.docker.com/_/microsoft-windows-iotcore-insider)
- [mcr.microsoft.com/windows/insider](https://hub.docker.com/_/microsoft-windows-insider)

有关详细信息，[请参阅使用 Windows 预览体验计划使用容器](../deploy-containers/insider-overview.md)。

### <a name="windows-server-core-vs-nanoserver"></a>Windows Server Core 与 Nanoserver

`Windows Server Core` 和 `Nanoserver` 是目标最常见的基本映像。 这些映像之间的主要区别在于，Nanoserver 的 API 图面大大减少。 Nanoserver 映像中缺少 PowerShell、WMI 和 Windows 服务堆栈。

构建 Nanoserver 是为了提供足够的 API 图面来运行依赖于 .NET core 或其他新式开源框架的应用程序。 与较小的 APi 图面相比，Nanoserver 映像的磁盘占用量明显小于 Windows 基础映像的其余部分。 请记住，你始终可以在自己认为合适的条件下在 Nano Server 上添加层。 有关此情况的示例，请查看 [.NET Core Nano Server Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1909/amd64/Dockerfile)。
