



# Windows Server 容器管理

**这是初步内容，可能还会更改。**

容器生命周期包括启动、停止和删除容器等操作。 在执行这些操作时，你可能还需要检索容器映像的列表、管理容器网络并限制容器资源。 本文档将详细介绍使用 PowerShell 的基本容器管理任务。

有关使用 Docker 管理 Windows 容器的文档，请参阅 Docker 文档[与容器结合使用](https://docs.docker.com/userguide/usingdocker/)。

## PowerShell

### 创建容器

创建新容器时，你需要将用作容器基础的容器映像名称。 使用 `Get-ContainerImage` 命令可以找到该名称。

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version         IsOSImage
----              ---------    -------         ---------
NanoServer        CN=Microsoft 10.0.10584.1000 True
WindowsServerCore CN=Microsoft 10.0.10584.1000 True
```

使用 `New-Container` 命令可创建新容器。 此外可以使用 `-ContainerComputerName` 参数为容器提供 NetBIOS 名称。

```powershell
PS C:\> New-Container -ContainerImageName WindowsServerCore -Name demo -ContainerComputerName demo

Name State Uptime   ParentImageName
---- ----- ------   ---------------
demo  Off   00:00:00 WindowsServerCore
```

创建容器后，将网络适配器添加到该容器。

```powershell
PS C:\> Add-ContainerNetworkAdapter -ContainerName demo
```

为了将容器的网络适配器连接到虚拟交换机，需要使用交换机名称。 使用 `Get-VMSwitch` 返回虚拟交换机的列表。

```powershell
PS C:\> Get-VMSwitch

Name SwitchType NetAdapterInterfaceDescription
---- ---------- ------------------------------
DHCP External   Microsoft Hyper-V Network Adapter
NAT  NAT
```

使用 `Connect-ContainerNetworkAdapter` 将网络适配器连接到虚拟交换机。 **请注意** – 此操作也可以在使用 –SwitchName 参数创建该容器时完成。

```powershell
PS C:\> Connect-ContainerNetworkAdapter -ContainerName demo -SwitchName NAT
```

### 启动容器

若要启动此容器，将枚举一个表示该容器的 PowerShell 对象。 通过将 `Get-Container` 的输出放入 PowerShell 变量中可以完成此操作。

```powershell
PS C:\> $container = Get-Container -Name demo
```

然后，可以将此数据与 `Start-Container` 命令结合使用来启动该容器。

```powershell
PS C:\> Start-Container $container
```

以下脚本将启动主机上的所有容器。

```powershell
PS C:\> Get-Container | Start-Container
```

### 与容器连接

可以使用 PowerShell Direct 连接到容器。 如果你需要手动执行某个任务（如安装软件、启动进程或容器疑难解答），这可能会有用。 由于 PowerShell Direct 正在使用中，因此可以创建与该容器的 PowerShell 会话，而不考虑网络配置。 有关 PowerShell Direct 的详细信息，请参阅 [PowerShell Direct 博客](http://blogs.technet.com/b/virtualization/archive/2015/05/14/powershell-direct-running-powershell-inside-a-virtual-machine-from-the-hyper-v-host.aspx)。

若要创建与该容器的交互式会话，请使用 `Enter-PSSession` 命令。

 ```powershell
PS C:\> Enter-PSSession -ContainerName demo -RunAsAdministrator
 ```

请注意，创建远程 PowerShell 会话后，shell 提示用于反映容器名称的更改。

```powershell
[demo]: PS C:\>
```

命令也可以针对容器运行，而无需创建持久性 PowerShell 会话。 若要执行此操作，请使用 `Invoke-Command`。

以下示例在容器中创建了一个名为“Application”的文件夹。

```powershell

PS C:\> Invoke-Command -ContainerName demo -ScriptBlock {New-Item -ItemType Directory -Path c:\application }

Directory: C:\
Mode                LastWriteTime         Length Name                                                 PSComputerName
----                -------------         ------ ----                                                 --------------
d-----       10/28/2015   3:31 PM                application                                          demo
```

### 停止容器

若要停止此容器，将需要一个表示该容器的 PowerShell 对象。 通过将 `Get-Container` 的输出放入 PowerShell 变量中可以完成此操作。

```powershell
PS C:\> $container = Get-Container -Name demo
```

然后，可以将其与 `Stop-Container` 命令结合使用来停止该容器。

```powershell
PS C:\> Stop-Container $container
```

以下内容将停止主机上的所有容器。

```powershell
PS C:\> Get-Container | Stop-Container
```

### 删除容器

当不再需要容器时，可以将其删除。 若要删除容器，该容器需要处于停止状态，并且需要创建表示该容器的 PowerShell 对象。

```powershell
PS C:\> $container = Get-Container -Name demo
```

若要删除容器，请使用 `Remove-Container` 命令。

```powershell
PS C:\> Remove-Container $container -Force
```

以下内容将删除主机上的所有容器。

```powershell
PS C:\> Get-Container | Remove-Container -Force
```

## Docker

### 创建容器

使用 `docker run` 通过 Docker 创建容器。

```powershell
PS C:\> docker run -p 80:80 windowsservercoreiis
```

有关 Docker run 命令的详细信息，请参阅 [Docker run 参考}( https://docs.docker.com/engine/reference/run/)。

### 停止容器

使用 `docker stop` 命令通过 Docker 停止容器。

```powershell
PS C:\> docker stop tender_panini

tender_panini
```

此示例使用 Docker 停止所有运行的容器。

```powershell
PS C:\> docker stop $(docker ps -q)

fd9a978faac8
b51e4be8132e
```

### 删除容器

若要使用 Docker 删除容器，请使用 `docker rm` 命令。

```powershell
PS C:\> docker rm prickly_pike

prickly_pike
```

使用 Docker 删除所有容器。

```powershell
PS C:\> docker rm $(docker ps -a -q)

dc3e282c064d
2230b0433370
```

有关 Docker rm 命令的详细信息，请参阅 [Docker rm 参考](https://docs.docker.com/engine/reference/commandline/rm/)。






<!--HONumber=Feb16_HO4-->


