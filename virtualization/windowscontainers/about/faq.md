---
title: Windows 容器常见问题解答
description: Windows Server 容器常见问题
keywords: docker, 容器
author: PatrickLang
ms.date: 10/25/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
ms.openlocfilehash: 405b2abc43a4ae2c546de351679deb755e4a9317
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910797"
---
# <a name="frequently-asked-questions-about-containers"></a>有关容器的常见问题

## <a name="whats-the-difference-between-linux-and-windows-server-containers"></a>Linux 和 Windows Server 容器之间的区别是什么？

Linux 和 Windows Server 都在其内核和核心操作系统内实现类似的技术。 区别在于在容器内运行的平台和工作负荷。  

当客户使用 Windows Server 容器时，它们可以与现有 Windows 技术（例如 .NET、ASP.NET 和 PowerShell）集成。

## <a name="what-are-the-prerequisites-for-running-containers-on-windows"></a>在 Windows 上运行容器的先决条件是什么？

已将容器引入 Windows Server 2016 平台。 若要使用容器，你将需要 Windows Server 2016 或 Windows 10 周年更新（版本1607）或更高版本。 有关详细信息，请阅读[系统要求](../deploy-containers/system-requirements.md)。

## <a name="what-are-wcow-and-lcow"></a>什么是 WCOW 和 LCOW？

WCOW 是 "windows 容器 on Windows" 的缩写。 LCOW 是 "适用于 Windows 上的 Linux 容器" 的缩写。

## <a name="how-are-containers-licensed-is-there-a-limit-to-the-number-of-containers-i-can-run"></a>如何许可容器？ 我可以运行的容器数是否有限制？

Windows 容器映像[EULA](../images-eula.md)说明了依赖于具有有效许可主机操作系统的用户的使用情况。 允许用户运行的容器数取决于主机操作系统版本和运行容器的[隔离模式](../manage-containers/hyperv-container.md)，以及这些容器是否正在运行以进行开发/测试或生产。

|主机 OS                                                         |进程隔离的容器限制                   |Hyper-v-隔离容器限制                   |
|----------------------------------------------------------------|---------------------------------------------------|---------------------------------------------------|
|Windows Server Standard                                         |无限制                                          |2                                                  |
|Windows Server Datacenter                                       |无限制                                          |无限制                                          |
|Windows 10 专业版和企业版                                   |无限制 *（仅用于测试或开发用途）*|无限制 *（仅用于测试或开发用途）*|
|Windows 10 IoT 核心版和企业版                             |受                                         |受                                          |

Windows Server 容器映像使用情况是通过读取该[版本](/windows-server/get-started-19/editions-comparison-19.md)支持的虚拟化来宾数量来确定的。 <br/>

>[!NOTE]
>Windows IoT 版本上 \*容器的生产使用情况取决于你是否同意使用 Windows 10 Core 运行时映像的 Microsoft 商业使用条款或 Windows 10 IoT 企业版设备许可证（"Windows IoT 商业协议"）。 Windows IoT 商业协议中的其他条款和限制适用于你在生产环境中使用容器映像的情况。 请阅读[容器映像的 EULA](../images-eula.md) ，了解完全允许的内容和不允许的内容。

## <a name="as-a-developer-do-i-have-to-rewrite-my-app-for-each-type-of-container"></a>作为开发人员，我是否必须为每种类型的容器重新编写我的应用？

否。 Windows 容器映像在 Windows Server 容器和 Hyper-v 隔离上都是通用的。 在你启动容器时选择容器类型。 从开发人员的角度来看，Windows Server 容器和 Hyper-v 隔离是同一项的两种风格。 它们提供相同的开发、编程和管理体验，它们是开放的且可扩展的，并且提供与 Docker 相同级别的集成和支持。

开发人员可以使用 Windows Server 容器创建容器映像并将其部署为 Hyper-v 隔离，反之亦然，而无需除了指定适当的运行时标志之外的任何更改。

与嵌套配置相比，Windows Server 容器提供了更高的密度和性能（例如，加速时间缩短）和更快的运行时性能。 Hyper-v 隔离，其名称为 true，提供更高的隔离性，确保在一个容器中运行的代码无法危害或影响主机操作系统或在同一主机上运行的其他容器。 这适用于具有托管不受信任的代码（包括 SaaS 应用程序和计算托管）要求的多租户方案。

## <a name="can-i-run-windows-containers-in-process-isolated-mode-on-windows-10"></a>能否在 Windows 10 上以进程隔离模式运行 Windows 容器？

从 Windows 10 2018 10 月版更新开始，你可以运行具有进程隔离的 Windows 容器，但在使用 `docker run`运行容器时，必须首先通过使用 `--isolation=process` 标志直接请求进程隔离。 在 Windows 10 专业版、Windows 10 企业版、Windows 10 IoT 核心版和 Windows 10 IoT 企业版上，进程隔离兼容。

如果要以这种方式运行 Windows 容器，需要确保主机运行的是 Windows 10 build 17763 +，并且有一个带有引擎18.09 或更高版本的 Docker 版本。

> [!WARNING]
> 除了 IoT 核心和 IoT 企业主机（在接受其他条款和限制之后），此功能仅适用于开发和测试。 应该继续使用 Windows Server 作为生产部署的主机。 通过使用此功能，还必须确保主机和容器版本标记相匹配，否则容器可能无法启动或出现未定义的行为。

## <a name="how-do-i-make-my-container-images-available-on-air-gapped-machines"></a>如何实现使我的容器映像可用于有气流的计算机上？

Windows 容器基本映像包含其分发受许可证限制的项目。 当你在这些映像上构建并将其推送到专用或公用注册表时，你会注意到基本层从未推送。 相反，我们将使用一个指向位于 Azure 云存储中的真正基本层的外部层概念。

当你有一个只能从专用容器注册表的地址提取映像的有气流的计算机时，这可能会使某些功能复杂化。 在这种情况下，尝试遵循外层来获取基本映像将不起作用。 若要替代外部层行为，可以在 Docker 守护程序中使用 `--allow-nondistributable-artifacts` 标志。

> [!IMPORTANT]
> 使用此标志不会妨碍你义务遵守 Windows 容器基本映像许可证的条款;不能发布用于公共或第三方再发行的 Windows 内容。 允许在你自己的环境中使用。

## <a name="additional-feedback"></a>其他反馈

想要向 FAQ 添加一些内容？ 在 "评论" 部分中打开新的反馈问题，或者使用 GitHub 为此页设置拉取请求。
