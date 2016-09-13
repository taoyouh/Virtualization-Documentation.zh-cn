---
title: "Windows 容器映像"
description: "使用 Windows 容器创建和管理容器映像。"
keywords: "docker, 容器"
author: neilpeterson
manager: timlt
ms.date: 08/22/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: d8163185-9860-4ee4-9e96-17b40fb508bc
translationtype: Human Translation
ms.sourcegitcommit: eccce83d7e376be592694162f54ccb67be9d3c12
ms.openlocfilehash: bb99d0c15d6d1dd9e126fde05207431153b4f94a

---

# Windows 容器映像

**这是初步内容，可能还会更改。** 

>Windows 容器通过 Docker 进行管理。 Windows 容器文档是对可在 [docs.docker.com](https://docs.docker.com/) (docs.docker.com) 上查找到的文档的补充。

容器映像用于部署容器。 这些映像可以包括应用程序以及所有应用程序依赖关系。 例如，你可以开发已使用 Nano Server、IIS 以及在 IIS 中运行的应用程序预配置的容器映像。 然后，此容器映像可以存储在容器注册表中供以后使用，也可以部署在任何 Windows 容器主机上（可以部署在本地、云中，甚至可以部署到容器服务），还可以用作新容器映像的基础。

### 安装映像

使用 Windows 容器前，需安装基本映像。 可通过将 Windows Server Core 或 Nano Server 作为基础操作系统获取基本映像。 有关支持的配置方面的信息，请参阅 [Windows 容器系统要求](../deployment/system_requirements.md)(#windows-容器系统要求)。

若要安装 Windows Server Core 基本映像，请运行以下内容：

```none
docker pull microsoft/windowsservercore
```

若要安装 Nano Server 基本映像，请运行以下内容：

```none
docker pull microsoft/nanoserver
```

### 列出映像

```none
docker images

REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
microsoft/windowsservercore   latest              02cb7f65d61b        9 weeks ago         7.764 GB
microsoft/nanoserver          latest              3a703c6e97a2        9 weeks ago         969.8 MB
```

### 创建新映像

可以从现有容器创建新容器映像。 为此，请使用 `docker commit` 命令。 以下示例将创建一个名为“windowsservercoreiis”的新容器映像。

```none
docker commit 475059caef8f windowsservercoreiis
```

### 删除映像

如果有任何容器（即使处于停止状态）与映像存在依赖关系，则不能删除容器映像。

当使用 Docker 删除映像时，可以按映像名称或 ID 引用这些映像。

```none
docker rmi windowsservercoreiis
```

### 映像依赖关系

若要通过 Docker 查看映像依赖关系，可以使用 `docker history` 命令。

```none
docker history windowsservercoreiis

IMAGE               CREATED             CREATED BY          SIZE                COMMENT
2236b49aaaef        3 minutes ago       cmd                 171.2 MB
6801d964fda5        2 weeks ago                             0 B
```

### Docker Hub

Docker Hub 注册表包含预生成的映像，可将这些映像下载到容器主机上。 下载这些映像后，它们可用作 Windows 容器应用程序的基础。

若要查看 Docker Hub 中可用的映像列表，请使用 `docker search` 命令。 注意 - 在从 Docker Hub 中提取这些依赖的容器映像之前，需要安装 Windows Server Core 或 Nano Server 基本操作系统映像映像。

其中大多数映像拥有 Windows Server Core 和 Nano Server 版本。 若要获取特定版本，只需添加标记“:windowsservercore”或“:nanoserver”。 默认情况下，“最新”标记将返回 Windows Server Core 版本，除非仅有 Nano Server 版本可用。


```none
docker search *

NAME                     DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
microsoft/sample-django  Django installed in a Windows Server Core ...   1                    [OK]
microsoft/dotnet35       .NET 3.5 Runtime installed in a Windows Se...   1         [OK]       [OK]
microsoft/sample-golang  Go Programming Language installed in a Win...   1                    [OK]
microsoft/sample-httpd   Apache httpd installed in a Windows Server...   1                    [OK]
microsoft/iis            Internet Information Services (IIS) instal...   1         [OK]       [OK]
microsoft/sample-mongodb MongoDB installed in a Windows Server Core...   1                    [OK]
microsoft/sample-mysql   MySQL installed in a Windows Server Core b...   1                    [OK]
microsoft/sample-nginx   Nginx installed in a Windows Server Core b...   1                    [OK]
microsoft/sample-node    Node installed in a Windows Server Core ba...   1                    [OK]
microsoft/sample-python  Python installed in a Windows Server Core ...   1                    [OK]
microsoft/sample-rails   Ruby on Rails installed in a Windows Serve...   1                    [OK]
microsoft/sample-redis   Redis installed in a Windows Server Core b...   1                    [OK]
microsoft/sample-ruby    Ruby installed in a Windows Server Core ba...   1                    [OK]
microsoft/sample-sqlite  SQLite installed in a Windows Server Core ...   1                    [OK]
```

### Docker 请求

若要从 Docker Hub 下载映像，请使用 `docker pull`。 有关详细信息，请参阅 [Docker Pull on Docker.com](https://docs.docker.com/engine/reference/commandline/pull/)（Docker.com 上的 Docker 请求）。

```none
docker pull microsoft/aspnet

Using default tag: latest
latest: Pulling from microsoft/aspnet
f9e8a4cc8f6c: Pull complete

b71a5b8be5a2: Download complete
```

现在运行 `docker images` 时可以看到该映像。

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
microsoft/aspnet    latest              b3842ee505e5        5 hours ago         101.7 MB
windowsservercore   10.0.14300.1000     6801d964fda5        2 weeks ago         0 B
windowsservercore   latest              6801d964fda5        2 weeks ago         0 B
```

> 如果 Docker 请求失败，请确保最新累积更新已应用到容器主机。 TP5 更新可以在 [KB3157663]( https://support.microsoft.com/en-us/kb/3157663) 中找到。

### Docker 推送

此外，还可以将容器映像上载到 Docker Hub 或 Docker 受信任注册表。 上载这些映像后，可以下载这些映像并在不同的 Windows 容器环境中重新使用。

若要将容器映像上载到 Docker Hub，请先登录到注册表。 有关详细信息，请参阅 [Docker Login on Docker.com]( https://docs.docker.com/engine/reference/commandline/login/)（Docker.com 上的 Docker 登录）。

```none
docker login

Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: username
Password:

Login Succeeded
```

登录到 Docker hub 或 Docker 受信任注册表后，请使用 `docker push` 上载容器映像。 可以按名称或 ID 引用容器映像。 有关详细信息，请参阅 [Docker Push on Docker.com]( https://docs.docker.com/engine/reference/commandline/push/)（Docker.com 上的 Docker 推送）。

```none
docker push username/containername

The push refers to a repository [docker.io/username/containername]
b567cea5d325: Pushed
00f57025c723: Pushed
2e05e94480e9: Pushed
63f3aa135163: Pushed
469f4bf35316: Pushed
2946c9dcfc7d: Pushed
7bfd967a5e43: Pushed
f64ea92aaebc: Pushed
4341be770beb: Pushed
fed398573696: Pushed
latest: digest: sha256:ae3a2971628c04d5df32c3bbbfc87c477bb814d5e73e2787900da13228676c4f size: 2410
```

此时容器映像便已存在，可通过 `docker pull` 进行访问。






<!--HONumber=Sep16_HO1-->


