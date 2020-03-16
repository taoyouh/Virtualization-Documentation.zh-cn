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
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/15/2020
ms.locfileid: "79402878"
---
# <a name="docker-engine-on-windows"></a>Windows 上的 Docker 引擎

Docker 引擎和客户端不包括在 Windows 中，需要单独进行安装和配置。 此外，Docker 引擎可以接受多种自定义配置。 例如，可以配置守护程序接受传入请求的方式、默认网络选项及调试/日志设置。 在 Windows 上，这些配置可以在配置文件中指定，或者通过使用 Windows 服务控制管理器指定。 本文档详细说明了如何安装和配置 Docker 引擎，还提供了一些常用配置示例。

## <a name="install-docker"></a>安装 Docker

需要使用 Docker 才能使用 Windows 容器。 Docker 由 Docker 引擎 (dockerd.exe) 和 Docker 客户端 (docker.exe) 组成。 获取安装内容的最简单方法是在快速入门指南中，这将帮助你设置和运行第一个容器。

- [安装 Docker](../quick-start/set-up-environment.md)

有关脚本化安装，请参阅[使用脚本安装 DOCKER EE](https://docs.docker.com/install/windows/docker-ee/#use-a-script-to-install-docker-ee)。

你需要安装容器映像，然后才能使用 Docker。 有关详细信息，请参阅[容器基本映像的文档](../manage-containers/container-base-images.md)。

## <a name="configure-docker-with-a-configuration-file"></a>使用配置文件配置 Docker

在 Windows 上配置 Docker 引擎的首选方法是使用配置文件。 可在“C:\ProgramData\Docker\config\daemon.json”中找到配置文件。 如果文件尚不存在，可以创建它。

>[!NOTE]
>并非每个可用的 Docker 配置选项都适用于 Windows 上的 Docker。 下面的示例演示了应用的配置选项。 有关 Docker 引擎配置的详细信息，请参阅[docker 守护程序配置文件](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)。

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

只需将所需的配置更改添加到配置文件。 例如，以下示例将 Docker 引擎配置为接受端口2375上的传入连接。 其他所有配置选项将使用默认值。

```json
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

同样，下面的示例将 Docker 守护程序配置为在备用路径中保留图像和容器。 如果未指定，则默认值为 `c:\programdata\docker`。

```json
{    
    "data-root": "d:\\docker"
}
```

下面的示例将 Docker 守护程序配置为仅接受通过端口2376的安全连接。

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

还可以通过使用 `sc config`修改 Docker 服务来配置 Docker 引擎。 使用此方法时将直接在 Docker 服务上设置 Docker 引擎的标记。 在命令提示符（cmd.exe 而非 PowerShell）中运行以下命令：

```cmd
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

>[!NOTE]
>如果你的 daemon 文件已包含 `"hosts": ["tcp://0.0.0.0:2375"]` 项，则不需要运行此命令。

## <a name="common-configuration"></a>通用配置

以下配置文件示例演示了通用的 Docker 配置。 这些配置可以并入单个配置文件。

### <a name="default-network-creation"></a>默认网络创建

若要将 Docker 引擎配置为不创建默认 NAT 网络，请使用以下配置。

```json
{
    "bridge" : "none"
}
```

有关详细信息，请参阅[管理 Docker 网络](../container-networking/network-drivers-topologies.md)。

### <a name="set-docker-security-group"></a>设置 Docker 安全组

登录到 Docker 主机并在本地运行 Docker 命令时，这些命令通过命名管道运行。 默认情况下，只有管理员组的成员才可以通过此命名管道访问 Docker 引擎。 若要指定具有此访问权限的安全组，请使用 `group` 标记。

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

有关详细信息，请参阅[Docker.com 上的 Windows 配置文件](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)。

## <a name="how-to-uninstall-docker"></a>如何卸载 Docker

本部分将介绍如何从 Windows 10 或 Windows Server 2016 系统卸载 Docker 和执行 Docker 系统组件的完全清除。

>[!NOTE]
>必须从提升的 PowerShell 会话中运行这些说明中的所有命令。

### <a name="prepare-your-system-for-dockers-removal"></a>为 Docker 的删除准备系统

卸载 Docker 之前，请确保系统上没有运行任何容器。

运行以下 cmdlet 以检查正在运行的容器：

```powershell
# Leave swarm mode (this will automatically stop and remove services and overlay networks)
docker swarm leave --force

# Stop all running containers
docker ps --quiet | ForEach-Object {docker stop $_}
```

删除 Docker 之前，最好从系统中删除所有容器、容器映像、网络和卷。 可以通过运行以下 cmdlet 来执行此操作：

```powershell
docker system prune --volumes --all
```

### <a name="uninstall-docker"></a>卸载 Docker

接下来，需要实际卸载 Docker。

在 Windows 10 上卸载 Docker

- 在 Windows 10 计算机上，中转到 "**设置**" > **应用**
- 在 "**应用 & 功能**" 下，查找**用于 Windows 的 Docker**
- 切换到**用于 Windows 的 Docker** > **卸载**

在 Windows Server 2016 上卸载 Docker：

在已提升权限的 PowerShell 会话中，使用**Uninstall**和**uninstall-module** cmdlet 从系统中删除 Docker 模块及其相应的包管理提供程序，如以下示例中所示：

```powershell
Uninstall-Package -Name docker -ProviderName DockerMsftProvider
Uninstall-Module -Name DockerMsftProvider
```

>[!TIP]
>你可以找到用于安装 Docker 的包提供程序，`PS C:\> Get-PackageProvider -Name *Docker*`

### <a name="clean-up-docker-data-and-system-components"></a>清理 Docker 数据和系统组件

卸载 Docker 后，需要删除 Docker 的默认网络，以便在 Docker 丢失后，其配置不会保留在系统中。 可以通过运行以下 cmdlet 来执行此操作：

```powershell
Get-HNSNetwork | Remove-HNSNetwork
```

运行以下 cmdlet 以从系统中删除 Docker 的程序数据：

```powershell
Remove-Item "C:\ProgramData\Docker" -Recurse
```

你可能还需要删除 windows 上与 Docker/容器关联的 Windows 可选功能。

这包括 "容器" 功能，此功能在安装了 Docker 的任何 Windows 10 或 Windows Server 2016 上自动启用。 这还可能包括“Hyper-V”功能，安装 Docker 时可在 Windows 10 上自动启用该功能，但必须在 Windows Server 2016 上显式启用该功能。

>[!IMPORTANT]
>[Hyper-v 功能](https://docs.microsoft.com/virtualization/hyper-v-on-windows/about/)是一项通用的虚拟化功能，只允许使用容器。 在禁用 Hyper-v 功能之前，请确保系统上没有其他需要 Hyper-v 的虚拟化组件。

删除 Windows 10 上的 Windows 功能：

- 转到 **"控制面板"**  > **程序**" > "**程序和功能**" > **启用或禁用 Windows 功能**。
- 查找要禁用的功能的名称，在本例中为**容器**和（可选） **hyper-v**。
- 取消选中要禁用的功能名称旁边的框。
- 选择 **"确定"**

删除 Windows Server 2016 上的 Windows 功能：

在已提升权限的 PowerShell 会话中，运行以下 cmdlet 以从系统中禁用**容器**和（可选） **hyper-v**功能：

```powershell
Remove-WindowsFeature Containers
Remove-WindowsFeature Hyper-V
```

### <a name="reboot-your-system"></a>重新启动系统

若要完成卸载和清理，请从已提升权限的 PowerShell 会话中运行以下 cmdlet，以重新启动系统：

```powershell
Restart-Computer -Force
```
