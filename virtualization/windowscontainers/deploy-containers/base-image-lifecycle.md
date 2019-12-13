---
title: 基本映像服务生命周期
description: 有关 Windows 容器基本映像生命周期的信息。
keywords: windows 容器，容器，生命周期，版本信息，基本映像，容器基础映像
author: Heidilohr
ms.author: helohr
ms.date: 06/17/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: bb5e5fabadde421de9d420edd2fc921457432930
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909987"
---
# <a name="base-image-servicing-lifecycles"></a>基本映像服务生命周期

Windows 容器基本映像基于半年频道版本或 Windows Server 的长期服务通道版本。 本文将介绍这两个通道中不同版本的基本映像的持续时间。

半年频道是每年两次的功能更新版本，每个版本包含十八个月的服务时间线。 这样，客户就可以更快地利用新的操作系统功能，无论是在应用程序（尤其是在容器和微服务上构建的应用程序），还是软件定义的混合数据中心。 有关详细信息，请参阅[Windows Server 半年频道概述](https://docs.microsoft.com/windows-server/get-started/semi-annual-channel-overview)。

对于 Server Core 映像，客户还可以使用长期服务通道，每隔两到三年发布 Windows Server 的新主版本。 长期服务通道版本接收五年的主流支持和5年的扩展支持。 此通道适用于需要更长维护服务选项和功能稳定性的系统。

下表列出了每种类型的基础映像、其服务通道以及其支持的持续时间。

|Base image                       |服务渠道|版本|操作系统内部版本|可用性|主要支持结束日期|延期支持日期|
|---------------------------------|-----------------|-------|--------|------------|---------------------------|---------------------|
|Server Core、Nano Server、Windows|半年      |1909   |18363   |2019/11/12  |2021/05/11                 |N/A                  |
|Server Core、Nano Server、Windows|半年      |1903   |18362   |05/21/2019  |2020/12/08                 |N/A                  |
|服务器核心                      |长期        |1809   |17763   |2018/11/13  |2024/01/09                 |2029/01/09           |
|Server Core、Nano Server、Windows|半年      |1809   |17763   |2018/11/13  |05/12/2020                 |N/A                  |
|服务器核心，Nano Server         |半年      |1803   |17134   |2018/04/30  |2019/11/12                 |N/A                  |
|服务器核心，Nano Server         |半年      |1709   |16299   |2017/10/17  |04/09/2019                 |N/A                  |
|服务器核心                      |长期        |1607   |14393   |2016/10/15  |2022/01/11                 |2027/01/11           |
|Nano Server                      |半年      |1607   |14393   |2016/10/15  |10/09/2018                 |N/A                  |

有关服务要求和其他附加信息，请参阅 [Windows 生命周期常见问题解答](https://support.microsoft.com/help/18581/lifecycle-faq-windows-products)、 [windows Server 版本信息](https://docs.microsoft.com/windows-server/get-started/windows-server-release-info)和[windows 基本操作系统映像 Docker 中心存储库](https://hub.docker.com/_/microsoft-windows-base-os-images)。
