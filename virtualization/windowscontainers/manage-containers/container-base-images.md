---
title: Windows 容器基础映像
description: 概述 Windows 容器基础映像以及何时使用它们。
keywords: docker, 容器, 哈希
author: patricklang
ms.date: 09/25/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 88e6e080-cf8f-41d8-a301-035959dc5ce0
ms.openlocfilehash: 9884cc0ae2d2f398d2dc2fb1997a70493a6de6c0
ms.sourcegitcommit: 16744984ede5ec94cd265b6bff20aee2f782ca88
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/28/2020
ms.locfileid: "76764182"
---
# <a name="container-base-images"></a>容器基础映像

Windows 提供了四个容器基础映像，用户可以基于它们进行构建。 每个基础映像都是 Windows 操作系统的一种不同风格，其磁盘占用量不同，并携带不同数量的 Windows API 集。

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
                        <p>支持传统的 .NET Framework 应用程序。</p>
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
                        <p>针对 .NET Core 应用程序构建。</p>
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
                        <p>专为 IoT 应用程序构建。</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="image-discovery"></a>映像发现

所有 Windows 容器基础映像均可通过 [Docker Hub](https://hub.docker.com/_/microsoft-windows-base-os-images) 发现。 Windows 容器基础映像本身由 Microsoft 容器注册表 (MCR) [mcr.microsoft.com](https://azure.microsoft.com/en-us/services/container-registry/) 提供。 这就是针对 Windows 容器基础映像的拉取命令如下所示的原因：

```code
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

MCR 没有其自己的目录体验，它用于支持现有的目录，例如 Docker Hub。 由于 Azure 已覆盖全球并可与 Azure CDN 配合使用，因此 MCR 能够提供一致且快速的映像拉取体验。 网络内性能增强、与 MCR（Microsoft 容器映像来源）和 Azure 市场紧密集成，并且 Azure 中以部署包格式提供容器的服务的数量在不断增加，这一切使得在 Azure 中运行其工作负荷的 Azure 客户受益匪浅。

## <a name="choosing-a-base-image"></a>选择基础映像

如何选择用作构建基础的合适基础映像？ 对大多数用户而言，`Windows Server Core` 和 `Nanoserver` 将是最适合使用的映像。

### <a name="guidelines"></a>指南

 虽然你可以自由决定使用哪个映像，但也可以根据下面提供的一些指导原则来进行选择：

- **你的应用程序是否需要完整的 .NET Framework？** 如果此问题的答案为“是”，则应选择 `Windows Server Core`。
- **你是否基于 .NET Core 构建 Windows 应用？** 如果此问题的答案为“是”，则应选择 `Nanoserver`。
- **你是否在构建 IoT 应用程序？** 如果此问题的答案为“是”，则应选择 `IoT Core`。
- **Windows Server Core 容器映像是否缺少你的应用所需的依赖项？** 如果此问题的答案为“是”，则应尝试选择 `Windows`。 此映像远远大于其他基础映像，但它携带了许多核心 Windows 库（例如 GDI 库）。
- **你是 Windows 预览体验成员吗？** 如果是，则应考虑使用映像的预览体验版。 请参阅下文中的“适用于 Windows 预览体验成员的基础映像”。

> [!TIP]
> 许多 Windows 用户希望将依赖于 .NET 的应用程序容器化。 除了此处所述的四个基础映像，Microsoft 还发布了多个 Windows 容器映像（例如 [.NET Framework](https://hub.docker.com/_/microsoft-dotnet-framework) 映像和 [ASP .NET](https://hub.docker.com/_/microsoft-dotnet-framework-aspnet/) 映像），这些映像预配置了常用的 Microsoft 框架。

### <a name="base-images-for-windows-insiders"></a>适用于 Windows 预览体验成员的基础映像

Microsoft 提供了每个容器基础映像的“预览体验”版本。 这些预览体验版容器映像中包含了我们在容器映像方面最新最棒的功能开发。 当运行的主机使用 Windows 预览体验版（无论是 Windows 预览体验版还是 Windows Server 预览体验版）时，最好使用这些映像。 Docker Hub 中提供了以下预览体验版映像：

- [mcr.microsoft.com/windows/servercore/insider](https://hub.docker.com/_/microsoft-windows-servercore-insider)
- [mcr.microsoft.com/windows/nanoserver/insider](https://hub.docker.com/_/microsoft-windows-nanoserver-insider)
- [mcr.microsoft.com/windows/iotcore/insider](https://hub.docker.com/_/microsoft-windows-iotcore-insider)
- [mcr.microsoft.com/windows/insider](https://hub.docker.com/_/microsoft-windows-insider)

若要了解详细信息，请阅读[将容器与 Windows 预览体验计划配合使用](../deploy-containers/insider-overview.md)。

### <a name="windows-server-core-vs-nanoserver"></a>Windows Server Core 与 Nanoserver

`Windows Server Core` 和 `Nanoserver` 是要使用的最常见基础映像。 这两种映像之间的主要区别是，Nanoserver 中的 API 图面要小得多。 Nanoserver 映像中缺少 PowerShell、WMI 和 Windows 服务堆栈。

Nanoserver 用于提供恰好足够的 API 图面来运行依赖于 .NET Core 或其他新式开源框架的应用。 API 图面较小的好处是，Nanoserver 映像的磁盘占用量远远小于其余的 Windows 基础映像。 请记住，你始终可以在自己认为合适的条件下在 Nano Server 上添加层。 有关此情况的示例，请查看 [.NET Core Nano Server Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1909/amd64/Dockerfile)。
