---
title: Windows 和 Windows 10 上的 Linux 容器
description: 容器部署快速入门
keywords: docker，容器 LCOW
author: taylorb-microsoft
ms.date: 11/8/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 4202908f8797a2b98ab657c45cd9a6b33191bd6f
ms.sourcegitcommit: 9dfef8d261f4650f47e8137a029e893ed4433a86
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/09/2018
ms.locfileid: "6224896"
---
# <a name="windows-and-linux-containers-on-windows-10"></a>Windows 和 Windows 10 上的 Linux 容器

本练习将演练创建并运行 Windows 10 上的 Windows 和 Linux 容器。 完成操作后，你将有：

1. 安装适用于 Windows 的 Docker
2. 运行的简单的 Windows 容器
3. 运行简单使用 Windows (LCOW) 的 Linux 容器的 Linux 容器

本快速入门特定于 Windows 10。 在此页面左侧的目录中，可以找到其他快速入门文档。

***HYPER-V 隔离：*** Windows Server 容器要求在 Windows 10 上的 HYPER-V 隔离才能为开发人员提供的相同内核版本和配置时，将在生产中使用，更多有关 HYPER-V 隔离可以找到[关于 Windows 容器](../about/index.md)页面上。

**先决条件：**

- 一个运行 Windows 10 Fall Creators Update （版本 1709年） 或更高版本 （专业版或企业）[可以运行 HYPER-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-requirements)的物理计算机系统

> 如果你不希望在本教程中运行 LCOW，Windows 容器将无法运行 Windows 10 周年更新 （版本 1607年） 或更高版本。

## <a name="1-install-docker-for-windows"></a>1. 安装适用于 Windows 的 Docker

[下载适用于 Windows 的 Docker](https://store.docker.com/editions/community/docker-ce-desktop-windows)和运行安装程序 （你将需要登录。 创建一个帐户如果你已经没有）。 Docker 文档中提供了[详细的安装说明](https://docs.docker.com/docker-for-windows/install)。

> 如果你已经安装 Docker，请确保你有 18.02 或更高版本来支持 LCOW 版本。 检查通过运行`docker -v`或检查*有关 Docker*。

> 在实验性功能选项*Docker 设置 > 守护程序*必须激活运行 LCOW 容器。

## <a name="2-switch-to-windows-containers"></a>2. 切换到 Windows 容器

安装后，适用于 Windows 的 Docker 默认为运行 Linux 容器。 通过使用 Docker 任务栏菜单或通过在 PowerShell 提示符下运行以下命令来切换到 Windows 容器：`& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon`。

![](./media/docker-for-win-switch.png)
> 请注意，Windows 容器模式允许 Windows 容器除了 LCOW 容器。

## <a name="3-install-base-container-images"></a>3. 安装基本容器映像

Windows 容器是从基本映像中构建的。 以下命令将拉取 Nano Server 基本映像。

```
docker pull microsoft/nanoserver
```

拉取映像后，运行 `docker images` 将返回已安装的映像的列表，此例中为 Nano Server 映像。

```
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> 可在此处 ([EULA](../images-eula.md)) 阅读 Windows 容器操作系统映像 EULA。

## <a name="4-run-your-first-windows-container"></a>4.运行你的第一个 Windows 容器

对于此简单示例，将创建和部署一个“Hello World”容器映像。 为获得最佳体验，请在升级后的Windows CMD shell 或 PowerShell中运行这些命令。
> Windows PowerShell ISE 不适用于与容器的交互式会话。 即使容器正在运行，也会显示为挂起。

首先，从 `nanoserver` 映像启动一个具有交互式会话的容器。 启动容器后，容器中将显示一个命令行界面。  

```
docker run -it microsoft/nanoserver cmd
```

在容器中创建一个简单的“Hello World”脚本。

```
powershell.exe Add-Content C:\helloworld.ps1 'Write-Host "Hello World"'
```   

完成后，退出该容器。

```
exit
```

现在从修改后的容器创建一个新的容器映像。 若要查看容器列表，请运行以下项并记住容器 ID。

```
docker ps -a
```

运行以下命令以创建 HelloWorld 映像。 将 <containerid> 替换为你的容器 ID。

```
docker commit <containerid> helloworld
```

完成后，现在你就具有一个包含“hello world”脚本的自定义映像了。 通过执行以下命令可以看到该映像。

```
docker images
```

最后，要运行该容器，请使用 `docker run` 命令。

```
docker run --rm helloworld powershell c:\helloworld.ps1
```

`docker run` 命令的结果是，从 HelloWorld 映像创建 Hyper-V 容器，然后执行一个“Hello World”脚本（输出回显到该界面），然后停止并删除容器。
Windows 10 和容器快速入门的后续部分将深入探讨在 Windows 10 上的容器中创建和部署应用程序。

## <a name="run-your-first-lcow-container"></a>运行你的第一个 LCOW 容器

对于此示例中，将部署 BusyBox 容器。 首先，尝试运行 Hello World BusyBox 映像。

```
docker run --rm busybox echo hello_world
```

请注意，这可以返回错误，当 Docker 尝试拉取映像。 发生这种情况是因为 Dockers 需要通过指令`--platform`标志来确认图像和主机操作系统进行相应地匹配。 由于 Windows 容器模式中的默认平台，Windows 将添加`--platform linux`标志来拉取和运行该容器。

```
docker run --rm --platform linux busybox echo hello_world
```

后已通过图像提取与平台所示，`--platform`标志不再需要。 运行而无需它的命令对此进行测试。

```
docker run --rm busybox echo hello_world
```

运行`docker images`返回的已安装的映像的列表。 在此情况下，Windows 和 Linux 映像。

```
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

## <a name="next-steps"></a>后续步骤

额外功能： 在运行 LCOW 上看到 Docker 的相应[博客文章](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/)

继续学习下一个教程以查看[生成示例应用](./building-sample-app.md)的示例
