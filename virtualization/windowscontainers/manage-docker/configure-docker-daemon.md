---
title: 在 Windows 中配置 Docker
description: 在 Windows 中配置 Docker
keywords: docker, 容器
author: PatrickLang
ms.date: 05/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
ms.openlocfilehash: c84a6652b5918238ee8ef6e1fa7a9b2aa596aefd
ms.sourcegitcommit: 8eedfdc1fda9d0abb36e28dc2b5fb39891777364
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/15/2020
ms.locfileid: "79402878"
---
# <a name="docker-engine-on-windows"></a>Windows 上的 Docker 引擎

Windows 中不含 Docker 引擎和客户端，它们需要单独安装和配置。 此外，Docker 引擎可以接受多种自定义配置。 例如，可以配置守护程序接受传入请求的方式、默认网络选项及调试/日志设置。 在 Windows 上，这些配置可以在配置文件中指定，或者通过使用 Windows 服务控制管理器指定。 本文档详述如何安装和配置 Docker 引擎，并提供一些常用配置的示例。

## <a name="install-docker"></a>安装 Docker

若要使用 Window 容器，则需要安装 Docker。 Docker 由 Docker 引擎 (dockerd.exe) 和 Docker 客户端 (docker.exe) 组成。 若要将一切内容安装好，最简单的方法是参阅本快速入门指南，了解如何设置所有项目并运行首个容器。

- [安装 Docker](../quick-start/set-up-environment.md)

若要进行脚本化安装，请参阅[使用脚本安装 Docker EE](https://docs.docker.com/install/windows/docker-ee/#use-a-script-to-install-docker-ee)。

需要先安装容器映像，然后才能使用 Docker。 如需详细信息，请参阅[有关容器基础映像的文档](../manage-containers/container-base-images.md)。

## <a name="configure-docker-with-a-configuration-file"></a>使用配置文件配置 Docker

在 Windows 上配置 Docker 引擎的首选方法是使用配置文件。 可在“C:\ProgramData\Docker\config\daemon.json”中找到配置文件。 如果该文件不存在，可以创建它。

>[!NOTE]
>并非所有可用的 Docker 配置选项都适用于 Windows 上的 Docker。 以下示例演示了适用的配置选项。 有关 Docker 引擎配置的详细信息，请参阅 [Docker 守护程序配置文件](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)。

```json
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

只需将想要进行的配置更改添加到配置文件即可。 例如，以下示例将 Docker 引擎配置为接受端口 2375 上的传入连接。 其他所有配置选项将使用默认值。

```json
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

同样，以下示例将 Docker 守护程序配置为将图像和容器保存在备用路径。 如果未指定，则默认设置为 `c:\programdata\docker`。

```json
{    
    "data-root": "d:\\docker"
}
```

以下示例将 Docker 守护程序配置为仅接受通过端口 2376 进行的安全连接。

```json
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

```cmd
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

>[!NOTE]
>如果 daemon.json 文件已经包含 `"hosts": ["tcp://0.0.0.0:2375"]` 条目，则无需运行此命令。

## <a name="common-configuration"></a>通用配置

以下配置文件示例演示了通用的 Docker 配置。 这些配置可以并入单个配置文件。

### <a name="default-network-creation"></a>创建默认网络

若要将 Docker 引擎配置为不创建默认 NAT 网络，请使用以下配置。

```json
{
    "bridge" : "none"
}
```

有关详细信息，请参阅[管理 Docker 网络](../container-networking/network-drivers-topologies.md)。

### <a name="set-docker-security-group"></a>设置 Docker 安全组

登录到 Docker 主机并在本地运行 Docker 命令后，这些命令将通过命名管道运行。 默认情况下，只有管理员组的成员才可以通过此命名管道访问 Docker 引擎。 若要指定具有此访问权限的安全组，请使用 `group` 标记。

```json
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

## <a name="how-to-uninstall-docker"></a>如何卸载 Docker

此部分介绍如何卸载 Docker，并全面清理 Windows 10 或 Windows Server 2016 系统中的 Docker 系统组件。

>[!NOTE]
>必须从提升的 PowerShell 会话运行这些指令中的所有命令。

### <a name="prepare-your-system-for-dockers-removal"></a>准备你的系统以删除 Docker

在卸载 Docker 之前，请确保系统上没有运行任何容器。

运行以下 cmdlet，检查是否有正在运行的容器：

```powershell
# Leave swarm mode (this will automatically stop and remove services and overlay networks)
docker swarm leave --force

# Stop all running containers
docker ps --quiet | ForEach-Object {docker stop $_}
```

最好在删除 Docker 之前也从系统中删除所有容器、容器映像、网络和卷。 为此，可以运行以下 cmdlet：

```powershell
docker system prune --volumes --all
```

### <a name="uninstall-docker"></a>卸载 Docker

接下来，需要实际卸载 Docker。

在 Windows 10 上卸载 Docker

- 在 Windows 10 计算机上转到“设置” > “应用”  
- 在“应用和功能”下面，查找“适用于 Windows 的 Docker”  
- 转到“适用于 Windows 的 Docker” > “卸载”  

在 Windows Server 2016 上卸载 Docker

从提升的 PowerShell 会话中，使用 **Uninstall-Package** 和 **Uninstall-Module** cmdlet 从系统中删除 Docker 模块及其相应的程序包管理提供程序，如以下示例所示：

```powershell
Uninstall-Package -Name docker -ProviderName DockerMsftProvider
Uninstall-Module -Name DockerMsftProvider
```

>[!TIP]
>可以查找曾通过 `PS C:\> Get-PackageProvider -Name *Docker*` 用于安装 Docker 的程序包提供程序

### <a name="clean-up-docker-data-and-system-components"></a>清理 Docker 数据和系统组件

在卸载 Docker 后，需删除 Docker 的默认网络。这样，在卸载 Docker 后，这些网络的配置就不会保留在系统上。 为此，可以运行以下 cmdlet：

```powershell
Get-HNSNetwork | Remove-HNSNetwork
```

运行以下 cmdlet，从系统中删除 Docker 的程序数据：

```powershell
Remove-Item "C:\ProgramData\Docker" -Recurse
```

可能还需要删除 Windows 上与 Docker/容器关联的 Windows 可选功能。

这包括“容器”功能，安装 Docker 时会在任何 Windows 10 或 Windows Server 2016 上自动启用该功能。 这还可能包括“Hyper-V”功能，安装 Docker 时可在 Windows 10 上自动启用该功能，但必须在 Windows Server 2016 上显式启用该功能。

>[!IMPORTANT]
>[Hyper-V 功能](https://docs.microsoft.com/virtualization/hyper-v-on-windows/about/)是一种常规虚拟化功能，该功能所启用的远远不止是容器。 禁用 Hyper-V 功能之前，请确保系统上没有其他虚拟化组件需要 Hyper-V。

若要在 Windows 10 上删除 Windows 功能，请执行以下操作：

- 转到“控制面板” > “程序” > “程序和功能” > “打开或关闭 Windows 功能。    
- 查找想要禁用的一项或多项功能的名称，在本例中为“容器”和（可选）“Hyper-V”。  
- 取消选中要禁用的功能名称旁边的框。
- 选择“确定” 

若要在 Windows Server 2016 上删除 Windows 功能，请执行以下操作：

从提升的 PowerShell 会话中运行以下 cmdlet，禁用系统中的“容器”和（可选）“Hyper-V”功能：  

```powershell
Remove-WindowsFeature Containers
Remove-WindowsFeature Hyper-V
```

### <a name="reboot-your-system"></a>重启系统

若要完成卸载和清理操作，请从提升的 PowerShell 会话运行以下 cmdlet，重启系统：

```powershell
Restart-Computer -Force
```
