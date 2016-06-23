---
title: Windows 容器工作正在进行中
description: Windows 容器工作正在进行中
keywords: docker, containers
author: scooley
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 5d9f1cb4-ffb3-433d-8096-b085113a9d7b
---

# 正在进行的工作

如果在此处没有看到你的问题已解决或仍有问题，请将其发布到[论坛](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)。

-----------------------

## 常规功能

### 容器和主机版本号匹配
Windows 容器要求操作系统映像匹配容器主机的版本和修补程序级别。 不匹配会导致容器和/或主机出现潜在的不稳定性和/或不可预测的行为。

如果针对 Windows 容器主机操作系统安装更新，将需要更新容器基础操作系统映像，以便具有匹配的更新。
<!-- Can we give examples of behavior or errors?  Makes it more searchable -->

**解决方法：**   
下载并安装匹配容器主机的操作系统版本和修补程序级别的容器基础映像。

### 默认的防火墙行为
在容器主机和容器环境中，你只具有容器主机的防火墙。 容器主机中配置的所有防火墙规则将传播到其所有容器。

### Windows 容器启动缓慢
如果你的容器的启动时间超过 30 秒，有可能正在执行许多重复的病毒扫描。

Windows Defender 等许多反恶意软件解决方案可能正在不必要地扫描容器映像内的文件（包括所有操作系统二进制文件）和容器操作系统映像中的文件。  在创建新容器，且从反恶意软件的角度来看，所有“容器文件”似乎是以前尚未扫描的新文件时，可能会发生此情况。  因此当容器内的进程试图读取这些文件时，反恶意软件组件会先扫描它们，然后再对文件进行访问。  实际上，将容器映像导入或提取到服务器时，已对这些文件进行了扫描。 从 Windows Server 技术预览版 5 开始，将推出基础结构，以便反恶意软件解决方案（包括 Windows Defender）能够识别这些情况并可相应地采取措施以避免多次扫描。 可使用[此处](https://msdn.microsoft.com/en-us/windows/hardware/drivers/ifs/anti-virus-optimization-for-windows-containers)所介绍的指南更新反恶意软件解决方案并避免多个扫描。 

### 如果内存限制为 < 48MB，启动/停止有时会失败。
当内存限制为小于 48MB 时，Windows 容器会遇到随机的不一致错误。

运行以下 PowerShell 并重复执行启动、停止操作多次会导致启动或停止失败。

```PowerShell
new-container "Test" -containerimagename "WindowsServerCore" -MaximumBytes 32MB
start-container test
stop-container test
```

**解决方法：**  
将内存值更改为 48MB。 


### 当 4 核 VM 上的处理器计数为 1 或 2 时，启动容器会失败

Windows 容器无法启动，并带有错误：  
`failed to start: This operation returned because the timeout period expired. (0x800705B4).`

当 4 核 VM 上的处理器计数设置为 1 或 2 时，将发生此情况。

``` PowerShell
new-container "Test2" -containerimagename "WindowsServerCore"
Set-ContainerProcessor -ContainerName test2 -Maximum 2
Start-Container test2

Start-Container : 'test2' failed to start.
'test2' failed to start: This operation returned because the timeout period expired. (0x800705B4).
'test2' failed to start. (Container ID 133E9DBB-CA23-4473-B49C-441C60ADCE44)
'test2' failed to start: This operation returned because the timeout period expired. (0x800705B4). (Container ID
133E9DBB-CA23-4473-B49C-441C60ADCE44)
At line:1 char:1
+ Start-Container test2
+ ~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : OperationTimeout: (:) [Start-Container], VirtualizationException
    + FullyQualifiedErrorId : OperationTimeout,Microsoft.Containers.PowerShell.Cmdlets.StartContainer
PS C:\> Set-ContainerProcessor -ContainerName test2 -Maximum 3
PS C:\> Start-Container test2
```

**解决方法：**  
增加可用于容器的处理器、不显式指定可用于容器的处理器或减少可用于 VM 的处理器。

--------------------------

## 网络

### 网络隔离舱隔离及含义
每个容器使用网络隔离舱提供隔离。 所有附加到给定容器的容器网络适配器（终结点），都将驻留在同一个网络隔离舱中。 你可能无法使用同一 IP 地址或端口访问两个不同的容器终结点，具体取决于所使用的网络模式（驱动程序）。 此外，Windows 防火墙规则不识别隔离舱或容器，因此采用的任何防火墙规则将应用到容器主机上的所有容器，而不考虑特定的终结点。

*** 透明网络 ***


***NAT 网络***可以使用按每个终结点应用的 NAT 端口转发规则来公开附加到单个容器的多个终结点。 当映射到同一内部端口（容器内）时，这些转发规则必须使用不同的外部端口（容器主机上）。  但是，如上所述，所应用的防火墙规则将跨容器主机作用于全局范围。



### 静态 NAT 映射可能与通过 Docker 的端口映射发生冲突
从 Windows Server 技术预览版 5 开始，NAT 创建规则和端口映射规则已集成到 *ContainerNetwork* cmdlet 和 docker 命令中。 Windows 主机网络服务 (HNS) 将以你的名义管理 NAT。 但有可能发生这样的情况，即外部客户端可能会尝试使用 HNS 创建的同一 NAT 来创建相同的端口映射规则。


下面的示例描述了端口 80 上产生的与静态映射的冲突以及发生这种冲突时 docker 报告的错误。
```
C:\Users\Administrator>docker run -it -p 80:80 windowsservercore cmd
docker: Error response from daemon: failed to create endpoint berserk_bassi on network nat: hnsCall failed in Win32: The remote procedure call failed. (0x6be).
```


***缓解措施*** 由于 HNS 将会管理 NAT，所以通常情况下，不大可能会执行这种操作。 应使用 `docker run -p <external>:<internal>` 或 Add-ContainerNetworkAdapterStaticMapping 创建所有端口转发/映射规则。 但是，如果 HNS 不自动清理映射，可通过使用 PowerShell 删除端口映射来解决这个错误。 这将删除上例中产生的端口 80 上的冲突。
```powershell
Get-NetNatStaticMapping | ? ExternalPort -eq 80 | Remove-NetNatStaticMapping
```


### Windows 容器未获得 IP
如果你要连接到具有 DHCP VM 开关的 Windows 容器，容器主机可以接收 IP，但该容器不可以。

容器获取了 169.254.***.*** APIPA IP 地址。

**解决办法：**这是共享内核的副作用。  所有容器实际上具有相同的 MAC 地址。

在承载容器主机 VM 的物理主机上启用 MAC 地址欺骗。

使用 PowerShell 可以实现此操作
```
Get-VMNetworkAdapter -VMName "[YourVMNameHere]"  | Set-VMNetworkAdapter -MacAddressSpoofing On
```
--------------------------

## 应用程序兼容性

存在许多有关可在 Windows 容器中使用哪些应用程序的问题，我们决定为应用程序兼容性信息单独编写[一篇文章](../reference/app_compat.md)。

其中还包含一些最常见的问题。

### 事件日志在容器内不可用

`get-eventlog Application` 等 PowerShell 命令和用于查询事件日志的 API 将返回类似于这样的错误：
```
get-eventlog : Cannot open log Application on machine .. Windows has not provided an error code.
At line:1 char:1
```

作为解决方法，可以将此步骤添加到 Dockerfile 中。 使用所包含的此步骤生成的映像将启用事件日志记录。
```
RUN powershell.exe -command Set-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Control\WMI\Autologger\EventLog-Application Start 1
```


### Localdb 实例 API 方法调用中出现异常错误
Localdb 实例 API 方法调用中出现异常错误

### RTerm 不起作用
已安装 RTerm，但在 Windows Server 容器中无法启动。

错误：  
```
'C:\Program' is not recognized as an internal or external command,
operable program or batch file.
```


### 容器：未安装 Visual C++ 运行时 x64/x86 2015

观察到的行为：容器内：
```
C:\build\vcredist_2015_x64.exe /q /norestart
C:\build>echo %errorlevel%
0
C:\build>wmic product get
No Instance(s) Available.
```

这是重复数据删除筛选器的互操作问题。 重复数据删除会检查重命名目标，以查看其是否是删除重复数据的文件。 它发出的创建将失败（带有 `STATUS_IO_REPARSE_TAG_NOT_HANDLED`），因为 Windows Server 容器筛选器位于重复数据删除之上。


有关可以容器化哪些应用程序的详细信息，请参阅[应用程序兼容性文章](../reference/app_compat.md)。

--------------------------


## Docker 管理

### 并非所有 Docker 命令都起作用
* Docker 执行程序无法在 Hyper-V 容器中运行。

如果任何不在此列表中的内容无法运行（或者如果某个命令的运行方式与预期迥然不同），请通过[论坛](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)告知我们。

### 粘贴到交互式 Docker 会话中的命令限制为 50 个字符
粘贴到交互式 Docker 会话中的命令限制为 50 个字符。  
如果将命令行复制到交互式 Docker 会话中，它当前限制为 50 个字符。 粘贴的字符串会简单截断。

这不是设计使然，我们正致力于提升该限制。

### Net use 错误
Net use 返回系统错误 1223，而不是提示输入用户名或密码

**解决方法：**  
运行 net use 时指定用户名和密码。

``` PowerShell
net use S: \\your\sources\here /User:shareuser [yourpassword]
``` 


--------------------------



## 远程桌面 

Windows 容器无法通过 TP5 中的 RDP 会话进行管理/与之交互。

--------------------------

### 无法使用“exit”退出 Nano Server 容器主机中的容器。
如果你尝试退出 Nano Server 容器主机中的容器，使用“exit”将断开与 Nano Server 容器主机的连接，而且不会退出该容器。

**解决方法：**改为使用 Exit-PSSession 退出该容器。

随时在[论坛](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)中发布功能请求。 


--------------------------


## 用户和域

### 本地用户
可以创建本地用户帐户并将其用于在容器中运行 Windows 服务和应用程序。


### 域成员身份
容器无法加入 Active Directory 域，并且无法以域用户的身份运行服务或应用程序、服务帐户或计算机帐户。 

容器旨在快速启动到环境几乎不可知的已知一致状态。 加入域并从域应用组策略设置将增加启动容器的时间、随时间推移更改其运行方式并限制在开发人员之间以及跨部署移动或共享映像的功能。

我们会谨慎考虑有关服务和应用程序如何使用 Active Directory 的反馈以及在容器中部署这些内容的交集。 如果你具有最适合你的内容的相关详细信息，请在[论坛](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)中与我们进行共享。 

我们正在积极地寻找解决方案以支持这些类型的方案。


<!--HONumber=May16_HO4-->


