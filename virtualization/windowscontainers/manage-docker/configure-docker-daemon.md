---
title: 在 Windows 中配置 Docker
description: 在 Windows 中配置 Docker
keywords: docker, 容器
author: PatrickLang
ms.date: 08/23/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
ms.openlocfilehash: bbc405fc2a490cfe5082be112fde724707e24785
ms.sourcegitcommit: 21d93e5febd9b1b47ae1aa59d08086e6ec1691e0
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/28/2019
ms.locfileid: "9121049"
---
# <a name="docker-engine-on-windows"></a>Windows 上的 Docker 引擎

Docker 引擎和客户端包含在 Windows 并不需要进行安装和单独配置。 此外，Docker 引擎可以接受多种自定义配置。 例如，可以配置守护程序接受传入请求的方式、默认网络选项及调试/日志设置。 在 Windows 上，这些配置可以在配置文件中指定，或者通过使用 Windows 服务控制管理器指定。 本文档详细介绍了如何安装和配置 Docker 引擎，并且还提供了一些常用的配置的示例。


## <a name="install-docker"></a>安装 Docker
若要使用 Window 容器，则需要安装 Docker。 Docker 由 Docker 引擎 (dockerd.exe) 和 Docker 客户端 (docker.exe) 组成。 快速入门指南中提供了安装所有内容的最简方法。 它们可以帮助你获得的所有内容设置和运行你的第一个容器。 

* [Windows Server 2019 上的 Windows 容器](../quick-start/quick-start-windows-server.md)
* [Windows 10 上的 Windows 容器](../quick-start/quick-start-windows-10.md)

若要进行脚本化安装，请参阅[使用脚本安装 Docker EE](https://docs.docker.com/install/windows/docker-ee/#use-a-script-to-install-docker-ee)。

可以使用 Docker 容器映像需要先进行安装。 有关详细信息，请参阅[映像使用快速入门指南](../quick-start/quick-start-images.md)。

## <a name="configure-docker-with-configuration-file"></a>使用配置文件配置 Docker

在 Windows 上配置 Docker 引擎的首选方法是使用配置文件。 可在“C:\ProgramData\Docker\config\daemon.json”中找到配置文件。 如果此文件还不存在，可以创建此文件。

注意：并非所有可用的 Docker 配置选项在 Windows 上的 Docker 中都适用。 以下示例列出了可用的选项。 有关 Docker 引擎配置的完整文档，请参阅 [Docker 守护程序配置文件](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)。

```
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
    "data-root": "",
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

```
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

同样，此示例将配置 Docker 守护程序以将图像和容器保存在备用路径。 如果未指定，默认路径为 c:\programdata\docker。

```
{    
    "data-root": "d:\\docker"
}
```

同样，此示例将 Docker 守护程序配置为仅接受通过端口 2376 的安全连接。

```
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

## <a name="configure-docker-on-the-docker-service"></a>在 Docker 服务上配置 Docker

还可以通过使用 `sc config` 修改 Docker 服务来配置 Docker 引擎。 使用此方法时将直接在 Docker 服务上设置 Docker 引擎的标记。 在命令提示符（cmd.exe 而非 PowerShell）中运行以下命令：


```
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

注意：如果 daemon.json 文件已经包含 `"hosts": ["tcp://0.0.0.0:2375"]` 条目，则无需运行此命令。

## <a name="common-configuration"></a>通用配置

以下配置文件示例演示了通用的 Docker 配置。 这些配置可以并入单个配置文件。

### <a name="default-network-creation"></a>创建默认网络 

若要将 Docker 引擎配置为不创建默认 NAT 网络，请运行以下操作。 有关详细信息，请参阅[管理 Docker 网络](../container-networking/network-drivers-topologies.md)。

```
{
    "bridge" : "none"
}
```

### <a name="set-docker-security-group"></a>设置 Docker 安全组

登录到 Docker 主机并在本地运行 Docker 命令后，这些命令将通过命名管道运行。 默认情况下，只有管理员组的成员才可以通过此命名管道访问 Docker 引擎。 若要指定具有此访问权限的安全组，请使用 `group` 标记。

```
{
    "group" : "docker"
}
```

## <a name="proxy-configuration"></a>代理配置

若要设置 `docker search` 和 `docker pull` 的代理信息，请使用 `HTTP_PROXY` 或 `HTTPS_PROXY` 名称以及代理信息的一个值创建 Windows 环境变量。 可使用类似于以下的命令通过 PowerShell 完成此操作：

```powershell
[Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://username:password@proxy:port/", [EnvironmentVariableTarget]::Machine)
```

设置变量后，重启 Docker 服务。

```powershell
Restart-Service docker
```

有关详细信息，请参阅 [Docker.com 上的 Windows 配置文件](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)。

## <a name="uninstall-docker"></a>卸载 Docker
*使用本部分中的步骤卸载 Docker，并全面清理 Windows 10 或 Windows Server 2016 系统中的 Docker 系统组件。*

> 注意：以下步骤中的所有命令都必须从**提升的** PowerShell 会话中运行。

### <a name="step-1-prepare-your-system-for-dockers-removal"></a>步骤 1：准备你的系统以删除 Docker 
最好在删除 Docker 之前确保系统上未运行容器。 下面是用于执行此操作的一些有效命令：
```
# Leave swarm mode (this will automatically stop and remove services and overlay networks)
docker swarm leave --force

# Stop all running containers
docker ps --quiet | ForEach-Object {docker stop $_}
```
最好在删除 Docker 之前也从系统中删除所有容器、容器映像、网络和卷：
```
docker system prune --volumes --all
```

### <a name="step-2-uninstall-docker"></a>步骤 2：卸载 Docker 

#### ***<a name="steps-to-uninstall-docker-on-windows-10"></a>在 Windows 10 上卸载 Docker 的步骤：10:***
- 在 Windows 10 计算机上转到 **“设置”>“应用”**
- 在 **“应用和功能”** 下面，查找 **“适用于 Windows 的 Docker”**
- 单击 **“适用于 Windows 的 Docker”>“卸载”**

#### ***<a name="steps-to-uninstall-docker-on-windows-server-2016"></a>在 Windows Server 2016 上卸载 Docker 的步骤：16:***
从提升的 PowerShell 会话中，使用 `Uninstall-Package` 和 `Uninstall-Module` cmdlet 从系统中删除 Docker 模块及其相应的程序包管理提供程序。 
> 提示：你可以查找曾用于安装 Docker 的程序包提供程序 `PS C:\> Get-PackageProvider -Name *Docker*`

*例如*：
```
Uninstall-Package -Name docker -ProviderName DockerMsftProvider
Uninstall-Module -Name DockerMsftProvider
```

### <a name="step-3-cleanup-docker-data-and-system-components"></a>步骤 3：清理 Docker 数据和系统组件
删除 Docker 的*默认网络*，以便在卸载 Docker 的同时在你的系统上清除这些网络的配置：
```
Get-HNSNetwork | Remove-HNSNetwork
```
从系统中删除 Docker 的*程序数据*：
```
Remove-Item "C:\ProgramData\Docker" -Recurse
```
你可能还需要删除 Windows 上与 Docker/容器关联的 *Windows 可选功能*。 

至少，这包括“容器”功能，安装 Docker 时会在任何 Windows 10 或 Windows Server 2016 上自动启用该功能。 这还可能包括“Hyper-V”功能，安装 Docker 时可在 Windows 10 上自动启用该功能，但必须在 Windows Server 2016 上显式启用该功能。

> **关于禁用 HYPER-V 的重要说明：**[Hyper-V 功能](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/about/)是一种常规虚拟化功能，该功能所启用的远远不止是容器！ 禁用 Hyper-V 功能之前，请确保系统上没有其他虚拟化组件需要该功能。

#### ***<a name="steps-to-remove-windows-features-on-windows-10"></a>在 Windows 10 上删除 Windows 功能的步骤：10:***
- 在 Windows 10 计算机上，转到 **“控制面板”>“程序”>“程序和功能”>“打开或关闭 Windows 功能”**
- 查找你想要禁用的功能的名称，在本例中为 **“容器”** 和（可选）**“Hyper-V”**
- **取消选中**你想要禁用的功能名称旁边的框
- 单击 **“确定”**

#### ***<a name="steps-to-remove-windows-features-on-windows-server-2016"></a>在 Windows Server 2016 上删除 Windows 功能的步骤：16:***
从提升的 PowerShell 会话中，使用以下命令禁用系统中的 **“容器”** 和（可选）**“Hyper-V”** 功能：
```
Remove-WindowsFeature Containers
Remove-WindowsFeature Hyper-V 
```

### <a name="step-4-reboot-your-system"></a>步骤 4：重新启动你的系统
若要完成这些卸载/清理步骤，请从提升的 PowerShell 会话中运行：
```
Restart-Computer -Force
```
