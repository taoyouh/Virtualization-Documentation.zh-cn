---
author: scooley
redirect_url: ../quick_start/manage_docker
---


# PowerShell 与 Docker 在管理 Windows 容器方面的比较

通过使用内置 Windows 工具（在此预览版中是 PowerShell）和开源管理工具（例如 Docker）管理 Windows 容器的方法有很多种。  
下面提供了分别概述这两种方法的指南：
* [使用 Docker 管理 Windows 容器](../quick_start/manage_docker.md)
* [使用 PowerShell 管理 Windows 容器](../quick_start/manage_powershell.md)

此页面是比较 Docker 工具和 PowerShell 管理工具的更深入参考。

## 适用于容器的 PowerShell 与 Hyper-V VM

你可以使用 PowerShell cmdlet 创建、运行 Windows 容器，并与之交互。 已经内置了运行所需的所有内容。

如果你已使用 Hyper-V PowerShell，则会非常熟悉 cmdlet 设计。 大量的工作流与使用 Hyper-V 模块管理虚拟机的方式类似。 你拥有 `New-Container`、`Get-Container`、`Start-Container` 和 `Stop-Container`，而没有 `New-VM`、`Get-VM`、`Start-VM` 和 `Stop-VM`。 有许多特定于容器的 cmdlet 和参数，但是 Windows 容器的常规生命周期和管理与 Hyper-V VM 的常规生命周期和管理大致相似。

## PowerShell 管理与 Docker 相比如何？

Containers PowerShell cmdlet 公开与 Docker API 有些许不同的 API；一般情况下，cmdlet 在操作中更为具体。 某些 Docker 命令在 PowerShell 中具有相当简单的并行命令：

| Docker 命令| PowerShell Cmdlet|
|----|----|
| `docker ps -a`| `Get-Container`|
| `docker images`| `Get-ContainerImage`|
| `docker rm`| `Remove-Container`|
| `docker rmi`| `Remove-ContainerImage`|
| `docker create`| `New-Container`|
| `docker commit <容器 ID>`| `New-ContainerImage -Container <容器>`|
| `docker load &lt;tarball&gt;`| `Import-ContainerImage <AppX 程序包>`|
| `docker save`| `Export-ContainerImage`|
| `docker start`| `Start-Container`|
| `docker stop`| `Stop-Container`|

PowerShell cmdlet 并非就是完美的奇偶校验，并且有很多命令我们没有提供 PowerShell 替换*（值得注意的是 `docker build` 和 `docker cp`）。 但你需要注意的是，`docker run` 没有单个单行替换。

\* 可能还会更改。

### 但我需要 docker run！发生什么事了？

我们在此处执行几项操作以便让已经知道如何使用 PowerShell 的用户对交互模型更熟悉一些。 当然，如果你习惯 docker 的操作方式，这会产生一些心理变化。

1.  PowerShell 模型中容器的生命周期略有不同。 在容器 PowerShell 模块中，我们公开了 `New-Container`（会创建已停止的新容器）和 `Start-Container` 的更具体操作。

  在创建和启动容器之间，你还可以配置容器的设置；对于 TP3，我们计划公开的另一项唯一配置是设置容器的网络连接功能。 使用 (Add/Remove/Connect/Disconnect/Get/Set)-ContainerNetworkAdapter cmdlet。

2.  当前无法在启动时传递要在容器内运行的命令。但是，你仍可以使用 `Enter-PSSession -ContainerId -ContainerId <正在运行的容器的 ID>` 获取正在运行的容器的交互式 PowerShell 会话，并且你可以使用 `Invoke-Command -ContainerId <容器 ID> -ScriptBlock { 要在容器内运行的代码 }` 或 `Invoke-Command -ContainerId <容器 ID> -FilePath <脚本路径>` 在正在运行的容器中执行命令。  
这两种命令均支持高特权操作的可选 `-RunAsAdministrator` 标志。


## 需要注意的问题和已知问题

1.  目前，Containers cmdlet 不了解通过 Docker 创建的任何容器或映像，而 Docker 也对通过 PowerShell 创建的容器和映像一无所知。 如果已在 Docker 中创建它，则使用 Docker 进行管理；如果已通过 PowerShell 创建它，则使用 PowerShell 进行管理。

2.  为改善最终用户体验，我们有很多工作要做 - 更完善的错误消息、更精良的进程报告、无效事件字符串等。 如果你碰巧遇到希望获取更多或更完善信息的情况，请随时将建议发送到论坛。

## 快速浏览

以下介绍了一些常见工作流。

这假定你已经安装了名为“ServerDatacenterCore”的操作系统容器映像，并（使用 New-VMSwitch）创建了名为“Virtual Switch”的虚拟交换机。

``` PowerShell
### 1. Enumerating images
# At this point, you can enumerate the images on the system:
Get-ContainerImage

# Get-ContainerImage also accepts filters.
# For example, this will return all container images whose Name starts with S (case-insensitive):
Get-ContainerImage -Name S*

# You can save the results of this to a variable.
# (If you're not familiar with PowerShell, the "$" denotes a variable.)
$baseImage = Get-ContainerImage -Name ServerDatacenterCore
$baseImage

### 2. Creating and enumerating containers
# Now, we can create a new container using this image:
New-Container -Name "MyContainer" -ContainerImage $baseImage -SwitchName "Virtual Switch"

# Now we can enumerate all containers.
Get-Container

# Similarly, we can save this container to a variable:
$container1 = Get-Container -Name "MyContainer"

### 3. Starting containers, interacting with running containers, and stopping containers
# Now let's go ahead and start the container.
Start-Container -Name "MyContainer"

# (We could've also started this container using "Start-Container -Container $container1".)

# With the container now running, let's go ahead and enter an interactive PowerShell session:
Enter-PSSession -ContainerId $container1.Id

# This should eventually bring up a PowerShell prompt from inside the container.
# You can try all the things that you did in the interactive cmd prompt given by "docker run -it".
# For now, just to prove we've been here, we can create a new file:
cd \
mkdir Test
cd Test
echo "hello world" > hello.txt
exit

# Now we should be back in the outside world. Even though we've exited the PowerShell session,
# the container itself is still running, as you can see by printing out the container again:
$container1

# Before we can commit this container to a new image, we need to stop the container.
# Let's do that now.
Stop-Container -Container $container1

### 4. Creating a new container image
# And now let's commit it to a new image.
$image1 = New-ContainerImage -Container $container1 -Publisher Test -Name Image1 -Version 1.0

# Enumerate all the images again, for sanity's sake:
Get-ContainerImage

# Rinse and repeat! Make another container based on the new image.
$container2 = New-Container -Name "MySecondContainer" -ContainerImage $image1 -SwitchName "Virtual Switch"

# (If you like, you can start the second container and verify that the new file
# "\Test\hello.txt" is there as expected.)

### 5. Removing a container
# The first container we created is now stopped. Let's get rid of it:
Remove-Container -Container $container1

# And confirm that it's gone:
Get-Container

### 6. Exporting, removing, and importing images
# For images that aren't the base OS image, we can export them into an .appx package file.
Export-ContainerImage -Image $image1 -Path "C:\exports"
# This should create a .appx file in the C:\exports folder.
# If you've given your image the same publisher, name, and version we used earlier,
# you'd expect the resulting .appx to be named "CN=Test_Image1_1.0.0.0.appx".

# Before we can try importing the image again, we need to remove the image.
# (If you have any running containers that depend on this image, you'll want to stop them first.)
Remove-ContainerImage -Image $image1

# Now let's import the image again:
Import-ContainerImage -Path C:\exports\CN=Test_Image1_1.0.0.0.appx

# We'd previously created a container dependent on this image. You should be able to start it:
Start-Container -Container $container2 
```

### 生成自己的示例

你可以使用 `Get-Command -Module Containers` 查看所有 Containers cmdlet。 还有其他若干 cmdlet 未在此作介绍，留待你自己去了解。    
**注意** 这不会返回作为核心 PowerShell 的一部分的 `Enter-PSSession` 和 `Invoke-Command` cmdlet。

你还可以使用 `Get-Help [cmdlet name]` 获取有关任何 cmdlet 的帮助，使用 `[cmdlet name] -?` 也可实现此目的。 如今，帮助输出会自动生成并直接告诉你命令语法；随着离最终确定 cmdlet 设计越来越近，我们会添加更多文档。

PowerShell ISE 能够更好地发现语法，但在没有熟练使用 PowerShell 前，不用去涉猎它。 如果在支持它的 SKU 上运行，请尝试启动 ISE、打开“命令”窗格，然后选择“容器”模块，这会向你显示 cmdlet 及其参数集的图形表示形式。

PS：以下是编写一些我们已在 ersatz `docker run`中看到的 cmdlet 的 PowerShell 函数，仅用于证明上述操作可以执行。 （澄清一点，这只是证明概念，并非进行活动开发。）

``` PowerShell
function Run-Container ([string]$ContainerImageName, [string]$Name="fancy_name", [switch]$Remove, [switch]$Interactive, [scriptblock]$Command) {
    $image = Get-ContainerImage -Name $ContainerImageName
    $container = New-Container -Name $Name -ContainerImage $image
    Start-Container $container

    if ($Interactive) {
         Start-Process powershell ("-NoExit", "-c", "Enter-PSSession -ContainerId $($container.Id)") -Wait
    } else {
        Invoke-Command -ContainerId $container.Id -ScriptBlock $Command
    }

    Stop-Container $container

    if ($Remove) {
        Remove-Container $container -Force
    }
} 
```

## Docker

Windows 容器可以通过 Docker 命令管理。 虽然 Windows 容器应该与其对应的 Linux 容器相媲美，并且通过 Docker 具有相同的管理体验，但是有一些 Docker 命令就是与 Windows 容器不兼容。 其他命令尚未进行测试（我们正努力解决此问题）。

为了不复制在 Docker 中提供的 API 文档，<a href="https://docs.docker.com/engine/reference/commandline/cli/" >此处</a>是指向它的管理 API 的链接。 其演练很出色。

我们在我们的“正在进行的工作”文档中跟踪适用于/不适用于 Docker API 的内容。






<!--HONumber=Apr16_HO4-->


