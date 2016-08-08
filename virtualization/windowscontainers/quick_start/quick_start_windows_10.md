---
title: "Windows 10 上的 Windows 容器"
description: "容器部署快速入门"
keywords: "docker, 容器"
author: neilpeterson
manager: timlt
ms.date: 07/13/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
translationtype: Human Translation
ms.sourcegitcommit: 6c7ce9f1767c6c6391cc6d33a553216bd815ff72
ms.openlocfilehash: bd93f5a73268b552710304d7da568e1497239679

---

# Windows 10 上的 Windows 容器

**这是初步内容，可能还会更改。** 

本练习将演练 Windows 10（预览体验成员版本 14372 及更高版本）上 Windows 容器功能的基本部署和使用。 完成操作后，你将已安装容器角色，并部署简单的 Hyper-V 容器。 在开始本快速入门之前，请先熟悉基本容器概念和术语。 可以在 [Quick Start Introduction](./quick_start.md)（快速入门简介）上找到此信息。 

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

完成备份后，运行以下命令以修复 Windows 容器技术预览中的已知问题。  

 ```none
Set-ItemProperty -Path 'HKLM:SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers' -Name VSmbDisableOplocks -Type DWord -Value 1 -Force
```

## 2.安装 Docker

若要使用 Window 容器，则需要安装 Docker。 Docker 由 Docker 引擎和 Docker 客户端组成。 此示例中将会安装这两者。 为此，请运行以下命令： 

以 zip 存档形式下载 Docker 引擎和客户端。

```none
Invoke-WebRequest "https://get.docker.com/builds/Windows/x86_64/docker-1.12.0.zip" -OutFile "$env:TEMP\docker-1.12.0.zip" -UseBasicParsing
```

将 zip 存档展开到 Program Files，存档内容已经位于 Docker 目录中。

```none
Expand-Archive -Path "$env:TEMP\docker-1.12.0.zip" -DestinationPath $env:ProgramFiles
```

将 Docker 目录添加到系统路径。

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";$env:ProgramFiles\docker\", [EnvironmentVariableTarget]::Machine)
```

重启 PowerShell 会话以识别已修改的路径。

若要将 Docker 安装为一个 Windows 服务，请运行以下命令。

```none
& $env:ProgramFiles\docker\dockerd.exe --register-service
```

安装完成后，可以启动该服务。

```none
Start-Service Docker
```

## 3.安装基本容器映像

Windows 容器是从模板或映像部署的。 需要先下载容器基本操作系统映像，才能部署容器。 以下命令将下载 Nano Server 基本映像。
    
> 此过程适用于高于 14372 的 Windows 预览体验成员版本并且在“docker pull”起作用之前临时使用。

下载 Nano Server 基本映像。 

```none
Start-BitsTransfer https://aka.ms/tp5/6b/docker/nanoserver -Destination nanoserver.tar.gz
```

安装基本映像。

```none  
docker load -i nanoserver.tar.gz
```

在此阶段，运行 `docker images` 将返回一个已安装映像列表，此例中为 Nano Server 映像。

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14300.1030     3f5112ddd185        3 weeks ago         810.2 MB
```

若要继续，需要使用“最新”版本来标记此映像。 为此，请运行以下命令。

```none
docker tag microsoft/nanoserver:10.0.14300.1030 nanoserver:latest
```

有关 Windows 容器映像的深入信息，请参阅[管理容器映像](../management/manage_images.md)。

## 4.部署第一个容器

对于此简单示例，将创建和部署一个“Hello World”容器映像。 为获得最佳体验，请在升级后的 Windows 命令行界面运行这些命令。

首先，从 `nanoserver` 映像启动一个具有交互式会话的容器。 启动容器后，容器中将显示一个命令行界面。  

```none
docker run -it nanoserver cmd
```

在容器中创建一个简单的“Hello World”脚本。

```none
powershell.exe Add-Content C:\helloworld.ps1 'Write-Host "Hello World"'
```   

完成后，退出该容器。

```none
exit
```

现在从修改后的容器创建一个新的容器映像。 若要查看容器列表，请运行以下项并记住容器 ID。

```none
docker ps -a
```

运行以下命令以创建 HelloWorld 映像。 将 <containerid> 替换为你的容器 ID。

```none
docker commit <containerid> helloworld
```

完成后，现在你就具有一个包含“hello world”脚本的自定义映像了。 通过执行以下命令可以看到该映像。

```none
docker images
```

最后，要运行该容器，请使用 `docker run` 命令。

```none
docker run --rm helloworld powershell c:\helloworld.ps1
```

`docker run` 命令的结果是，从 HelloWorld 映像创建 Hyper-V 容器，然后执行一个“Hello World”脚本（输出回显到该界面），然后停止并删除容器。 Windows 10 和容器快速入门的后续部分将深入探讨在 Windows 10 上的容器中创建和部署应用程序。

## 后续步骤

[Windows Server 上的 Windows 容器](./quick_start_windows_server.md)





<!--HONumber=Aug16_HO1-->


