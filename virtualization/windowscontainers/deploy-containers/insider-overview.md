---
title: 将容器与 Windows 预览体验计划配合使用
description: 了解如何开始在 windows 预览体验计划中使用 windows 容器
keywords: docker、容器、预览体验成员、windows
author: cwilhit
ms.openlocfilehash: 137209a66c3d0b907003498fe78a04a57a140130
ms.sourcegitcommit: d0411b05d1ef7328a770766b84fd0743f9d9c237
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2019
ms.locfileid: "10254381"
---
# <a name="use-containers-with-the-windows-insider-program"></a>将容器与 Windows 预览体验计划配合使用

此练习将向你说明如何在 Windows Insider Preview 计划的最新 Windows Server 预览体验成员版本上部署和使用 Windows 容器功能。 在此练习中，你将安装容器角色并部署基本操作系统映像的预览版本。 如果你需要熟悉容器，可在[关于容器](../about/index.md)中找到此信息。

> [!NOTE]
> 此内容特定于 Windows Server 预览体验计划中的 Windows Server 容器。 如果您正在寻找使用 Windows 容器的非预览体验说明，请参阅[入门](../quick-start/set-up-environment.md)指南。

## <a name="join-the-windows-insider-program"></a>加入 Windows 预览体验计划

若要运行 Windows 容器的预览体验计划版本，你必须拥有运行 windows 预览体验计划的最新 Windows Server 内部版本的主机和/或 Windows 预览体验计划中最新版本的 Windows 10。 加入[Windows 预览体验计划](https://insider.windows.com/GettingStarted)，并查看使用条款。

> [!IMPORTANT]
> 您必须使用 windows Server 预览体验计划中的 Windows Server 内部版本或 windows 10 版本的 windows 预览体验计划预览程序，以使用下面介绍的基本图像。 如果你没有使用这些版本中的其中一个，使用这些基本映像将导致启动容器失败。

## <a name="install-docker"></a>安装 Docker

<!-- start tab view -->
# [<a name="windows-server-insider"></a>Windows Server 预览体验成员](#tab/Windows-Server-Insider)

安装 Docker EE 将用到 OneGet 提供程序 PowerShell 模块。 提供程序将启用计算机上的容器功能，并安装 Docker EE - 此操作需要重启计算机。 打开提升的 PowerShell 会话并运行下列命令。

> [!NOTE]
> 在 Windows Server 预览体验计划内部版本中安装 Docker EE 需要不同于用于非预览体验成员版本的 OneGet 提供程序。 如果已安装 Docker EE 和 DockerMsftProvider OneGet 提供程序，请在继续操作前先将它们移除。

```powershell
Stop-Service docker
Uninstall-Package docker
Uninstall-Module DockerMsftProvider
```

安装适用于 Windows 预览体验成员版本的 OneGet PowerShell 模块。

```powershell
Install-Module -Name DockerProvider -Repository PSGallery -Force
```

使用 OneGet 安装 Docker EE 预览版的最新版本。

```powershell
Install-Package -Name docker -ProviderName DockerProvider -RequiredVersion Preview
```

完成安装后，重启计算机。

```powershell
Restart-Computer -Force
```

# [<a name="windows-10-insider"></a>Windows 10 预览体验成员](#tab/Windows-10-Insider)

在 Windows 10 预览体验计划中，Docker 边缘通过与 Docker 桌面稳定的同一安装程序安装。 下载[Docker 桌面](https://store.docker.com/editions/community/docker-ce-desktop-windows)并运行安装程序。 您将需要登录。 如果尚未有帐户，请创建一个帐户。 [Docker 文档](https://docs.docker.com/docker-for-windows/install)中提供了更详细的安装说明。

安装后，打开 Docker 的设置并切换到 "边缘" 通道。

![](./media/docker-edge-instruction.png)

---
<!-- stop tab view -->

## <a name="pull-an-insider-container-image"></a>提取预览体验成员容器图像

使用 Windows 容器前，需安装基本映像。 通过成为 Windows 预览体验计划的一部分，你可以对基本映像使用我们的最新版本。 你可以在[容器基本图像](../manage-containers/container-base-images.md)文档中阅读有关可用的基本图像的详细信息。

若要拉取 Nano Server 预览体验成员基本映像，请运行以下内容：

```console
docker pull mcr.microsoft.com/nanoserver/insider
```

若要拉取 Windows Server Core 预览体验成员基本映像，请运行以下内容：

```console
docker pull mcr.microsoft.com/windows/servercore/insider
```

> [!IMPORTANT]
> 请阅读 Windows 容器操作系统映像[EULA](../images-eula.md )和 Windows 预览体验计划[使用条款](https://www.microsoft.com/software-download/windowsinsiderpreviewserver)。
