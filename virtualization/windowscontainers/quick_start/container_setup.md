



# 将 Windows 容器主机部署到新的 Hyper-V 虚拟机

本文档演示了使用 PowerShell 脚本部署新的 Hyper-V 虚拟机，然后将其配置为 Windows 容器主机的步骤。

若要介绍将 Windows 容器主机脚本化部署到现有虚拟或物理系统的步骤，请参阅[就地 Windows 容器主机部署](./inplace_setup.md)。

**请在安装容器操作系统映像前阅读：**Microsoft Windows Server 预发行软件的许可条款（“许可条款”）适用于使用 Microsoft Windows 容器操作系统映像补充（“补充软件”）。 通过下载和使用补充软件，即表示你同意许可条款，并且如果你未接受许可条款，则无法使用它。 Windows Server 预发行版软件和补充软件均由 Microsoft Corporation 授权。

若要完成此快速入门中的 **Windows Server** 和 **Hyper-V 容器**练习，需要以下内容。

* 运行 Windows 10 版本 10586 或更高版本/Windows Server Technical Preview 4 或更高版本的系统。
* 已启用的 Hyper-V 角色（[请参阅说明](https://msdn.microsoft.com/virtualization/hyperv_on_windows/quick_start/walkthrough_install#UsingPowerShell))）。
* 容器主机映像的 20GB 可用存储，操作系统基础映像和设置脚本。
* Hyper-V 主机上的管理员权限。

> 运行 Hyper-V 容器的虚拟化容器主机将需要嵌套虚拟化。 物理主机和虚拟主机都需要运行支持嵌套虚拟化的操作系统。 有关详细信息，请参阅 [Windows Server 2016 Technical Preview](https://technet.microsoft.com/library/dn765471.aspx#BKMK_nested) 上的 Hyper-V 中的新增功能。

## 在新虚拟机中设置新的容器主机

Windows 容器包含多个组件，如 Windows 容器主机和容器操作系统基础映像。 我们已整合了一个可用于为你下载和配置这些项的脚本。 按照这些步骤部署新的 Hyper-V 虚拟机并将此系统配置为 Windows 容器主机。

以管理员身份启动 PowerShell 会话。 通过右键单击 PowerShell 图标并选择“以管理员身份运行”，或通过从任何 PowerShell 会话运行以下命令，可完成此操作。

``` powershell
PS C:\> start-process powershell -Verb runAs
```

在下载和运行脚本前，请确保已创建外部 Hyper-V 虚拟交换机。 如果没有虚拟交换机，此脚本将失败。

运行以下内容来返回外部虚拟交换机的列表。 如果未返回任何内容，请创建新的外部虚拟交换机，然后继续转到本指南的下一步。

```powershell
PS C:\> Get-VMSwitch | where {$_.SwitchType -eq “External”}
```

使用以下命令来下载配置脚本。 还可以从此位置手动下载该脚本 - [配置脚本](https://aka.ms/tp4/New-ContainerHost)。

``` PowerShell
PS C:\> wget -uri https://aka.ms/tp4/New-ContainerHost -OutFile c:\New-ContainerHost.ps1
```

运行以下命令来创建和配置容器主机，其中 `&lt;containerhost&gt;` 将是虚拟机名称。

``` powershell
PS C:\> powershell.exe -NoProfile c:\New-ContainerHost.ps1 -VMName testcont -WindowsImage ServerDatacenterCore -Hyperv
```

当脚本开始时，系统将提示你输入密码。 这将是分配给 Administrator 帐户的密码。

接下来，系统将要求你阅读并接受许可条款。

```
Before installing and using the Windows Server Technical Preview 4 with Containers virtual machine you must:
    1. Review the license terms by navigating to this link: http://aka.ms/tp4/containerseula
    2. Print and retain a copy of the license terms for your records.
By downloading and using the Windows Server Technical Preview 4 with Containers virtual machine you agree to such
license terms. Please confirm you have accepted and agree to the license terms.
[N] No  [Y] Yes  [?] Help (default is "N"):
```

然后，此脚本将开始下载和配置 Windows 容器组件。 由于下载较大，此过程可能需要很长一段时间。 完成后，你的虚拟机将进行配置，并随时供你使用 PowerShell 和 Docker 创建并管理 Windows 容器和 Windows 容器映像。

当配置脚本完成时，使用在配置过程中指定的密码登录到虚拟机，并确保虚拟机具有有效的 IP 地址。 完成这些项目后，你的系统应已为 Windows 容器准备就绪。

## 后续步骤：开始使用容器

现在你有一个运行 Windows 容器功能的 Windows Server 2016 系统，请跳到以下指南来开始使用 Windows Server 和 Hyper-V 容器。

[快速入门：Windows 容器和 PowerShell](./manage_powershell.md)  
[快速入门：Windows 容器和 Docker](./manage_docker.md)






<!--HONumber=Feb16_HO4-->


