---
title: Windows 10 上的 windows 和 Linux 容器
description: 容器部署快速入门
keywords: docker、容器、LCOW
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: a664b5b8eb87adffdf7eba3ffca9f4194128df80
ms.sourcegitcommit: e61db4d98d9476a622e6cc8877650d9e7a6b4dd9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/13/2019
ms.locfileid: "10288125"
---
# <a name="get-started-run-your-first-windows-container"></a>入门：运行你的第一个 Windows 容器

本主题介绍了如何在设置你的环境之后运行你的第一个 Windows 容器，如下所述[： "准备 Windows for 容器](./set-up-environment.md)"。 若要运行容器，首先需要安装基本映像，它为你的容器提供了一种基本的操作系统服务层。 然后，创建并运行基于基本映像的容器映像。 有关详细信息，请参阅。

## <a name="install-a-container-base-image"></a>安装容器基映像

从容器图像创建所有容器。 Microsoft 提供了一些称为基本映像的 starter 映像（有关更多详细信息，请参阅[容器基础图像](../manage-containers/container-base-images.md)）。 此过程将提取（下载和安装）轻型 Nano Server 基本映像。

1. 打开命令提示符窗口（如内置命令提示符、PowerShell 或[Windows 终端](https://www.microsoft.com/p/windows-terminal-preview/9n0dx20hk701?activetab=pivot:overviewtab)），然后运行以下命令下载并安装基本映像：

   ```console
   docker pull mcr.microsoft.com/windows/nanoserver:1903
   ```

   > [!TIP]
   > 如果看到一条错误消息`no matching manifest for unknown in the manifest list entries`，请确保 Docker 未配置为运行 Linux 容器。

2. 图像下载完成后，请在等待时阅读[EULA](../images-eula.md) -通过查询本地 docker 图像存储库来验证系统中是否存在该文件。 运行该命令`docker images`将返回已安装映像的列表。

   下面是显示 Nano Server 映像的输出示例。

   ```console
   REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
   microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
   ```

## <a name="run-a-windows-container"></a>运行 Windows 容器

对于此简单示例，将创建并部署 "Hello World" 容器图像。 为获得最佳体验，请在提升的命令提示符窗口中运行这些命令（但不要使用 Windows PowerShell ISE），因为容器似乎挂起，因此它不适用于容器的交互式会话。

1. 通过在命令提示符窗口中输入以下命令`nanoserver` ，从映像启动包含交互式会话的容器：

   ```console
   docker run -it mcr.microsoft.com/windows/nanoserver:1903 cmd.exe
   ```
2. 启动容器后，命令提示符窗口会将上下文更改为容器。 在容器内，我们将创建一个简单的 "Hello World" 文本文件，然后通过输入以下命令退出容器：

   ```cmd
   echo "Hello World!" > Hello.txt
   exit
   ```   

3. 通过运行[docker ps](https://docs.docker.com/engine/reference/commandline/ps/)命令获取刚刚退出的容器的容器 ID：

   ```console
   docker ps -a
   ```

4. 创建一个新的 "HelloWorld" 图像，其中包含你运行的第一个容器中的更改。 若要执行此操作，请运行[docker commit](https://docs.docker.com/engine/reference/commandline/commit/)命令`<containerid>` ，并将其替换为容器的 ID：

   ```console
   docker commit <containerid> helloworld
   ```

   完成后，现在你就具有一个包含“hello world”脚本的自定义映像了。 可通过[docker 图像](https://docs.docker.com/engine/reference/commandline/images/)命令看到此操作。

   ```console
   docker images
   ```

   下面是输出示例：

   ```console
   REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
   helloworld                             latest              a1064f2ec798        10 seconds ago      258MB
   mcr.microsoft.com/windows/nanoserver   1903                2b9c381d0911        3 weeks ago         256MB
   ```

5. 最后，通过在命令行（cmd.exe）停止时通过使用[docker 运行](https://docs.docker.com/engine/reference/commandline/run/)命令和自动删除容器的`--rm`参数运行新容器。

   ```console
   docker run --rm helloworld cmd.exe /s /c type Hello.txt
   ```

   结果是，容器是通过 "HelloWorld" 图像创建的，在读取文件并将文件内容输出到外壳的容器中启动了 cmd.exe 的实例，并且容器已停止且已被删除。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [了解如何 containerize 示例应用](./building-sample-app.md)
