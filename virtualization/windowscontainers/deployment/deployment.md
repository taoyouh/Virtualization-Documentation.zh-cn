# 容器主机部署 - Windows Server

**这是初步内容，可能还会更改。**

部署 Windows 容器主机具有不同的步骤，具体取决于操作系统和主机系统类型（物理或虚拟）。 本文档中的步骤用于在物理系统或虚拟系统上将 Windows 容器主机部署到 Windows Server 2016 或 Windows Server Core 2016。 若要将 Windows 容器主机安装到 Nano Server，请参阅[容器主机部署 - Nano Server](./deployment_nano.md)。

有关系统要求的详细信息，请参阅 [Windows 容器主机系统要求](./system_requirements.md)。

PowerShell 脚本也可用于自动部署 Windows 容器主机。
- [在新的 Hyper-V 虚拟机中部署容器主机](../quick_start/container_setup.md)。
- [将容器主机部署到现有系统](../quick_start/inplace_setup.md)。

# Windows Server 主机

此表中列出的步骤可用于将容器主机部署到 Windows Server 2016 和 Windows Server 2016 Core 中。 所含内容是 Windows Server 和 Hyper-V 容器所需的配置。

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
<td>[创建虚拟交换机](#vswitch)</td>
<td>容器连接到虚拟交换机以获取网络连接。</td>
</tr>
<tr>
<td>[配置 NAT](#nat)</td>
<td>如果使用网络地址转换配置虚拟交换机，则 NAT 本身需要配置。</td>
</tr>
<tr>
<td>[安装容器操作系统映像](#img)</td>
<td>操作系统映像提供容器部署的基础。</td>
</tr>
<tr>
<td>[安装 Docker](#docker)</td>
<td>可选步骤，但需要执行此步骤才能使用 Docker 创建和管理 Windows 容器。</td>
</tr>
</table>

使用 Hyper-V 容器时需要采取这些步骤。 请注意，标记有 * 的步骤仅在容器主机本身即是 Hyper-V 虚拟机时才需要。

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:100%" cellpadding="5" cellspacing="5">
<tr valign="top">
<td width="30%"><strong>部署操作</strong></td>
<td width="70%"><strong>详细信息</strong></td>
</tr>
<tr>
<td>[启用 Hyper-V 角色](#hypv) </td>
<td>仅在将使用 Hyper-V 容器时需要 Hyper-V。</td>
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
<td>[禁用动态内存 *](#dyn)</td>
<td>如果容器主机本身是 Hyper-V 虚拟机，则必须禁用动态内存。</td>
</tr>
<tr>
<td>[配置 MAC 地址欺骗 *](#mac)</td>
<td>如果虚拟化容器主机，将需要启用 MAC 欺骗。</td>
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

### <a name=vswitch></a>创建虚拟交换机

需要将每个容器连接到虚拟交换机，以便通过网络进行通信。 使用 `New-VMSwitch` 命令创建虚拟交换机。 容器支持类型为 `External` 或 `NAT` 的虚拟交换机。 有关 Windows 容器网络的详细信息，请参阅[容器网络](../management/container_networking.md)。

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

### <a name=img></a>安装操作系统映像

操作系统映像用作任何 Windows Server 或 Hyper-V 容器的基础。 此映像用于部署容器，然后可以修改该容器，并将其捕获到新的容器映像中。 已将 Windows Server Core 和 Nano Server 作为基础操作系统创建操作系统映像。

可使用 ContainerProvider PowerShell 模块找到并安装容器操作系统映像。 在使用此模块之前，需要安装它。 可以使用以下命令安装此模块。

```powershell
PS C:\> Install-PackageProvider ContainerProvider -Force
```

使用 `Find-ContainerImage` 从 PowerShell OneGet 程序包管理器中返回映像列表：
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

```powershell
PS C:\> Install-ContainerImage -Name WindowsServerCore -Version 10.0.10586.0

Downloaded in 0 hours, 2 minutes, 28 seconds.
```

问题：Save-ContainerImage 和 Install-ContainerImage cmdlet 无法通过远程 PowerShell 会话使用 WindowsServerCore 容器映像。<br />解决方法：使用远程桌面登录计算机并直接使用 Save-ContainerImage cmdlet。

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


## Hyper V 容器主机

### <a name=hypv></a>启用 Hyper-V 角色

如果要部署 Hyper-V 容器，则需要在容器主机上启用 Hyper-V 角色。 可使用 `Install-WindowsFeature` 命令在 Windows Server 2016 或 Windows Server 2016 Core 上安装 Hyper-V 角色。 如果容器主机本身是 Hyper-V 虚拟机，则需要首先启用嵌套虚拟化。 若要执行此操作，请参阅[配置嵌套虚拟化](#nest)。

```powershell
PS C:\> Install-WindowsFeature hyper-v
```

### <a name=nest></a>配置嵌套虚拟化

如果容器主机本身将在 Hyper-V 虚拟机上运行，并且还将托管 Hyper-V 容器，则需要启用嵌套虚拟化。 可以使用以下 PowerShell 命令完成此操作。

注意 - 在运行此命令时，必须关闭虚拟机。

```powershell
PS C:\> Set-VMProcessor -VMName <VM Name> -ExposeVirtualizationExtensions $true
```

### <a name=proc></a>配置虚拟处理器

如果容器主机本身将在 Hyper-V 虚拟机上运行，并且还将托管 Hyper-V 容器，该虚拟机将需要至少两个处理器。 这可以通过虚拟机的设置或使用以下命令进行配置。

注意 - 在运行此命令时，必须关闭虚拟机。

```poweshell
PS C:\> Set-VMProcessor –VMName <VM Name> -Count 2
```

### <a name=dyn></a>禁用动态内存

如果容器主机本身是 Hyper-V 虚拟机，则必须在容器主机虚拟机上禁用动态内存。 这可以通过虚拟机的设置或使用以下命令进行配置。

注意 - 在运行此命令时，必须关闭虚拟机。

```poweshell
PS C:\> Set-VMMemory <VM Name> -DynamicMemoryEnabled $false
```

### <a name=mac></a>配置 MAC 地址欺骗

最后，如果容器主机在 Hyper-V 虚拟机内部运行，必须启用 MAC 欺骗。 这使每个容器都可以接收 IP 地址。 若要启用 MAC 地址欺骗，请在 Hyper-V 主机上运行以下命令。 VMName 属性将是容器主机的名称。

```powershell
PS C:\> Get-VMNetworkAdapter -VMName <VM Name> | Set-VMNetworkAdapter -MacAddressSpoofing On
```





<!--HONumber=Feb16_HO1-->
