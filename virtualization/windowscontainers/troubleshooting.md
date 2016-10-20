# 疑难解答

设置计算机或运行容器时遇到问题？ 我们创建了一个 PowerShell 脚本来检查常见问题。 请先试一试，查看它所找到的内容并分享结果。

```PowerShell
Invoke-WebRequest https://raw.githubusercontent.com/Microsoft/Virtualization-Documentation/master/windows-server-container-tools/Debug-ContainerHost/Debug-ContainerHost.ps1 | Invoke-Expression
```
其运行的所有测试以及常见解决方案的列表位于脚本的[自述文件](https://github.com/Microsoft/Virtualization-Documentation/blob/master/windows-server-container-tools/Debug-ContainerHost/README.md)中。

如果这对找到问题的根源没有帮助，请继续在[容器论坛](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)上发布脚本的输出。 这是从社区（包含 Windows 预览体验成员和开发人员）获得帮助的最佳位置。


## 查找日志
存在多个用于管理 Windows 容器的服务。 下一节将介绍为每个服务获取日志的位置。

### Docker 引擎
Docker 引擎会将事件记录到 Windows“应用程序”事件日志中，而不是某个文件中。 使用 Windows PowerShell 可以轻松读取、排序和筛选这些日志

例如，这将显示过去 5 分钟的 Docker 引擎日志（从最早的开始）。

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-5) | Sort-Object Time 
```

也可以很容易通过管道将其转换为 CSV 文件，以便其他工具或电子表格进行读取。

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-30)  | Sort-Object Time | Export-CSV ~/last30minutes.CSV
```

#### 启用调试日志记录
还可以在 Docker 引擎上启用调试级别的日志记录。 如果常规日志没有足够的详细信息，这可能有助于进行故障排除。

首先，打开提升的命令提示符，然后运行 `sc.exe qc docker` 为 Docker 服务获取当前命令行。
例如：
```none
C:\> sc.exe qc docker
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: docker
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : "C:\Program Files\Docker\dockerd.exe" --run-service
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : Docker Engine
        DEPENDENCIES       :
        SERVICE_START_NAME : LocalSystem
```

获取当前 `BINARY_PATH_NAME`，然后对其进行修改：
- 在结尾处添加 -D
- 用 \ 转义每个 "
- 将整个命令括在 " 中

然后运行 `sc.exe config docker binpath= `，后跟新字符串。 例如： 
```none
sc.exe config docker binpath= "\"C:\Program Files\Docker\dockerd.exe\" --run-service -D"
```


现在，重启 Docker 服务
```none
sc.exe stop docker
sc.exe start docker
```

这将记录更多内容到应用程序事件日志中，因此最好在完成排除故障后立即删除 `-D` 选项。 使用上述不含 `-D` 的相同步骤，然后重启服务以禁用调试日志记录。


### 主机容器服务
Docker 引擎依赖于 Windows 特定的主机容器服务。 它具有单独的日志: 
- Microsoft-Windows-Hyper-V-Compute-Admin
- Microsoft-Windows-Hyper-V-Compute-Operational

它们在事件查看器中可见，也可以使用 PowerShell 查询。

例如：
```PowerShell
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Admin
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Operational 
```



<!--HONumber=Oct16_HO3-->


