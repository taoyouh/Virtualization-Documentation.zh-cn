---
title: Windows 10 上的 windows 和 Linux 容器
description: 容器部署快速入门
keywords: docker、容器、LCOW
author: cwilhit
ms.date: 09/11/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 3d651a4a68acefa25f1b647b1b33618bbfb91ae9
ms.sourcegitcommit: 868a64eb97c6ff06bada8403c6179185bf96675f
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2019
ms.locfileid: "10129358"
---
# <a name="get-started-run-your-first-container"></a>入门：运行你的第一个容器

在[上一段](./set-up-environment.md)中，我们为运行容器配置了环境。 本练习将介绍如何提取容器图像并运行它。

## <a name="install-container-base-image"></a>安装容器基映像

将从`container images`实例化所有容器。 Microsoft 提供了多个 "初学者" 图像`base images`（称为），可供选择。 以下命令将拉取 Nano Server 基本映像。

```console
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

> [!TIP]
> 如果您看到一条错误消息`no matching manifest for unknown in the manifest list entries`，指出，请确保 Docker 未配置为运行 Linux 容器。

在拉入图像后，可以通过查询本地 docker 图像存储库来验证计算机上是否存在该图像。 运行此命令`docker images`将返回已安装映像的列表-在此情况下，为 Nano Server 映像。

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> [!IMPORTANT]
> 请阅读 Windows 容器操作系统映像[EULA](../images-eula.md)。

## <a name="run-your-first-windows-container"></a>运行你的第一个 Windows 容器

对于此简单示例，将创建并部署 "Hello World" 容器图像。 为获得最佳体验，请在提升的 Windows CMD shell 或 PowerShell 中运行这些命令。

> Windows PowerShell ISE 不适用于与容器的交互式会话。 即使容器正在运行，也会显示为挂起。

首先，从 `nanoserver` 映像启动一个具有交互式会话的容器。 启动容器后，将在容器内显示命令外壳。  

```console
docker run -it mcr.microsoft.com/windows/nanoserver:1809 cmd.exe
```

在容器内，我们将创建一个简单的 "Hello World" 文本文件。

```cmd
echo "Hello World!" > Hello.txt
```   

完成后，退出该容器。

```cmd
exit
```

从已修改的容器中创建新的容器图像。 若要查看正在运行或已退出的容器的列表，请运行以下并记录容器 id。

```console
docker ps -a
```

运行以下命令以创建 HelloWorld 映像。 将 `<containerid>` 替换为你的容器 ID。

```console
docker commit <containerid> helloworld
```

完成后，现在你就具有一个包含“hello world”脚本的自定义映像了。 通过执行以下命令可以看到该映像。

```console
docker images
```

最后，使用`docker run`命令运行容器。

```console
docker run --rm helloworld cmd.exe /s /c type Hello.txt
```

`docker run`命令的结果是，容器是从 "HelloWorld" 映像创建的，在容器中启动了 cmd 的一个实例，并执行了文件的读取（输出回显到 shell），然后停止并删除容器。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [了解如何 containerize 示例应用](./building-sample-app.md)
