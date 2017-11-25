---
title: "容器部署快速入门 - 映像"
description: "容器部署快速入门"
keywords: "docker, 容器"
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 479e05b1-2642-47c7-9db4-d2a23592d29f
ms.openlocfilehash: 58bd491ccae7cd9d91088d9f180367623320345e
ms.sourcegitcommit: 456485f36ed2d412cd708aed671d5a917b934bbe
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/08/2017
---
# <a name="automating-builds-and-saving-images"></a>自动生成和保存映像

在之前的 Windows Server 快速入门中，已从预先创建的 .Net Core 示例创建了 Windows 容器。 本练习中将详细介绍手动创建自定义容器映像、使用 Dockerfile 自动创建容器映像以及将容器映像存储在 Docker Hub 公共注册表中。

本快速入门仅适用于 Windows Server 2016 上的 Windows Server 容器且会使用 Windows Server Core 容器基本映像。 此页面左侧的目录中提供其他快速入门文档。

**先决条件：**

- 一个运行 Windows Server 2016 的计算机系统（物理或虚拟）。
- 使用 Windows 容器功能和 Docker 配置此系统。 有关这些步骤的演练，请参阅 [Windows Server 上的 Windows 容器](./quick-start-windows-server.md)。
- 一个用于将容器映像推送到 Docker Hub 的 Docker ID。 如果还没有 Docker ID，请在 [Docker 云](https://cloud.docker.com/)中进行注册。

## <a name="1-container-image---manual"></a>1.容器映像 - 手动

为获得最佳体验，请从 Windows 命令行界面 (cmd.exe) 演练此练习。

手动创建容器映像的第一步是部署容器。 对于此示例，请从预创建的 IIS 映像部署 IIS 容器。 部署容器后，你将从该容器内部进入 shell 会话。 使用 `-it` 标识初始交互式会话。 有关 Docker Run 命令的详细信息，请参阅 [Docker.com 上的 Docker Run 参考](https://docs.docker.com/engine/reference/run/)。 

> 取决于 Windows Server Core 基本映像的大小，此步骤可能需要一些时间。

```
docker run -d --name myIIS -p 80:80 microsoft/iis
```

现在，容器将在后台运行。 该容器包含的默认命令 `ServiceMonitor.exe` 会监视 IIS 进度，并在 IIS 停止时自动停止容器。 若要了解如何创建该映像的详细信息，请参阅 GitHub 上的 [Microsoft docker-iis](https://github.com/Microsoft/iis-docker)。

接下来，在容器中启动交互式 cmd。 这将允许你在运行的容器中运行命令，而无需停止 IIS 或 ServiceMonitor。

```
docker exec -i myIIS cmd 
```

接下来，你可以对运行中的容器进行更改。 运行以下命令来删除 IIS 初始屏幕。

```
del C:\inetpub\wwwroot\iisstart.htm
```

以下内容用来将默认 IIS 站点替换为新的静态站点。

```
echo "Hello World From a Windows Server Container" > C:\inetpub\wwwroot\index.html
```

从不同的系统，浏览到容器主机的 IP 地址。 你应该可以看到“Hello World”应用程序。

**注意：**如果你使用的是 Azure，将需要一个网络安全组规则来允许通过端口 80 通信。 有关详细信息，请参阅 [Create Rule in a Network Security Group](https://azure.microsoft.com/en-us/documentation/articles/virtual-networks-create-nsg-arm-pportal/#create-rules-in-an-existing-nsg)（在网络安全组中创建规则）。

![](media/hello.png)

返回到容器中，退出交互式容器会话。

```
exit
```

现在可以将此修改的容器捕获到新的容器映像中。 为此，你将需要容器名称。 使用 `docker ps -a` 命令可查找到该名称。

```
docker ps -a

CONTAINER ID     IMAGE                             COMMAND   CREATED             STATUS   PORTS   NAMES
489b0b447949     microsoft/iis   "cmd"     About an hour ago   Exited           pedantic_lichterman
```

若要创建新的容器映像，请使用 `docker commit` 命令。 Docker commit 采用“docker commit container-name new-image-name”的形式。 注意 - 将此示例中的容器名称替换为实际容器名称。

```
docker commit pedantic_lichterman modified-iis
```

若要验证是否已创建了新的映像，请使用 `docker images` 命令。  

```
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
modified-iis        latest              3e4fdb6ed3bc        About a minute ago   10.17 GB
microsoft/iis       windowsservercore   c26f4ceb81db        2 weeks ago          9.48 GB
windowsservercore   10.0.14300.1000     dbfee88ee9fd        8 weeks ago          9.344 GB
windowsservercore   latest              dbfee88ee9fd        8 weeks ago          9.344 GB
```

现在可以部署此映像了。 生成的容器中将包括所有捕获的修改。

## <a name="2-container-image---dockerfile"></a>2.容器映像 - Dockerfile

通过上一练习，已手动创建和修改容器，并已将其捕获到新容器映像中。 Docker 包含使用 Dockerfile 自动执行此过程的方法。 本练习将产生与上一练习几乎相同的结果，但是这次该过程将自动执行。 此练习需要 Docker ID。 如果还没有 Docker ID，请在 [Docker 云]( https://cloud.docker.com/)中进行注册。

在容器主机上，创建目录 `c:\build`，并在此目录中创建一个名为 `Dockerfile` 的文件。 注意 - 该文件不应具有文件扩展名。

```
powershell new-item c:\build\Dockerfile -Force
```

使用记事本打开 Dockerfile。

```
notepad c:\build\Dockerfile
```

将以下文本复制到 Dockerfile 并保存该文件。 这些命令指示 Docker 使用 `microsoft/iis` 为基本，创建新的映像。 然后 dockerfile 会运行在 `RUN` 指示中指定的命令，在本例中，已使用新内容更新了 index.html 文件。 

有关 Dockerfile 的详细信息，请参阅 [Windows 上的 Dockerfile](../manage-docker/manage-windows-dockerfile.md)。

```
FROM microsoft/iis
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
```

`docker build` 命令将启动映像生成过程。 `-t` 参数会指示此生成过程将新映像命名为 `iis-dockerfile`。 **使用 Docker 帐户的用户名替换“用户”**。 如果还没有 Docker 帐户，请在 [Docker 云](https://cloud.docker.com/)中进行注册。

```
docker build -t <user>/iis-dockerfile c:\Build
```

完成后，你可以验证是否已使用 `docker images` 命令创建了映像。

```
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
iis-dockerfile      latest              8d1ab4e7e48e        2 seconds ago       9.483 GB
microsoft/iis       windowsservercore   c26f4ceb81db        2 weeks ago         9.48 GB
windowsservercore   10.0.14300.1000     dbfee88ee9fd        8 weeks ago         9.344 GB
windowsservercore   latest              dbfee88ee9fd        8 weeks ago         9.344 GB
```

现在，使用以下命令部署容器，并再次使用 Docker ID 替换“用户”。

```
docker run -d -p 80:80 <user>/iis-dockerfile ping -t localhost
```

创建容器后，浏览到容器主机的 IP 地址。 你应该可以看到 Hello World 应用程序。

![](media/dockerfile2.png)

返回到容器主机上，使用 `docker ps` 来获取容器的名称，然后使用 `docker rm` 来删除容器。 注意 - 将此示例中的容器名称替换为实际容器名称。

获取容器名称。

```
docker ps

CONTAINER ID   IMAGE            COMMAND               CREATED              STATUS              PORTS                NAMES
c1dc6c1387b9   iis-dockerfile   "ping -t localhost"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp   cranky_brown
```

删除容器。

```
docker rm -f <container name>
```

## <a name="3-docker-push"></a>3.Docker 推送

Docker 容器映像可以存储在容器注册表中。 图像存储在注册表中后，可以跨多个不同的容器主机检索以供将来使用。 Docker 在 [Docker Hub](https://hub.docker.com/) 中提供公共注册表来存储容器映像。

在此练习中，会将自定义 hello world 映像推送到你在 Docker Hub 上的帐户中。

首先，使用 `docker login command` 登录到 docker 帐户。

```
docker login

Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.

Username: user
Password: Password

Login Succeeded
```

登录后，可将容器映像推送到 Docker Hub。 为此，请使用 `docker push` 命令。 **将“用户”替换为 Docker ID**。 

```
docker push <user>/iis-dockerfile
```

现在可以使用 `docker pull` 将容器映像从 Docker Hub 下载到任意 Windows 容器主机。 在本教程中，我们将删除现有映像，并将它从 Docker Hub 向下拉取。 

```
docker rmi <user>/iis-dockerfile
```

运行 `docker images` 将显示映像已删除。

```
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

[Windows 10 上的 Windows 容器](./quick-start-windows-10.md)
