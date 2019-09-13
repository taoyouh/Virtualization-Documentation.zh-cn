---
title: Windows 10 上的 windows 和 Linux 容器
description: 容器部署快速入门
keywords: docker、容器、LCOW
author: cwilhit
ms.date: 09/11/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 5f0922a1ee2588b6e5a06091fe34e07ceadf89cb
ms.sourcegitcommit: 868a64eb97c6ff06bada8403c6179185bf96675f
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2019
ms.locfileid: "10129359"
---
# <a name="get-started-configure-your-environment-for-containers"></a>入门：为容器配置你的环境

此快速入门演示了如何：

> [!div class="checklist"]
> * 设置容器的环境
> * 运行第一个容器图像
> * Containerize 简单的 .NET core 应用程序

## <a name="prerequisites"></a>系统必备

<!-- start tab view -->
# [<a name="windows-server"></a>Windows Server](#tab/Windows-Server)

请确保满足以下要求：

- 运行 Windows Server 2016 或更高版本的一台计算机系统（物理或虚拟）。

> [!NOTE]
> 如果你使用的是 Windows Server 2019 预览体验计划预览版，请更新到[Window server 2019 评估](https://www.microsoft.com/evalcenter/evaluate-windows-server-2019 )。

# [<a name="windows-10-professional-and-enterprise"></a>Windows 10 专业版和企业版](#tab/Windows-10-Client)

请确保满足以下要求：

- 运行 Windows 10 专业版或企业版的一台物理计算机系统（版本1607）或更高版本。
- 应启用[hyper-v](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) 。

> [!NOTE]
>  从 Windows 10 年10月更新2018开始，我们不再允许用户在 Windows 10 企业版或专业版（适用于开发/测试目的）的进程隔离模式下运行 Windows 容器。 请参阅[常见问题](../about/faq.md)了解详细信息。 
> 
> Windows Server 容器默认情况下在 Windows 10 上使用 Hyper-v 隔离，以便为开发人员提供将在生产中使用的相同内核版本和配置。 在我们的文档的 "[概念](../manage-containers/hyperv-container.md)" 区域中了解有关 hyper-v 隔离的详细信息。

---
<!-- stop tab view -->

## <a name="install-docker"></a>安装 Docker

Docker 是用于处理 Windows 容器的权威 native。 Docker 可为用户提供用于管理给定主机、生成容器、删除容器等的容器的 CLI。 了解有关我们的文档的[概念](../manage-containers/configure-docker-daemon.md)领域的 Docker 的详细信息。

<!-- start tab view -->
# [<a name="windows-server"></a>Windows Server](#tab/Windows-Server)

在 Windows Server 上，Docker 是通过 Microsoft 发布的[OneGet 提供商 PowerShell 模块](https://github.com/oneget/oneget)安装的，该模块由 Microsoft 名为[DockerMicrosoftProvider](https://github.com/OneGet/MicrosoftDockerProvider)。 此提供商：

- 在你的计算机上启用容器功能
- 在你的计算机上安装 Docker 引擎和客户端。

若要安装 Docker，请打开一个提升的 PowerShell 会话，然后从[PowerShell 库](https://www.powershellgallery.com/packages/DockerMsftProvider)中安装 Docker-Microsoft PackageManagement 提供商。

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

接下来，使用 PackageManagement PowerShell 模块安装最新版本的 Docker。

```powershell
Install-Package -Name docker -ProviderName DockerMsftProvider
```

PowerShell 询问是否信任包源“DockerDefault”时，键入 `A` 以继续进行安装。 安装完成后，必须重新启动计算机。

```powershell
Restart-Computer -Force
```

> [!TIP]
> 如果想要在以后更新 Docker：
>  - 查看已安装的版本，查看时使用 `Get-Package -Name Docker -ProviderName DockerMsftProvider`
>  - 查找当前版本，查找时使用 `Find-Package -Name Docker -ProviderName DockerMsftProvider`
>  - 当你准备就绪后，进行升级，升级时使用 `Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force`，后跟 `Start-Service Docker`

# [<a name="windows-10-professional-and-enterprise"></a>Windows 10 专业版和企业版](#tab/Windows-10-Client)

在 Windows 10 专业版和企业版上，Docker 是通过经典安装程序安装的。 下载[Docker 桌面](https://store.docker.com/editions/community/docker-ce-desktop-windows)并运行安装程序。 您将需要登录。 如果尚未有帐户，请创建一个帐户。 [Docker 文档](https://docs.docker.com/docker-for-windows/install)中提供了更详细的安装说明。

安装后，Docker 桌面默认为运行 Linux 容器。 使用 Docker 任务栏菜单或通过在 PowerShell 提示符中运行以下命令，切换到 Windows 容器：

```console
& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
```

![](./media/docker-for-win-switch.png)

---
<!-- stop tab view -->

## <a name="next-steps"></a>后续步骤

现在已正确配置你的环境，请按照链接了解如何拉取和运行容器。

> [!div class="nextstepaction"]
> [运行第一个容器](./run-your-first-container.md)
