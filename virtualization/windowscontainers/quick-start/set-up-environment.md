---
title: Windows 10 上的 Windows 和 Linux 容器
description: 设置适用于容器的 Windows 10 或 Windows Server，然后继续操作，运行第一个容器映像。
keywords: docker, 容器, LCOW
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 2c52dd96b3bf2402d41ec5b178af36521d00a649
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909557"
---
# <a name="get-started-prep-windows-for-containers"></a>入门：准备适用于容器的 Windows

本教程介绍如何执行以下操作：

- 设置适用于容器的 Windows 10 或 Windows Server
- 运行第一个容器映像
- 容器化简单的 .NET Core 应用程序

## <a name="prerequisites"></a>必备条件

<!-- start tab view -->
# <a name="windows-server"></a>[Windows Server](#tab/Windows-Server)

若要在 Windows Server 上运行容器，需要一台运行 Windows Server（半年频道）、Windows Server 2019 或 Windows Server 2016 的物理服务器或虚拟机。

若要进行测试，可以下载 [Window Server 2019 评估版](https://www.microsoft.com/evalcenter/evaluate-windows-server-2019 )或 [Windows Server Insider Preview](https://insider.windows.com/for-business-getting-started-server/) 的副本。

# <a name="windows-10"></a>[Windows 10](#tab/Windows-10-Client)

若要在 Windows 10 上运行容器，需要以下各项：

- 一个运行 Windows 10 专业版或企业版（含周年更新（版本 1607）或更高版本）的物理计算机系统。
- [Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) 应已启用。

> [!NOTE]
>  从 Windows 10 的 2018 年 10 月更新版开始，我们不再允许用户在 Windows 10 企业版或专业版上以进程隔离模式运行 Windows 容器进行开发/测试。 有关详细信息，请参阅[常见问题解答](../about/faq.md)。 
> 
> Windows Server 容器在 Windows 10 上默认使用 Hyper-V 隔离，为开发人员提供在生产中使用的相同内核版本和配置。 有关 Hyper-V 隔离的详细信息，请参阅文档的[概念](../manage-containers/hyperv-container.md)部分。

---
<!-- stop tab view -->

## <a name="install-docker"></a>安装 Docker

第一步是安装 Docker，这是使用 Windows 容器所必需的。 Docker 为容器提供标准的运行时环境，该环境具有通用的 API 和命令行接口 (CLI)。

如需更多配置详细信息，请参阅 [Windows 上的 Docker 引擎](../manage-docker/configure-docker-daemon.md)。

<!-- start tab view -->
# <a name="windows-server"></a>[Windows Server](#tab/Windows-Server)

若要在 Windows Server 上安装 Docker，可以使用由 Microsoft 发布的 [OneGet 提供程序 PowerShell 模块](https://github.com/oneget/oneget)（称为 [DockerMicrosoftProvider](https://github.com/OneGet/MicrosoftDockerProvider)）。 此提供程序启用 Windows 中的容器功能，并安装 Docker 引擎和客户端。 以下是操作方法：

1. 打开提升的 PowerShell 会话，从 [PowerShell 库](https://www.powershellgallery.com/packages/DockerMsftProvider)安装 Docker-Microsoft PackageManagement 提供程序。

   ```powershell
   Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
   ```

   如果系统提示安装 NuGet 提供程序，还请键入 `Y` 进行安装。

2. 使用 PackageManagement PowerShell 模块安装最新版本的 Docker。

   ```powershell
   Install-Package -Name docker -ProviderName DockerMsftProvider
   ```

   PowerShell 询问是否信任包源“DockerDefault”时，键入 `A` 以继续进行安装。
3. 在安装完成后，请重启计算机。

   ```powershell
   Restart-Computer -Force
   ```

如果希望稍后更新 Docker，请执行以下操作：

- 使用 `Get-Package -Name Docker -ProviderName DockerMsftProvider` 查看已安装的版本
- 使用 `Find-Package -Name Docker -ProviderName DockerMsftProvider` 查找当前版本
- 准备就绪后，使用 `Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force` 进行升级，随后执行 `Start-Service Docker`

# <a name="windows-10"></a>[Windows 10](#tab/Windows-10-Client)

可以使用以下步骤在 Windows 10 专业版和企业版上安装 Docker。 

1. 下载并安装 [Docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows)，创建免费的 Docker 帐户（如果还没有该帐户）。 如需更多详细信息，请参阅 [Docker 文档](https://docs.docker.com/docker-for-windows/install)。

2. 在安装过程中，将默认容器类型设置为 Windows 容器。 若要在安装完成后进行切换，可以在 Windows 系统任务栏中使用 Docker 项（如下所示），也可以在 PowerShell 提示符下使用以下命令：

   ```console
   & $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
   ```

![Docker 系统任务栏菜单，显示“切换到 Windows 容器”命令。](./media/docker-for-win-switch.png)

---
<!-- stop tab view -->

## <a name="next-steps"></a>后续步骤

正确配置环境后，可以单击链接，了解如何运行容器。

> [!div class="nextstepaction"]
> [运行第一个容器](./run-your-first-container.md)
