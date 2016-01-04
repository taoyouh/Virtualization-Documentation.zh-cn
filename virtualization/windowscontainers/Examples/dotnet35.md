# 创建 .NET 3.5 Server Core 容器映像

本指南详细介绍如何创建包含 .NET 3.5 Framework 的 Windows Server Core 容器。 在开始本练习之前，你将需要 Windows Server 2016 .iso 文件，或访问 Windows Server 2016 媒体。

## 准备媒体

在创建支持 .NET 3.5 的容器映像之前，需要暂存 .NET 3.5 程序包以供在容器内使用。 对于本示例，`Microsoft-windows-netfx3-ondemand-package.cab` 文件将从 Windows Server 2016 媒体复制到容器主机。

在容器主机上创建一个名为 `dotnet3.5\source` 的目录。

```powershell
New-Item -ItemType Directory c:\dotnet3.5\source
```

将 `Microsoft-windows-netfx3-ondemand-package.cab` 文件复制到此目录中。 可以在 Windows Server 2016 媒体的 sources\sxs 文件夹下找到此文件。

```powershell
$file = "d:\sources\sxs\Microsoft-windows-netfx3-ondemand-package.cab"
Copy-Item -Path $file -Destination c:\dotnet3.5\source
```

或者，如果容器主机在 Hyper-V 虚拟机中运行，并且使用快速启动脚本部署，则可以运行以下命令。 请注意，此命令在 Hyper-V 主机而不是容器主机上运行。

```powershell
$vm = "<Container Host VM Name>"
$iso = "$((Get-VMHost).VirtualHardDiskPath)".TrimEnd("\") + "\WindowsServerTP4.iso"
Mount-DiskImage -ImagePath $iso
$ISOImage = Get-DiskImage -ImagePath $iso | Get-Volume
$ISODrive = "$([string]$iSOImage.DriveLetter):"
Get-VM -Name $vm | Enable-VMIntegrationService -Name "Guest Service Interface"
Copy-VMFile -Name $vm -SourcePath "$iSODrive\sources\sxs\microsoft-windows-netfx3-ondemand-package.cab" -DestinationPath "c:\dotnet3.5\source\microsoft-windows-netfx3-ondemand-package.cab" -FileSource Host -CreateFullPath
Dismount-DiskImage -ImagePath $iso
```

现在可以创建包含 .NET 3.5 Framework 的容器映像。 可以使用 PowerShell 或 Docker 完成此操作。 下面是这两种方法的示例。

## 创建映像 - PowerShell

若要使用 PowerShell 创建新映像，则需要创建容器、针对所有所需更改对其进行修改，然后将其捕获到新映像中。

从 Windows Server Core 基础映像创建新容器。

```powershell
New-Container -Name dotnet35 -ContainerImageName windowsservercore -SwitchName “Virtual Switch”
```

使用新容器创建共享文件夹。 这将用于使 .NET 3.5 cab 文件在新容器内部可访问。 请注意，运行以下命令时，容器必须停止。

```powershell
Add-ContainerShareFolder -ContainerName dotnet35 -SourcePath C:\dotnet3.5\source -DestinationPath c:\sxs
```

启动容器并运行以下命令来安装 .NET 3.5。

```powershell
Start-Container dotnet35
Invoke-Command -ContainerName dotnet35 -ScriptBlock {Add-WindowsFeature -Name NET-Framework-Core -Source c:\sxs} -RunAsAdministrator
```

安装完成后，停止容器。

```powershell
Stop-Container dotnet35
```

若要从此容器创建映像，请在容器主机上运行以下命令。

```powershell
New-ContainerImage -ContainerName dotnet35 -Name dotnet35 -Publisher Demo -Version 1.0
```

运行 `Get-ContainerImages` 来查看新映像。 现在可以使用此映像运行预安装了 .NET 3.5 Framework 的容器。

```powershell
Get-ContainerImages
```

## 创建映像 - Docker

若要使用 Docker 创建新映像，请使用有关如何创建新映像的说明创建 dockerfile。 此 dockerfile 将随后运行，从而生成新的容器映像。 请注意，以下命令在容器主机 VM 上运行。

创建 dockerfile 并在记事本中打开。

```powershell
New-Item C:\dotnet3.5\dockerfile -Force
Notepad C:\dotnet3.5\dockerfile
```

将此文本复制到 dockerfile 并进行保存。

```powershell
FROM windowsservercore
ADD source /sxs
RUN powershell -Command "& {Add-WindowsFeature -Name NET-Framework-Core -Source c:\sxs}"
```

运行将使用 dockerfile 的 `docker build` 并创建新容器映像。

```powershell
Docker build -t dotnet35 C:\dotnet3.5\
```

运行“docker images”来查看新映像。 现在可以使用此映像运行预安装了 .NET 3.5 Framework 的容器。

```powershell
docker images
```




