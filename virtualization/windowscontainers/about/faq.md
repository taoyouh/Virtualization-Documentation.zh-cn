---
title: "Windows 容器常见问题解答"
description: "Windows 容器常见问题解答"
keywords: "docker, 容器"
author: PatrickLang
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
translationtype: Human Translation
ms.sourcegitcommit: 2ab9a4b09a2db72e5e2be71ced5d5400761a5ad8
ms.openlocfilehash: b084bf179d9360e4a72e8e88b4fec80eafb2906c

---

# 常见问题

## 关于 Windows 容器

**什么是 Windows Server 容器？**

Windows Server 容器是一种轻型的操作系统虚拟化方法，用于将单独的应用程序或服务与在同一个容器主机上运行的其他服务分离。 为了启用此功能，每个容器都有其自己的操作系统、进程、文件系统、注册表和 IP 地址的视图。  

**什么是 Hyper-V 容器？**

可以将 Hyper-V 容器视为在 Hyper-V 分区内运行的 Windows Server 容器。

Hyper-V 容器在高效、高密度的 Windows Server 容器和高度隔离的硬件虚拟化 Hyper-V 虚拟器之间提供了额外的部署选项。 对于来自不同信任边界的应用程序运行在同一个主机上的环境，可能需要其他隔离。 Hyper-V 容器将使用经优化的虚拟化和 Windows Server 操作系统提供更高的隔离，可将容器互相隔离和与主机操作系统隔离。 两个容器部署选项都使用相同的管理 API、工具和映像格式。 部署时，客户只需选择最符合他们要求的部署模式。

**Linux 和 Windows Server 容器之间的区别是什么？**

Linux 和 Windows Server 容器很相似，两者都在其内核和核心操作系统内实现类似的技术。 区别在于在容器内运行的平台和工作负荷。  
如果客户使用的是 Windows Server 容器，则可以与 .NET、ASP.NET、PowerShell 等现有 Windows 技术集成。

**作为开发人员，我是否必须为每种类型的容器重新编写我的应用？**

否，Windows 容器映像在 Windows Server 容器和 Hyper-V 容器上通用。 在你启动容器时选择容器类型。 从开发人员的角度看，Windows Server 容器和 Hyper-V 容器是同一事物的两种风格。 它们提供相同的开发、编程和管理体验，它们是开放的且可扩展，并且将通过 Docker 包含相同级别的集成和支持。

开发人员可以使用 Windows Server 容器创建容器映像并将其部署为 Hyper-V 容器，反之亦然，而无需除指定相应的运行时标志之外的任何更改。

在注重速度的情况下，Windows Server 容器将提供更高的密度和性能（例如与嵌套配置相比，更低的起转时间、更快的运行时性能）。 Hyper-V 容器提供更好的隔离，从而确保在一个容器中运行的代码无法危害或影响主机操作系统或在同一个主机上运行的其他容器。 这对于多租户方案很有用（带有对托管不受信任代码的要求），包括 SaaS 应用程序和计算托管。

**Hyper-V/Windows Server 容器是否是附加设备，或它们是否将集成在 Windows Server 内？**

容器功能将集成到 Windows Server 2016 中。  

**Windows Server 容器和 Drawbridge 之间的关系是什么？**

Drawbridge 是帮助我们获得对容器的有价值见解的众多研究项目之一。  Windows Server 2016 中的许多容器技术都来源于我们在 Drawbridge 中的经验，我们很高兴将把世界级的容器技术带给 Windows Server 2016 的客户。

**Windows Server 容器和 Hyper-V 容器的先决条件是什么？**

Window Server 容器和 Hyper-V 容器都需要 Windows Server 2016。 这些技术将不适用以前版本的 Windows。


## Windows 容器管理

**Hyper-V 容器是否还可用于 Docker 生态系统？**

是，Hyper-V 容器将通过 Docker 提供与 Windows Server 容器相同级别的集成和管理。  目标是打造开放、一致、跨平台的体验。  
Docker 平台还将大大简化和增强跨容器选项工作的体验。 使用 Windows Server 容器开发的应用程序无需更改即可部署为 Hyper-V 容器。


## Microsoft 的开放生态系统

**Microsoft 是否正在参与开放容器计划 (OCI)？**

为了保证封装格式保持通用，Docker 最近组织了开放容器计划 (OCI)，旨在确保容器封装保持开放和以基础为导向的格式，其中 Microsoft 是创始成员之一。

**Microsoft 是否真的正在与 Docker 合作？**

是。  
我们与 Docker 的合作关系使开发人员可以使用相同的 Docker 工具集创建、管理和部署 Windows Server 和 Linux 容器。 面向 Windows Server 的开发人员将不再需要在使用各种 Windows Server 技术和生成容器化的应用程序之间进行选择。  

Docker 包含两方面，即项目的开源组和 Docker 公司。 我们考虑此合作关系同时包含这两者。 Docker 的成功部分是因为围绕 Docker 容器技术所生成的充满活力的生态系统。 Microsoft 正在致力于 Docker 项目，使其支持 Windows Server 容器和 Hyper-V 容器。  

有关详细信息，请参阅[新的 Windows Server 容器和对 Docker 的 Azure 支持](http://azure.microsoft.com/blog/2014/10/15/new-windows-server-containers-and-azure-support-for-docker/?WT.mc_id=Blog_ServerCloud_Announce_TTD)博客文章。



<!--HONumber=Nov16_HO1-->


