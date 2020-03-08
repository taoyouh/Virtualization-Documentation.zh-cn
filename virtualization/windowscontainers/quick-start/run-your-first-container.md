---
title: Windows 10 上的 windows 和 Linux 容器
description: 容器部署快速入门
keywords: docker，容器，LCOW
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 551d405d836cfb16b587ef78bc2d5f5abbd8648f
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/06/2020
ms.locfileid: "78853811"
---
# <a name="get-started-run-your-first-windows-container"></a>入门：运行第一个 Windows 容器

本主题介绍如何在设置环境后（如[入门：为容器准备 Windows](./set-up-environment.md)中所述）运行第一个 Windows 容器。 若要运行容器，请首先安装基本映像，该映像为容器提供基本的操作系统服务层。 然后，创建并运行基于基本映像的容器映像。 有关详细信息，请参阅。

## <a name="install-a-container-base-image"></a>安装容器基本映像

所有容器都从容器映像创建。 Microsoft 提供了几个称为基本映像的 starter 映像（有关详细信息，请参阅[容器基本映像](../manage-containers/container-base-images.md)）。 此过程会拉取（下载和安装）轻型 Nano Server 基本映像。

1. 打开 "命令提示符" 窗口（例如内置命令提示符、PowerShell 或[Windows 终端](https://www.microsoft.com/p/windows-terminal-preview/9n0dx20hk701?activetab=pivot:overviewtab)），然后运行以下命令下载并安装基本映像：

   ```console
   docker pull mcr.microsoft.com/windows/nanoserver:1903
   ```

   > [!TIP]
   > 如果看到错误消息 "`no matching manifest for unknown in the manifest list entries`"，请确保 Docker 未配置为运行 Linux 容器。

2. 下载完成后，请在等待时阅读[EULA](../images-eula.md) ，并通过查询本地 docker 映像存储库来验证是否存在系统。 运行命令 `docker images` 返回已安装的映像的列表。

   下面是显示 Nano Server 映像的输出示例。

   ```console
   REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
   microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
   ```

## <a name="run-a-windows-container"></a>运行 Windows 容器

对于这个简单的示例，将创建并部署 "Hello World" 容器映像。 为了获得最佳体验，请在已提升权限的命令提示符窗口中运行这些命令（但不要使用 Windows PowerShell ISE），它不适用于与容器交互会话，因为容器看似挂起）。

1. 在命令提示符窗口中输入以下命令，从 `nanoserver` 映像启动包含交互式会话的容器：

   ```console
   docker run -it mcr.microsoft.com/windows/nanoserver:1903 cmd.exe
   ```
2. 启动容器后，命令提示符窗口会将上下文更改为容器。 在容器中，我们将创建一个简单的 "Hello World" 文本文件，然后通过输入以下命令退出容器：

   ```cmd
   echo "Hello World!" > Hello.txt
   exit
   ```   

3. 通过运行[docker ps](https://docs.docker.com/engine/reference/commandline/ps/)命令获取刚刚退出的容器的容器 ID：

   ```console
   docker ps -a
   ```

4. 创建新的 "HelloWorld" 映像，其中包含所运行的第一个容器中的更改。 为此，请运行[docker commit](https://docs.docker.com/engine/reference/commandline/commit/)命令，将 `<containerid>` 替换为容器的 ID：

   ```console
   docker commit <containerid> helloworld
   ```

   完成后，现在你就具有一个包含“hello world”脚本的自定义映像了。 可以通过[docker images](https://docs.docker.com/engine/reference/commandline/images/)命令查看。

   ```console
   docker images
   ```

   下面是输出示例：

   ```console
   REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
   helloworld                             latest              a1064f2ec798        10 seconds ago      258MB
   mcr.microsoft.com/windows/nanoserver   1903                2b9c381d0911        3 weeks ago         256MB
   ```

5. 最后，使用带有 `--rm` 参数的[docker run](https://docs.docker.com/engine/reference/commandline/run/)命令运行新容器，该参数在命令行（cmd.exe）停止后自动删除容器。

   ```console
   docker run --rm helloworld cmd.exe /s /c type Hello.txt
   ```

   结果是从 "HelloWorld" 映像创建了一个容器，cmd.exe 的实例是在读取文件并将文件内容输出到 shell 的容器中启动的，然后该容器停止并被删除。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [了解如何容器化示例应用](./building-sample-app.md)
