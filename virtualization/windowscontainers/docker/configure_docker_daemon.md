---
title: "在 Windows 中配置 Docker"
description: "在 Windows 中配置 Docker"
keywords: "docker, 容器"
author: neilpeterson
manager: timlt
ms.date: 08/23/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
translationtype: Human Translation
ms.sourcegitcommit: 38d9f06af87cf1d69529d28e30cab60f16e0982b
ms.openlocfilehash: 185831094b63a1b7fb1931db7fb82a6c59c2b060

---

# Windows 上的 Docker 引擎

Windows 中不含 Docker 引擎和客户端，需要单独进行安装和配置。 此外，Docker 引擎可以接受多种自定义配置。 例如，可以配置守护程序接受传入请求的方式、默认网络选项及调试/日志设置。 在 Windows 上，这些配置可以在配置文件中指定，或者通过使用 Windows 服务控制管理器指定。 此文档将详细阐述如何安装和配置 Docker 引擎，还会提供一些通用配置的示例。


## 安装 Docker
若要使用 Window 容器，则需要安装 Docker。 Docker 由 Docker 引擎 (dockerd.exe) 和 Docker 客户端 (docker.exe) 组成。 快速入门指南中提供了安装所有内容的最简方法。 指南将帮助设置所有项目并运行首个容器。 

* [Windows Server 2016 上的 Windows 容器](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/quick_start/quick_start_windows_server)
* [Windows 10 上的 Windows 容器](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/quick_start/quick_start_windows_10)


### 手动安装
若要改用 Docker 引擎和客户端的开发中版本，可遵循后续步骤。 这将安装 Docker 引擎和客户端。 否则，请跳到下一节。

> 如果已安装 Docker for Windows，请务必在执行以下手动安装步骤之前将其删除。 

下载 Docker 引擎

Https://master.dockerproject.org 始终提供最新版本。 此示例使用 v1.13 开发分支提供的最新内容。 

```none
Invoke-WebRequest "https://download.docker.com/components/engine/windows-server/cs-1.12/docker.zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

将 Zip 存档扩展到 Program Files。

```
Expand-Archive -Path "$env:TEMP\docker.zip" -DestinationPath $env:ProgramFiles
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

## 使用配置文件配置 Docker

在 Windows 上配置 Docker 引擎的首选方法是使用配置文件。 可在“c:\ProgramData\docker\config\daemon.json”中找到配置文件。 如果此文件还不存在，可以创建此文件。

注意：并非所有可用的 Docker 配置选项在 Windows 上的 Docker 中都适用。 以下示例列出了可用的选项。 有关 Docker 引擎配置的完整文档，请参阅 [Docker 守护程序配置文件](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)。

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

只需将想要进行的配置更改添加到配置文件即可。 例如，此示例中将 Docker 引擎配置为接受端口 2375 传入的连接。 其他所有配置选项将使用默认值。

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

## 在 Docker 服务上配置 Docker

还可以通过使用 `sc config` 修改 Docker 服务来配置 Docker 引擎。 使用此方法时将直接在 Docker 服务上设置 Docker 引擎的标记。 在命令提示符（cmd.exe 而非 PowerShell）中运行以下命令：


```none
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

注意：如果 daemon.json 文件已经包含 `"hosts": ["tcp://0.0.0.0:2375"]` 条目，则无需运行此命令。

## 通用配置

以下配置文件示例演示了通用的 Docker 配置。 这些配置可以并入单个配置文件。

### 创建默认网络 

若要将 Docker 引擎配置为不创建默认 NAT 网络，请运行以下操作。 有关详细信息，请参阅[管理 Docker 网络](../management/container_networking.md)。

```none
{
    "bridge" : "none"
}
```

### 设置 Docker 安全组

登录到 Docker 主机并在本地运行 Docker 命令后，这些命令将通过命名管道运行。 默认情况下，只有管理员组的成员才可以通过此命名管道访问 Docker 引擎。 若要指定具有此访问权限的安全组，请使用 `group` 标记。

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



<!--HONumber=Oct16_HO3-->


