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
ms.openlocfilehash: 354469199f3c7e886760e8a391edccde067986af
ms.sourcegitcommit: c48dcfe43f73b96e0ebd661164b6dd164c775bfa
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "9610287"
---
# <a name="docker-engine-on-windows"></a>Windows 上的 Docker 引擎

Docker 引擎和客户端不包含在 Windows，并且需要进行安装和单独配置。 此外，Docker 引擎可以接受多种自定义配置。 例如，可以配置守护程序接受传入请求的方式、默认网络选项及调试/日志设置。 在 Windows 上，这些配置可以在配置文件中指定，或者通过使用 Windows 服务控制管理器指定。 本文档详细介绍了如何安装和配置 Docker 引擎，并且还提供了一些常用的配置的示例。

## <a name="install-docker"></a>安装 Docker

若要使用 Windows 容器需要 Docker。 Docker 由 Docker 引擎 (dockerd.exe) 和 Docker 客户端 (docker.exe) 组成。 获取安装的所有内容的最简单方法是快速入门指南，这将帮助你获取所有内容设置和运行你的第一个容器中。

- [在 Windows Server 2019 上的 Windows 容器](../quick-start/quick-start-windows-server.md)
- [Windows 10 上的 Windows 容器](../quick-start/quick-start-windows-10.md)

脚本化安装，请参阅[使用脚本安装 Docker EE](https://docs.docker.com/install/windows/docker-ee/#use-a-script-to-install-docker-ee)。

你可以使用 Docker 之前，你将需要安装容器映像。 有关详细信息，请参阅[使用图像的快速入门指南](../quick-start/quick-start-images.md)。

## <a name="configure-docker-with-a-configuration-file"></a>使用配置文件配置 Docker

在 Windows 上配置 Docker 引擎的首选方法是使用配置文件。 可在“C:\ProgramData\Docker\config\daemon.json”中找到配置文件。 如果它尚不存在，你可以创建此文件。

>[!NOTE]
>并非所有可用的 Docker 配置选项适用于 Windows 上的 Docker。 下面的示例显示执行应用的配置选项。 有关 Docker 引擎配置的详细信息，请参阅[Docker 守护程序配置文件](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)。

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

只需将所需的配置更改添加到配置文件。 例如，下面的示例配置 Docker 引擎，使接受端口 2375年传入的连接。 其他所有配置选项将使用默认值。

```json
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

同样，下面的示例将配置 Docker 守护程序以将图像和容器保存在备用路径。 如果未指定，默认值是`c:\programdata\docker`。

```json
{    
    "data-root": "d:\\docker"
}
```

下面的示例配置 Docker 守护程序以仅接受通过端口 2376年的安全的连接。

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

也可以通过修改 Docker 服务配置 Docker 引擎`sc config`。 使用此方法时将直接在 Docker 服务上设置 Docker 引擎的标记。 在命令提示符（cmd.exe 而非 PowerShell）中运行以下命令：

```cmd
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

>[!NOTE]
>你无需运行此命令，如果 daemon.json 文件已经包含`"hosts": ["tcp://0.0.0.0:2375"]`条目。

## <a name="common-configuration"></a>常见的配置

以下配置文件示例演示了通用的 Docker 配置。 这些配置可以并入单个配置文件。

### <a name="default-network-creation"></a>创建默认网络

若要配置 Docker 引擎，以便它不会创建默认 NAT 网络，请使用以下配置。

```json
{
    "bridge" : "none"
}
```

有关详细信息，请参阅[管理 Docker 网络](../container-networking/network-drivers-topologies.md)。

### <a name="set-docker-security-group"></a>设置 Docker 安全组

当你登录到 Docker 主机并在本地运行 Docker 命令时，将通过命名管道运行这些命令。 默认情况下，只有管理员组的成员才可以通过此命名管道访问 Docker 引擎。 若要指定具有此访问权限的安全组，请使用 `group` 标记。

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

本部分将告诉你如何卸载 Docker，并从 Windows 10 或 Windows Server 2016 系统执行完整的 Docker 系统组件清理。

>[!NOTE]
>你必须从提升的 PowerShell 会话这些说明中运行所有命令。

### <a name="prepare-your-system-for-dockers-removal"></a>准备你的系统以删除 Docker 的

卸载 Docker 之前，请确保你的系统上运行任何容器。

运行下列 cmdlet 检查正在运行的容器：

```powershell
# Leave swarm mode (this will automatically stop and remove services and overlay networks)
docker swarm leave --force

# Stop all running containers
docker ps --quiet | ForEach-Object {docker stop $_}
```

它也是好的做法删除所有容器、 容器映像、 网络和卷系统中删除 Docker 之前。 你可以通过运行以下 cmdlet 来执行此操作：

```powershell
docker system prune --volumes --all
```

### <a name="uninstall-docker"></a>卸载 Docker

接下来，你将需要实际卸载 Docker。

Windows 10 上卸载 Docker

- 转到**设置** > Windows 10 计算机上的**应用**
- 在**应用 & 功能**下, 找到**用于 Windows 的 Docker**
- 转到**适用于 Windows 的 Docker** > **卸载**

要卸载 Windows Server 2016 上的 Docker:

从提升的 PowerShell 会话，用于**卸载程序包**和**卸载模块**cmdlet 从你的系统中删除 Docker 模块及其相应的程序包管理提供程序在下面的示例所示：

```powershell
Uninstall-Package -Name docker -ProviderName DockerMsftProvider
Uninstall-Module -Name DockerMsftProvider
```

>[!TIP]
>你可以查找曾用于安装 Docker 的程序包提供程序 `PS C:\> Get-PackageProvider -Name *Docker*`

### <a name="clean-up-docker-data-and-system-components"></a>清理 Docker 数据和系统组件

卸载 Docker 后，你将需要删除 Docker 的默认网络，使其配置不会保留在系统上 Docker 后消失。 你可以通过运行以下 cmdlet 来执行此操作：

```powershell
Get-HNSNetwork | Remove-HNSNetwork
```

运行以下 cmdlet 从系统中删除 Docker 的计划数据：

```powershell
Remove-Item "C:\ProgramData\Docker" -Recurse
```

你可能还需要删除 Windows 上与 Docker/容器关联的 Windows 可选功能。

这包括"容器"功能，安装 Docker 时会在任何 Windows 10 或 Windows Server 2016 上自动启用该功能。 这还可能包括“Hyper-V”功能，安装 Docker 时可在 Windows 10 上自动启用该功能，但必须在 Windows Server 2016 上显式启用该功能。

>[!IMPORTANT]
>[HYPER-V 功能](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/about/)是一种常规虚拟化功能，使不仅仅容器。 之前禁用 HYPER-V 功能，请确保没有其他虚拟化的组件系统上需要 HYPER-V。

若要删除 Windows 10 上的 Windows 功能：

- 转到**控制面板** > **程序** > **程序和功能** > **打开或关闭 Windows 功能**。
- 查找你想要禁用的功能的功能的名称-在此情况下，**容器**和**HYPER-V**（可选）。
- 取消选中你想要禁用的功能名称旁边的框。
- 选择 **"确定"**

若要删除 Windows Server 2016 上的 Windows 功能：

从提升的 PowerShell 会话，请运行以下 cmdlet，若要禁用的**容器**和 （可选） 你的系统中的**HYPER-V**功能：

```powershell
Remove-WindowsFeature Containers
Remove-WindowsFeature Hyper-V
```

### <a name="reboot-your-system"></a>重新启动系统

若要完成卸载和清理，请从提升的 PowerShell 会话中重新启动系统运行以下 cmdlet:

```powershell
Restart-Computer -Force
```
