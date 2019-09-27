---
title: Windows 容器常见问题解答
description: Windows Server 容器常见问题
keywords: docker, 容器
author: PatrickLang
ms.date: 05/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
ms.openlocfilehash: af12aff787cf178ff80d5db15cc926266816882f
ms.sourcegitcommit: 579349d7bc6a7dbf68445339c468ad8d2b87d7de
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/26/2019
ms.locfileid: "10152728"
---
# <a name="frequently-asked-questions-about-containers"></a>有关容器的常见问题

## <a name="what-are-wcow-and-lcow"></a>什么是 WCOW 和 LCOW？

WCOW 是 "Windows 上的 Windows 容器" 的简短。 LCOW 是 "Windows 上的 Linux 容器" 的简短。

## <a name="whats-the-difference-between-linux-and-windows-server-containers"></a>Linux 与 Windows Server 容器有何区别？

Linux 和 Windows Server 都在其内核和核心操作系统中实现了类似的技术。 区别在于在容器内运行的平台和工作负荷。  

当客户使用 Windows Server 容器时，它们可以与现有 Windows 技术（如 .NET、ASP.NET 和 PowerShell）集成。

## <a name="as-a-developer-do-i-have-to-rewrite-my-app-for-each-type-of-container"></a>作为开发人员，我是否必须为每种类型的容器重写我的应用？

不需要。 Windows 容器映像在 Windows Server 容器和 Hyper-v 隔离中通用。 在你启动容器时选择容器类型。 从开发人员的角度来看，Windows Server 容器和 Hyper-v 隔离是同一内容的两种风格。 它们提供相同的开发、编程和管理体验，并且具有开放和可扩展的功能，并且包含与 Docker 的相同级别的集成和支持。

开发人员可以使用 Windows Server 容器创建容器图像，并在 Hyper-v 隔离中部署它，反之亦然，除了指定相应的运行时标志之外，没有任何更改。

Windows Server 容器可提供更大的密度和性能，以实现速度较长的功能，例如，较低的旋转时间和更快的运行时性能与嵌套配置相比较。 Hyper-v 隔离（对于其名称为 true）提供更大隔离，确保在一个容器中运行的代码无法危害或影响在同一主机上运行的主机操作系统或其他容器。 这对于具有托管不受信任代码的要求的多租户方案（包括 SaaS 应用程序和计算托管）很有用。

## <a name="what-are-the-prerequisites-for-running-containers-on-windows"></a>在 Windows 上运行容器有哪些先决条件？

已将容器引入 Windows Server 2016 的平台。 若要使用容器，你需要 Windows Server 2016 或 Windows 10 周年更新（版本1607）或更高版本。

## <a name="can-i-run-windows-containers-in-process-isolated-mode-on-windows-10-enterprise-or-professional"></a>是否可以在 Windows 10 企业版或专业版的进程隔离模式下运行 Windows 容器？

从2018年10月的更新开始，你可以运行具有进程隔离的 Windows 容器，但必须首先通过使用运行容器时使用`--isolation=process`标志来直接请求进程隔离。 `docker run`

如果你希望以这种方式运行 Windows 容器，你需要确保你的主机运行的是 Windows 10 内部版本 17763 +，并且你有一个具有引擎18.09 或更高版本的 Docker 版本。

> [!WARNING]
> 此功能仅适用于开发和测试。 你应继续使用 Windows Server 作为生产部署的主机。 通过使用此功能，你还必须确保你的主机和容器版本标记匹配，否则容器可能无法启动或表现出未定义的行为。

## <a name="how-do-i-make-my-container-images-available-on-air-gapped-machines"></a>如何让我的容器图像在 air-有空隙的计算机上可用？

Windows 容器基映像包含其分布受许可证限制的项目。 当你在这些映像上构建并将它们推送到私有或公共注册表时，你会注意到基本层从未推送。 相反，我们使用指向驻留在 Azure 云存储中的真正基本层的外国层的概念。

如果您有一个有气流的计算机，而该计算机只能从您的私人容器注册表的地址提取图像，这可能会使操作变得复杂。 在这种情况下，尝试关注外来图层以获取基本图像将不起作用。 若要覆盖外部层行为，可以使用 Docker 后台`--allow-nondistributable-artifacts`程序中的标志。

> [!IMPORTANT]
> 此标志的使用不会妨碍你的义务遵守 Windows 容器基本映像许可证的条款;您不得发布用于公共或第三方再发行的 Windows 内容。 允许在您自己的环境中使用。

## <a name="is-microsoft-participating-in-the-open-container-initiative-oci"></a>Microsoft 是否正在参与开放容器计划 (OCI)？

为保证打包格式保持通用，Docker 最近组织了 "开放容器倡议" （OCI），它旨在确保容器封装保持开放且采用基础 led 格式，将 Microsoft 作为其公开成员之一。

## <a name="additional-feedback"></a>其他反馈

想要向常见问题添加内容？ 在 "评论" 部分中打开新的反馈问题，或使用 GitHub 设置此页面的拉取请求。
