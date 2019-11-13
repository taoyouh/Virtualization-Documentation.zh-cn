---
title: Containerize .NET Core 应用
description: 了解如何使用容器构建 .NET core 应用示例
keywords: docker, 容器
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: fab0dc46ddcc8c82a010d408032e5f3c4cea8d69
ms.sourcegitcommit: e61db4d98d9476a622e6cc8877650d9e7a6b4dd9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/13/2019
ms.locfileid: "10288065"
---
# <a name="containerize-a-net-core-app"></a>Containerize .NET Core 应用

本主题介绍了如何将现有的现有 .NET 应用作为 Windows 容器进行打包，如 "入门" 中所述，在设置你的环境之后[：准备用于容器的窗口](set-up-environment.md)和运行第一个容器，如[运行你的第一个 windows 容器](run-your-first-container.md)中所述。

你还需要在计算机上安装 Git 源控制系统。 若要安装它，请访问[Git](https://git-scm.com/download)。

## <a name="clone-the-sample-code-from-github"></a>克隆 GitHub 中的示例代码

所有容器示例源代码均保留在名`windows-container-samples`为的文件夹中的 "[虚拟化-文档](https://github.com/MicrosoftDocs/Virtualization-Documentation)git 存储库（称为存储库）" 下。

1. 打开 PowerShell 会话并将目录更改为要存储该存储库的文件夹。 （其他命令提示符窗口类型同样适用，但我们的示例命令使用 PowerShell。）
2. 将存储库复制到当前工作目录：

   ```PowerShell
   git clone https://github.com/MicrosoftDocs/Virtualization-Documentation.git
   ```

3. 使用以下命令导航到 "创建`Virtualization-Documentation\windows-container-samples\asp-net-getting-started` Dockerfile" 下找到的示例目录。

   [Dockerfile](https://docs.docker.com/engine/reference/builder/)类似于生成文件，它是指示容器引擎如何构建容器映像的指令列表。

   ```Powershell
   # Navigate into the sample directory
   Set-Location -Path Virtualization-Documentation\windows-container-samples\asp-net-getting-started

   # Create the Dockerfile for our project
   New-Item -Name Dockerfile -ItemType file
   ```

## <a name="write-the-dockerfile"></a>编写 Dockerfile

打开您刚刚创建的 Dockerfile，用您喜欢的任何文本编辑器，然后添加以下内容：

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

我们编写了 dockerfile 来执行_多阶段生成_。 在执行 dockerfile 时，它将使用临时容器`build-env`和 .net CORE 2.1 SDK 生成示例应用，然后将输出二进制文件复制到仅包含 .net core 2.1 运行时的另一个容器中，以便我们将最终容器的大小降到最低。

## <a name="build-and-run-the-app"></a>生成并运行应用

编写 Dockerfile 后，我们可以在我们的 Dockerfile 上指向 Docker，并告诉它构建并运行我们的图像：

1. 在命令提示符窗口中，导航到 dockerfile 驻留的目录，然后运行[docker build](https://docs.docker.com/engine/reference/commandline/build/)命令以从 dockerfile 构建容器。

   ```Powershell
   docker build -t my-asp-app .
   ```

2. 若要运行新生成的容器，请运行[docker "运行](https://docs.docker.com/engine/reference/commandline/run/)" 命令。

   ```Powershell
   docker run -d -p 5000:80 --name myapp my-asp-app
   ```

   让我们 dissect 此命令：

   * `-d` 告诉 Docker tun 运行容器 "已分离"，这意味着不会将任何控制台挂钩到容器内的控制台。 容器在后台运行。 
   * `-p 5000:80` 告诉 Docker 将主机上的端口5000映射到容器中的端口80。 每个容器都获得自己的 IP 地址。 默认情况下，ASP .NET 在端口80上侦听。 通过端口映射，我们可以在映射端口上转到主机的 IP 地址，并且 Docker 会将所有流量转发到容器内的目标端口。
   * `--name myapp` 通知 Docker 为此容器提供一个方便的名称来进行查询（而无需在运行时通过 Docker 器查找分配的 contaienr ID）。
   * `my-asp-app` 是希望 Docker 运行的图像。 这是`docker build`进程的 culmination 生成的容器图像。

3. 打开 web 浏览器 web 浏览器，然后`http://localhost:5000`导航到要查看容器化的应用程序，如以下屏幕截图所示：

   >![ASP.NET 核心网页，从容器中的本地主机运行](media/SampleAppScreenshot.png)

## <a name="next-steps"></a>后续步骤

1. 下一步是使用 Azure 容器注册表将已容器的 ASP.NET web 应用发布到专用注册表。 这使您可以在您的组织中部署它。

   > [!div class="nextstepaction"]
   > [创建私人容器注册表](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell)

   当你转到将[容器映像推送到注册表](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell#push-image-to-registry)的分区时，请指定你刚刚打包（`my-asp-app`）的 ASP.NET 应用的名称和你的容器注册表（例如： `contoso-container-registry`）：

   ```PowerShell
   docker tag my-asp-app contoso-container-registry.azurecr.io/my-asp-app:v1
   ```

   若要查看更多应用示例及其关联的 dockerfiles，请参阅[其他容器示例](../samples.md)。

2. 将应用发布到容器注册表后，下一步是将应用部署到使用 Azure Kubernetes 服务创建的 Kubernetes 群集。

   > [!div class="nextstepaction"]
   > [创建 Kubernetes 群集](https://docs.microsoft.com/azure/aks/windows-container-cli)
