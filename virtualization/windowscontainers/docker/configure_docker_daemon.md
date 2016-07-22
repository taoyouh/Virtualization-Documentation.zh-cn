---
title: "在 Windows 中配置 Docker"
description: "在 Windows 中配置 Docker"
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 07/15/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
translationtype: Human Translation
ms.sourcegitcommit: 475240afdf97af117519cfaa287f1e4fec8837a5
ms.openlocfilehash: 5b86442643fb5937b62a67d144ae0d1c98373b41

---

# Windows 上的 Docker 守护程序

Windows 中不含 Docker 引擎，需要单独进行安装和配置。 此外，Docker 守护程序可以接受多种自定义配置。 例如，可以配置守护程序接受传入请求的方式、默认网络选项及调试/日志设置。 在 Windows 上，这些配置可以在配置文件中指定，或者通过使用 Windows 服务控制管理器指定。 此文档将详细阐述如何安装和配置 docker 守护程序，还会提供一些通用配置的示例。

## 安装 Docker

若要使用 Window 容器，则需要安装 Docker。 Docker 由 Docker 引擎和 Docker 客户端组成。 此示例中将会安装这两者。

为 Docker 可执行文件创建文件夹。

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

下载 Docker 守护程序。

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile $env:ProgramFiles\docker\dockerd.exe
```

下载 Docker 客户端。

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile $env:ProgramFiles\docker\docker.exe
```

将 Docker 目录添加到系统路径。 添加完成后，重启 PowerShell 会话以识别已修改的路径。

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

若要将 Docker 安装为一个 Windows 服务，请运行以下命令。

```none
dockerd --register-service
```

安装完成后，可以启动该服务。

```none
Start-Service Docker
```

安装容器映像后 Docker 才可以使用。 有关详细信息，请参阅[管理容器映像](../management/manage_images.md)。

## Docker 配置文件

在 Windows 上配置 Docker 守护程序的首选方法是使用配置文件。 可在“c:\ProgramData\docker\config\daemon.json”中找到配置文件。 如果此文件还不存在，可以创建此文件。

注意：并非所有可用的 Docker 配置选项在 Windows 上的 Docker 中都适用。 以下示例列出了可用的选项。 若要查看有关 Docker 守护程序配置的完整文档（包括适用于 Linux 的相应文档），请参阅 [Docker 守护程序]( https://docs.docker.com/v1.10/engine/reference/commandline/daemon/)。

```none
{
    "authorization-plugins": [],
    "dns": [],
    "dns-opts": [],
    "dns-search": [],
    "exec-opts": [],
    "storage-driver": "",
    "storage-opts": [],
    "labels": [],
    "log-driver": "", 
    "mtu": 0,
    "pidfile": "",
    "graph": "",
    "cluster-store": "",
    "cluster-advertise": "",
    "debug": true,
    "hosts": [],
    "log-level": "",
    "tlsverify": true,
    "tlscacert": "",
    "tlscert": "",
    "tlskey": "",
    "group": "",
    "default-ulimits": {},
    "bridge": "",
    "fixed-cidr": "",
    "raw-logs": false,
    "registry-mirrors": [],
    "insecure-registries": [],
    "disable-legacy-registry": false
}
```

只需将想要进行的配置更改添加到配置文件即可。 例如，此示例中将 Docker 守护程序配置为接受端口 2375 传入的连接。 其他所有配置选项将使用默认值。

```none
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

同样，此示例将 Docker 守护程序配置为仅接受通过端口 2376 的安全连接。

```none
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

## 服务控制管理器

还可以通过使用 `sc config` 修改 Docker 服务来配置 Docker 守护程序。 使用此方法时将直接在 Docker 服务上设置 Docker 守护程序的标记。


```none
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

## 通用配置

以下配置文件示例演示了通用的 Docker 配置。 这些配置可以并入单个配置文件。

### 创建默认网络 

若要将 Docker 守护程序配置为不创建默认 NAT 网络，请运行以下操作。 有关详细信息，请参阅[管理 Docker 网络](../management/container_networking.md)。

```none
{
    "bridge" : "none"
}
```

### 设置 Docker 安全组

登录到 Docker 主机并在本地运行 Docker 命令后，这些命令将通过命名管道运行。 默认情况下，只有管理员组的成员可以通过此命名管道访问 Docker 守护程序。 若要指定具有此访问权限的安全组，请使用 `group` 标记。

```none
{
    "group" : "docker"
}
```

## 代理配置

若要设置 `docker search` 和 `docker pull` 的代理信息，请使用 `HTTP_PROXY` 或 `HTTPS_PROXY` 名称以及代理信息的一个值创建 Windows 环境变量。 可使用类似于以下的命令通过 PowerShell 完成此操作：

```none
[Environment]::SetEnvironmentVariable("HTTP_PROXY”, “http://username:password@proxy:port/”, [EnvironmentVariableTarget]::Machine)
```

设置变量后，重启 Docker 服务。

```none
restart-service docker
```

有关详细信息，请参阅 [Docker.com 上的守护程序套接字选项](https://docs.docker.com/v1.10/engine/reference/commandline/daemon/#daemon-socket-option)。

## 收集日志
Docker 守护程序会将事件记录到 Windows“应用程序”事件日志中，而不是某个文件中。 使用 Windows PowerShell 可以轻松读取、排序和筛选这些日志

例如，这将显示过去 5 分钟的 Docker 守护程序日志（从最早的开始）。
```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-5) | Sort-Object Time 
```

也可以很容易通过管道将其转换为 CSV 文件，以便其他工具或电子表格进行读取。
```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-30)  | Sort-Object Time | Export-CSV ~/last30minutes.csv ```



<!--HONumber=Jul16_HO3-->


