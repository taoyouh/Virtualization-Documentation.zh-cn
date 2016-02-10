# 正在进行的工作

如果在此处没有看到你的问题已解决或仍有问题，请将其发布到[论坛](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)。

-----------------------


## 常规功能

### 容器和主机版本号匹配

Windows 容器要求操作系统映像匹配容器主机的版本和修补程序级别。 不匹配会导致容器和/或主机出现潜在的不稳定性和/或不可预测的行为。

如果针对 Windows 容器主机操作系统安装更新，将需要更新容器基础操作系统映像，以便具有匹配的更新。


**解决方法：**   
下载并安装匹配容器主机的操作系统版本和修补程序级别的容器基础映像。

### 所有非 C:/ 驱动器在容器中均可见

可用于容器主机的所有非 C:/ 驱动器将自动映射到运行的新 Windows 容器中。

此时，无法有选择性地将文件夹映射到容器中，因为将自动映射驱动器的临时解决方法。

**解决方法：**  
我们正在努力解决它。 将来，会提供文件夹共享。

### 默认的防火墙行为

在容器主机和容器环境中，你只具有容器主机的防火墙。 容器主机中配置的所有防火墙规则将传播到其所有容器。

### Windows 容器启动缓慢

如果你的容器的启动时间超过 30 秒，有可能正在执行许多重复的病毒扫描。

Windows Defender 等许多反恶意软件解决方案可能正在不必要地扫描容器映像内的文件（包括所有操作系统二进制文件）和容器操作系统映像中的文件。 在创建新容器，且从反恶意软件的角度来看，所有“容器文件”似乎是以前尚未扫描的新文件时，可能会发生此情况。 因此当容器内的进程试图读取这些文件时，反恶意软件组件会先扫描它们，然后再对文件进行访问。 实际上，将容器映像导入或提取到服务器时，已对这些文件进行了扫描。 在将来的预览版中，将推出新的基础结构，以便反恶意软件解决方案（包括 Windows Defender）能够识别这些情况并可相应地采取措施以避免多次扫描。

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
“无法启动：由于超时时间已过，该操作返回。 (0x800705B4)。”

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

### 受限制的网络隔离舱

在此版本中，我们支持每个容器一个网络隔离舱。 这意味着，如果你拥有一个具有多个网络适配器的容器，将不能访问每个适配器上的同一个网络端口（例如属于同一个容器 192.168.0.1:80 和 192.168.0.2:80）。

**解决方法：**  
如果容器需要公开多个终结点，请使用 NAT 端口映射。

### Windows 容器未获得 IP

如果你要连接到具有 DHCP VM 开关的 Windows 容器，容器主机可以接收 IP，但该容器不可以。

容器获得 169.254.***.*** APIPA IP 地址。

**解决方法：**
这是共享内核的副作用。 所有容器实际上具有相同的 MAC 地址。

支持在容器主机上进行 MAC 地址欺骗。

使用 PowerShell 可以实现此操作
```
Get-VMNetworkAdapter -VMName "[YourVMNameHere]"  | Set-VMNetworkAdapter -MacAddressSpoofing On
```
### 不支持 HTTPS 和 TLS

Window Server 容器和 Hyper-V 容器不支持 HTTPS 或 TLS。 我们致力于在将来实现这一点。

--------------------------


## 应用程序兼容性

存在许多有关可在 Windows 容器中使用哪些应用程序的问题，我们决定为应用程序兼容性信息单独编写[一篇文章](../reference/app_compat.md)。

其中还包含一些最常见的问题。

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

观察到的行为：
在容器中：
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

### Docker 客户端不安全

在此预览版中，Docker 通信是公开的（如果你知道查看位置）。

### 并非所有 Docker 命令都起作用

* Docker 执行程序无法在 Hyper-V 容器中运行。
* 尚不支持与 DockerHub 相关的命令。

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

Windows 容器无法通过 TP4 中的 RDP 会话进行管理/与之交互。

--------------------------


## PowerShell 管理

### 并非所有 *-PSSession 都具有 containerid 参数

这是正确的。 我们正计划将来对 cimsession 提供完全支持。

### 无法使用“exit”退出 Nano Server 容器主机中的容器。

如果你尝试退出 Nano Server 容器主机中的容器，使用“exit”将断开与 Nano Server 容器主机的连接，而且不会退出该容器。

**解决方法：**
改为使用 Exit-PSSession 退出该容器。

随时在[论坛](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)中发布功能请求。


--------------------------



## 用户和域

### 本地用户

可以创建本地用户帐户并将其用于在容器中运行 Windows 服务和应用程序。


### 域成员身份

容器无法加入 Active Directory 域，并且无法以域用户的身份运行服务或应用程序、服务帐户或计算机帐户。

容器旨在快速启动到环境几乎不可知的已知一致状态。 加入域并从域应用组策略设置将增加启动容器的时间、随时间推移更改其运行方式并限制在开发人员之间以及跨部署移动或共享映像的功能。

我们会谨慎考虑有关服务和应用程序如何使用 Active Directory 的反馈以及在容器中部署这些内容的交集。如果你具有最适合你的内容的相关详细信息，请在[论坛](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)中与我们进行共享。

我们正在积极地寻找解决方案以支持这些类型的方案。



<!--HONumber=Feb16_HO1-->
