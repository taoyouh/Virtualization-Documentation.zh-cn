---
title: "Windows 容器文档"
description: "Windows 容器文档"
keywords: "docker, 容器"
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 74c9d604-0915-4d89-bc69-0263b76bc66b
translationtype: Human Translation
ms.sourcegitcommit: f721639b1b10ad97cc469df413d457dbf8d13bbe
ms.openlocfilehash: 1ee40d330234f8800ba73d0c4abe36859cfa2989

---

# Windows 容器文档

Windows 容器提供操作系统级别的虚拟化，允许多个独立的应用程序在单个系统上运行。 该功能附带两种不同类型的容器运行时，每个都有不同程度的应用程序隔离。 Windows Server 容器通过命名空间实现隔离并处理隔离。 Hyper-V 容器在轻型虚拟机中封装每个容器。 此文档集提供了有关管理操作的快速入门指南、部署指南和技术详细信息。

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:90%" cellpadding="25" cellspacing="5">
<tr>
<td ><center>![](media/try.png)</center></td>
<td>**快速启动**<br /><br />
Windows Server 快速入门<br /><br />
<ul>
<li>[步骤1 – 概念和术语](quick_start/quick_start.md)<br /><br /></li>
<li>[步骤 2 – 配置 Windows Server 和第一个容器](quick_start/quick_start_windows_server.md)<br /><br /></li>
<li>[步骤 3 – 创建并推送容器映像](quick_start/quick_start_images.md)<br /><br /></li>
</ul>
Windows 10 快速入门<br /><br />
<ul>
<li>[步骤1 – 概念和术语](quick_start/quick_start.md)<br /><br /></li>
<li>[步骤 2 – 配置 Windows 10 和第一个容器](quick_start/quick_start_windows_10.md)<br /><br /></li>
</ul>
</td>
</tr>
<tr>
<td ><center>![](media/1.png)</center></td>
<td>**部署**<br /><br />
了解如何在 Windows Server 2016 和 Nano Server 上部署 Windows 容器。<br /><br />
<ul>
<li>[系统要求](deployment/system_requirements.md)<br /><br /></li>
<li>[部署容器主机 - Windows Server](deployment/deployment.md)<br /><br /></li>
<li>[部署容器主机 - Nano Server](deployment/deployment_nano.md)<br /><br /></li>
<li>[防病毒优化](https://msdn.microsoft.com/en-us/windows/hardware/drivers/ifs/anti-virus-optimization-for-windows-containers)<br /><br /></li>
</ul>
</td>
</tr>

<tr>
<td ><center>![](media/explore.png)</center></td>
<td>**Windows 上的 Docker**<br /><br />
了解如何在 Windows 上管理 Docker。<br /><br />
<ul>
<li>[Windows 上的 Docker 引擎](docker/configure_docker_daemon.md)<br /><br /></li>
<li>[Windows 上的 Dockerfile](docker/manage_windows_dockerfile.md)<br /><br /></li>
<li>[管理容器数据](management/manage_data.md)<br /><br /></li>
<li>[优化 Dockerfile](docker/optimize_windows_dockerfile.md)<br /><br /></li>
<li>[容器网络](management/container_networking.md)<br /><br /></li>
</ul>
</td>
</tr>

<tr>
<td ><center>![](media/video.png)</center></td>
<td>**观看**<br /><br />
对来自 Windows 容器团队的演示和访谈感兴趣？<br /><br />
<ul>
<li>[容器频道](https://channel9.msdn.com/Blogs/containers)</li>
</ul>
<br />
</td>
</tr>

<tr>
<td ><center>![](media/question.png)</center></td>
<td>**社区**<br /><br />
与社区交互、试用示例，以及查找其他资源。<br /><br />
<ul>
<li>[容器论坛](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)<br /><br /></li>
<li>[容器资源](https://msdn.microsoft.com/virtualization/community/community_overview)<br /><br /></li>
</ul>
</td>
</tr>
</table>



<!--HONumber=Sep16_HO4-->


