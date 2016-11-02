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
ms.sourcegitcommit: ffdf89b0ae346197b9ae631ee5260e0565261c55
ms.openlocfilehash: 80a2efcb9fff58d357dc1630a335665fc91147cb

---

# Windows Server 上的 Windows 容器

本练习将演练 Windows Server 上 Windows 容器功能的基本部署和使用。 完成操作后，你将已安装容器角色，并部署简单的 Windows Server 容器。 在开始本快速入门之前，请先熟悉基本容器概念和术语。 可以在 [Quick Start Introduction](./quick_start.md)（快速入门简介）上找到此信息。

本快速入门特定于 Windows Server 2016 上的 Windows Server 容器。 此页面左侧的目录中提供其他快速入门文档。

**先决条件：**

一个运行 Windows Server 2016 的计算机系统（物理或虚拟）。 如果使用的是 Windows Server 2016 TP5，请更新为 [Window Server 2016 Evaluation](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016 )。 

> 需要安装关键更新，才能让 Windows 容器功能正常运作。 请在进行本教程所述操作前安装所有更新。

## 1.安装 Docker

安装 Docker 将用到 [OneGet 提供程序 PowerShell 模块](https://github.com/oneget/oneget)。 提供程序将启用计算机上的容器功能，并安装 Docker - 此操作需要重启计算机。 若要使用 Window 容器，则需要安装 Docker。 其中包括 Docker 引擎和 Docker 客户端。

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

可以通过运行以下项安装 Windows 更新，确保 Windows Server 系统为最新：

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

```none
docker run microsoft/sample-dotnet
```

容器将启动，请打印 hello world 消息，然后退出。

```none
       Welcome to .NET Core!
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
```

有关 Docker Run 命令的深入信息，请参阅 [Docker.com 上的 Docker Run 参考]( https://docs.docker.com/engine/reference/run/)。

## 后续步骤

[Windows Server 上的容器映像](./quick_start_images.md)

[Windows 10 上的 Windows 容器](./quick_start_windows_10.md)



<!--HONumber=Oct16_HO4-->


