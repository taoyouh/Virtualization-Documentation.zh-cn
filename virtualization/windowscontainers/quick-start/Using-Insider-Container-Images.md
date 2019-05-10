
# <a name="using-insider-container-images"></a>使用预览体验成员容器映像

此练习将向你说明如何在 Windows Insider Preview 计划的最新 Windows Server 预览体验成员版本上部署和使用 Windows 容器功能。 在此练习中，你将安装容器角色并部署基本操作系统映像的预览版本。 如果你需要熟悉容器，可在[关于容器](../about/index.md)中找到此信息。

本快速入门特定于 Windows Server Insider Preview 计划上的 Windows Server 容器。 请先熟悉此计划，然后再继续此快速入门。

## <a name="prerequisites"></a>先决条件：

- 加入 [Windows 预览体验计划](https://insider.windows.com/GettingStarted)并查看使用条款。
- 一个运行 Windows 预览体验计划中最新 Windows Server 版本和/或 Windows 预览体验计划中最新 Windows 10 版本的计算机系统（物理或虚拟）。

> [!IMPORTANT]
> 你必须使用 Windows Server Insider Preview 计划中的 Windows Server 版本或 Windows Insider Preview 计划使用的基本映像中的 Windows 10 版本如下所述。 如果你没有使用这些版本中的其中一个，使用这些基本映像将导致启动容器失败。

## <a name="install-docker-enterprise-edition-ee"></a>安装 Docker 企业版 (EE)

若要使用 Window 容器，则需要安装 Docker EE。 Docker EE 由 Docker 引擎和 Docker 客户端组成。

安装 Docker EE 将用到 OneGet 提供程序 PowerShell 模块。 提供程序将启用计算机上的容器功能，并安装 Docker EE - 此操作需要重启计算机。 打开提升的 PowerShell 会话并运行下列命令。

> [!NOTE]
> 与 Windows Server 会员版本安装 Docker EE 需要比于非会员版本使用不同的 OneGet 提供程序。 如果已安装 Docker EE 和 DockerMsftProvider OneGet 提供程序，请在继续操作前先将它们移除。

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

使用 Windows 容器前，需安装基本映像。 加入 Windows 预览体验成员计划后，你还可以测试我们的最新版本的基本映像。 预览体验成员基本映像现在有 4 个基于 Windows Server 的基本映像可用。 请参阅下表，查看各个基本映像应用于什么用途：

| 基本操作系统映像                       | 用法                      |
|-------------------------------------|----------------------------|
| mcr.microsoft.com/windows/servercore         | 生产和开发 |
| mcr.microsoft.com/windows/nanoserver              | 生产和开发 |
| mcr.microsoft.com/windows/servercore/insider | 仅开发           |
| mcr.microsoft.com/windows/nanoserver/insider        | 仅开发           |

若要拉取 Nano Server 预览体验成员基本映像，请运行以下内容：

```console
docker pull mcr.microsoft.com/nanoserver/insider
```

若要拉取 Windows Server Core 预览体验成员基本映像，请运行以下内容：

```console
docker pull mcr.microsoft.com/windows/servercore/insider
```

> [!IMPORTANT]
> 请阅读 Windows 容器操作系统映像[EULA](../EULA.md )和 Windows 预览体验计划[使用条款](https://www.microsoft.com/software-download/windowsinsiderpreviewserver)。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [生成并运行示例应用程序](./Nano-RS3-.NET-Core-and-PS.md)
