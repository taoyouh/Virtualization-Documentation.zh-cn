---
title: Windows 容器要求
description: Windows 容器要求。
keywords: 元数据, 容器
author: enderb-ms
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: 4706ea24da1d5ca61b94dfd141883aa2d04ad906
ms.sourcegitcommit: 7c3af076eb8bad98e1c3de0af63dacd842efcfa3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2018
ms.locfileid: "1844046"
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
<td><center>Nano Server<a href="#warn-1">*</a></center></td>
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

> [!Warning]  
> <span id="warn-1">从 Windows Server 版本 1709 开始，Nano Server 不再作为容器主机提供。</span>


### <a name="memory-requirements"></a>内存要求
可以通过[资源控制](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/resource-controls)或重载容器主机来配置容器可用内存限值。  下面列出了启动容器和运行基本命令（ipconfig、dir 等）所需的最小内存量。  __请注意，这些值未考虑容器之间的资源共享或来自在容器中运行的应用程序的要求。  例如，具有 512MB 可用内存的主机可以在 Hyper-V 隔离下运行多个 Server Core 容器，因为这些容器会共享资源。__

#### <a name="windows-server-2016"></a>WindowsServer 2016
| 基本映像  | Windows Server 容器 | Hyper-V 隔离    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 40MB                     | 130MB + 1GB 页面文件 |
| 服务器核心 | 50MB                     | 325MB + 1GB 页面文件 |

#### <a name="windows-server-version-1709"></a>Windows Server 版本 1709
| 基本映像  | Windows Server 容器 | Hyper-V 隔离    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 30MB                     | 110MB + 1GB 页面文件 |
| 服务器核心 | 45MB                     | 360MB + 1GB 页面文件 |


### <a name="nano-server-vs-windows-server-core"></a>Nano Server 与 Windows Server 核心

如何在 Windows Server Core 和 Nano Server 之间进行选择？ 虽然可以随意使用它们之中的任何一个进行生成，但如果你发现应用程序需要与 .NET Framework 完全兼容，则应使用 [Windows Server Core](https://hub.docker.com/r/microsoft/windowsservercore/)。 另一方面，如果你的应用程序是针对云生成的并且使用 .NET Core，则应使用 [Nano Server](https://hub.docker.com/r/microsoft/nanoserver/)。 这是因为 Nano Server 的设计目标是尽可能减小占用空间，因此删除了几个不重要的库。 考虑在 Nano Server 上生成时，应注意以下几点：

- 已删除服务堆栈
- 未包含 .NET Core（但你可以使用 [.NET Core Nano Server 映像](https://hub.docker.com/r/microsoft/dotnet/)）
- 已删除 PowerShell
- 已删除 WMI
- 自 Windows Server 版本 1709 起，在用户上下文环境下运行应用程序时，需要管理员权限的命令将会失败。 可以通过用户标记（即 docker run - 用户 ContainerAdministrator）指定容器管理员帐户，不过未来我们计划将管理员帐户从 NanoServer 中完全移除。

以上是最显著的差异，其他差异并未一一列出。 还有一些缺失的其他组件并未在此列出。 请记住，你始终可以在自己认为合适的条件下在 Nano Server 上添加层。 有关此情况的示例，请查看 [.NET Core Nano Server Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.0/sdk/nanoserver/amd64/Dockerfile)。

