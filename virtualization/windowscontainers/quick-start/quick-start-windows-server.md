---
title: Windows Server 上的 Windows 容器
description: 容器部署快速入门
keywords: docker, 容器
author: cwilhit
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
ms.openlocfilehash: df0470fd44c823fe14411d6d3d29188e980d1780
ms.sourcegitcommit: 34d8b2ca5eebcbdb6958560b1f4250763bee5b48
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "9621195"
---
# <a name="windows-containers-on-windows-server"></a>Windows Server 上的 Windows 容器

此练习将演练基本部署和使用 Windows 容器功能在 Windows Server 2019 和 Windows Server 2016 上。

在本快速入门中，你将完成：

1. 启用 Windows Server 中的容器功能
2. 安装 Docker
3. 运行简单的 Windows 容器

如果你需要熟悉容器，可在[关于容器](../about/index.md)中找到此信息。

本快速入门仅适用于 Windows Server 2019 和 Windows Server 2016 上的 Windows Server 容器。 此页面左侧的目录中提供其他快速入门文档，包括 Windows 10 中的容器。

## <a name="prerequisites"></a>系统必备

请确保你满足以下要求：
- 运行 Windows Server 2019 一台计算机系统 （物理或虚拟）。 如果你使用的 Windows Server 2019 Insider Preview，请更新[Window Server 2019 Evaluation](https://www.microsoft.com/evalcenter/evaluate-windows-server-2019 )。

> 为了让 Windows 容器功能正常运作需要安装关键更新。 请在进行本教程所述操作前安装所有更新。

若要在 Azure 上部署，可使用此[模板](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-server-container-tools/containers-azure-template)轻松进行部署。

<br/>
<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVirtualization-Documentation%2Flive%2Fwindows-server-container-tools%2Fcontainers-azure-template%2Fazuredeploy.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>


## <a name="install-docker"></a>安装 Docker

若要安装 Docker，我们将使用的适用于提供程序，以在此情况下执行安装- [MicrosoftDockerProvider](https://github.com/OneGet/MicrosoftDockerProvider) [OneGet 提供程序 PowerShell 模块](https://github.com/oneget/oneget)。 该提供程序将在计算机上启用容器功能。 还将安装 Docker，它要求重新启动。 若要使用 Window 容器，则需要安装 Docker。 其中包括 Docker 引擎和 Docker 客户端。

打开提升的 PowerShell 会话并运行下列命令。

首先，从 [PowerShell 库](https://www.powershellgallery.com/packages/DockerMsftProvider)安装 Docker-Microsoft PackageManagement 提供程序。

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

接下来，使用 PackageManagement PowerShell 模块安装最新版本的 Docker。

```powershell
Install-Package -Name docker -ProviderName DockerMsftProvider
```

PowerShell 询问是否信任包源“DockerDefault”时，键入 `A` 以继续进行安装。 完成安装后，重启计算机。

```powershell
Restart-Computer -Force
```

> [!TIP]
> 如果你希望稍后更新 Docker:
>  - 查看已安装的版本，查看时使用 `Get-Package -Name Docker -ProviderName DockerMsftProvider`
>  - 查找当前版本，查找时使用 `Find-Package -Name Docker -ProviderName DockerMsftProvider`
>  - 当你准备就绪后，进行升级，升级时使用 `Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force`，后跟 `Start-Service Docker`

## <a name="install-windows-updates"></a>安装 Windows 更新

运行以下命令，确保 Windows Server 系统保持最新状态：

```console
sconfig
```

将出现一个文本配置菜单，可以选择其中的选项 6 下载并安装更新：

```console
===============================================================================
                         Server Configuration
===============================================================================

1) Domain/Workgroup:                    Workgroup:  WORKGROUP
2) Computer Name:                       WIN-HEFDK4V68M5
3) Add Local Administrator
4) Configure Remote Management          Enabled

5) Windows Update Settings:             DownloadOnly
6) Download and Install Updates
7) Remote Desktop:                      Disabled
...
```

出现提示时，选择选项 A 下载所有更新。

## <a name="deploy-your-first-container"></a>部署你的第一个容器

对于此练习，你将从 Docker Hub 注册表下载预先创建的 .NET 示例映像，并部署运行 .Net Hello World 应用程序的简单容器。  

使用 `docker run` 部署 .Net 容器。 这也可下载容器映像，可能需要几分钟时间。 具体取决于你的 Windows Server，运行以下命令下面的主机版本。

#### <a name="windows-server-2019"></a>Windows Server 2019

```console
docker run microsoft/dotnet-samples:dotnetapp-nanoserver-1809
```

#### <a name="windows-server-2016"></a>Windows Server 2016

```console
docker run microsoft/dotnet-samples:dotnetapp-nanoserver-sac2016
```

容器启动后，请打印 hello world 消息，然后退出。

```console
         Hello from .NET Core!
    __________________
                      \
                       \
                          ....
                          ....'
                           ....
                        ..........
                    .............'..'..
                 ................'..'.....
               .......'..........'..'..'....
              ........'..........'..'..'.....
             .'....'..'..........'..'.......'.
             .'..................'...   ......
             .  ......'.........         .....
             .                           ......
            ..    .            ..        ......
           ....       .                 .......
           ......  .......          ............
            ................  ......................
            ........................'................
           ......................'..'......    .......
        .........................'..'.....       .......
     ........    ..'.............'..'....      ..........
   ..'..'...      ...............'.......      ..........
  ...'......     ...... ..........  ......         .......
 ...........   .......              ........        ......
.......        '...'.'.              '.'.'.'         ....
.......       .....'..               ..'.....
   ..       ..........               ..'........
          ............               ..............
         .............               '..............
        ...........'..              .'.'............
       ...............              .'.'.............
      .............'..               ..'..'...........
      ...............                 .'..............
       .........                        ..............
        .....


**Environment**
Platform: .NET Core
OS: Microsoft Windows 10.0.17763
```

有关 Docker Run 命令的深入信息，请参阅 [Docker.com 上的 Docker Run 参考](https://docs.docker.com/engine/reference/run/)。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [了解如何自动执行容器生成和保存图像](./quick-start-images.md)
