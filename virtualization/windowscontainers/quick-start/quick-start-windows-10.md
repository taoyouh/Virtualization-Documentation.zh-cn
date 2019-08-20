---
title: Windows 10 上的 windows 和 Linux 容器
description: 容器部署快速入门
keywords: docker、容器、LCOW
author: taylorb-microsoft
ms.date: 08/16/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: aa7a20d914fdb65597c0f31ef6d53b91f6497be4
ms.sourcegitcommit: 2f8fd4b2e7113fbb7c323d89f3c72df5e1a4437e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/20/2019
ms.locfileid: "10044957"
---
# <a name="windows-containers-on-windows-10"></a>Windows 10 上的 Windows 容器

该练习将指导你在 Windows 10 上创建和运行 Windows 容器。

在此快速入门中, 你将完成以下操作:

1. 安装 Docker 桌面
2. 运行简单的 Windows 容器

本快速入门特定于 Windows 10。 可在此页面左侧的目录中找到其他快速入门文档。

## <a name="prerequisites"></a>系统必备
请确保满足以下要求:
- 运行 Windows 10 专业版或企业版的一台物理计算机系统 (版本 1607) 或更高版本。 
- 请确保已启用[hyper-v](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) 。

***Hyper-v 隔离:*** Windows Server 容器在 Windows 10 上需要 Hyper-v 隔离, 以便为开发人员提供将在生产中使用的相同内核版本和配置, 可以在 "[关于 Windows 容器](../about/index.md)" 页面上找到有关 hyper-v 隔离的更多信息。

> [!NOTE]
> 在 Windows 10 月更新2018的版本中, 我们不再允许用户在 Windows 10 企业版或专业版 (适用于开发/测试目的) 的进程隔离模式下运行 Windows 容器。 请参阅[常见问题](../about/faq.md)了解详细信息。

## <a name="install-docker-desktop"></a>安装 Docker 桌面

下载[Docker 桌面](https://store.docker.com/editions/community/docker-ce-desktop-windows)并运行安装程序 (您将需要登录。 如果尚未有帐户, 请创建一个帐户。 Docker 文档中提供了[详细的安装说明](https://docs.docker.com/docker-for-windows/install)。

## <a name="switch-to-windows-containers"></a>切换到 Windows 容器

安装 Docker 桌面默认为运行 Linux 容器。 使用 Docker 任务栏菜单或通过在 PowerShell 提示符中运行以下命令, 切换到 Windows 容器:

```console
& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
```

![](./media/docker-for-win-switch.png)

## <a name="install-base-container-images"></a>安装基本容器映像

Windows 容器是从基本映像中构建的。 以下命令将拉取 Nano Server 基本映像。

```console
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

> [!NOTE]
> 如果看到一条错误消息`no matching manifest for unknown in the manifest list entries`, 请确保你不希望提取 Linux 容器。

拉取映像后，运行 `docker images` 将返回已安装的映像的列表，此例中为 Nano Server 映像。

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> [!IMPORTANT]
> 请阅读 Windows 容器操作系统映像[EULA](../images-eula.md)。

## <a name="run-your-first-windows-container"></a>运行你的第一个 Windows 容器

对于此简单示例, 将创建并部署 "Hello World" 容器图像。 为获得最佳体验，请在升级后的Windows CMD shell 或 PowerShell中运行这些命令。

> Windows PowerShell ISE 不适用于与容器的交互式会话。 即使容器正在运行，也会显示为挂起。

首先，从 `nanoserver` 映像启动一个具有交互式会话的容器。 启动容器后，容器中将显示一个命令行界面。  

```console
docker run -it mcr.microsoft.com/windows/nanoserver:1809 cmd.exe
```

在容器内, 我们将创建一个简单的 "Hello World" 文本文件。

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

`docker run`命令的结果是, 在 hyper-v 隔离下运行的容器是从 "HelloWorld" 映像创建的, 在容器中启动了 cmd 的一个实例, 并执行了文件的读取 (输出回显到 shell), 然后执行容器已停止并已删除。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [了解如何构建示例应用](./building-sample-app.md)
