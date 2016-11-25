---
title: "Windows 10 上的 Windows 容器"
description: "容器部署快速入门"
keywords: "docker, 容器"
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
translationtype: Human Translation
ms.sourcegitcommit: 9b99982abfbbda12758bb1c922ed1bd431ecca20
ms.openlocfilehash: c9f3a7669ae82e0b3a91956336d67225687c715b

---

# Windows 10 上的 Windows 容器

本练习将演练 Windows 10 专业版或企业版（周年纪念版）上 Windows 容器功能的基本部署和用法。 完成操作后，你将已安装容器角色，并部署简单的 Hyper-V 容器。 在开始本快速入门之前，请先熟悉基本容器概念和术语。 可以在 [Quick Start Introduction](./quick_start.md)（快速入门简介）上找到此信息。

本快速入门特定于 Windows 10 上的 Hyper-V 容器。 此页面左侧的目录中提供其他快速入门文档。

**先决条件：**

- 一个运行 Windows 10 周年纪念版（专业版或企业版）的物理计算机系统。   
- 本快速入门可以在 Windows 10 虚拟机上运行，但需要启用嵌套虚拟化。 可以在[嵌套虚拟化指南](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting)中找到相关详细信息。

> 必须安装关键更新，Windows 容器才会工作。 
> 若要检查 OS 版本，请运行 `winver.exe`，并将显示的版本与 [Windows 10 更新历史记录](https://support.microsoft.com/en-us/help/12387/windows-10-update-history)进行比较。 
> 请确保拥有 14393.222 或更高版本再继续操作。

## 1.安装容器功能

需要在使用 Windows 容器之前启用容器功能。 要执行此操作，在**提升的** PowerShell 会话中运行以下命令。

如果收到一条错误，显示 `Enable-WindowsOptionalFeature` 不存在，请仔细检查是否以管理员身份运行 PowerShell。

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

> 如果以前使用的是Windows 10 上的 Hyper-V 容器和 Technical Preview 5 容器基本映像，请务必重新启用 Oplocks。 请运行以下命令：  `Set-ItemProperty -Path 'HKLM:SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers' -Name VSmbDisableOplocks -Type DWord -Value 0 -Force`

## 2.安装 Docker

若要使用 Window 容器，则需要安装 Docker。 Docker 由 Docker 引擎和 Docker 客户端组成。 此示例中将会安装这两者。 为此，请运行以下命令：

以 zip 存档形式下载 Docker 引擎和客户端。

```none
Invoke-WebRequest "https://master.dockerproject.org/windows/amd64/docker-1.13.0-dev.zip" -OutFile "$env:TEMP\docker-1.13.0-dev.zip" -UseBasicParsing
```

将 zip 存档展开到 Program Files，存档内容已经位于 Docker 目录中。

```none
Expand-Archive -Path "$env:TEMP\docker-1.13.0-dev.zip" -DestinationPath $env:ProgramFiles
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
Start-Service Docker
```

## 3.安装基本容器映像

Windows 容器是从模板或映像部署的。 需要先下载容器基本操作系统映像，才能部署容器。 以下命令将下载 Nano Server 基本映像。

拉取 Nano Server 基本映像。

```none
docker pull microsoft/nanoserver
```

拉取映像后，运行 `docker images` 将返回已安装的映像的列表，此例中为 Nano Server 映像。

```none
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

有关 Windows 容器映像的深入信息，请参阅[管理容器映像](../management/manage_images.md)。

> 可在此处 ([EULA](../Images_EULA.md)) 阅读 Windows 容器操作系统映像 EULA。

## 4.部署第一个容器

对于此简单示例，将创建和部署一个“Hello World”容器映像。 为获得最佳体验，请在升级后的Windows CMD shell 或 PowerShell中运行这些命令。

> Windows PowerShell ISE 不适用于与容器的交互式会话。 即使容器正在运行，也会显示为挂起。

首先，从 `nanoserver` 映像启动一个具有交互式会话的容器。 启动容器后，容器中将显示一个命令行界面。  

```none
docker run -it microsoft/nanoserver cmd
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

`docker run` 命令的结果是，从 HelloWorld 映像创建 Hyper-V 容器，然后执行一个“Hello World”脚本（输出回显到该界面），然后停止并删除容器。
Windows 10 和容器快速入门的后续部分将深入探讨在 Windows 10 上的容器中创建和部署应用程序。

## 后续步骤

[Windows Server 上的 Windows 容器](./quick_start_windows_server.md)



<!--HONumber=Nov16_HO2-->


