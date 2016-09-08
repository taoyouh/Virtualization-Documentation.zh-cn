---
title: "Windows Server 上的 Windows 容器"
description: "容器部署快速入门"
keywords: "docker, 容器"
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
translationtype: Human Translation
ms.sourcegitcommit: 39e480b8bf3f90cfe9b7d4d17141b9dbec5f93e5
ms.openlocfilehash: 6fae5625731c936903c808e286dbb9b4abcddd06

---

# Windows Server 上的 Windows 容器

**这是初步内容，可能还会更改。**

本练习将演练 Windows Server 上 Windows 容器功能的基本部署和使用。 完成操作后，你将已安装容器角色，并部署简单的 Windows Server 容器。 在开始本快速入门之前，请先熟悉基本容器概念和术语。 可以在 [Quick Start Introduction](./quick_start.md)（快速入门简介）上找到此信息。

本快速入门特定于 Windows Server 2016 上的 Windows Server 容器。 此页面左侧的目录中提供其他快速入门文档。

**先决条件：**

一个运行 [Windows Server 2016 Technical Preview 5](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-technical-preview) 的计算机系统（物理或虚拟）。

Azure 中提供了完全配置的 Windows Server 映像。 若要使用此映像，请通过单击下面的按钮来部署虚拟机。 部署将需要大约 10 分钟。 完成后，登录到 Azure 的虚拟机，跳到本教程的第四步。 

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVirtualization-Documentation%2Fmaster%2Fwindows-server-container-tools%2Fcontainers-azure-template%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

## 1.安装容器功能

需要在使用 Windows 容器之前启用容器功能。 要执行此操作，在提升的 PowerShell 会话中运行以下命令。

```none
Install-WindowsFeature containers
```

功能安装完成后，重启计算机。

```none
Restart-Computer -Force
```

## 2.安装 Docker

若要使用 Window 容器，则需要安装 Docker。 Docker 由 Docker 引擎和 Docker 客户端组成。 此示例中将会安装这两者。

以 zip 存档形式下载 Docker 引擎和客户端。

```none
Invoke-WebRequest "https://get.docker.com/builds/Windows/x86_64/docker-1.12.0.zip" -OutFile "$env:TEMP\docker-1.12.0.zip" -UseBasicParsing
```

将 Zip 存档扩展到 Program Files。

```none
Expand-Archive -Path "$env:TEMP\docker-1.12.0.zip" -DestinationPath $env:ProgramFiles
```

将 Docker 目录添加到系统路径。

```none
# For quick use, does not require shell to be restarted.
$env:path += ";c:\program files\docker"

# For persistent use, will apply even after a reboot. 
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

若要将 Docker 安装为一个 Windows 服务，请运行以下命令。

```none
dockerd --register-service
```

安装完成后，可以启动该服务。

```none
Start-Service docker
```

## 3.安装基本容器映像

Windows 容器是从模板或映像部署的。 需要先下载基本操作系统映像，才能部署容器。 以下命令将下载 Windows Server Core 基本映像。

```none
docker pull microsoft/windowsservercore
```

此过程可能需要花费一些时间，因此稍作休息，待拉取完成后返回继续。

拉取映像后，运行 `docker images` 将返回已安装的映像的列表，此例中为 Windows Server Core 映像。

```none
docker images

REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
microsoft/windowsservercore   latest              02cb7f65d61b        8 weeks ago         7.764 GB
```

有关 Windows 容器映像的深入信息，请参阅[管理容器映像](../management/manage_images.md)。

## 4.部署第一个容器

对于此练习，你将从 Docker Hub 注册表下载预先创建的 IIS 映像，并部署运行 IIS 的简单容器。  

若要在 Docker Hub 中搜索 Windows 容器映像，请运行 `docker search Microsoft`。  

```none
docker search microsoft

NAME                                         DESCRIPTION
microsoft/aspnet                             ASP.NET is an open source server-side Web ...
microsoft/dotnet                             Official images for working with .NET Core...
mono                                         Mono is an open source implementation of M...
microsoft/azure-cli                          Docker image for Microsoft Azure Command L...
microsoft/iis                                Internet Information Services (IIS) instal...
microsoft/mssql-server-2014-express-windows  Microsoft SQL Server 2014 Express installe...
microsoft/nanoserver                         Nano Server base OS image for Windows cont...
microsoft/windowsservercore                  Windows Server Core base OS image for Wind...
microsoft/oms                                Monitor your containers using the Operatio...
microsoft/dotnet-preview                     Preview bits for microsoft/dotnet image
microsoft/dotnet35
microsoft/applicationinsights                Application Insights for Docker helps you ...
microsoft/sample-redis                       Redis installed in Windows Server Core and...
microsoft/sample-node                        Node installed in a Nano Server based cont...
microsoft/sample-nginx                       Nginx installed in Windows Server Core and...
microsoft/sample-httpd                       Apache httpd installed in Windows Server C...
microsoft/sample-dotnet                      .NET Core running in a Nano Server container
microsoft/sqlite                             SQLite installed in a Windows Server Core ...
...
```

使用 `docker pull` 下载 IIS 映像。  

```none
docker pull microsoft/iis
```

可以通过 `docker images` 命令验证映像下载。 请注意，你将在此处看到基本映像 (windowsservercore) 和 IIS 映像。

```none
docker images

REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
microsoft/iis                 latest              accd044753c1        11 days ago         7.907 GB
microsoft/windowsservercore   latest              02cb7f65d61b        8 weeks ago         7.764 GB
```

使用 `docker run` 部署 IIS 容器。

```none
docker run -d -p 80:80 microsoft/iis ping -t localhost
```

此命令将 IIS 映像作为后台服务 (-d) 运行，并对网络进行配置，以便容器主机的端口 80 映射到容器的端口 80。
有关 Docker Run 命令的深入信息，请参阅 [Docker.com 上的 Docker Run 参考]( https://docs.docker.com/engine/reference/run/)。


可通过运行 `docker ps` 命令看到正在运行的容器。 记住容器名称，以便在后面的步骤中使用。

```none
docker ps

CONTAINER ID  IMAGE          COMMAND              CREATED             STATUS             PORTS               NAME
09c9cc6e4f83  microsoft/iis  "ping -t localhost"  About a minute ago  Up About a minute  0.0.0.0:80->80/tcp  big_jang
```

从不同的计算机中，打开 Web 浏览器并输入容器主机的 IP 地址。 如果已正确配置所有内容，你应看到 IIS 初始屏幕。 这是从 Windows 容器中托管的 IIS 实例提供的。

**注意：**如果正在使用 Azure，将需要虚拟机的外部 IP 地址和配置的网络安全。 有关详细信息，请参阅 [Create Rule in a Network Security Group]( https://azure.microsoft.com/en-us/documentation/articles/virtual-networks-create-nsg-arm-pportal/#create-rules-in-an-existing-nsg)（在网络安全组中创建规则）。

![](media/iis1.png)

返回到容器主机上，使用 `docker rm` 命令删除容器。 注意 - 将此示例中的容器名称替换为实际容器名称。

```none
docker rm -f big_jang
```
## 后续步骤

[Windows Server 上的容器映像](./quick_start_images.md)

[Windows 10 上的 Windows 容器](./quick_start_windows_10.md)



<!--HONumber=Aug16_HO4-->


