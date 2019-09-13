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
ms.sourcegitcommit: 868a64eb97c6ff06bada8403c6179185bf96675f
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2019
ms.locfileid: "10129257"
---
# <a name="docker-engine-on-windows"></a>Windows 上的 Docker 引擎

Docker 引擎和客户端未包含在 Windows 中，并且需要单独安装和配置。 此外，Docker 引擎可以接受多种自定义配置。 例如，可以配置守护程序接受传入请求的方式、默认网络选项及调试/日志设置。 在 Windows 上，这些配置可以在配置文件中指定，或者通过使用 Windows 服务控制管理器指定。 此文档详细介绍了如何安装和配置 Docker 引擎，还提供了一些常用配置的示例。

## <a name="install-docker"></a>安装 Docker

需要 Docker 才能使用 Windows 容器。 Docker 由 Docker 引擎 (dockerd.exe) 和 Docker 客户端 (docker.exe) 组成。 获取安装的所有内容的最简单方法是在 "快速入门指南" 中，它将帮助你设置和运行第一个容器中的所有内容。

- [安装 Docker](../quick-start/set-up-environment.md)

有关脚本安装，请参阅[使用脚本安装 DOCKER EE](https://docs.docker.com/install/windows/docker-ee/#use-a-script-to-install-docker-ee)。

在可以使用 Docker 之前，你需要安装容器图像。 有关详细信息，请参阅[容器基本图像的文档](../manage-containers/container-base-images.md)。

## <a name="configure-docker-with-a-configuration-file"></a>使用配置文件配置 Docker

在 Windows 上配置 Docker 引擎的首选方法是使用配置文件。 可在“C:\ProgramData\Docker\config\daemon.json”中找到配置文件。 如果此文件尚不存在，则可以创建它。

>[!NOTE]
>并非每个可用 Docker 配置选项都适用于 Windows 上的 Docker。 以下示例显示了适用的配置选项。 有关 Docker 引擎配置的详细信息，请参阅[docker 后台程序配置文件](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)。

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

你只需要将所需的配置更改添加到配置文件。 例如，以下示例将 Docker 引擎配置为接受端口2375上的传入连接。 其他所有配置选项将使用默认值。

```json
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

同样，以下示例将 Docker 后台程序配置为将图像和容器保留在备用路径中。 如果未指定，则默认为`c:\programdata\docker`。

```json
{    
    "data-root": "d:\\docker"
}
```

以下示例将 Docker 后台程序配置为仅通过端口2376接受安全连接。

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

也可以通过修改带有`sc config`的 docker 服务来配置 docker 引擎。 使用此方法时将直接在 Docker 服务上设置 Docker 引擎的标记。 在命令提示符（cmd.exe 而非 PowerShell）中运行以下命令：

```cmd
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

>[!NOTE]
>如果你的守护程序. json 文件已包含该`"hosts": ["tcp://0.0.0.0:2375"]`条目，则无需运行此命令。

## <a name="common-configuration"></a>常见配置

以下配置文件示例演示了通用的 Docker 配置。 这些配置可以并入单个配置文件。

### <a name="default-network-creation"></a>默认网络创建

若要配置 Docker 引擎以使其不创建默认 NAT 网络，请使用以下配置。

```json
{
    "bridge" : "none"
}
```

有关详细信息，请参阅[管理 Docker 网络](../container-networking/network-drivers-topologies.md)。

### <a name="set-docker-security-group"></a>设置 Docker 安全组

当你已登录到 Docker 主机且本地运行了 Docker 命令时，这些命令将通过命名管道运行。 默认情况下，只有管理员组的成员才可以通过此命名管道访问 Docker 引擎。 若要指定具有此访问权限的安全组，请使用 `group` 标记。

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

本部分将告诉你如何从 Windows 10 或 Windows Server 2016 系统中卸载 Docker 和执行 Docker 系统组件的完全清理。

>[!NOTE]
>你必须从提升的 PowerShell 会话中运行这些指令中的所有命令。

### <a name="prepare-your-system-for-dockers-removal"></a>为您的系统准备要删除的 Docker

在卸载 Docker 之前，请确保你的系统上未运行任何容器。

运行以下 cmdlet 检查运行中的容器：

```powershell
# Leave swarm mode (this will automatically stop and remove services and overlay networks)
docker swarm leave --force

# Stop all running containers
docker ps --quiet | ForEach-Object {docker stop $_}
```

在删除 Docker 之前，最好从系统中删除所有容器、容器映像、网络和卷。 你可以通过运行以下 cmdlet 来执行此操作：

```powershell
docker system prune --volumes --all
```

### <a name="uninstall-docker"></a>卸载 Docker

接下来，你将需要实际卸载 Docker。

在 Windows 10 上卸载 Docker

- 转到 Windows 10 计算机上的 "**设置** > "**应用**
- 在 "**应用 & 功能**" 下，查找**窗口的 Docker**
- 转到**Windows** > **卸载**的 Docker

要在 Windows Server 2016 上卸载 Docker，请执行以下操作：

从提升的 PowerShell 会话中，使用**卸载程序包**和**卸载模块**cmdlet 从你的系统中删除 Docker 模块及其相应的程序包管理提供程序，如下例所示：

```powershell
Uninstall-Package -Name docker -ProviderName DockerMsftProvider
Uninstall-Module -Name DockerMsftProvider
```

>[!TIP]
>你可以找到用于安装 Docker 的程序包提供程序 `PS C:\> Get-PackageProvider -Name *Docker*`

### <a name="clean-up-docker-data-and-system-components"></a>清理 Docker 数据和系统组件

卸载 Docker 后，你需要删除 Docker 的默认网络，以便在 Docker 离开后，其配置不会保留在系统上。 你可以通过运行以下 cmdlet 来执行此操作：

```powershell
Get-HNSNetwork | Remove-HNSNetwork
```

运行以下 cmdlet 以从你的系统中删除 Docker 的程序数据：

```powershell
Remove-Item "C:\ProgramData\Docker" -Recurse
```

你可能还需要删除 Windows 上与 Docker/容器关联的 Windows 可选功能。

这包括 "容器" 功能，该功能在安装了 Docker 时在任何 Windows 10 或 Windows Server 2016 上自动启用。 这还可能包括“Hyper-V”功能，安装 Docker 时可在 Windows 10 上自动启用该功能，但必须在 Windows Server 2016 上显式启用该功能。

>[!IMPORTANT]
>[Hyper-v 功能](https://docs.microsoft.com/virtualization/hyper-v-on-windows/about/)是一个通用虚拟化功能，支持的不仅仅是容器。 在禁用 Hyper-v 功能之前，请确保系统上没有任何需要 Hyper-v 的虚拟化组件。

若要删除 Windows 10 上的 Windows 功能，请执行以下操作：

- 转到 **"** > 控制面板**程序** > " 程序**和功能** > **打开或关闭 Windows 功能**。
- 查找要禁用的功能或功能的名称，在本例中为**容器**和（可选） **hyper-v**。
- 取消选中要禁用的功能名称旁边的框。
- 选择 **"确定"**

若要删除 Windows Server 2016 上的 Windows 功能，请执行以下操作：

从提升的 PowerShell 会话中，运行以下 cmdlet 以从你的系统中禁用**容器**和（可选） **hyper-v**功能：

```powershell
Remove-WindowsFeature Containers
Remove-WindowsFeature Hyper-V
```

### <a name="reboot-your-system"></a>重新启动系统

若要完成卸载和清理，请从提升的 PowerShell 会话中运行以下 cmdlet 以重启你的系统：

```powershell
Restart-Computer -Force
```
