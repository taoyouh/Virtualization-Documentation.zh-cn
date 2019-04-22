---
title: Windows 容器常见问题解答
description: Windows Server 容器常见问题解答
keywords: docker, 容器
author: PatrickLang
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
ms.openlocfilehash: 69783f0fc3dcc80eb9614031dc6c9b2c35eeefd1
ms.sourcegitcommit: a5ff22c205149dac4fc05325ef3232089826f1ef
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/22/2019
ms.locfileid: "9380231"
---
# <a name="frequently-asked-questions"></a>常见问题解答

## <a name="general"></a>概要

### <a name="what-is-wcow-what-is-lcow"></a>什么是 WCOW？ 什么是 LCOW？

WCOW 是 Windows 和 LCOW 上的 Windows 容器的缩写词是 Windows 上的 Linux 容器的缩写词。

### <a name="what-is-the-difference-between-linux-and-windows-server-containers"></a>Linux 和 Windows Server 容器之间的区别是什么？

Linux 和 Windows Server 容器很相似，它们都实现类似的技术在其内核和核心操作系统内。 区别在于在容器内运行的平台和工作负荷。  

当客户使用 Windows Server 容器时，则可以与现有 Windows 技术集成等.NET、 ASP.NET、 PowerShell 和详细信息。

### <a name="as-a-developer-do-i-have-to-rewrite-my-app-for-each-type-of-container"></a>作为开发人员，我是否需要重新编写我的应用的每种类型的容器？

不需要。 Windows 容器映像都是通用的 Windows Server 容器和 HYPER-V 隔离。 在你启动容器时选择容器类型。 从开发人员的角度看，Windows Server 容器和 HYPER-V 隔离是同一事物的两种风格。 它们提供相同的开发、 编程和管理体验、 是开放的且可扩展并将包含相同级别的集成和与 Docker 的支持。

开发人员可以创建使用 Windows Server 容器的容器映像，并将其部署中 HYPER-V 隔离或反之亦然而无需除指定相应的运行时标志之外的任何更改。

键，如时间和与嵌套配置相比更快的运行时性能较低注重速度时，Windows Server 容器提供更高的密度和性能。 HYPER-V 隔离，设置为 true 将它的名称，提供更好的隔离，从而在一个容器中运行的代码无法危害或影响主机操作系统或在同一主机上运行的其他容器。 这可用于带有对托管不受信任的代码，包括 SaaS 应用程序和计算托管要求多租户方案。

### <a name="what-are-the-prerequisites-for-running-containers-on-windows"></a>在 Windows 上运行容器的先决条件是什么？

容器引入到 Windows Server 2016 平台。 若要使用容器，你将需要 Windows Server 2016 或 Windows 10 周年更新 （版本 1607年） 或更高版本。

### <a name="can-i-run-windows-containers-in-process-isolated-mode-on-windows-10-enterprise-or-professional"></a>可以运行 Windows 容器在进程隔离模式下在 Windows 10 企业版或专业版上？

开始使用 Windows 10 2018 年 10 月更新，我们不会再不允许用户在运行的进程隔离的 Windows 容器。 但是，你必须直接请求的进程隔离通过使用`--isolation=process`标志运行通过容器时`docker run`。

如果这是一个感兴趣，你需要确保你的主机运行 Windows 10 版本 17763 +，并且具有引擎 18.09 的 docker 版本或更高版本。

> [!WARNING]
> 此功能仅适用于开发/测试。 你应该继续与主机的 Windows Server 用于生产部署。
>
> 通过使用此功能，你还必须确保你的主机和容器版本标记匹配，否则容器可能无法启动或可能出现未定义的行为。

## <a name="windows-container-management"></a>Windows 容器管理

### <a name="how-do-i-make-my-container-images-available-on-air-gapped-machines"></a>如何使我的容器映像可用孤立的计算机上？

Windows 容器基本映像包含其分发许可证受限制的字体的项目。 当你在这些映像上生成和推送到私人或公共注册表时，你会注意到永远不会推送基层。 相反，我们使用指向驻留在 Azure 云存储中的真实基本层外层的概念。

当具有孤立的计算机，仅可以拉取的地址专用容器注册表中的图像时，这可以将一个问题。 尝试遵循外层，以获取基本映像都将在此情况下失败。 若要替代外层行为，你可以使用`--allow-nondistributable-artifacts`Docker 守护程序中的标志。

> [!IMPORTANT]
> 此标志的使用情况不应排除你义务遵守的 Windows 容器基本映像许可证; 条款你必须不发布公共或第三方重新分发的 Windows 内容。 允许你自己的环境内使用。

## <a name="microsofts-open-ecosystem"></a>Microsoft 的开放生态系统

### <a name="is-microsoft-participating-in-the-open-container-initiative-oci"></a>Microsoft 是否正在参与开放容器计划 (OCI)？

为了保证封装格式保持通用，Docker 最近组织了开放容器计划 (OCI)，旨在确保容器封装保持开放和以基础为导向的格式，其中 Microsoft 是创始成员之一。

> [!TIP]
> 已添加到常见问题解答建议？ 在备注部分中打开一个新的反馈问题或使用 GitHub 打开拉取请求针对这些文档 ！
