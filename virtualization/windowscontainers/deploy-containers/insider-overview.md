---
title: 将容器与 Windows 预览体验计划配合使用
description: 了解如何开始在 windows 预览体验计划中使用 windows 容器
keywords: docker、容器、预览体验成员、windows
author: cwilhit
ms.openlocfilehash: 92fb359df1c207b848fb985caf7f46852f6b4f90
ms.sourcegitcommit: 6080b2c5053720490d374f6fb0daa870d5ddd4e8
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/25/2019
ms.locfileid: "10257791"
---
# <a name="use-containers-with-the-windows-insider-program"></a>将容器与 Windows 预览体验计划配合使用

此练习将向你说明如何在 Windows Insider Preview 计划的最新 Windows Server 预览体验成员版本上部署和使用 Windows 容器功能。 在此练习中，你将安装容器角色并部署基本操作系统映像的预览版本。 如果你需要熟悉容器，可在[关于容器](../about/index.md)中找到此信息。

## <a name="join-the-windows-insider-program"></a>加入 Windows 预览体验计划

若要运行 Windows 容器的预览体验计划版本，你必须拥有运行 windows 预览体验计划的最新 Windows Server 内部版本的主机和/或 Windows 预览体验计划中最新版本的 Windows 10。 加入[Windows 预览体验计划](https://insider.windows.com/GettingStarted)，并查看使用条款。

> [!IMPORTANT]
> 您必须使用 windows Server 预览体验计划中的 Windows Server 内部版本或 windows 10 版本的 windows 预览体验计划预览程序，以使用下面介绍的基本图像。 如果你没有使用这些版本中的其中一个，使用这些基本映像将导致启动容器失败。

## <a name="install-docker"></a>安装 Docker

如果尚未安装 Docker，请按照[入门](../quick-start/set-up-environment.md)指南安装 docker。

## <a name="pull-an-insider-container-image"></a>提取预览体验成员容器图像

通过成为 Windows 预览体验计划的一部分，你可以对基本映像使用我们的最新版本。

若要拉取 Nano Server 预览体验成员基本映像，请运行以下内容：

```console
docker pull mcr.microsoft.com/nanoserver/insider
```

若要拉取 Windows Server Core 预览体验成员基本映像，请运行以下内容：

```console
docker pull mcr.microsoft.com/windows/servercore/insider
```

"Windows" 和 "IoTCore" 基本图像也具有可供提取的预览体验成员版本。 有关[容器基本图像](../manage-containers/container-base-images.md)文档中可用的预览体验计划的详细信息，请参阅。

> [!IMPORTANT]
> 请阅读 Windows 容器操作系统映像[EULA](../images-eula.md )和 Windows 预览体验计划[使用条款](https://www.microsoft.com/software-download/windowsinsiderpreviewserver)。
