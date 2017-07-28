# 使用或不使用 .NET Core 2.0 或 PowerShell Core 6 生成和运行应用程序

此版本中的 Nano Server 基本操作系统容器映像删除了 .NET Core 和 PowerShell，但支持 .NET Core 和 PowerShell 作为基础 Nano Server 容器上的附加分层容器。  

如果你的容器计划运行本机代码或开放框架，如 Node.js、Python、Ruby 等，则基本 Nano Server 容器足已。  此版本与 Windows Server 2016 版本的一个细微差别，此版本可能因为[占用空间节省](https://docs.microsoft.com/en-us/windows-server/get-started/nano-in-semi-annual-channel)而无法运行某些本机代码。 如果你注意到任何回归问题，请在[论坛](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)中告知我们。 

若要从 Dockerfile 生成容器，请使用 docker 版本，若要运行它，请使用 docker 运行。  以下命令将下载 Nano Server 容器基本操作系统映像（这可能需要几分钟）并打印在主机控制台打印“Hello World!” 消息。

```none
docker run microsoft/nanoserver-insider cmd /c echo Hello World!
```

你可以使用 [Windows 上的 Dockerfile](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-docker/manage-windows-dockerfile)，通过 Dockerfile 语法（如 FROM、RUN、COPY、ADD、CMD 等）生成更复杂的应用程序。尽管你将无法通过该基本映像立即运行特定命令，你现在能够创建仅包含让你的应用程序工作所需的内容的容器映像。

由于 .NET Core 和 PowerShell 在基本 Nano Server 容器操作系统映像中不可用，使用压缩 zip 格式的内容生成容器会比较难。 使用在 Docker 17.05 中提供的[多阶段版本](https://docs.docker.com/engine/userguide/eng-image/multistage-build/)功能，你可以充分利用另一个容器中的 PowerShell 解压缩内容并复制到 Nano 容器。 这种方法可用于创建 .NET Core 容器和 PowerShell 容器。 

可以通过使用此命令拉取 PowerShell 容器：

```none
docker pull microsoft/nanoserver-insider-powershell
```

可以通过使用此命令拉取 .NET Core 容器：

```none
docker pull microsoft/nanoserver-insider-dotnet
```

下面是一些有关我们如何利用多阶段版本创建这些容器映像的示例。

## 基于 .NET Core 2.0 部署应用
在其他位置生成 .NET Core 应用程序且你想要在容器中运行它时，可以利用预览体验成员版本中的 .NET Core 2.0 容器映像运行你的 .NET Core 应用。  你可以在 [.NET Core GitHub](https://github.com/dotnet/dotnet-docker-nightly) 找到有关如何使用 .NET Core 容器映像运行 .NET Core 应用程序的详细信息。  如果你在容器内部开发应用程序，应改为使用 .NET Core SDK。  对于高级用户，你可以使用 .NET Core 2.0 版本、Dockerfile 和在 [dotnet-docker-nightly](https://github.com/dotnet/dotnet-docker-nightly/tree/master/2.0) 中指定的 URL 生成你自己的 .NET Core 2.0 容器。 若要执行该操作，可以使用 Windows Server Core 容器完成下载和解压缩功能。  Dockerfile 示例与 [.NET Core 运行时 Dockerfile](https://github.com/dotnet/dotnet-docker-nightly/blob/master/2.0/runtime/nanoserver-insider/Dockerfile) 相同。


使用此 Dockerfile，可以使用以下命令生成 .NET Core 2.0 容器。

```none
docker build -t nanoserverdnc -f Dockerfile-dotnetRuntime .
```

## 在容器中运行 PowerShell Core 6
使用相同的[多阶段版本](https://docs.docker.com/engine/userguide/eng-image/multistage-build/)方法，可以使用[此 PowerShell Dockerfile](https://github.com/PowerShell/PowerShell/blob/master/docker/release/nanoserver-insider/Dockerfile) 生成 PowerShell Core 6 容器。


然后发出 docker 版本以创建 PowerShell 容器映像。

```none 
docker build -t nanoserverPowerShell6 -f Dockerfile-PowerShell6 .
```

你可以在 [PowerShell GitHub](https://github.com/PowerShell/PowerShell/tree/master/docker/release) 找到详细信息。  值得一提的是，PowerShell zip 包含生成 PowerShell Core 6 所需的 .NET Core 2.0 的子集。  如果你的 PowerShell 模块依赖于 .NET Core 2.0，可以安全地在 Nano .NET Core 容器上生成 PowerShell 容器，而不是在基本 Nano 容器上，即 使用 Dockerfile 中的 FROM microsoft/nanoserver-insider-dotnet。 

## 后续步骤
- 使用在 Docker 中心可用的其中一个基于 Nano Server 的新容器映像，即基本 Nano Server 映像、具有 .NET Core 2.0 的 Nano 和具有 PowerShell Core 6 的 Nano
- 使用本指南中的 Dockerfile 示例内容，基于新的 Nano Server 容器基本操作系统映像生成你自己的容器映像。 
