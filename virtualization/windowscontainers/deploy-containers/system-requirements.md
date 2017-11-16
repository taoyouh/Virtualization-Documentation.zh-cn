---
title: "Windows 容器要求"
description: "Windows 容器要求。"
keywords: "元数据, 容器"
author: enderb-ms
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: f4ee9346db77e29f9d3366634b8b6ad07d0fec08
ms.sourcegitcommit: 380dd8e78780995b96def2e2ec6e22e3387e82e0
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/09/2017
---
# <a name="windows-container-requirements"></a>Windows 容器要求

本指南列出 Windows 容器主机的要求。

## <a name="os-requirements"></a>操作系统要求

- Windows 容器功能仅适用于 Windows Server 版本 1709、Windows Server 2016（Core 和附带桌面体验的版本）以及 Windows 10 专业版和企业版（周年纪念版）。
- 运行 Hyper-V 容器之前必须安装 Hyper-V 角色
- Windows Server 容器主机必须将 Windows 安装到 c:\。 如果将仅部署 Hyper-V 容器，则不会应用此限制。

## <a name="virtualized-container-hosts"></a>虚拟化的容器主机

如果 Windows 容器主机将从 Hyper-V 虚拟机运行，并且还将承载 Hyper-V 容器，则需要启用嵌套虚拟化。 嵌套的虚拟化具有以下要求：

- 至少 4 GB RAM 可用于虚拟化的 Hyper-V 主机。
- 主机系统使用 Windows Server 版本 1709、Windows Server 2016 或 Windows 10，虚拟机使用 Windows Server（Full、Core）。
- 带有 Intel VT-x 处理器（此功能目前只适用于 Intel 处理器）。
- 容器主机虚拟机还需要至少 2 个虚拟处理器。

## <a name="supported-base-images"></a>支持的基本映像

Windows 容器提供两种容器基本映像，Windows Server Core 和 Nano Server。 并非所有配置都支持这两个操作系统映像。 下表详细介绍所支持的配置。

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>主机操作系统</center></th>
<th><center>Windows Server 容器</center></th>
<th><center>Hyper-V 容器</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>Windows Server 2016（Standard 或 Datacenter）</center></td>
<td><center>Server Core/Nano Server</center></td>
<td><center>Server Core/Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Nano Server</center></td>
<td><center> Nano Server</center></td>
<td><center>Server Core/Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Windows 10 专业版/企业版</center></td>
<td><center>不可用</center></td>
<td><center>Server Core/Nano Server</center></td>
</tr>
</tbody>
</table>

### <a name="nano-server-vs-windows-server-core"></a>Nano Server 与 Windows Server Core

如何在 Windows Server Core 和 Nano Server 之间进行选择？ 虽然可以随意使用它们之中的任何一个进行生成，但如果你发现应用程序需要与 .NET Framework 完全兼容，则应使用 [Windows Server Core](https://hub.docker.com/r/microsoft/windowsservercore/)。 另一方面，如果你的应用程序是针对云生成的并且使用 .NET Core，则应使用 [Nano Server](https://hub.docker.com/r/microsoft/nanoserver/)。 这是因为 Nano Server 的设计目标是尽可能减小占用空间，因此删除了几个不重要的库。 考虑在 Nano Server 上生成时，应注意以下几点：

- 已删除服务堆栈
- 未包含 .NET Core（但你可以使用 [.NET Core Nano Server 映像](https://hub.docker.com/r/microsoft/dotnet/)）
- 已删除 PowerShell
- 已删除 WMI

以上是最大的差异，但并未详尽列出所有差异。 还有一些缺失的其他组件并未在此列出。 请记住，你始终可以在自己认为合适的条件下在 Nano Server 上添加层。 有关此情况的示例，请查看 [.NET Core Nano Server Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.0/sdk/nanoserver/amd64/Dockerfile)。

## <a name="matching-container-host-version-with-container-image-versions"></a>将容器主机版本与容器映像版本相匹配
### <a name="windows-server-containers"></a>Windows Server 容器
由于 Windows Server 容器和基础主机共享一个内核，因此容器基本映像必须与主机基本映像相匹配。  如果版本不同，容器虽然可以启动，但其功能完整性得不到保证。 因此不支持不匹配的版本。  Windows 操作系统具有 4 个级别的版本，主要版本、次要版本、内部版本和修订版本 – 例如 10.0.14393.0。 只有在发布新版本的操作系统后，内部版本号才会改变。 应用 Windows 更新后，会相应更新修订版本号。 如果内部版本号不同（例如 10.0.14300.1030 (Technical Preview 5) 和 10.0.14393 (Windows Server 2016 RTM)），则会阻止 Windows Server 容器启动。 如果内部版本号相同但修订版本号不同（例如 10.0.14393 (Windows Server 2016 RTM) 和 10.0.14393.206 (Windows Server 2016 GA)），则不会阻止 Windows Server 容器启动。 即使技术上没有阻止容器启动，但此配置可能无法在所有环境下正常运行，因此不支持配置到产品环境。 

若要查看 Windows 主机已安装的版本类型，可以查询 HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion。  若要查看基础映像使用的版本类型，可以查看 Docker 中心上的标记或查看映像说明提供的映像哈希表。  [Windows 10 更新历史记录](https://support.microsoft.com/en-us/help/12387/windows-10-update-history)页面列出了每个内部版本和修订版本发布的时间。

在此示例中，14393 是主要内部版本号，321 是修订版本号。
```none
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Users\Administrator> (Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\').BuildLabEx
14393.321.amd64fre.rs1_release_inmarket.161004-2338
```

### <a name="hyper-v-isolation-for-containers"></a>容器 Hyper-V 隔离
无论带有还是不带有 Hyper-V 隔离，Windows 容器都可运行。  Hyper-V 隔离使用优化的虚拟机在容器周围创造安全边界。  Hyper-V 隔离容器与标准 Windows 容器不同，后者在容器和主机之间共享内核，而 Hyper-V 隔离容器则是各自使用自己的 Windows 内核实例。  出于此原因，容器主机和映像中可以有不同的操作系统版本（请参阅下面的兼容性矩阵）。  

若要运行带有 Hyper-V 隔离的容器，只需在你的 Docker 运行命令中添加标记“--isolation=hyper-v”。

### <a name="compatibility-matrix"></a>兼容性矩阵
只要配置受支持，无论修订版本号是多少，2016 GA (10.0.14393.206) 之后的 Windows Server 版本都可以运行 Windows Server Core 或 Nano Server 的 Windows Server 2016 GA 映像。    

为了获得由 Windows 更新提供的全部功能、可靠性和安全保障，需要保证所有系统为最新版本，这一点至关重要。  
