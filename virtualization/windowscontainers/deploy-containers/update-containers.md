---
title: 更新 Windows Server 容器
description: Windows 如何跨多个版本运行内部版本和容器
keywords: 元数据, 容器, 版本
author: heidilohr
ms. author: helohr
manager: lizross
ms.date: 03/10/2020
ms.openlocfilehash: 12a60398a12437ea733b2da31aae853a7afd2c57
ms.sourcegitcommit: 8eedfdc1fda9d0abb36e28dc2b5fb39891777364
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/10/2020
ms.locfileid: "79037414"
---
# <a name="update-windows-server-containers"></a>更新 Windows Server 容器

作为每个月提供 Windows Server 服务的一部分，我们会定期发布更新的 Windows Server 基本操作系统容器映像。 通过这些更新，你可以自动生成更新的容器映像，或通过拉取最新版本来手动更新它们。 Windows Server 容器没有类似于 Windows Server 的服务堆栈。 不能像使用 Windows Server 那样在容器中获取更新。 因此，每个月都将重新生成包含更新的 Windows Server 基本操作系统容器映像，并发布更新的容器映像。

其他容器映像（如 .NET 或 IIS）将根据更新的基本 OS 容器映像重建，并每月发布。

## <a name="how-to-get-windows-server-container-updates"></a>如何获取 Windows Server 容器更新

我们将更新 Windows Server 基本操作系统容器映像，使其与 Windows 维护步调一致。 已更新的容器映像将在每月的第二个星期二发布，这有时称为我们的 "B" 发行版，前缀编号基于发行月份。 例如，我们将我们的2月份更新 "2B" 和三月份更新 "3B"。 这是每月更新事件，是唯一包含新安全修补程序的定期发布。

托管这些容器的服务器（称为容器主机或只是 "主机"）可在除 "B" 版本以外的其他更新事件中提供服务。 若要了解有关 Windows update 维护节奏的详细信息，请参阅[windows update 维护节奏](https://techcommunity.microsoft.com/t5/windows-it-pro-blog/windows-10-update-servicing-cadence/ba-p/222376)博客文章。

新的 Windows Server 基本操作系统容器映像将在 Microsoft 容器注册表（MCR）中每月的第二个星期二完成10： 00 (PST 后立即生效，特色标记面向最新的 B 版本。 示例包括：

- ltsc2019 [（LTSC）](/windows-server/get-started-19/servicing-channels-19#long-term-servicing-channel-ltsc)： mcr.microsoft.com/windows/servercore:ltsc2019 docker pull
- 1909 [（SAC）](/windows-server/get-started-19/servicing-channels-19#semi-annual-channel)： docker pull mcr.microsoft.com/windows/servercore:1909

如果你对 Docker 中心比 MCR 更熟悉，[此博客文章](https://azure.microsoft.com/blog/microsoft-syndicates-container-catalog/)将提供更详细的说明。  

对于每个版本，各自的容器映像还会发布，其中包含修订号的两个附加标记和针对特定容器映像修订的知识库文章编号。 例如：

- docker pull mcr.microsoft.com/windows/servercore:10.0.17763.1040
- docker pull mcr.microsoft.com/windows/servercore:1809-KB4546852

这些示例结合了2019年2月18日的安全更新版本，提取 Windows Server 服务器核心容器映像。  

有关 Windows Server 基本操作系统容器映像、版本及其各自标记的完整列表，请参阅 Docker Hub 上的此[Windows 基本 os 容器映像](https://hub.docker.com/_/microsoft-windows-base-os-images)。

Microsoft 在 Azure Marketplace 上发布的每月服务 Windows Server 映像还附带预装的基本 OS 容器映像。 在我们的[Windows Server Azure Marketplace 定价页](https://azuremarketplace.microsoft.com/marketplace/apps/microsoftwindowsserver.windowsserver?tab=PlansAndPrice)上查找更多。 通常，我们会在 "B" 发布后的五个工作日更新这些映像。

有关 Windows Server 映像和版本的完整列表，请参阅[Azure Marketplace 上的 Windows server 版本更新历史记录](https://support.microsoft.com/help/4497947/windows-server-release-on-azure-marketplace-update-history)。

## <a name="host-and-container-version-compatibility"></a>主机和容器版本兼容性

Windows 容器有两种隔离模式：进程隔离模式和 Hyper-v 隔离模式。 当涉及主机和容器版本兼容性时，hyper-v 隔离更为灵活。 若要了解详细信息，请参阅[版本兼容性](version-compatibility.md)和[隔离模式](../manage-containers/hyperv-container.md)。 本部分将重点介绍进程隔离的容器，除非另行说明。

当你用每月更新更新容器主机或容器映像时，只要主机和容器映像都受支持（Windows Server 1809 或更高版本），则主机和容器映像修订不需要与容器的启动和正常运行。

但是，在2020年2月11日的安全更新版本（也称为 "2B"）或更高的每月安全更新版本中使用 Windows Server 容器时，可能会遇到问题。 有关更多详细信息，请参阅[此 Microsoft 支持部门文章](https://support.microsoft.com/help/4542617/you-might-encounter-issues-when-using-windows-server-containers-with-t)。 这些问题是由于安全更改导致的，需要在用户模式和内核模式之间进行更改，以确保应用程序的安全性。 这些问题仅发生在进程隔离容器上，因为进程隔离容器与容器主机共享内核模式。 这意味着没有更新用户模式组件的容器映像是不安全的，不与新的受保护内核接口兼容。

我们已于2020年2月18日发布了一个修补程序。 此新版本已建立 "新基线"。 这一新基线遵循以下规则：

- 在2B 之前的主机和容器的任意组合都将起作用。
- 在2B 之后的主机和容器的任意组合都将起作用。
- 新基线的不同面上的任何主机和容器组合都不起作用。 例如，3B 主机和1B 容器将不起作用。

让我们使用三月2020每月的安全更新版本作为示例，演示这些新的兼容性规则如何工作。 在下表中，2020年3月版安全更新版本称为 "3B，2月2020更新为" 2B，1月2020更新为 "1B"。

| Host | 容器 | 兼容性 |
|---|---|---|
| 亿美元 | 亿美元 | 是 |
| 亿美元 | 之 | 是 |
| 亿美元 | 1B 或更早版本 | 是 |
| 之 | 亿美元 | 是 |
| 之 | 之 | 是 |
| 之 | 1B 或更早版本 | 是 |
| 1B 或更早版本 | 亿美元 | 是 |
| 1B 或更早版本 | 之 | 是 |
| 1B 或更早版本 | 1B 或更早版本 | 是 |

对于参考，下表列出了基本 OS 容器映像的版本号，每月将 Windows Server 2016 中的不同主要 OS 版本中的每月安全更新发布到最新的 Windows Server 1909 版本。

| Windows Server 版本（浮动标记） | 1/14/20 版（1B）的更新版本| 2/18/20 版本的更新版本（2B） | 3/10/20 版（3B）的更新版本 |
|---|---|---|---|
| Windows Server 2016 （ltsc2016） | 10.0.14393.3443 (B4534271) | 10.0.14393.3506 (KB4546850) | 即将发布的容器 |
| Windows Server，版本1803（1803） | 10.0.17134.1246 (KB4534293) | 10.0.17134.1305 (KB4546851)  | 此版本已达到支持的结尾。 有关详细信息，请参阅[基本映像服务生命周期](base-image-lifecycle.md)。|
| Windows Server 2019 （ltsc2019） | 10.0.17763.973 (KB4534273) | 10.0.17763.1040 (KB4546852) | 10.0.17763.1098 (KB4538461) |
| Windows Server，版本1903（1903） |10.0.18362.592 (KB4528760) | 10.0.18362.658 (KB4546853) | 10.0.18362.719 (KB4540673) |
| Windows Server，版本1909（1909） | 10.0.18363.592 (KB4528760) | 10.0.18363.658 (KB4546853) | 10.0.18363.719 (KB4540673) |

## <a name="troubleshoot-host-and-container-image-mismatches"></a>排除主机和容器映像不匹配问题

在开始之前，请确保熟悉[版本兼容性](version-compatibility.md)信息。 此信息可帮助你确定问题是否是由不匹配的修补程序引起的。 如果已建立了不匹配的修补程序作为原因，则可以按照本部分中的说明进行操作以解决此问题。

### <a name="query-the-version-of-your-container-host"></a>查询容器主机的版本

如果可以访问容器主机，则可以运行 `ver` 命令来获取其操作系统版本。 例如，如果在运行 Windows Server 2019 的系统上运行 `ver` 并使用最新的2月版2020安全更新版本，则会看到以下内容：

```batch
Microsoft Windows [Version 10.0.17763.1039]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\>ver 

Microsoft Windows [Version 10.0.17763.1039]
```

>[!NOTE]
>此示例中的修订号显示为1039，而不是1040，因为2月2020安全更新版本没有适用于 Windows Server 的带外2B 版本。 对于具有修订号1040的容器，只有带外2B 版本。

如果你不能直接访问容器主机，请咨询你的 IT 管理员。如果正在云上运行，请查看云提供商的网站，了解它们正在运行的容器主机操作系统版本。 例如，如果你使用的是 Azure Kubernetes Service （AKS），则可以在[AKS 发行说明](https://github.com/Azure/AKS/releases)中找到主机操作系统版本。

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

    在此示例中，容器 OS 版本显示为 `10.0.17763.1039`。

    如果已运行容器，还可以在容器本身中运行 `ver` 命令来获取版本。 例如，使用最新的2月版2020安全更新版本在 Windows Server 2019 的服务器核心容器映像中运行 `ver` 将显示以下内容：

    ```batch
    Microsoft Windows [Version 10.0.17763.1040]
    (c) 2020 Microsoft Corporation. All rights reserved.

    C:\>ver

    Microsoft Windows [Version 10.0.17763.1040]
    ```
