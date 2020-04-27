---
title: Windows 容器常见问题解答
description: Windows Server 容器常见问题解答
keywords: docker, 容器
author: PatrickLang
ms.date: 10/25/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
ms.openlocfilehash: 405b2abc43a4ae2c546de351679deb755e4a9317
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/24/2020
ms.locfileid: "74910797"
---
# <a name="frequently-asked-questions-about-containers"></a>有关容器的常见问题

## <a name="whats-the-difference-between-linux-and-windows-server-containers"></a>Linux 和 Windows Server 容器的区别是什么？

Linux 和 Windows Server 都在其内核和核心操作系统中实现了类似的技术。 区别在于在容器内运行的平台和工作负荷。  

客户使用 Windows Server 容器时，这些容器可以与 .NET、ASP.NET 和 PowerShell 之类的现有 Windows 技术集成。

## <a name="what-are-the-prerequisites-for-running-containers-on-windows"></a>在 Windows 上运行容器的先决条件是什么？

已将容器引入装有 Windows Server 2016 的平台。 若要使用容器，需要安装 Windows Server 2016 或 Windows 10 周年更新版（版本 1607）或更高版本。 请阅读[系统要求](../deploy-containers/system-requirements.md)，了解详细信息。

## <a name="what-are-wcow-and-lcow"></a>WCOW 和 LCOW 是什么？

WCOW 是“Windows containers on Windows”（Windows 上的 Windows 容器）的缩写。 LCOW 是“Linux containers on Windows”（Windows 上的 Linux 容器）的缩写。

## <a name="how-are-containers-licensed-is-there-a-limit-to-the-number-of-containers-i-can-run"></a>如何许可容器？ 我可以运行的容器数是否有限制？

Windows 容器映像 [EULA](../images-eula.md) 描述的使用情况有一个前提：用户安装了获得有效许可的主机 OS。 允许用户运行的容器数取决于主机 OS 版本和容器运行时使用的[隔离模式](../manage-containers/hyperv-container.md)，以及运行这些容器是为了开发/测试还是为了生产。

|主机 OS                                                         |进程隔离的容器的限制                   |Hyper-V 隔离的容器的限制                   |
|----------------------------------------------------------------|---------------------------------------------------|---------------------------------------------------|
|Windows Server Standard                                         |无限制                                          |2                                                  |
|Windows Server Datacenter                                       |无限制                                          |无限制                                          |
|Windows 10 专业版和企业版                                   |无限制（仅适用于测试或开发目的） |无限制（仅适用于测试或开发目的） |
|Windows 10 IoT 核心版和企业版                             |无限制*                                         |无限制*                                          |

Windows Server 容器映像使用情况通过读取该[版本](/windows-server/get-started-19/editions-comparison-19.md)支持的虚拟化来宾数量来确定。 <br/>

>[!NOTE]
>\*IoT 版 Windows 上容器的生产使用情况取决于你是否已同意适用于 Windows 10 核心版运行时映像的 Microsoft 商业使用条款或 Windows 10 IoT 企业版设备许可证（“Windows IoT 商业协议”）。 Windows IoT 商业协议中的其他条款和限制适用于在生产环境中使用容器映像的情况。 请阅读[容器映像 EULA](../images-eula.md)，确切了解允许的内容和不允许的内容。

## <a name="as-a-developer-do-i-have-to-rewrite-my-app-for-each-type-of-container"></a>作为开发人员，我是否必须针对每种类型的容器重新编写我的应用？

否。 Windows 容器映像通用于 Windows Server 容器和 Hyper-V 隔离。 在你启动容器时选择容器类型。 从开发人员的角度看，Windows Server 容器和 Hyper-V 隔离是同一事物的两种风格。 它们提供相同的开发、编程和管理体验，它们是开放且可扩展的，并通过 Docker 提供相同级别的集成和支持。

开发人员可以使用 Windows Server 容器创建容器映像并通过 Hyper-V 隔离的方式对其进行部署，反之亦然，无需进行任何更改，只需指定相应的运行时标志即可。

在速度很重要的情况下，Windows Server 容器可提供更高的密度和性能。例如，与嵌套配置相比，可以缩短启动时间，提高运行时性能。 Hyper-V 隔离正如其名称所暗示的那样，可以提供更好的隔离，确保在一个容器中运行的代码无法危害或影响主机操作系统或在同一个主机上运行的其他容器。 这适用于要求托管不受信任代码的多租户方案（包括 SaaS 应用程序和计算托管）。

## <a name="can-i-run-windows-containers-in-process-isolated-mode-on-windows-10"></a>能否在 Windows 10 上以进程隔离模式运行 Windows 容器？

从 Windows 10 2018 年 10 月版更新开始，可以运行实施了进程隔离的 Windows 容器，但在使用 `--isolation=process` 运行容器时，必须首先使用 `docker run` 标志直接请求进程隔离。 进程隔离在 Windows 10 专业版、Windows 10 企业版、Windows 10 IoT 核心版和 Windows 10 IoT 企业版上是兼容的。

如果要以这种方式运行 Windows 容器，则需确保主机运行的是 Windows 10 版本 17763+，并且你有一个引擎为 18.09 或更高版本的 Docker 版本。

> [!WARNING]
> 在 IoT 核心版和 IoT 企业版主机上，在接受其他条款和限制之后即可使用此功能，但除此之外，此功能仅适用于开发和测试。 应该继续使用 Windows Server 作为进行生产部署的主机。 使用此功能，还必须确保主机和容器版本标记匹配，否则容器可能无法启动或出现未定义的行为。

## <a name="how-do-i-make-my-container-images-available-on-air-gapped-machines"></a>如何使我的容器映像可以在气隙计算机上使用？

Windows 容器基础映像包含其分发受许可证限制的项目。 在这些映像上进行构建并将其推送到专用或公共注册表时，你会注意到，我们从不推送基础层， 而是使用“外部层”这样一个概念，让其指向位于 Azure 云存储中的真正基础层。

如果气隙计算机只能从专用容器注册表的地址拉取映像，这可能会使事情复杂化。 在这种情况下，尝试沿外部层获取基础映像会失败。 若要重写外部层行为，可以在 Docker 守护程序中使用 `--allow-nondistributable-artifacts` 标志。

> [!IMPORTANT]
> 使用此标志不会妨碍你根据义务遵守 Windows 容器基础映像许可证的条款；你不得发布可以通过公共方式或第三方方式再次分发的 Windows 内容。 可以在你自己的环境中使用。

## <a name="additional-feedback"></a>其他反馈

想要向常见问题解答部分添加内容？ 请在评论部分提交新的反馈问题，或者通过 GitHub 针对此页设置一个拉取请求。
