---
title: Windows 10 上的 windows 和 Linux 容器
description: 为容器设置 Windows 10 或 Windows Server，然后继续运行第一个容器图像。
keywords: docker、容器、LCOW
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 2c52dd96b3bf2402d41ec5b178af36521d00a649
ms.sourcegitcommit: e61db4d98d9476a622e6cc8877650d9e7a6b4dd9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/13/2019
ms.locfileid: "10288115"
---
# <a name="get-started-prep-windows-for-containers"></a>入门：为容器准备窗口

本教程介绍了如何：

- 设置 Windows 10 或 Windows Server for 容器
- 运行第一个容器图像
- Containerize 简单的 .NET core 应用程序

## <a name="prerequisites"></a>系统必备

<!-- start tab view -->
# [<a name="windows-server"></a>Windows Server](#tab/Windows-Server)

若要在 Windows Server 上运行容器，需要物理服务器或运行 Windows Server （半年频道）、Windows Server 2019 或 Windows Server 2016 的虚拟机。

对于测试，你可以下载[Window server 2019 评估版](https://www.microsoft.com/evalcenter/evaluate-windows-server-2019 )或[Windows Server 预览体验计划预览版](https://insider.windows.com/for-business-getting-started-server/)的副本。

# [<a name="windows-10"></a>Windows10](#tab/Windows-10-Client)

若要在 Windows 10 上运行容器，你需要以下各项：

- 运行 Windows 10 专业版或企业版的一台物理计算机系统（版本1607）或更高版本。
- 应启用[hyper-v](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) 。

> [!NOTE]
>  从 Windows 10 年10月更新2018开始，我们不再允许用户在 Windows 10 企业版或专业版（适用于开发/测试目的）的进程隔离模式下运行 Windows 容器。 请参阅[常见问题](../about/faq.md)了解详细信息。 
> 
> Windows Server 容器默认情况下在 Windows 10 上使用 Hyper-v 隔离，以便为开发人员提供将在生产中使用的相同内核版本和配置。 在我们的文档的 "[概念](../manage-containers/hyperv-container.md)" 区域中了解有关 hyper-v 隔离的详细信息。

---
<!-- stop tab view -->

## <a name="install-docker"></a>安装 Docker

第一步是安装 Docker，它是使用 Windows 容器所必需的。 Docker 通过一个通用 API 和命令行界面（CLI）为容器提供标准的运行时环境。

有关详细配置的详细信息，请参阅[Windows 上的 Docker 引擎](../manage-docker/configure-docker-daemon.md)。

<!-- start tab view -->
# [<a name="windows-server"></a>Windows Server](#tab/Windows-Server)

若要在 Windows Server 上安装 Docker，你可以使用 Microsoft 发布的[OneGet 提供商 PowerShell 模块](https://github.com/oneget/oneget)，称为[DockerMicrosoftProvider](https://github.com/OneGet/MicrosoftDockerProvider)。 此提供程序支持 Windows 中的容器功能并安装 Docker 引擎和客户端。 操作方法如下：

1. 打开一个提升了权限的 PowerShell 会话，并从[PowerShell 库](https://www.powershellgallery.com/packages/DockerMsftProvider)安装 Docker-Microsoft PackageManagement 提供商。

   ```powershell
   Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
   ```

   如果系统提示你安装 NuGet 提供程序，请键入`Y`进行安装。

2. 使用 PackageManagement PowerShell 模块安装最新版本的 Docker。

   ```powershell
   Install-Package -Name docker -ProviderName DockerMsftProvider
   ```

   PowerShell 询问是否信任包源“DockerDefault”时，键入 `A` 以继续进行安装。
3. 安装完成后，重新启动计算机。

   ```powershell
   Restart-Computer -Force
   ```

如果想要在以后更新 Docker：

- 查看已安装的版本，查看时使用 `Get-Package -Name Docker -ProviderName DockerMsftProvider`
- 查找当前版本，查找时使用 `Find-Package -Name Docker -ProviderName DockerMsftProvider`
- 当你准备就绪后，进行升级，升级时使用 `Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force`，后跟 `Start-Service Docker`

# [<a name="windows-10"></a>Windows10](#tab/Windows-10-Client)

你可以通过使用以下步骤在 Windows 10 专业版和企业版版上安装 Docker。 

1. 下载并安装[Docker 桌面](https://store.docker.com/editions/community/docker-ce-desktop-windows)，如果尚无空闲的 docker 帐户，则创建一个。 有关更多详细信息，请参阅[Docker 文档](https://docs.docker.com/docker-for-windows/install)。

2. 在安装过程中，将默认容器类型设置为 Windows 容器。 若要在安装完成后进行切换，你可以使用 Windows 系统任务栏中的 Docker 项目（如下所示），或者在 PowerShell 提示符中使用以下命令：

   ```console
   & $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
   ```

![显示 "切换到 Windows 容器" 命令的 Docker 系统托盘菜单。](./media/docker-for-win-switch.png)

---
<!-- stop tab view -->

## <a name="next-steps"></a>后续步骤

现在已正确配置你的环境，请按照链接了解如何运行容器。

> [!div class="nextstepaction"]
> [运行第一个容器](./run-your-first-container.md)
