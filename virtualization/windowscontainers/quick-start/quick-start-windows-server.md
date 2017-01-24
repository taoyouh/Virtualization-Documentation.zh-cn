---
title: "Windows Server 上的 Windows 容器"
description: "容器部署快速入门"
keywords: "docker, 容器"
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
translationtype: Human Translation
ms.sourcegitcommit: 54eff4bb74ac9f4dc870d6046654bf918eac9bb5
ms.openlocfilehash: 31a55ead81e05f7dee1d1f4f8a2101114d1ca017

---

# Windows Server 上的 Windows 容器

本练习将演练 Windows Server 2016 上 Windows 容器功能的基本部署和使用。 在本练习中，你将安装容器角色并部署简单的 Windows Server 容器。 在开始本快速入门之前，请先熟悉基本容器概念和术语。 可以在[快速入门简介](./index.md)上找到此信息。

本快速入门特定于 Windows Server 2016 上的 Windows Server 容器。 此页面左侧的目录中提供其他快速入门文档，包括 Windows 10 中的容器。

**先决条件：**

一个运行 Windows Server 2016 的计算机系统（物理或虚拟）。 如果使用的是 Windows Server 2016 TP5，请更新为 [Window Server 2016 Evaluation](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016 )。 

> 需要安装关键更新，才能让 Windows 容器功能正常运作。 请在进行本教程所述操作前安装所有更新。

若要在 Azure 上部署，可使用此[模板](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-server-container-tools/containers-azure-template)轻松进行部署。<br/>
<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVirtualization-Documentation%2Fmaster%2Fwindows-server-container-tools%2Fcontainers-azure-template%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>


## 1.安装 Docker

安装 Docker 将用到 [OneGet 提供程序 PowerShell 模块](https://github.com/oneget/oneget)。 该提供程序将在计算机上启用容器功能。 还将安装 Docker，它要求重新启动。 若要使用 Window 容器，则需要安装 Docker。 其中包括 Docker 引擎和 Docker 客户端。

打开提升的 PowerShell 会话并运行下列命令。

首先，安装 OneGet PowerShell 模块。

```none
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

接下来使用 OneGet 安装最新版的 Docker。
```none
Install-Package -Name docker -ProviderName DockerMsftProvider
```

PowerShell 询问是否信任包源“DockerDefault”时，键入 A 继续进行安装。 完成安装后，重启计算机。

```none
Restart-Computer -Force
```

## 2.安装 Windows 更新

运行以下命令，确保 Windows Server 系统保持最新状态：

```none
sconfig
```

将出现一个文本配置菜单，可以选择其中的选项 6 下载并安装更新：

```none
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

## 3.部署第一个容器

对于此练习，你将从 Docker Hub 注册表下载预先创建的 .NET 示例映像，并部署运行 .Net Hello World 应用程序的简单容器。  

使用 `docker run` 部署 .Net 容器。 这也可下载容器映像，可能需要几分钟时间。

```console
docker run microsoft/dotnet-samples:dotnetapp-nanoserver
```

容器启动后，请打印 hello world 消息，然后退出。

```console
         Dotnet-bot: Welcome to using .NET Core!
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
Platform: .NET Core 1.0
OS: Microsoft Windows 10.0.14393
```

有关 Docker Run 命令的深入信息，请参阅 [Docker.com 上的 Docker Run 参考]( https://docs.docker.com/engine/reference/run/)。

## 后续步骤

[Windows Server 上的容器映像](./quick-start-images.md)

[Windows 10 上的 Windows 容器](./quick-start-windows-10.md)



<!--HONumber=Jan17_HO4-->


