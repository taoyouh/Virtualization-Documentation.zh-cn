## 容器主机部署

**这是初步内容，可能还会更改。**

部署 Windows 容器主机具有不同的步骤，具体取决于操作系统和主机系统类型（物理或虚拟）。 本文档将详细介绍 Windows Server 2016 和 Nano Server 到物理和虚拟系统的部署选项。

有关系统要求的详细信息，请参阅 [Windows 容器主机系统要求](./system_requirements.md)。

PowerShell 脚本可用于自动执行 Windows 容器主机的部署。
- [在新的 Hyper-V 虚拟机中部署容器主机](../quick_start/container_setup.md)。
- [将容器主机部署到现有系统](../quick_start/inplace_setup.md)。
- [在 Azure 中部署容器主机](../quick_start/azure_setup.md)。

### Windows Server 主机

此表中列出的步骤可用于将容器主机部署到 Windows Server 2016 TP4 和 Windows Server Core 2016。 所含内容是 Windows Server 和 Hyper-V 容器所需的配置。

\* 仅在将部署 Hyper-V 容器时需要。  
\*\* 仅在 Docker 用于创建和管理容器时需要。

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:100%" cellpadding="5" cellspacing="5">
<tr valign="top">
<td width="30%"><strong>部署操作</strong></td>
<td width="70%"><strong>详细信息</strong></td>
</tr>
<tr>
<td>[安装容器功能](#role)</td>
<td>容器功能支持使用 Windows Server 和 Hyper-V 容器。</td>
</tr>
<tr>
<td>[启用嵌套虚拟化 *](#nest)</td>
<td>如果容器主机本身是 Hyper-V 虚拟机，则需要启用嵌套虚拟化。</td>
</tr>
<tr>
<td>[配置虚拟处理器 *](#proc)</td>
<td>如果容器主机本身是 Hyper-V 虚拟机，则需要配置至少两个虚拟处理器。</td>
</tr>
<tr>
<td>[启用 Hyper-V 角色 *](#hypv) </td>
<td>仅在将使用 Hyper-V 容器时需要 Hyper-V。</td>
</tr>
<tr>
<td>[创建虚拟交换机](#vswitch)</td>
<td>容器连接到虚拟交换机以获取网络连接。</td>
</tr>
<tr>
<td>[配置 NAT](#nat)</td>
<td>如果使用网络地址转换配置虚拟交换机，则 NAT 本身需要配置。</td>
</tr>
<tr>
<td>[配置 MAC 地址欺骗](#mac)</td>
<td>如果虚拟化容器主机，将需要启用 MAC 欺骗。</td>
</tr>
<tr>
<td>[安装容器操作系统映像](#img)</td>
<td>操作系统映像提供容器部署的基础。</td>
</tr>
<tr>
<td>[安装 Docker **](#docker)</td>
<td>此步骤是可选步骤，但需要执行此步骤才能使用 Docker 创建和管理 Windows 容器。</td>
</tr>
</table>

### Nano Server 主机

此表中列出的步骤可用于将容器主机部署到 Nano Server。 所含内容是 Windows Server 和 Hyper-V 容器所需的配置。

\* 仅在将部署 Hyper-V 容器时需要。  
\*\* 仅在 Docker 用于创建和管理容器时需要。

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:100%" cellpadding="5" cellspacing="5">
<tr valign="top">
<td width="30%"><strong>部署操作</strong></td>
<td width="70%"><strong>详细信息</strong></td>
</tr>
<tr>
<td>[为容器准备 Nano Server](#nano)</td>
<td>使用容器和 Hyper-V 功能准备 Nano Server VHD。</td>
</tr>
<tr>
<td>[启用嵌套虚拟化 *](#nest)</td>
<td>如果容器主机本身是 Hyper-V 虚拟机，则需要启用嵌套虚拟化。</td>
</tr>
<tr>
<td>[配置虚拟处理器 *](#proc)</td>
<td>如果容器主机本身是 Hyper-V 虚拟机，则需要配置至少两个虚拟处理器。</td>
</tr>
<tr>
<td>[创建虚拟交换机](#vswitch)</td>
<td>容器连接到虚拟交换机以获取网络连接。</td>
</tr>
<tr>
<td>[配置 NAT](#nat)</td>
<td>如果使用网络地址转换配置虚拟交换机，则 NAT 本身需要配置。</td>
</tr>
<tr>
<td>[配置 MAC 地址欺骗](#mac)</td>
<td>如果虚拟化容器主机，将需要启用 MAC 欺骗。</td>
</tr>
<tr>
<td>[安装容器操作系统映像](#img)</td>
<td>操作系统映像提供容器部署的基础。</td>
</tr>
<tr>
<td>[安装 Docker **](#docker)</td>
<td>此步骤是可选步骤，但需要执行此步骤才能使用 Docker 创建和管理 Windows 容器。 </td>
</tr>
</table>

## 部署步骤

### <a name=role></a>安装容器功能

可以使用 Windows Server Manager 或 PowerShell 在 Windows Server 2016 或 Windows Server 2016 Core 上安装容器功能。

若要使用 PowerShell 安装角色，请在提升的 PowerShell 会话中运行以下命令。

```powershell
PS C:\> Install-WindowsFeature containers
```
当容器角色安装完成时，需要重新启动系统。

```powershell
PS C:\> shutdown /r 
```

重新启动系统后，使用 `Get-ContainerHost` 命令验证容器角色是否已成功安装：

```powershell
PS C:\> Get-ContainerHost

Name            ContainerImageRepositoryLocation
----            --------------------------------
WIN-LJGU7HD7TEP C:\ProgramData\Microsoft\Windows\Hyper-V\Container Image Store
```

### <a name=nano></a> 准备 Nano Server

部署 Nano Server 涉及到创建准备的虚拟硬盘驱动器，其中包括 Nano Server 操作系统和其他功能包。 本指南快速详细介绍准备 Nano Server 虚拟硬盘驱动器，可用于 Windows 容器。

若要了解 Nano Server 的详细信息以及不同的 Nano Server 部署选项，请参阅 [Nano Server 文档](https://technet.microsoft.com/en-us/library/mt126167.aspx)。

创建名为 `nano` 的文件夹。

```powershell
PS C:\> New-Item -ItemType Directory c:\nano
```

在 Windows Server 媒体上，从 Nano Server 文件夹中找到 `NanoServerImageGenerator.psm1` 和 `Convert-WindowsImage.ps1` 文件。 将这些文件复制到 `c:\nano`。

```powershell
#Set path to Windows Server 2016 Media
PS C:\> $WindowsMedia = "C:\Users\Administrator\Desktop\TP4 Release Media"

PS C:\> Copy-Item $WindowsMedia\NanoServer\Convert-WindowsImage.ps1 c:\nano
PS C:\> Copy-Item $WindowsMedia\NanoServer\NanoServerImageGenerator.psm1 c:\nano
```
运行以下内容来创建 Nano Server 虚拟硬盘驱动器。 `–Containers` 参数指示将安装容器程序包，而 `–Compute` 参数负责 Hyper-V 程序包。 仅在将创建 Hyper-V 容器时需要 Hyper-V。

```powershell
PS C:\> Import-Module C:\nano\NanoServerImageGenerator.psm1
PS C:\> New-NanoServerImage -MediaPath $WindowsMedia -BasePath c:\nano -TargetPath C:\nano\NanoContainer.vhdx -MaxSize 10GB -GuestDrivers -ReverseForwarders -Compute -Containers
```
操作完成后，从 `NanoContainer.vhdx` 文件创建虚拟机。 此虚拟机将运行 Nano Server 操作系统，并带有可选程序包。

### <a name=nest></a>配置嵌套虚拟化

如果容器主机本身将在 Hyper-V 虚拟机上运行，并且还将托管 Hyper-V 容器，则需要启用嵌套虚拟化。 可以使用以下 PowerShell 命令完成此操作。

> 在运行此命令时，必须关闭虚拟机。

```powershell
PS C:\> Set-VMProcessor -VMName <container host vm> -ExposeVirtualizationExtensions $true
```

### <a name=proc></a>配置虚拟处理器

如果容器主机本身将在 Hyper-V 虚拟机上运行，并且还将托管 Hyper-V 容器，该虚拟机将需要至少两个处理器。 这可以通过虚拟机的设置或使用以下 PowerShell 脚本进行配置。

```poweshell
PS C:\> Set-VMProcessor –VMName <VM Name> -Count 2
```

### <a name=hypv></a>启用 Hyper-V 角色

如果要部署 Hyper-V 容器，则需要在容器主机上启用 Hyper-V 角色。 如果容器主机是虚拟机，请确保已启用嵌套虚拟化。 可使用以下 PowerShell 命令在 Windows Server 2016 或 Windows Server 2016 Core 上安装 Hyper-V 角色。 对于 Nano Server 配置，请参阅[准备 Nano Server](#nano)。

```powershell
PS C:\> Install-WindowsFeature hyper-v
```

### <a name=vswitch></a>创建虚拟交换机

需要将每个容器连接到虚拟交换机，以便通过网络进行通信。 虚拟交换机使用 `New-VMSwitch` 命令创建。 容器支持类型为 `External` 或 `NAT` 的虚拟交换机。

此示例创建一个名为“Virtual Switch”、类型为 NAT 且 Nat 子网为 172.16.0.0/12 的虚拟交换机。

```powershell
PS C:\> New-VMSwitch -Name "Virtual Switch" -SwitchType NAT -NATSubnetAddress 172.16.0.0/12
```

### <a name=nat></a>配置 NAT

除了创建虚拟交换机，如果交换机类型为 NAT，还需要创建 NAT 对象。 使用 `New-NetNat` 命令完成此操作。 此示例创建一个 NAT 对象，其名称为 `ContainerNat`，并且地址前缀与分配给容器交换机的 NAT 子网匹配。

```powershell
PS C:\> New-NetNat -Name ContainerNat -InternalIPInterfaceAddressPrefix "172.16.0.0/12"

Name                             : ContainerNat
ExternalIPInterfaceAddressPrefix :
InternalIPInterfaceAddressPrefix : 172.16.0.0/12
IcmpQueryTimeout                 : 30
TcpEstablishedConnectionTimeout  : 1800
TcpTransientConnectionTimeout    : 120
TcpFilteringBehavior             : AddressDependentFiltering
UdpFilteringBehavior             : AddressDependentFiltering
UdpIdleSessionTimeout            : 120
UdpInboundRefresh                : False
Store                            : Local
Active                           : True
```

<a name=mac></a>最后，如果容器主机在 Hyper-V 虚拟机内部运行，必须启用 MAC 欺骗。这使每个容器都可以接收 IP 地址。若要启用 MAC 地址欺骗，请在 Hyper-V 主机上运行以下命令。VMName 属性将是容器主机的名称。

```powershell
PS C:\> Get-VMNetworkAdapter -VMName <contianer host vm> | Set-VMNetworkAdapter -MacAddressSpoofing On
```

### <a name=img></a>安装操作系统映像

操作系统映像用作任何 Windows Server 或 Hyper-V 容器的基础。 此映像用于部署容器，然后可以修改该容器，并将其捕获到新的容器映像中。 已将 Windows Server Core 和 Nano Server 作为基础操作系统创建操作系统映像。

可使用 ContainerProvider PowerShell 模块找到并安装容器操作系统映像。 在使用此模块之前，需要安装它。 以下命令可用于安装该模块。

```powershell
PS C:\> Install-PackageProvider ContainerProvider -Force
```

从 PowerShell OneGet 程序包管理器中返回映像列表：
```powershell
PS C:\> Find-ContainerImage

Name                 Version                 Description
----                 -------                 -----------
NanoServer           10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
WindowsServerCore    10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
```

若要下载和安装 Nano Server 基础操作系统映像，请运行以下内容。

```powershell
PS C:\> Install-ContainerImage -Name NanoServer -Version 10.0.10586.0
Downloaded in 0 hours, 0 minutes, 10 seconds.
```

此外，此命令下载和安装 Windows Server Core 基础操作系统映像。

> **问题：**Save-ContainerImage 和 Install-ContainerImage cmdlet 无法通过远程 PowerShell 会话使用 WindowsServerCore 容器映像。<br />**解决方法：**使用远程桌面登录到计算机并直接使用 Save-ContainerImage cmdlet。

```powershell
PS C:\> Install-ContainerImage -Name WindowsServerCore -Version 10.0.10586.0
Downloaded in 0 hours, 2 minutes, 28 seconds.
```

验证是否已使用 `Get-ContainerImage` 命令安装映像。

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version      IsOSImage
----              ---------    -------      ---------
NanoServer        CN=Microsoft 10.0.10586.0 True
WindowsServerCore CN=Microsoft 10.0.10586.0 True
```
有关容器映像管理的详细信息，请参阅 [Windows 容器映像](../management/manage_images.md)。


### <a name=docker></a>安装 Docker

Windows 中未附带 Docker 守护程序和命令行接口，并且这些功能不与 Windows 容器功能一起安装。 Docker 不是使用 Windows 容器的要求。 如果你希望安装 Docker，请按照本文 [Docker 和 Windows](./docker_windows.md) 中的说明进行操作。




