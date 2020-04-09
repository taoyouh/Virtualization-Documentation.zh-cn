---
title: Windows 容器疑难解答
description: Windows 容器和 Docker 的疑难解答提示、自动化脚本和日志信息
keywords: docker, 容器, 疑难解答, 日志
author: PatrickLang
ms.date: 12/19/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ebd79cd3-5fdd-458d-8dc8-fc96408958b5
ms.openlocfilehash: 752df5de208887b149460a204bbb6ff74393e809
ms.sourcegitcommit: b140ac14124e4bee3c7f31a7f8274d4a0ccb2dda
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/08/2020
ms.locfileid: "80929975"
---
# <a name="troubleshooting"></a>故障排除

设置计算机或运行容器时遇到问题？ 我们创建了一个 PowerShell 脚本来检查常见问题。 请先试一试，查看它所找到的内容并分享结果。

```PowerShell
Invoke-WebRequest https://aka.ms/Debug-ContainerHost.ps1 -UseBasicParsing | Invoke-Expression
```
其运行的所有测试以及常见解决方案的列表位于脚本的[自述文件](https://github.com/Microsoft/Virtualization-Documentation/blob/live/windows-server-container-tools/Debug-ContainerHost/README.md)中。

如果这对找到问题的根源没有帮助，请继续在[容器论坛](https://social.msdn.microsoft.com/Forums/home?forum=windowscontainers)上发布脚本的输出。 这是从社区（包含 Windows 预览体验成员和开发人员）获得帮助的最佳位置。


### <a name="finding-logs"></a>查找日志
有多个服务用于管理 Windows 容器。 下一节将介绍为每个服务获取日志的位置。

## <a name="docker-container-logs"></a>Docker 容器日志 
`docker logs` 命令从 STDOUT/STDERR 获取容器的日志，这是适用于 Linux 应用程序的标准应用程序日志存放位置。 Windows 应用程序通常不会记录到 STDOUT/STDERR;相反，它们将记录到 ETW、事件日志或日志文件等。 

Github 上现在提供了一个 Microsoft 支持的开源工具的[日志监视器](https://github.com/microsoft/windows-container-tools/tree/master/LogMonitor)。 日志监视器将 Windows 应用程序日志桥接到 STDOUT/STDERR。 日志监视器是通过配置文件配置的。 

### <a name="log-monitor-usage"></a>日志监视器使用情况

LogMonitor 和 LogMonitorConfig 应同时包含在相同的 LogMonitor 目录中。 

日志监视器可用于 SHELL 使用模式：

```
SHELL ["C:\\LogMonitor\\LogMonitor.exe", "cmd", "/S", "/C"]
CMD c:\windows\system32\ping.exe -n 20 localhost
```

或入口点使用模式：

```
ENTRYPOINT C:\LogMonitor\LogMonitor.exe c:\windows\system32\ping.exe -n 20 localhost
```

这两个示例用法都包装了 dism.exe 应用程序。 其他应用程序（如[IIS）。ServiceMonitor]( https://github.com/microsoft/IIS.ServiceMonitor)）可以使用类似的方式与日志监视器嵌套：

```
COPY LogMonitor.exe LogMonitorConfig.json C:\LogMonitor\
WORKDIR /LogMonitor
SHELL ["C:\\LogMonitor\\LogMonitor.exe", "powershell.exe"]
 
# Start IIS Remote Management and monitor IIS
ENTRYPOINT      Start-Service WMSVC; `
                    C:\ServiceMonitor.exe w3svc;
```


日志监视器将已包装的应用程序作为子进程启动，并监视应用程序的 STDOUT 输出。

请注意，在 SHELL 使用模式下，应在 SHELL 形式而不是 exec 格式中指定 CMD/ENTRYPOINT 指令。 使用 CMD/ENTRYPOINT 指令的 exec 格式时，不会启动 SHELL，并且不会在容器中启动日志监视工具。

可在[日志监视器 wiki](https://github.com/microsoft/windows-container-tools/wiki)上找到更多使用情况信息。 可在[github](https://github.com/microsoft/windows-container-tools/tree/master/LogMonitor/src/LogMonitor/sample-config-files)存储库中找到关键 Windows 容器方案（IIS 等）的配置文件示例。 可在此[博客文章](https://techcommunity.microsoft.com/t5/Containers/Windows-Containers-Log-Monitor-Opensource-Release/ba-p/973947)中找到其他上下文。

## <a name="docker-engine"></a>Docker 引擎
Docker 引擎会将事件记录到 Windows“应用程序”事件日志中，而不是某个文件中。 使用 Windows PowerShell 可以轻松读取、排序和筛选这些日志

例如，这将显示过去 5 分钟的 Docker 引擎日志（从最早的开始）。

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-5) | Sort-Object Time 
```

也可以很容易通过管道将其转换为 CSV 文件，以便其他工具或电子表格进行读取。

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-30)  | Sort-Object Time | Export-CSV ~/last30minutes.CSV
```

### <a name="enabling-debug-logging"></a>启用调试日志记录
还可以在 Docker 引擎上启用调试级别的日志记录。 如果常规日志没有足够的详细信息，这可能有助于进行故障排除。

首先，打开提升的命令提示符，然后运行 `sc.exe qc docker` 为 Docker 服务获取当前命令行。
示例：
```
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

然后运行 `sc.exe config docker binpath=`，后跟新字符串。 例如： 
```
sc.exe config docker binpath= "\"C:\Program Files\Docker\dockerd.exe\" --run-service -D"
```


现在，重启 Docker 服务
```
sc.exe stop docker
sc.exe start docker
```

这将记录更多内容到应用程序事件日志中，因此最好在完成排除故障后立即删除 `-D` 选项。 使用上述不含 `-D` 的相同步骤，然后重启服务以禁用调试日志记录。

上述操作的替代方法是在调试模式下从提升的 PowerShell 提示符运行 docker 守护程序，并将输出直接捕获到文件中。
```PowerShell
sc.exe stop docker
<path\to\>dockerd.exe -D > daemon.log 2>&1
```

### <a name="obtaining-stack-dump"></a>获取堆栈转储

通常，这仅在 Microsoft 支持或 docker 开发人员明确请求时才有用。 它可用于帮助诊断 docker 似乎已挂起的情况。 

下载 [docker signal.exe](https://github.com/moby/docker-signal)。

用法：
```PowerShell
docker-signal --pid=$((Get-Process dockerd).Id)
```

输出文件将位于正在运行的数据根目录中。 默认目录是 `C:\ProgramData\Docker`。 可以通过运行 `docker info -f "{{.DockerRootDir}}"` 来确认实际目录。

文件将被 `goroutine-stacks-<timestamp>.log`。

请注意，`goroutine-stacks*.log` 不包含个人信息。


## <a name="host-compute-service"></a>主机计算服务
Docker 引擎依赖于 Windows 特定的主机计算服务。 它具有单独的日志: 
- Microsoft-Windows-Hyper-V-Compute-Admin
- Microsoft-Windows-Hyper-V-Compute-Operational

它们在事件查看器中可见，还可以通过 PowerShell 进行查询。

例如：
```PowerShell
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Admin
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Operational 
```

### <a name="capturing-hcs-analyticdebug-logs"></a>捕获 HCS 分析/调试日志

若要对“Hyper-V 计算”启用分析/调试日志，请将日志保存到 `hcslog.evtx`。

```PowerShell
# Enable the analytic logs
wevtutil.exe sl Microsoft-Windows-Hyper-V-Compute-Analytic /e:true /q:true

# <reproduce your issue>

# Export to an evtx
wevtutil.exe epl Microsoft-Windows-Hyper-V-Compute-Analytic <hcslog.evtx>

# Disable
wevtutil.exe sl Microsoft-Windows-Hyper-V-Compute-Analytic /e:false /q:true
```

### <a name="capturing-hcs-verbose-tracing"></a>捕获 HCS 详细跟踪信息

通常，这些信息仅在 Microsoft 支持请求时才有用。 

下载 [HcsTraceProfile.wprp](https://github.com/MicrosoftDocs/Virtualization-Documentation/blob/master/windows-server-container-tools/wpr-profiles/HcsTraceProfile.wprp)

```PowerShell
# Enable tracing
wpr.exe -start HcsTraceProfile.wprp!HcsArgon -filemode

# <reproduce your issue>

# Capture to HcsTrace.etl
wpr.exe -stop HcsTrace.etl "some description"
```

向你的支持联系人提供 `HcsTrace.etl`。
