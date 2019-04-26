---
title: 适用于 Docker 的 PowerShell
description: 如何使用 PowerShell 管理 Docker 容器
keywords: docker, 容器, powershell
author: PatrickLang
ms.date: 12/19/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 4a0e907d-0d07-42f8-8203-2593391071da
ms.openlocfilehash: 71d91e12ae843bf96e1b4001915ffb55bb352ffd
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/26/2019
ms.locfileid: "9576848"
---
### <a name="powershell-for-docker"></a>适用于 Docker 的 PowerShell

通过我们与你、与来自论坛、Twitter、GitHub 的用户的对话甚至当面交流，我们发现有一个问题出现的频率远远大于其他任何问题 - 即，为什么我在 PowerShell 中看不到 Docker 容器？ 

在我们与你们讨论其利、弊以及各种选择时，我们得出了一个结论，即容器 PowerShell 模块需要更新... 因此我们将弃用已在 Windows Server 2016 的预览版本中发行的容器 PowerShell 模块，并且已经开始着手将它替换为新的适用于 Docker 的模块。  此新模块的开发正在进行中，但我们采用了与过去不同的方式 - 我们将以开放的形式进行此工作。  我们对此模块的目标是将其打造成为一款社区协作产品，通过 Docker 引擎带来绝佳的 PowerShell 容器体验。  此新模块会直接构建于 Docker 引擎的 REST 接口顶层，使用户能够在 Docker CLI 和 PowerShell 之间进行选择，或同时选择两者。

构建出色的 PowerShell 模块绝非易事，要确保正确编码，还要保持对象、参数集和 cmdlet 名称之间的正确平衡，这些都非常重要。  因此，当我们着手开发此新模块时，我们希望你们 - 我们的最终用户和广大的 PowerShell 和 Docker 社区可以帮助塑造此模块。  哪些参数集对你来说很重要？  我们是否应采用等效于“docker run”的方法还是你应通过管道将 new-container 传递到 start-container - 你想要使用哪种...  了解有关此模块的详细信息和参与开发请 head 通过到我们的 GitHub 页面 (https://github.com/Microsoft/Docker-PowerShell/)并加入。

随着开发的进行，若我们取得了稳定的 alpha 质量模块，我们会将其发布在 PowerShell 库上，并且会在此页面上更新有关其使用方法的说明。
