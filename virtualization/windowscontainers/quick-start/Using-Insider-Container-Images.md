# 使用预览体验成员容器映像

此练习将向你说明如何在 Windows Insider Preview 计划的最新 Windows Server 预览体验成员版本上部署和使用 Windows 容器功能。 在此练习中，你将安装容器角色并部署基本操作系统映像的预览版本。 在开始本快速入门之前，请先熟悉基本容器概念和术语。 可以在[快速入门简介](./index.md)上找到此信息。

本快速入门特定于 Windows Server Insider Preview 计划上的 Windows Server 容器。 请先熟悉此计划，然后再继续此快速入门。

**先决条件：**

- 加入 [Windows 预览体验计划](https://insider.windows.com/GettingStarted)并查看使用条款。 
- 一个运行 Windows 预览体验计划中最新 Windows Server 版本和/或 Windows 预览体验计划中最新 Windows 10 版本的计算机系统（物理或虚拟）。

>你必须使用 Windows Server Insider Preview 计划中的 Windows Server 版本或 Windows Insider Preview 计划中的 Windows 10 版本才能使用如下所述的基本映像。 如果你没有使用这些版本中的其中一个，使用这些基本映像将导致启动容器失败。

## 安装 Docker
若要使用 Window 容器，则需要安装 Docker。 Docker 由 Docker 引擎和 Docker 客户端组成。 你还需要支持多阶段版本的 Docker 版本以获得使用容器优化的 Nano Server 映像的最佳体验。

安装 Docker 将用到 OneGet 提供程序 PowerShell 模块。 提供程序将启用计算机上的容器功能，并安装 Docker - 此操作需要重启计算机。 请注意，有多个通道可以在不同的情况下使用不同版本的 docker。 此练习中，我们将使用来自稳定通道的 Docker 的最新 Community Edition 版本。 如果你要测试 Docker 中的最新进展，也可以使用 Edge 通道。 

打开提升的 PowerShell 会话并运行下列命令。

>注意：在预览体验成员版本中安装 Docker 需要与截至现在通常使用的提供程序不同的提供程序。 请注意以下区别。

安装 OneGet PowerShell 模块。
```powershell
Install-Module -Name DockerMsftProviderInsider -Repository PSGallery -Force
```
使用 OneGet 安装最新版的 Docker。
```powershell
Install-Package -Name docker -ProviderName DockerMsftProviderInsider -RequiredVersion 17.06.0-ce
```
完成安装后，重启计算机。
```none
Restart-Computer -Force
```

## 安装基本容器映像

使用 Windows 容器前，需安装基本映像。 加入 Windows 预览体验计划后，你还可以测试我们的最新版本的基本映像。 预览体验成员基本映像现在有 4 个基于 Windows Server 的基本映像可用。 请参阅下表，查看各个基本映像应用于什么用途：

| 基本操作系统映像                       | 用法                      |
|-------------------------------------|----------------------------|
| microsoft/windowsservercore         | 生产和开发 |
| microsoft/nanoserver                | 生产和开发 |
| microsoft/windowsservercore-insider | 仅开发           |
| microsoft/nanoserver-insider        | 仅开发           |

若要拉取 Nano Server 预览体验成员基本映像，请运行以下内容：

```none
docker pull microsoft/nanoserver-insider
```

若要拉取 Windows Server Core 预览体验成员基本映像，请运行以下内容：

```none
docker pull microsoft/windowsservercore-insider
```

请阅读 Windows 容器操作系统映像 EULA（可在此处找到 - [EULA](../EULA.md )）和 Windows 预览体验计划使用条款（可在此处找到 - [使用条款](https://www.microsoft.com/en-us/software-download/windowsinsiderpreviewserver)）。 

## 后续步骤

[使用或不使用 .NET Core 2.0 或 PowerShell Core 6 生成和运行应用程序](./Nano-RS3-.NET-Core-and-PS.md)
