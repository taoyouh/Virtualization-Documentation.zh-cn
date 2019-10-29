---
title: Containerize .NET Core 应用
description: 了解如何使用容器构建 .NET core 应用示例
keywords: docker, 容器
author: cwilhit
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: db3caea3f7911ec6641930302198f976bd61240d
ms.sourcegitcommit: da762ce138467e50dce22d5086ad407138b38e48
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/29/2019
ms.locfileid: "10261826"
---
# <a name="containerize-a-net-core-app"></a>Containerize .NET Core 应用

此段假定你的开发环境已配置为使用容器。 如果你没有为容器配置环境，请访问 "[设置你的环境](./set-up-environment.md)" 以了解如何开始使用。

你将需要在计算机上安装 Git 源控制系统。 可在此处抓取： [Git](https://git-scm.com/download)

## <a name="clone-the-sample-code"></a>克隆示例代码

所有容器示例源代码均保留在名`windows-container-samples`为的文件夹中的 "[虚拟化-文档](https://github.com/MicrosoftDocs/Virtualization-Documentation)git 存储库" 下。 将此 git 存储库复制到你的 curent 工作目录。

```Powershell
git clone https://github.com/MicrosoftDocs/Virtualization-Documentation.git
```

导航到在 "" 下`Virtualization-Documentation\windows-container-samples\asp-net-getting-started`找到的示例目录，并创建 Dockerfile。 [Dockerfile](https://docs.docker.com/engine/reference/builder/)就像是一个 makefile，它是指示容器引擎必须如何生成容器映像的指令列表。

```Powershell
# navigate into the sample directory
Set-Location -Path Virtualization-Documentation\windows-container-samples\asp-net-getting-started

# create the Dockerfile for our project
New-Item -Name Dockerfile -ItemType file
```

## <a name="write-the-dockerfile"></a>编写 dockerfile

打开您刚刚创建的 dockerfile （通过您的任何别致的文本编辑器），并添加以下内容。

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:2.1 AS build-env
WORKDIR /app

COPY *.csproj ./
RUN dotnet restore

COPY . ./
RUN dotnet publish -c Release -o out

FROM mcr.microsoft.com/dotnet/core/aspnet:2.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "asp-net-getting-started.dll"]
```

让我们逐行分行，解释每条说明的功能。

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:2.1 AS build-env
WORKDIR /app
```

第一组行声明我们将在哪个基础映像上生成容器。 如果本地系统尚不具备此映像，则 Docker 将自动尝试并获取它。 该`mcr.microsoft.com/dotnet/core/sdk:2.1`版本随安装了 .net CORE 2.1 SDK 进行了打包，因此它将由构建面向版本2.1 的 ASP .net core 项目的任务组成。 下一条指令会将`/app`容器中的工作目录更改为，因此在此上下文下执行的所有命令都将执行。

```Dockerfile
COPY *.csproj ./
RUN dotnet restore
```

接下来，这些说明将 .csproj 文件复制到`build-env`容器的`/app`目录中。 复制此文件后，.NET 将从该文件中读取并获取项目所需的所有依赖项和工具。

```Dockerfile
COPY . ./
RUN dotnet publish -c Release -o out
```

一旦 .NET 已将所有依赖项提取到`build-env`容器中，下一个指令会将所有项目源文件复制到容器中。 然后，我们会告诉 .NET 使用 "发布" 配置发布应用程序，并在中指定输出路径。

编译应该成功。 现在，我们必须构建最终的映像。 

> [!TIP]
> 此快速入门基于源构建 .NET core 项目。 生成容器图像时，最好_仅_在容器映像中包含生产负载及其依赖项。 我们不希望最终图像中包含 .NET core SDK，因为我们只需要 .NET core 运行时，因此 dockerfile 将编写为使用与 SDK `build-env`一起打包的临时容器来构建应用。

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/aspnet:2.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "asp-net-getting-started.dll"]
```

由于我们的应用程序是 ASP.NET，因此我们将指定一个包含此运行时的映像。 我们随后将所有文件从临时容器的输出目录复制到最终容器中。 我们将容器配置为在容器启动时通过我们的新应用作为其入口点运行

我们编写了 dockerfile 来执行_多阶段生成_。 在执行 dockerfile 时，它将使用临时容器`build-env`，使用 .net CORE 2.1 SDK 生成示例应用，然后将输出二进制文件复制到仅包含 .net core 2.1 运行时的另一个容器中，以便我们最大程度地减少最终容器。

## <a name="run-the-app"></a>运行应用

编写 dockerfile 后，我们可以在我们的 dockerfile 上指向 Docker，并告诉它构建图像。 

>[!IMPORTANT]
>以下执行的命令需要在 dockerfile 驻留的目录中执行。

```Powershell
docker build -t my-asp-app .
```

若要运行容器，请执行下面的命令。

```Powershell
docker run -d -p 5000:80 --name myapp my-asp-app
```

让我们 dissect 此命令：

* `-d` 告诉 Docker tun 运行容器 "已分离"，这意味着不会将任何控制台挂钩到容器内的控制台。 容器在后台运行。 
* `-p 5000:80` 告诉 Docker 将主机上的端口5000映射到容器中的端口80。 每个容器都获得自己的 IP 地址。 默认情况下，ASP .NET 在端口80上侦听。 通过端口映射，我们可以在映射端口上转到主机的 IP 地址，并且 Docker 会将所有流量转发到容器内的目标端口。
* `--name myapp` 通知 Docker 为此容器提供一个方便的名称来进行查询（而无需在运行时通过 Docker 器查找分配的 contaienr ID）。
* `my-asp-app` 是希望 Docker 运行的图像。 这是`docker build`进程的 culmination 生成的容器图像。

打开 web 浏览器 web 浏览器，并`http://localhost:5000`导航到向的应用程序。

>![](media/SampleAppScreenshot.png)

## <a name="next-steps"></a>后续步骤

我们已成功将 ASP.NET web 应用进行了容器。 若要查看更多应用示例及其关联的 dockerfiles，请单击下面的按钮。

> [!div class="nextstepaction"]
> [查看更多容器示例](../samples.md)
