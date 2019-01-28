---
title: 生成示例应用
description: 了解如何利用容器生成示例应用
keywords: docker, 容器
author: cwilhit
ms.date: 07/25/2017
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 970f039e97ce0628c7a7f78c417017fc95570f82
ms.sourcegitcommit: 51da93c4548c5df7a9f01e54d46d81b338c874cf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/28/2019
ms.locfileid: "9031161"
---
# <a name="build-a-sample-app"></a>生成示例应用

此练习将演练如何获取示例 ASP.net 应用并将其转换为在容器中运行。 如果需要了解如何在 Windows 10 中使用容器进行启动和运行，请访问 [Windows 10 快速入门](./quick-start-windows-10.md)。

本快速入门特定于 Windows 10。 此页面左侧的目录中提供其他快速入门文档。 由于此教程的焦点在于容器，因此我们将忽略编写代码，而只关注容器。 如果要从头开始生成此教程，则可在 [ASP.NET Core 文档](https://docs.microsoft.com/en-us/aspnet/core/tutorials/first-mvc-app-xplat/)中找到它

如果未在计算机上安装 Git 源代码控制，则可从此处获取它：[Git](https://git-scm.com/download)

## <a name="getting-started"></a>入门

此示例项目是使用 [VSCode](https://code.visualstudio.com/) 设置的。 我们还将使用 Powershell。 请从 github 获取演示代码。 你可使用 git 来克隆存储库或直接从 [SampleASPContainerApp](https://github.com/cwilhit/SampleASPContainerApp) 下载该项目。

```Powershell
git clone https://github.com/cwilhit/SampleASPContainerApp.git
```

现在，让我们导航到项目目录，并创建 Dockerfile。 [Dockerfile](https://docs.docker.com/engine/reference/builder/) 与 makefile 类似：其中列出了生成容器映像必须遵循的指令。

```Powershell
#Create the dockerfile for our proj
New-Item C:/Your/Proj/Location/Dockerfile -type file
```

## <a name="writing-our-dockerfile"></a>编写 Dockerfile

现在让我们打开先前在项目根文件夹中创建的该 Dockerfile（使用你喜欢的任一文本编辑器），并向其中添加一些逻辑。 然后，我们将对其逐行细分并说明正在发生的情况。

```Dockerfile
FROM microsoft/aspnetcore-build:1.1 AS build-env
WORKDIR /app

COPY *.csproj ./
RUN dotnet restore

COPY . ./
RUN dotnet publish -c Release -o out

FROM microsoft/aspnetcore:1.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "MvcMovie.dll"]
```

第一组行声明我们将在哪个基础映像上生成容器。 如果本地系统尚不具备此映像，则 Docker 将自动尝试并获取它。 Aspnetcore-build 与用于编译项目的依赖项打包在一起。 我们随后将容器中的工作目录更改为“/app”，这样 dockerfile 中的所有后续命令都将在此目录中执行。

_注意_：由于必须生成项目，我们创建的第一个容器是临时容器，并且我们只会将该容器用于生成项目，然后在结束时将其丢弃。

```Dockerfile
FROM microsoft/aspnetcore-build:1.1 AS build-env
WORKDIR /app
```

接下来，我们将 .csproj 文件复制到临时容器的“/app”目录。 我们这么做是因为.csproj 文件包含我们项目所需的程序包引用的列表。

复制该文件后，dotnet 会从其中读取数据，然后在此基础上获取我们项目所需的所有依赖项和工具。

```Dockerfile
COPY *.csproj ./
RUN dotnet restore
```

获取所有依赖项之后，我们将其复制到临时容器。 然后，我们告知 dotnet 以某种版本配置发布我们的应用程序，并指定输出路径。

```Dockerfile
COPY . ./
RUN dotnet publish -c Release -o out
```

此时项目应该已经编译成功。 现在，我们需要生成最终容器。 由于我们的应用程序是 ASP.NET，我们需要指定具有这些库的映像作为源。 我们随后将所有文件从临时容器的输出目录复制到最终容器中。 我们将容器配置为以启动容器时编译的新 .dll 来运行。

_注意_：此最终容器的基础映像是类似的，但不同于以上的 ```FROM``` 命令：它不具有能够_生成_ ASP.NET 应用的库，而只具有能够运行此类应用的库。

```Dockerfile
FROM microsoft/aspnetcore:1.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "MvcMovie.dll"]
```

我们现在已成功执行所谓的_多阶段生成_。 我们使用了临时容器生成映像，然后将已发布的 dll 移到了另一个容器，以便最大限度减小最终结果占用的空间。 我们希望该容器具有正常运行所需的绝对最少的依赖项；如果先前一直使用的是第一个映像，那么它将与其他层打包在一起（用于生成 ASP.NET 应用），但这些层并不重要，因此会增加映像的大小。

## <a name="running-the-app"></a>运行该应用

编写好 dockerfile 后，剩下的所有工作就是告诉 Docker 生成我们的应用，然后运行容器。 我们指定要发布到的端口，然后为容器指定一个标记“myapp”。 在 PowerShell 中，执行以下命令。

_注意_：PowerShell 控制台的当前工作目录必须是上面创建的 dockerfile 所在的目录。

```Powershell
docker build -t myasp .
docker run -d -p 5000:80 --name myapp myasp
```

若要查看应用运行情况，我们需要访问运行该应用的地址。 通过运行以下命令即可获取 IP 地址。

```Powershell
 docker inspect -f "{{ .NetworkSettings.Networks.nat.IPAddress }}" myapp
```

运行以下命令即可获得正在运行的容器的 IP 地址。 输出应该会如以下示例所示。

```Powershell
 172.19.172.12
```

将此 IP 地址输入你选择的 Web 浏览器中，然后你将看到应用程序在容器中成功运行！

<center style="margin: 25px">![](media/SampleAppScreenshot.png)</center>

单击导航栏中的“MvcMovie”将会打开一个网页，你可以在其中输入、编辑和删除电影条目。

## <a name="next-steps"></a>后续步骤

我们已成功获取 ASP.NET Web 应用，使用 Docker 配置并生成了该应用，并已成功将其部署到正在运行的容器中。 但还有可以进一步执行的步骤！ 你可以进一步将该 Web 应用拆分为如下组件：运行 Web API 的容器、运行前端的容器以及运行 SQL Server 的容器。

既然你已挂起的容器，请有转和生成出色的容器化的软件 ！

> [!div class="nextstepaction"]
> [查看更多容器示例](../samples.md)
