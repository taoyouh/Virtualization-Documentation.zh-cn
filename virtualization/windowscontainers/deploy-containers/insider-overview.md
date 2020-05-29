---
title: 将容器与 Windows 预览体验计划配合使用
description: 了解如何开始将 Windows 容器与 Windows 预览体验计划配合使用
keywords: docker, 容器, 预览体验, windows
author: cwilhit
ms.openlocfilehash: 92fb359df1c207b848fb985caf7f46852f6b4f90
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909887"
---
# <a name="use-containers-with-the-windows-insider-program"></a>将容器与 Windows 预览体验计划配合使用

此练习将向你说明如何在 Windows Insider Preview 计划的最新 Windows Server 预览体验成员版本上部署和使用 Windows 容器功能。 在此练习中，你将安装容器角色并部署基本操作系统映像的预览版本。 如果你需要熟悉容器，可在[关于容器](../about/index.md)中找到此信息。

## <a name="join-the-windows-insider-program"></a>加入 Windows 预览体验计划

若要运行 Windows 容器的预览体验版本，你必须有一台运行 Windows 预览体验计划中最新 Windows Server 版本和/或 Windows 预览体验计划中最新 Windows 10 版本的主机。 加入 [Windows 预览体验计划](https://insider.windows.com/GettingStarted)并查看使用条款。

> [!IMPORTANT]
> 你必须使用 Windows Server Insider Preview 计划中的 Windows Server 版本或 Windows Insider Preview 计划中的 Windows 10 版本才能使用如下所述的基础映像。 如果你没有使用这些版本中的其中一个，使用这些基本映像将导致启动容器失败。

## <a name="install-docker"></a>安装 Docker

如果尚未安装 Docker，请按照[入门](../quick-start/set-up-environment.md)指南来安装 Docker。

## <a name="pull-an-insider-container-image"></a>拉取预览体验版容器映像

加入 Windows 预览体验成员计划后，你可以使用我们的最新版本的基础映像。

若要拉取 Nano Server 预览体验成员基本映像，请运行以下内容：

```console
docker pull mcr.microsoft.com/nanoserver/insider
```

若要拉取 Windows Server Core 预览体验成员基本映像，请运行以下内容：

```console
docker pull mcr.microsoft.com/windows/servercore/insider
```

“Windows”和“IoTCore”基础映像也具有可供拉取的预览体验版本。 可以在[容器基础映像](../manage-containers/container-base-images.md)文档中详细了解可用的预览体验版基础映像。

> [!IMPORTANT]
> 请阅读 Windows 容器操作系统映像 [EULA](../images-eula.md ) 和 Windows 预览体验计划[使用条款](https://www.microsoft.com/software-download/windowsinsiderpreviewserver)。
