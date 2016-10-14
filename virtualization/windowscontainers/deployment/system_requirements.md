---
title: "Windows 容器要求"
description: "Windows 容器要求。"
keywords: "元数据, 容器"
author: neilpeterson
manager: timlt
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
translationtype: Human Translation
ms.sourcegitcommit: 08355d7a7da50d0f244bd750508fd42084818d7f
ms.openlocfilehash: ac48bc7ee7b70483d8a368749aea0862c52f049c

---

# Windows 容器要求

本指南列出 Windows 容器主机的要求。

## 操作系统要求

- Windows 容器功能仅适用于 Windows Server 2016（核心和桌面体验）、Nano Server 和 Windows 10 专业版和企业版（周年纪念版）。
- 如果将运行 Hyper-V 容器，则需要安装 Hyper-V 角色。
- Windows Server 容器主机必须将 Windows 安装到 c:\. 如果仅部署 Hyper-V 容器，则不会应用此限制。

## 虚拟化的容器主机

如果 Windows 容器主机将从 Hyper-V 虚拟机运行，并且还将承载 Hyper-V 容器，则需要启用嵌套虚拟化。 嵌套的虚拟化具有以下要求：

- 至少 4 GB RAM 可用于虚拟化的 Hyper-V 主机。
- Windows Server 2016 或主机系统上的 Windows 10 以及 Windows Server（Full、Core）或虚拟机中的 Nano Server。
- 带有 Intel VT-x 处理器（此功能目前只适用于 Intel 处理器）。
- 容器主机虚拟机还需要至少 2 个虚拟处理器。

## 支持的基本映像

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
<td><center>Windows Server 2016（桌面）</center></td>
<td><center>Server Core/Nano Server</center></td>
<td><center>Server Core/Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Core</center></td>
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


<!--HONumber=Oct16_HO2-->


