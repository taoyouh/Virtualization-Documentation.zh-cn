
# <a name="using-insider-container-images"></a>使用预览体验成员容器映像

此练习将向你说明如何在 Windows Insider Preview 计划的最新 Windows Server 预览体验成员版本上部署和使用 Windows 容器功能。 在此练习中，你将安装容器角色并部署基本操作系统映像的预览版本。 如果你需要熟悉容器，可在[关于容器](../about/index.md)中找到此信息。

本快速入门特定于 Windows Server Insider Preview 计划上的 Windows Server 容器。 请先熟悉此计划，然后再继续此快速入门。

## <a name="prerequisites"></a>先决条件：

- 加入 [Windows 预览体验计划](https://insider.windows.com/GettingStarted)并查看使用条款。
- 一个运行 Windows 预览体验计划中最新 Windows Server 版本和/或 Windows 预览体验计划中最新 Windows 10 版本的计算机系统（物理或虚拟）。

> [!IMPORTANT]
> Windows 需要主机操作系统版本才能匹配容器操作系统版本。 如果你希望基于较新的 Windows 版本运行容器，请确保你具有等效的主机版本。 否则，你可以使用 Hyper-v 隔离在新的主机内部版本上运行较旧的容器。 您可以在我们的容器文档中阅读有关 Windows 容器版本兼容性的详细信息。

## <a name="install-docker-enterprise-edition-ee"></a>安装 Docker 企业版 (EE)

若要使用 Window 容器，则需要安装 Docker EE。 Docker EE 由 Docker 引擎和 Docker 客户端组成。

安装 Docker EE 将用到 OneGet 提供程序 PowerShell 模块。 提供程序将启用计算机上的容器功能，并安装 Docker EE - 此操作需要重启计算机。 打开提升的 PowerShell 会话并运行下列命令。

> [!NOTE]
> 在 Windows Server 预览体验计划内部版本中安装 Docker EE 需要不同于用于非预览体验成员版本的 OneGet 提供程序。 如果已安装 Docker EE 和 DockerMsftProvider OneGet 提供程序，请在继续操作前先将它们移除。

```powershell
Stop-Service docker
Uninstall-Package docker
Uninstall-Module DockerMsftProvider
```

安装适用于 Windows 预览体验成员版本的 OneGet PowerShell 模块。

```powershell
Install-Module -Name DockerProvider -Repository PSGallery -Force
```

使用 OneGet 安装 Docker EE 预览版的最新版本。

```powershell
Install-Package -Name docker -ProviderName DockerProvider -RequiredVersion Preview
```

完成安装后，重启计算机。

```powershell
Restart-Computer -Force
```

## <a name="install-base-container-image"></a>安装基本容器映像

使用 Windows 容器前，需安装基本映像。 加入 Windows 预览体验成员计划后，你还可以测试我们的最新版本的基本映像。 使用预览体验计划基本图像，现在有6个基于 Windows Server 的可用基础映像。 请参阅下表，查看各个基本映像应用于什么用途：

| 基本操作系统映像                       | 用法                      |
|-------------------------------------|----------------------------|
| mcr.microsoft.com/windows/servercore         | 生产和开发 |
| mcr.microsoft.com/windows/nanoserver              | 生产和开发 |
| mcr.microsoft.com/windows/              | 生产和开发 |
| mcr.microsoft.com/windows/servercore/insider | 仅开发           |
| mcr.microsoft.com/windows/nanoserver/insider        | 仅开发           |
| mcr.microsoft.com/windows/insider        | 仅开发           |

要提取服务器核心预览体验计划，请参阅[服务器核心预览体验中心](https://hub.docker.com/_/microsoft-windows-servercore-insider)存储库中的特色标记，以使用以下格式：

```console
docker pull mcr.microsoft.com/windows/servercore/insider:10.0.{build}.{revision}
```

若要提取 Nano Server 预览体验成员的基本图像，请参阅[Nano Server 预览体验成员 Docker 中心](https://store.docker.com/_/microsoft-windows-nanoserver-insider)存储库中的特色标记，以使用以下格式：

```console
docker pull mcr.microsoft.com/windows/nanoserver/insider:10.0.{build}.{revision}
```

若要提取 Windows 预览体验成员基本映像，请参阅[Windows 预览体验中心中心](https://store.docker.com/_/microsoft-windows-insider)存储库上的特色标记以使用以下格式：

```console
docker pull mcr.microsoft.com/windows/insider:10.0.{build}.{revision}
```

> [!IMPORTANT]
> 请阅读 Windows 容器操作系统映像[EULA](../EULA.md )和 Windows 预览体验计划[使用条款](https://www.microsoft.com/software-download/windowsinsiderpreviewserver)。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [构建和运行示例应用程序](./Nano-RS3-.NET-Core-and-PS.md)
