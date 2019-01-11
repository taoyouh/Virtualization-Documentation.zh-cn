---
title: 容器部署快速入门 - 映像
description: 容器部署快速入门
keywords: docker, 容器
author: cwilhit
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 479e05b1-2642-47c7-9db4-d2a23592d29f
ms.openlocfilehash: 0350e62deef06402991f505dd263db7fd506cba1
ms.sourcegitcommit: 1aef193cf56dd0870139b5b8f901a8d9808ebdcd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/11/2019
ms.locfileid: "9001583"
---
# <a name="automating-builds-and-saving-images"></a>自动生成和保存映像

在之前的 Windows Server 快速入门中，已从预先创建的 .Net Core 示例创建了 Windows 容器。 本练习将详细介绍如何使用 Dockerfile 自动创建容器映像以及将容器映像存储在 Docker Hub 公共注册表中。

本快速入门仅适用于 Windows Server 2016 上的 Windows Server 容器且会使用 Windows Server Core 容器基本映像。 此页面左侧的目录中提供其他快速入门文档。

## <a name="prerequisites"></a>先决条件

请确保你满足以下要求：

- 一个运行 Windows Server 2016 的计算机系统（物理或虚拟）。
- 使用 Windows 容器功能和 Docker 配置此系统。 有关这些步骤的演练，请参阅 [Windows Server 上的 Windows 容器](./quick-start-windows-server.md)。
- 一个用于将容器映像推送到 Docker Hub 的 Docker ID。 如果还没有 Docker ID，请在 [Docker 云](https://cloud.docker.com/)中进行注册。

## <a name="container-image---dockerfile"></a>容器映像-Dockerfile

尽管可以手动创建和修改容器，然后将其捕获到新容器镜像中，但是 Docker 还包含一种使用 Dockerfile 自动执行此过程的方法。 此练习需要 Docker ID。 如果还没有 Docker ID，请在 [Docker 云]( https://cloud.docker.com/)中进行注册。

在容器主机上，创建目录 `c:\build`，并在此目录中创建一个名为 `Dockerfile` 的文件。 注意 - 该文件不应具有文件扩展名。

```console
powershell new-item c:\build\Dockerfile -Force
```

使用记事本打开 Dockerfile。

```console
notepad c:\build\Dockerfile
```

将以下文本复制到 Dockerfile 并保存该文件。 这些命令指示 Docker 使用 `microsoft/iis` 为基本，创建新的映像。 然后 dockerfile 会运行在 `RUN` 指示中指定的命令，在本例中，已使用新内容更新了 index.html 文件。

有关 Dockerfile 的详细信息，请参阅 [Windows 上的 Dockerfile](../manage-docker/manage-windows-dockerfile.md)。

```dockerfile
FROM microsoft/iis
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
```

`docker build` 命令将启动映像生成过程。 `-t` 参数会指示此生成过程将新映像命名为 `iis-dockerfile`。 **使用 Docker 帐户的用户名替换“用户”**。 如果还没有 Docker 帐户，请在 [Docker 云](https://cloud.docker.com/)中进行注册。

```console
docker build -t <user>/iis-dockerfile c:\Build
```

完成后，你可以验证是否已使用 `docker images` 命令创建了映像。

```console
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
iis-dockerfile      latest              8d1ab4e7e48e        2 seconds ago       9.483 GB
microsoft/iis       windowsservercore   c26f4ceb81db        2 weeks ago         9.48 GB
windowsservercore   10.0.14300.1000     dbfee88ee9fd        8 weeks ago         9.344 GB
windowsservercore   latest              dbfee88ee9fd        8 weeks ago         9.344 GB
```

现在，使用以下命令部署容器，并再次使用 Docker ID 替换“用户”。

```console
docker run -d -p 80:80 <user>/iis-dockerfile ping -t localhost
```

创建容器后，浏览到容器主机的 IP 地址。 你应该可以看到 Hello World 应用程序。

![](media/dockerfile2.png)

返回到容器主机上，使用 `docker ps` 来获取容器的名称，然后使用 `docker rm` 来删除容器。 注意 - 将此示例中的容器名称替换为实际容器名称。

获取容器名称。

```console
docker ps

CONTAINER ID   IMAGE            COMMAND               CREATED              STATUS              PORTS                NAMES
c1dc6c1387b9   iis-dockerfile   "ping -t localhost"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp   cranky_brown
```

停止容器。

```console
docker stop <container name>
```

删除容器。

```console
docker rm -f <container name>
```

## <a name="docker-push"></a>Docker 推送

Docker 容器映像可以存储在容器注册表中。 图像存储在注册表中后，可以跨多个不同的容器主机检索以供将来使用。 Docker 在 [Docker Hub](https://hub.docker.com/) 中提供公共注册表来存储容器映像。

在此练习中，会将自定义 hello world 映像推送到你在 Docker Hub 上的帐户中。

首先，使用 `docker login command` 登录到 docker 帐户。

```console
docker login

Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.

Username: user
Password: Password

Login Succeeded
```

登录后，可将容器映像推送到 Docker Hub。 为此，请使用 `docker push` 命令。 **将“用户”替换为 Docker ID**。 

```console
docker push <user>/iis-dockerfile
```

为 Docker 推送到 Docker Hub 每一层，docker 将跳过已存在于在 Docker Hub，或其他注册表 （外层） 的层。  例如，将跳过，并不推送到 Docker Hub 上托管在 Microsoft 容器注册表中或从专用的公司注册，层中的最新版本的 Windows Server Core。

现在可以使用 `docker pull` 将容器映像从 Docker Hub 下载到任意 Windows 容器主机。 在本教程中，我们将删除现有映像，并将它从 Docker Hub 向下拉取。 

```console
docker rmi <user>/iis-dockerfile
```

运行 `docker images` 将显示映像已删除。

```console
docker images

REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
modified-iis              latest              51f1fe8470b3        5 minutes ago       7.69 GB
microsoft/iis             latest              e4525dda8206        3 hours ago         7.61 GB
```

最后，docker 拉取可用于将映像拉回容器主机。 使用 Docker 帐户的用户名替换“用户”。 

```
docker pull <user>/iis-dockerfile
```

## <a name="next-steps"></a>后续步骤

如果要了解如何打包示例 ASP.NET 应用程序，请访问下方链接中的 Windows 10 教程。

> [!div class="nextstepaction"]
> [Windows 10 上的容器](./quick-start-windows-10.md)
