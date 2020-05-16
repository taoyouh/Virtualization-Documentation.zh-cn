---
title: 容器化 .NET Core 应用
description: 了解如何使用容器生成示例 .NET Core 应用
keywords: docker, 容器
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: c9e175a7ced0f328e342f3cdd4f99adc717d5700
ms.sourcegitcommit: 62f4bcca4e07f2a34a927e5c4d786e505821d559
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/05/2020
ms.locfileid: "82784396"
---
# <a name="containerize-a-net-core-app"></a>容器化 .NET Core 应用

本主题介绍在按[入门：为容器准备 Windows](set-up-environment.md) 中所述设置环境并按[运行第一个 Windows 容器](run-your-first-container.md)中所述运行第一个容器后，如何将用于部署的现有示例 .NET 应用打包为 Windows 容器。

此外还需在计算机上安装 Git 源代码管理系统。 若要安装它，请访问 [Git](https://git-scm.com/download)。

## <a name="clone-the-sample-code-from-github"></a>克隆 GitHub 中的示例代码

所有容器示例源代码都保存在 [Virtualization-Documentation](https://github.com/MicrosoftDocs/Virtualization-Documentation) git 存储库（非正式名称为 repo）的名为 `windows-container-samples` 的文件夹中。

1. 打开 PowerShell 会话，将目录更改为要在其中存储此存储库的文件夹。 （其他命令提示符窗口类型也适用，但我们的示例命令使用 PowerShell。）
2. 将存储库克隆到当前工作目录：

   ```PowerShell
   git clone https://github.com/MicrosoftDocs/Virtualization-Documentation.git
   ```

3. 使用以下命令导航到 `Virtualization-Documentation\windows-container-samples\asp-net-getting-started` 下的示例目录并创建 Dockerfile。

   [Dockerfile](https://docs.docker.com/engine/reference/builder/) 类似于生成文件，是一个列表，其中包含的指令会指示容器引擎生成容器映像。

   ```Powershell
   # Navigate into the sample directory
   Set-Location -Path Virtualization-Documentation\windows-container-samples\asp-net-getting-started

   # Create the Dockerfile for our project
   New-Item -Name Dockerfile -ItemType file
   ```

## <a name="write-the-dockerfile"></a>编写 Dockerfile

打开你刚才用喜爱的任意文本编辑器创建的 Dockerfile，然后添加以下内容：

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

让我们逐行分解并解释每个指令的作用。

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:2.1 AS build-env
WORKDIR /app
```

第一组行声明我们将在哪个基础映像上生成容器。 如果本地系统尚不具备此映像，则 Docker 将自动尝试并获取它。 `mcr.microsoft.com/dotnet/core/sdk:2.1` 随安装的 .NET Core 2.1 SDK 一起打包，因此可生成面向版本 2.1 的 ASP .NET Core 项目。 下一条指令会将容器中的工作目录更改为 `/app`，因此，此后的所有命令都将在此上下文中执行。

```Dockerfile
COPY *.csproj ./
RUN dotnet restore
```

接下来，这些指令将 .csproj 文件复制到 `build-env` 容器的 `/app` 目录中。 复制该文件后，.NET 会从其中读取数据，然后在此基础上获取我们项目所需的所有依赖项和工具。

```Dockerfile
COPY . ./
RUN dotnet publish -c Release -o out
```

.NET 将所有依赖项拉取到 `build-env` 容器中后，下一条指令会将所有项目源文件复制到容器中。 然后，我们告知 .NET 以某种版本配置发布我们的应用程序，并指定输出路径。

编译应该成功。 现在，我们必须生成最终映像。 

> [!TIP]
> 本快速入门根据源来生成 .NET Core 项目。 生成容器映像时，最好是只在容器映像中包含  生产有效负载及其依赖项。 我们不希望在最终映像中包含 .NET Core SDK，因为我们只需要 .NET Core 运行时。因此，我们在编写 Dockerfile 时，使用与名为 `build-env` 的 SDK 一起打包的临时容器来生成应用。

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/aspnet:2.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "asp-net-getting-started.dll"]
```

由于我们的应用程序是 ASP.NET，因此我们指定包含该运行时的映像。 我们随后将所有文件从临时容器的输出目录复制到最终容器中。 我们对容器进行了配置，使得容器在启动并运行时将新应用作为入口点

我们编写了 Dockerfile 来执行多阶段生成  。 执行 Dockerfile 时，它会使用临时容器 `build-env`，通过 .NET Core 2.1 SDK 来生成示例应用，然后将输出的二进制文件复制到仅包含 .NET Core 2.1 运行时的另一个容器，以便将最终容器的大小降到最低。

## <a name="build-and-run-the-app"></a>生成并运行应用

编写 Dockerfile 后，我们可以将 Docker 指向我们的 Dockerfile，让其生成并运行我们的映像：

1. 在命令提示符窗口中，导航到 Dockerfile 所在的目录，然后运行 [docker build](https://docs.docker.com/engine/reference/commandline/build/) 命令，根据 Dockerfile 生成容器。

   ```Powershell
   docker build -t my-asp-app .
   ```

2. 若要运行新生成的容器，请运行 [docker run](https://docs.docker.com/engine/reference/commandline/run/) 命令。

   ```Powershell
   docker run -d -p 5000:80 --name myapp my-asp-app
   ```

   让我们仔细分析此命令：

   * `-d` 告知 Docker 以“分离”模式运行容器，这意味着没有任何控制台与容器中的控制台挂钩。 容器在后台运行。 
   * `-p 5000:80` 告知 Docker 将主机上的端口 5000 映射到容器中的端口 80。 每个容器均获取其自己的 IP 地址。 默认情况下，ASP .NET 在端口 80 上侦听。 我们可以通过端口映射转到已映射端口的主机 IP 地址，Docker 会将所有流量转发到容器内的目标端口。
   * `--name myapp` 告知 Docker 为此容器指定一个方便进行查询的名称（这样就不需要查找 Docker 在运行时分配的容器 ID）。
   * `my-asp-app` 是我们希望 Docker 运行的映像。 这是 `docker build` 过程结束时生成的容器映像。

3. 打开 Web 浏览器并导航到 `http://localhost:5000` 即可查看容器化应用程序，如以下屏幕截图所示：

   >![从容器中的 localhost 运行的 ASP.NET Core 网页](media/SampleAppScreenshot.png)

## <a name="next-steps"></a>后续步骤

1. 下一步是使用 Azure 容器注册表将容器化的 ASP.NET Web 应用发布到专用注册表。 这样就可以在组织中部署它。

   > [!div class="nextstepaction"]
   > [创建专用容器注册表](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell)

   转到[将容器映像推送到注册表](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell#push-image-to-registry)这一部分时，请指定刚打包的 ASP.NET 应用的名称 (`my-asp-app`) 以及容器注册表（例如 `contoso-container-registry`）：

   ```PowerShell
   docker tag my-asp-app contoso-container-registry.azurecr.io/my-asp-app:v1
   ```

   若要查看更多应用示例及其关联的 Dockerfile，请参阅[其他容器示例](../samples.md)。

2. 将应用发布到容器注册表后，下一步是将该应用部署到使用 Azure Kubernetes 服务创建的 Kubernetes 群集。

   > [!div class="nextstepaction"]
   > [创建 Kubernetes 群集](https://docs.microsoft.com/azure/aks/windows-container-cli)
