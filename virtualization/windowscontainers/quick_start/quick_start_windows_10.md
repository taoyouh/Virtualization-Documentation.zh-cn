---
title: "Windows 10 上的 Windows 容器"
description: "容器部署快速入门"
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 06/28/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
translationtype: Human Translation
ms.sourcegitcommit: 3fc388632dee4d714ab5a7869fa852c079c11910
ms.openlocfilehash: e2d86c6f1aba07c7c40d1f932b3884c99bfd8f0a

---

# Windows 10 上的 Windows 容器

**这是初步内容，可能还会更改。** 

本练习将演练 Windows 10（预览体验成员版本 14352 及更高版本）上 Windows 容器功能的基本部署和使用。 完成操作后，你将已安装容器角色，并部署简单的 Hyper-V 容器。 在开始本快速入门之前，请先熟悉基本容器概念和术语。 可以在 [Quick Start Introduction](./quick_start.md)（快速入门简介）上找到此信息。 

本快速入门特定于 Windows 10 上的 Hyper-V 容器。 此页面左侧的目录中提供其他快速入门文档。

**先决条件：**

- 一个运行 [Windows 10 预览体验成员版本](https://insider.windows.com/)的物理计算机系统。   
- 本快速入门可以在 Windows 10 虚拟机上运行，但需要启用嵌套虚拟化。 可以在[嵌套虚拟化指南](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting)中找到相关详细信息。

## 1.安装容器功能

需要在使用 Windows 容器之前启用容器功能。 要执行此操作，在提升的 PowerShell 会话中运行以下命令。 

```none
Enable-WindowsOptionalFeature -Online -FeatureName containers -All
```

由于 Windows 10 仅支持 Hyper-V 容器，因此还必须启用 Hyper-V 功能。 若要使用 PowerShell 启用 Hyper-V 功能，请在提升的 PowerShell 会话中运行以下命令。

```none
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

安装完成后，请重启计算机。

```none
Restart-Computer -Force
```

## 2.安装 Docker

若要使用 Window 容器，则需要安装 Docker。 Docker 由 Docker 引擎和 Docker 客户端组成。 此示例中将会安装这两者。 为此，请运行以下命令： 

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

将 Docker 目录添加到系统路径。

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

重启 PowerShell 会话以识别已修改的路径。

若要将 Docker 安装为一个 Windows 服务，请运行以下命令。

```none
dockerd --register-service
```

安装完成后，可以启动该服务。

```none
Start-Service Docker
```

## 3.安装基本容器映像

Windows 容器是从模板或映像部署的。 需要先下载容器基本操作系统映像，才能部署容器。 以下命令将下载 Nano Server 基本映像。
    
为当前 PowerShell 进程设置 PowerShell 执行策略。 这仅将影响当前 PowerShell 会话中执行的脚本，但在更改执行策略时，应非常审慎。

```none
Set-ExecutionPolicy Bypass -scope Process
```

安装容器映像包提供程序。

```none  
Install-PackageProvider ContainerImage -Force
```

然后，选择 Nano Server 映像。

```none
Install-ContainerImage -Name NanoServer
```

安装基本映像之后，需要重启 Docker 服务。

```none
Restart-Service docker
```

在此阶段，运行 `docker images` 将返回一个已安装映像列表，此例中为 Nano Server 映像。

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14300.1016     3f5112ddd185        3 weeks ago         810.2 MB
```

若要继续，需要使用“最新”版本来标记此映像。 为此，请运行以下命令。

```none
docker tag nanoserver:10.0.14300.1016 nanoserver:latest
```

有关 Windows 容器映像的深入信息，请参阅[管理容器映像](../management/manage_images.md)。

## 4.部署第一个容器

对于这个简单的示例，预先创建了 .NET Core 映像。 使用 `docker pull` 年龄下载此映像。

运行时，将启动一个容器，并将执行简单的 .NET Core 应用程序，然后容器将退出。 

```none
docker pull microsoft/sample-dotnet
```

可以通过运行 `docker images` 命令进行相应验证。

```none
docker images

REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
microsoft/sample-dotnet  latest              28da49c3bff4        41 hours ago        918.3 MB
nanoserver               10.0.14300.1016     3f5112ddd185        3 weeks ago         810.2 MB
nanoserver               latest              3f5112ddd185        3 weeks ago         810.2 MB
```

使用 `docker run` 命令运行容器。 下面的示例指定 `--rm` 参数，这将指示 Docker 引擎即时删除不再运行的容器。 

有关 Docker Run 命令的深入信息，请参阅 [Docker.com 上的 Docker Run 参考]( https://docs.docker.com/engine/reference/run/)。

```none
docker run --isolation=hyperv --rm microsoft/sample-dotnet
```

**注意** - 如果发生错误并指示超时事件，请运行以下 PowerShell 脚本，并重试该操作。

```none
Set-ItemProperty -Path 'HKLM:SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers' -Name VSmbDisableOplocks -Type DWord -Value 1 -Force
```

`docker run` 命令的结果是，从 sample-dotnet 映像创建 Hyper-V 容器，然后执行一个示例应用程序（输出回显到 shell），然后停止并删除容器。 Windows 10 和容器快速入门的后续部分将深入探讨在 Windows 10 上的容器中创建和部署应用程序。

## 后续步骤

[Windows Server 上的 Windows 容器](./quick_start_windows_server.md)





<!--HONumber=Jun16_HO4-->


