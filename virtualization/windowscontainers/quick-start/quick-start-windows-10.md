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
ms.openlocfilehash: dc500a7b6c0f8f078820407e6ed80ca5868bf4f3
ms.sourcegitcommit: 95cec99aa8e817d3e3cb2163bd62a32d9e8f7181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/18/2018
ms.locfileid: "8973647"
---
# <a name="windows-containers-on-windows-10"></a>Windows 10 上的 Windows 容器

> [!div class="op_single_selector"]
> - [Windows 上的 Linux 容器](quick-start-windows-10-linux.md)
> - [在 Windows 上的 Windows 容器](quick-start-windows-10.md)

本练习将演练创建并运行 Windows 10 上的 Windows 容器。

在本快速入门中，你将完成：

1. 安装适用于 Windows 的 Docker
2. 运行一个简单的 Windows 容器

本快速入门特定于 Windows 10。 可以在此页面左侧的目录中找到其他快速入门文档。

## <a name="prerequisites"></a>先决条件
请确保你满足以下要求：
- 一个运行 Windows 10 专业版或企业版周年更新 （版本 1607年） 或更高版本的物理计算机系统。 
- 请确保启用[HYPER-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) 。

***HYPER-V 隔离：*** Windows Server 容器要求对 Windows 10 使用 HYPER-V 隔离才能为开发人员提供的相同内核版本和配置时，将在生产中使用，更多有关 HYPER-V 隔离可以找到[关于 Windows 容器](../about/index.md)页面上。

> [!NOTE]
> 在 Windows 更新 2018 年 10 月版本中，我们不会再禁止用户从用于开发人员/测试目的，Windows 10 企业版或专业版上运行 Windows 容器进程隔离模式。 请参阅[常见问题](../about/faq.md)若要了解详细信息。

## <a name="install-docker-for-windows"></a>安装适用于 Windows 的 Docker

[Docker for Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows)下载并运行安装程序 （你将需要登录。 创建一个帐户如果你已经没有）。 Docker 文档中提供了[详细的安装说明](https://docs.docker.com/docker-for-windows/install)。

## <a name="switch-to-windows-containers"></a>切换到 Windows 容器

安装后，适用于 Windows 的 Docker 默认为运行 Linux 容器。 切换到 Windows 容器使用 Docker 任务栏菜单或通过在 PowerShell 中运行以下命令提示符下：

```console
& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
```

![](./media/docker-for-win-switch.png)

## <a name="install-base-container-images"></a>安装基本容器映像

Windows 容器是从基本映像中构建的。 以下命令将拉取 Nano Server 基本映像。

```console
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

拉取映像后，运行 `docker images` 将返回已安装的映像的列表，此例中为 Nano Server 映像。

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> [!IMPORTANT]
> 请阅读 Windows 容器操作系统映像[EULA](../images-eula.md)。

## <a name="run-your-first-windows-container"></a>运行你的第一个 Windows 容器

对于此简单示例，将创建 Hello World 容器映像的大小和部署。 为获得最佳体验，请在升级后的Windows CMD shell 或 PowerShell中运行这些命令。

> Windows PowerShell ISE 不适用于与容器的交互式会话。 即使容器正在运行，也会显示为挂起。

首先，从 `nanoserver` 映像启动一个具有交互式会话的容器。 启动容器后，容器中将显示一个命令行界面。  

```console
docker run -it mcr.microsoft.com/windows/nanoserver:1809 cmd.exe
```

在容器内，我们将创建一个简单的 Hello World 文本文件。

```cmd
echo "Hello World!" > Hello.txt
```   

完成后，退出该容器。

```cmd
exit
```

现在从修改后的容器创建一个新的容器映像。 若要查看容器列表，请运行以下项并记住容器 ID。

```console
docker ps -a
```

运行以下命令以创建 HelloWorld 映像。 将 <containerid> 替换为你的容器 ID。

```console
docker commit <containerid> helloworld
```

完成后，现在你就具有一个包含“hello world”脚本的自定义映像了。 通过执行以下命令可以看到该映像。

```console
docker images
```

最后，要运行该容器，请使用 `docker run` 命令。

```console
docker run --rm helloworld cmd.exe /s /c type Hello.txt
```

结果的`docker run`命令是从 HelloWorld 映像创建 HYPER-V 容器、 cmd 实例已启动容器中，并执行我们文件 （输出回显到该界面），然后停止并删除容器的读数。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [了解如何生成示例应用](./building-sample-app.md)