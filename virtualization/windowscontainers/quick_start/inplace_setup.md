# 将 Windows 容器主机部署到现有虚拟或物理系统

本文档介绍使用 PowerShell 脚本在现有物理或虚拟系统上部署并配置 Windows 容器角色的步骤。

若要介绍脚本化部署配置为 Windows 容器主机的新 Hyper-V 虚拟机的步骤，请参阅[新 Hyper-V Windows 容器主机](./container_setup.md)。

**请在安装容器操作系统映像前阅读：**Microsoft Windows Server 预发行软件的许可条款（“许可条款”）适用于使用 Microsoft Windows 容器操作系统映像补充（“补充软件”）。 通过下载和使用补充软件，即表示你同意许可条款，并且如果你未接受许可条款，则无法使用它。 Windows Server 预发行版软件和补充软件均由 Microsoft Corporation 授权。

为了完成本快速入门中的 Windows Server 容器和 Hyper-V 容器练习，需要以下内容。

* 运行 Windows Server Technical Preview 4 或更高版本的系统。
* 容器主机映像、基础操作系统映像和安装程序脚本有 10GB 的可用存储。
* 系统上的管理员权限。

## 为容器安装现有的虚拟机或裸机主机。

Windows 容器需要容器操作系统基础映像。 我们已整合了一个可用于为你下载并安装此映像的脚本。 按照以下步骤将你的系统配置为 Windows 容器主机。 有关详细信息，请参阅 [Windows Server 2016 Technical Preview](https://tnstage.redmond.corp.microsoft.com/en-US/library/dn765471.aspx#BKMK_nested) 上的 Hyper-V 中的新增功能。

以管理员身份启动 PowerShell 会话。 可通过从命令行运行以下命令来完成此操作。

``` powershell
PS C:\> powershell.exe
```

请确保这些窗口的标题为“管理员：Windows PowerShell”。 如果未说明是管理员，请运行此命令以使用管理权限运行：

``` powershell
PS C:\> start-process powershell -Verb runas
```

使用以下命令下载安装脚本。 还可以从此位置手动下载该脚本 - [配置脚本](https://aka.ms/tp4/Install-ContainerHost)。

``` PowerShell
PS C:\> wget -uri https://aka.ms/tp4/Install-ContainerHost -OutFile C:\Install-ContainerHost.ps1
```

下载完成后，执行该脚本。
``` PowerShell
PS C:\> C:\Install-ContainerHost.ps1 -HyperV
```

然后，此脚本将开始下载和配置 Windows 容器组件。 由于下载较大，此过程可能需要很长一段时间。 在此过程中，计算机可能重新启动。 完成后，你的计算机将进行配置，并随时供你使用 PowerShell 和 Docker 创建并管理 Windows 容器和 Windows 容器映像。

完成这些项目后，你的系统应已为 Windows 容器准备就绪。

## 后续步骤：开始使用容器

现在你有一个运行 Windows 容器功能的 Windows Server 2016 系统，请跳到以下指南来开始使用 Windows Server 和 Hyper-V 容器。

[快速入门：Windows 容器和 Docker](./manage_docker.md)

[快速入门：Windows 容器和 PowerShell](./manage_powershell.md)




<!--HONumber=Jan16_HO1-->
