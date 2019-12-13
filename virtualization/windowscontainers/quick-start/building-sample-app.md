---
title: 容器化 .NET Core 应用程序
description: 了解如何使用容器生成 .NET core 应用示例
keywords: docker, 容器
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: fab0dc46ddcc8c82a010d408032e5f3c4cea8d69
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910137"
---
# <a name="containerize-a-net-core-app"></a>容器化 .NET Core 应用程序

本主题介绍如何将现有的示例 .NET 应用打包为 Windows 容器，如 "[入门：准备 Windows](set-up-environment.md)容器" 中所述，并运行第一个容器，如[运行第一个 windows 容器](run-your-first-container.md)中所述。

你还需要在计算机上安装 Git 源代码管理系统。 若要安装它，请访问[Git](https://git-scm.com/download)。

## <a name="clone-the-sample-code-from-github"></a>克隆 GitHub 中的示例代码

所有容器示例源代码都保存在名为 `windows-container-samples`的文件夹中的[虚拟化文档](https://github.com/MicrosoftDocs/Virtualization-Documentation)git 存储库（非正式称为存储库）下。

1. 打开 PowerShell 会话，并将目录更改为要在其中存储此存储库的文件夹。 （其他 "命令提示符" 窗口类型也同样适用，但我们的示例命令使用 PowerShell。）
2. 将存储库克隆到当前工作目录：

   ```PowerShell
   git clone https://github.com/MicrosoftDocs/Virtualization-Documentation.git
   ```

3. 使用以下命令导航到 `Virtualization-Documentation\windows-container-samples\asp-net-getting-started` 中找到的示例目录，并创建 Dockerfile。

   [Dockerfile](https://docs.docker.com/engine/reference/builder/)类似于生成文件，它是指示容器引擎如何构建容器映像的指令列表。

   ```Powershell
   # Navigate into the sample directory
   Set-Location -Path Virtualization-Documentation\windows-container-samples\asp-net-getting-started

   # Create the Dockerfile for our project
   New-Item -Name Dockerfile -ItemType file
   ```

## <a name="write-the-dockerfile"></a>编写 Dockerfile

打开刚才用所需的任何文本编辑器创建的 Dockerfile，然后添加以下内容：

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

让我们逐行分解，并解释每个指令的作用。

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:2.1 AS build-env
WORKDIR /app
```

第一组行声明我们将在哪个基础映像上生成容器。 如果本地系统尚不具备此映像，则 Docker 将自动尝试并获取它。 `mcr.microsoft.com/dotnet/core/sdk:2.1` 随 .NET core 2.1 SDK 一起打包，因此，这是生成面向版本2.1 的 ASP .NET core 项目的任务。 下一条指令会将容器中的工作目录更改为 `/app`，因此，此操作后面的所有命令都将在此上下文中执行。

```Dockerfile
COPY *.csproj ./
RUN dotnet restore
```

接下来，这些说明将 .csproj 文件复制到 `build-env` 容器的 `/app` 目录中。 复制此文件后，.NET 将从该文件中读取该文件，然后获取项目所需的所有依赖项和工具。

```Dockerfile
COPY . ./
RUN dotnet publish -c Release -o out
```

.NET 将所有依赖项提取到 `build-env` 容器中后，下一条指令会将所有项目源文件复制到容器中。 然后，告知 .NET 使用发布配置发布应用程序，并在中指定输出路径。

编译应成功。 现在，我们必须生成最终映像。 

> [!TIP]
> 本快速入门基于源构建 .NET core 项目。 构建容器映像时，最好_只_在容器映像中包含生产有效负载及其依赖项。 我们不希望在最终映像中包括 .NET core SDK，因为我们只需要 .NET core 运行时，因此将 dockerfile 编写为使用与 SDK 一起打包的临时容器，`build-env` 来生成应用。

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/aspnet:2.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "asp-net-getting-started.dll"]
```

由于我们的应用程序是 ASP.NET 的，因此请指定包含此运行时的映像。 我们随后将所有文件从临时容器的输出目录复制到最终容器中。 当容器启动时，我们将容器配置为以新应用程序的入口点运行

我们编写了 dockerfile 来执行_多阶段生成_。 执行 dockerfile 时，它将使用临时容器 `build-env`，通过 .NET core 2.1 SDK 来构建示例应用，然后将输出的二进制文件复制到仅包含 .NET core 2.1 运行时的另一个容器，以便将最终容器的大小降到最低。

## <a name="build-and-run-the-app"></a>生成并运行应用

编写 Dockerfile 后，我们可以在我们的 Dockerfile 上指出 Docker，并告诉它构建并运行我们的映像：

1. 在命令提示符窗口中，导航到 dockerfile 所在的目录，然后运行[docker build](https://docs.docker.com/engine/reference/commandline/build/)命令以从 dockerfile 构建容器。

   ```Powershell
   docker build -t my-asp-app .
   ```

2. 若要运行新构建的容器，请运行[docker run](https://docs.docker.com/engine/reference/commandline/run/)命令。

   ```Powershell
   docker run -d -p 5000:80 --name myapp my-asp-app
   ```

   让我们仔细分析此命令：

   * `-d` 告知 Docker 运行运行容器 "已分离"，这意味着没有控制台挂钩到容器内的控制台。 容器在后台运行。 
   * `-p 5000:80` 指示 Docker 将主机上的端口5000映射到容器中的端口80。 每个容器均获取其自己的 IP 地址。 默认情况下，ASP .NET 在端口80上侦听。 端口映射允许我们转到映射端口中主机的 IP 地址，Docker 会将所有流量转发到容器内的目标端口。
   * `--name myapp` 通知 Docker 为此容器指定一个便于查询的名称（而不是在运行时由 Docker 指定 contaienr ID）。
   * `my-asp-app` 是我们希望 Docker 运行的映像。 这是 `docker build` 进程的结果生成的容器映像。

3. 打开 web 浏览器 web 浏览器并导航到 `http://localhost:5000` 查看容器化应用程序，如以下屏幕截图所示：

   >![从容器中的 localhost 运行的 ASP.NET Core 网页](media/SampleAppScreenshot.png)

## <a name="next-steps"></a>后续步骤

1. 下一步是使用 Azure 容器注册表将容器化的 ASP.NET web 应用发布到专用注册表。 这允许你在组织中部署它。

   > [!div class="nextstepaction"]
   > [创建专用容器注册表](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell)

   转到将[容器映像推送到注册表](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell#push-image-to-registry)的部分时，请指定刚刚打包（`my-asp-app`）的 ASP.NET 应用的名称和容器注册表（例如： `contoso-container-registry`）：

   ```PowerShell
   docker tag my-asp-app contoso-container-registry.azurecr.io/my-asp-app:v1
   ```

   若要查看更多应用示例及其关联的 dockerfile，请参阅[附加容器示例](../samples.md)。

2. 将应用发布到容器注册表后，下一步是将该应用部署到使用 Azure Kubernetes Service 创建的 Kubernetes 群集。

   > [!div class="nextstepaction"]
   > [创建 Kubernetes 群集](https://docs.microsoft.com/azure/aks/windows-container-cli)
