---
title: 更新 Windows Server 容器
description: Windows 如何跨多个版本运行内部版本和容器
keywords: 元数据, 容器, 版本
author: heidilohr
ms. author: helohr
manager: lizross
ms.date: 03/10/2020
ms.openlocfilehash: 84413f27bfce66e7d259c05795a280ed34b582ab
ms.sourcegitcommit: 6f216408434a437da87a72d582500a4ca6c2679c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/21/2020
ms.locfileid: "80112690"
---
# <a name="update-windows-server-containers"></a>更新 Windows Server 容器

作为 Windows Server 维护措施的一部分，我们每个月会定期发布更新的 Windows Server 基础操作系统容器映像。 使用这些更新，你可以自动生成更新的容器映像，或通过拉取最新版本来手动更新它们。 与 Windows Server 不同，Windows Server 容器没有维护堆栈。 你无法像 Windows Server 那样在容器中获取更新。 因此，我们每个月都会重建包含更新的 Windows Server 基础操作系统容器映像，并发布更新的容器映像。

其他容器映像（例如 .NET 或 IIS）将基于更新的基础操作系统容器映像进行重建，并每月发布一次。

## <a name="how-to-get-windows-server-container-updates"></a>如何获取 Windows Server 容器更新

我们按与 Windows 维护一致的节奏更新 Windows Server 基础操作系统容器映像。 更新的容器映像在每月的第二个星期二发布，我们有时将其称为“B”版本，它的前缀编号基于发布月份。 例如，我们将 2 月份更新称为“2B”，将三月份更新称为“3B”。 此每月更新事件是包含新安全修补程序的唯一定期发布。

托管这些容器的服务器（称为容器主机，或直接称为“主机”）可以在“B”版本以外的其他更新事件期间进行维护。 若要详细了解 Windows Update 维护节奏，请参阅 [Windows Update 维护节奏](https://techcommunity.microsoft.com/t5/windows-it-pro-blog/windows-10-update-servicing-cadence/ba-p/222376)博客文章。

新的 Windows Server 基础操作系统容器映像将在每月第二个星期二上午 10:00（太平洋标准时间）之后快速在 Microsoft 容器注册表 (MCR) 中上线，其功能标签面向最新的 B 版本。 一些示例如下：

- ltsc2019 [(LTSC)](/windows-server/get-started-19/servicing-channels-19#long-term-servicing-channel-ltsc):  docker pull mcr.microsoft.com/windows/servercore:ltsc2019
- 1909 [(SAC)](/windows-server/get-started-19/servicing-channels-19#semi-annual-channel): docker pull mcr.microsoft.com/windows/servercore:1909

如果你更熟悉 Docker Hub 而非 MCR，可以通过[此博客文章](https://azure.microsoft.com/blog/microsoft-syndicates-container-catalog/)获得更详细的说明。  

对于每个版本，还会发布相应的容器映像，其中包含两个额外的标记，分别用于修订版本号和面向特定容器映像修订版本的知识库文章编号。 例如：

- docker pull mcr.microsoft.com/windows/servercore:10.0.17763.1040
- docker pull mcr.microsoft.com/windows/servercore:1809-KB4546852

这些示例都拉取包含 2 月 18 日安全版本更新的 Windows Server 2019 Server Core 容器映像。  

有关 Windows Server 基础操作系统容器映像、版本及其各自标记的完整列表，请参阅 Docker Hub 中的 [Windows 基础操作系统容器映像](https://hub.docker.com/_/microsoft-windows-base-os-images)。

Microsoft 在 Azure 市场中发布的每月维护的 Windows Server 映像还附带了预先安装的基础操作系统容器映像。 可以在我们的 [Windows Server Azure 市场定价页面](https://azuremarketplace.microsoft.com/marketplace/apps/microsoftwindowsserver.windowsserver?tab=PlansAndPrice)上查找更多信息。 通常，我们会在“B”发布后的大约五个工作日内更新这些映像。

有关 Windows Server 映像和版本的完整列表，请参阅 [Azure 市场更新历史记录中的 Windows Server 版本](https://support.microsoft.com/help/4497947/windows-server-release-on-azure-marketplace-update-history)。

## <a name="host-and-container-version-compatibility"></a>主机和容器版本兼容性

对于 Windows 容器，有两种隔离模式：进程隔离和 Hyper-V 隔离。 当涉及主机和容器版本兼容性时，Hyper-V 隔离更为灵活。 若要了解详细信息，请参阅[版本兼容性](version-compatibility.md)和[隔离模式](../manage-containers/hyperv-container.md)。 除非另行说明，否则本部分介绍的都是进程隔离容器。

使用每月更新来更新容器主机或容器映像时，只要主机和容器映像都受支持（Windows Server 1809 或更高版本），容器就可以在主机和容器映像修订版不需匹配的情况下正常启动并运行。

但是，当使用的 Windows Server 容器安装了 2020 年 2 月 11 日安全更新版本（也称为“2B”）或更高的每月安全更新版本时，可能会遇到问题。 有关更多详细信息，请参阅[此 Microsoft 支持文章](https://support.microsoft.com/help/4542617/you-might-encounter-issues-when-using-windows-server-containers-with-t)。 这些问题是由于某个安全性更改导致的，该更改要求对用户模式与内核模式之间的接口进行更改，以确保应用程序的安全性。 这些问题仅发生在进程隔离容器上，因为进程隔离容器与容器主机共享内核模式。 这意味着没有更新用户模式组件的容器映像是不安全的，与新的受保护内核接口不兼容。

我们已于 2020 年 2 月 18 日发布了一个修补程序。 此新版本建立了一个“新基线”。 此新基线遵循以下规则：

- 主机和容器都早于 2B 的任何组合都可工作。
- 主机和容器都晚于 2B 的任何组合都可工作。
- 主机和容器处于新基线的不同端的任何组合都不可工作。 例如，3B 主机和 1B 容器将不可工作。

让我们使用 2020 年 3 月的每月安全更新版本作为示例，演示这些新的兼容性规则如何生效。 在下表中，2020 年 3 月的安全更新版本称为“3B”，2020 年 2 月的更新称为“2B”，2020 年 1 月的更新称为“1B”。

| 主机 | 容器 | 兼容性 |
|---|---|---|
| 3B | 3B | 是 |
| 3B | 2B | 是 |
| 3B | 1B 或更低版本 | 否 |
| 2B | 3B | 是 |
| 2B | 2B | 是 |
| 2B | 1B 或更低版本 | 否 |
| 1B 或更低版本 | 3B | 否 |
| 1B 或更低版本 | 2B | 否 |
| 1B 或更低版本 | 1B 或更低版本 | 是 |

作为参考，下表列出了各个主要操作系统版本（从 Windows Server 2016 到最新的 Windows Server 版本 1909）中具有 1B、2B 和 3B 每月安全更新版本的基础操作系统容器映像的版本号。

| Windows Server 版本（浮动标记） | 2020 年 1 月 14 日版本的更新版本 (1B)| 2020 年 2 月 18 日版本的更新版本 (2B) | 2020 年 3 月 10 日版本的更新版本 (3B) |
|---|---|---|---|
| Windows Server 2016 (ltsc2016) | 10.0.14393.3443 (KB4534271) | 10.0.14393.3506 (KB4546850) | 10.0.14393.3568 (KB4551573) |
| Windows Server 版本 1803 (1803) | 10.0.17134.1246 (KB4534293) | 10.0.17134.1305 (KB4546851)  | 对此版本的支持已终止。 有关详细信息，请参阅[基础映像维护生命周期](base-image-lifecycle.md)。|
| Windows Server 版本 1809 (1809)| 10.0.17763.973 (KB4534273) | 10.0.17763.1040 (KB4546852) | 10.0.17763.1098 (KB4538461) |
| Windows Server 2019 (ltsc2019) | 10.0.17763.973 (KB4534273) | 10.0.17763.1040 (KB4546852) | 10.0.17763.1098 (KB4538461) |
| Windows Server 版本 1903 (1903) |10.0.18362.592 (KB4528760) | 10.0.18362.658 (KB4546853) | 10.0.18362.719 (KB4540673) |
| Windows Server 版本 1909 (1909) | 10.0.18363.592 (KB4528760) | 10.0.18363.658 (KB4546853) | 10.0.18363.719 (KB4540673) |

## <a name="troubleshoot-host-and-container-image-mismatches"></a>排查主机和容器映像不匹配的问题

在开始之前，请确保熟悉[版本兼容性](version-compatibility.md)中的信息。 此信息可帮助你确定问题是不是由不匹配的修补程序引起的。 如果已确定不匹配的修补程序是导致问题的原因，则可按照本部分中的说明来解决问题。

### <a name="query-the-version-of-your-container-host"></a>查询容器主机的版本

如果可以访问容器主机，则可以运行 `ver` 命令来获取其操作系统版本。 例如，如果你在某个系统上运行 `ver`，而该系统运行包含最新的 2020 年 2 月安全更新版本的 Windows Server 2019 ，则会看到以下信息：

```batch
Microsoft Windows [Version 10.0.17763.1039]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\>ver 

Microsoft Windows [Version 10.0.17763.1039]
```

>[!NOTE]
>此示例中的修订版本号显示为 1039 而不是 1040，因为 2020 年 2 月安全更新版本没有适用于 Windows Server 的带外 2B 版本。 只有一个容器带外 2B 版本的修订版本号为 1040。

如果你不能直接访问容器主机，请咨询你的 IT 管理员。如果是在云中运行，请查看云提供商的网站，了解其正在运行的容器主机操作系统版本。 例如，如果你使用的是 Azure Kubernetes 服务 (AKS)，则可在 [AKS 发行说明](https://github.com/Azure/AKS/releases)中找到主机操作系统版本。

### <a name="query-the-version-of-your-container-image"></a>查询容器映像的版本

按照以下说明查找容器运行的版本：

1. 在 PowerShell 中运行以下 cmdlet：

    ```powershell
    docker images
    ```

    输出应如下所示：

     ```powershell
     REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
     mcr.microsoft.com/windows/servercore   ltsc2019            b456290f487c        4 weeks ago         4.84GB
     mcr.microsoft.com/windows              1809                58229ca44fa7        4 weeks ago         12GB
     mcr.microsoft.com/windows/nanoserver   1809                f519d4f3a868        4 weeks ago         251M

2. Run the `docker inspect` command for the Image ID of the container image that isn't working. This will tell you which version the container image is targeting.

   For example, let's say we `run docker inspect` for an ltsc 2019 container image:

   ```powershell
   docker inspect b456290f487c

       "Architecture": "amd64",

        "Os": "windows",

        "OsVersion": "10.0.17763.1039",

        "Size": 4841309825,

        "VirtualSize": 4841309825,
    ```

    在此示例中，容器操作系统版本显示为 `10.0.17763.1039`。

    如果已在运行容器，还可以在容器本身中运行 `ver` 命令来获取版本。 例如，在具有最新的 2020 年 2 月安全更新版本的 Windows Server 2019 的 Server Core 容器映像中运行 `ver` 会显示以下信息：

    ```batch
    Microsoft Windows [Version 10.0.17763.1040]
    (c) 2020 Microsoft Corporation. All rights reserved.

    C:\>ver

    Microsoft Windows [Version 10.0.17763.1040]
    ```
