---
title: Windows Container Requirements
description: Windows Container Requirements.
keywords: metadata, containers
author: enderb-ms
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: 89d66c7c5515532ab9bc7ebfcf0c79e59ebd7d28
ms.sourcegitcommit: 33ba39d65b08aa61d8a5f5fdf2822dc28d2e3b3a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/24/2017
---
# Windows container requirements

This guides list the requirements for a Windows container Host.

## OS Requirements

- The Windows container feature is only available on Windows Server 2016 (Core and with Desktop Experience), Nano Server, and Windows 10 Professional and Enterprise (Anniversary Edition).
- The Hyper-V role must be installed before running Hyper-V Containers
- Windows Server Container hosts must have Windows installed to c:\. This restriction does not apply if only Hyper-V Containers will be deployed.

## Virtualized Container Hosts

If a Windows container host will be run from a Hyper-V virtual machine, and will also be hosting Hyper-V Containers, nested virtualization will need to be enabled. Nested virtualization has the following requirements:

- At least 4 GB RAM available for the virtualized Hyper-V host.
- Windows Server 2016, or Windows 10 on the host system, and Windows Server (Full, Core) or Nano Server in the virtual machine.
- A processor with Intel VT-x (this feature is currently only available for Intel processors).
- The container host VM will also need at least 2 virtual processors.

## Supported Base Images

Windows Containers are offered with two container base images, Windows Server Core and Nano Server. Not all configurations support both OS images. This table details the supported configurations.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>Host Operating System</center></th>
<th><center>Windows Server Container</center></th>
<th><center>Hyper-V Container</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>Windows Server 2016 (Standard or Datacenter)</center></td>
<td><center>Server Core / Nano Server</center></td>
<td><center>Server Core / Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Nano Server</center></td>
<td><center> Nano Server</center></td>
<td><center>Server Core / Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Windows 10 Pro / Enterprise</center></td>
<td><center>Not Available</center></td>
<td><center>Server Core/Nano Server</center></td>
</tr>
</tbody>
</table>

### Nano Server 与 Windows Server Core

如何在 Windows Server Core 和 Nano Server 之间进行选择？ 虽然可以随意使用它们之中的任何一个进行生成，但如果你发现应用程序需要与 .NET Framework 完全兼容，则应使用 [Windows Server Core](https://hub.docker.com/r/microsoft/windowsservercore/)。 另一方面，如果你的应用程序是针对云生成的并且使用 .NET Core，则应使用 [Nano Server](https://hub.docker.com/r/microsoft/nanoserver/)。 这是因为 Nano Server 的设计目标是尽可能减小占用空间，因此删除了几个不重要的库。 考虑在 Nano Server 上生成时，应注意以下几点：

- 已删除服务堆栈
- 未包含 .NET Core（但你可以使用 [.NET Core Nano Server 映像](https://hub.docker.com/r/microsoft/dotnet/)）
- 已删除 PowerShell
- 已删除 WMI

以上是最大的差异，但并未详尽列出所有差异。 还有一些缺失的其他组件并未在此列出。 请记住，你始终可以在自己认为合适的条件下在 Nano Server 上添加层。 有关此情况的示例，请查看 [.NET Core Nano Server Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.0/sdk/nanoserver/amd64/Dockerfile)。

## 将容器主机版本与容器映像版本相匹配
### Windows Server Containers
Because Windows Server Containers and the underlying host share a single kernel, the container’s base image must match that of the host.  If the versions are different the container may start, but full functionally cannot be guaranteed. Therefore mismatched versions are not supported.  The Windows operating system has 4 levels of versioning, Major, Minor, Build and Revision – for example 10.0.14393.0. The build number only changes when new versions of the OS are published. The revision number is updated as Windows updates are applied. Windows Server Containers are blocked from starting when the build number is different - for example 10.0.14300.1030 (Technical Preview 5) and 10.0.14393 (Windows Server 2016 RTM). If the build number matches but the revision number is different, it is not blocked from starting - for example 10.0.14393 (Windows Server 2016 RTM) and 10.0.14393.206 (Windows Server 2016 GA). Even though they are not technically blocked, this is a configuration that may not function properly under all circumstances and thus cannot be supported for production environments. 

To check what version a Windows host has installed you can query HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion.  To check what version your base image is using you can review the tags on the Docker hub or the image hash table provided in the image description.  The [Windows 10 Update History](https://support.microsoft.com/en-us/help/12387/windows-10-update-history) page lists when each build and revision was released.

In this example 14393 is the major build number and 321 is the revision.
```none
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Users\Administrator> (Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\').BuildLabEx
14393.321.amd64fre.rs1_release_inmarket.161004-2338
```

### Hyper-V Isolation for Containers
Windows Containers can be run with or without Hyper-V isolation.  Hyper-V isolation creates a secure boundary around the container with an optimized VM.  Unlike standard Windows Containers, which share the kernel between containers and the host, each Hyper-V isolated container has its own instance of the Windows kernel.  出于此原因，容器主机和映像中可以有不同的操作系统版本（请参阅下面的兼容性矩阵）。  

若要运行带有 Hyper-V 隔离的容器，只需在你的 Docker 运行命令中添加标记“--isolation=hyper-v”。

### 兼容性矩阵
Windows Server builds after 2016 GA (10.0.14393.206) can run the Windows Server 2016 GA images of both Windows Server Core or Nano Server in a supported configuration regardless of the revision number.    

It is important to understand that in order to have the full functionality, reliability and security assurances provided with Windows updates you should maintain the latest versions on all systems.  
