---
title: 基础映像服务生命周期
description: 有关 Windows 容器基础映像服务生命周期的信息。
keywords: windows 容器, 容器, 生命周期, 版本信息, 基础映像, 容器基础映像
author: Heidilohr
ms.author: helohr
ms.date: 06/17/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 2dcd228af0984b55162894555fa21f9e02dd1934
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/15/2020
ms.locfileid: "81395740"
---
# <a name="base-image-servicing-lifecycles"></a>基础映像服务生命周期

> [!Note]  
> 为了让人们和组织将精力集中在保持业务连续性上，Microsoft 已经推迟了一些产品预定的支持和服务结束日期。 有关详细信息，请参阅自 2020 年 4 月 14 日以来的[支持和服务结束日期的生命周期更改](https://support.microsoft.com/en-us/help/4557164/lifecycle-changes-to-end-of-support-and-servicing-dates)条目。

Windows 容器基础映像基于 Windows Server 的半年频道版本或长期服务频道版本。 本文将介绍对这两个频道中不同版本的基础映像的支持会持续多长时间。

半年频道每年发布两次功能更新，每个版本的服务时间线为 18 个月。 这使得客户可以在应用程序（特别是基于容器和微服务的应用程序）中以及软件定义的混合数据中心中更快地利用新的操作系统功能。 有关详细信息，请参阅 [Windows Server 半年频道概述](https://docs.microsoft.com/windows-server/get-started/semi-annual-channel-overview)。

对于 Server Core 映像，客户还可以使用长期服务频道，该频道每隔两到三年为 Windows Server 发布新的主要版本。 长期服务频道版本享受 5 年的主要支持和 5 年的外延支持。 此频道适用于需要更长时间服务选项和功能稳定性的系统。

下表列出了每种类型的基础映像、其服务频道以及支持持续时间。

|Base image                       |服务频道|版本|操作系统内部版本|可用性|主要支持结束日期|外延支持日期|
|---------------------------------|-----------------|-------|--------|------------|---------------------------|---------------------|
|Server Core、Nano Server、Windows|半年      |1909   |18363   |2019/11/12  |2021/05/11                 |N/A                  |
|Server Core、Nano Server、Windows|半年      |1903   |18362   |2019/05/21  |2020/12/08                 |N/A                  |
|服务器核心                      |长期        |2019   |17763   |2018/11/13  |2024/01/09                 |2029/01/09           |
|Server Core、Nano Server、Windows|半年      |1809   |17763   |2018/11/13  |2020/11/10                 |N/A                  |
|Server Core、Nano Server         |半年      |1803   |17134   |2018/04/30  |2019/11/12                 |N/A                  |
|Server Core、Nano Server         |半年      |1709   |16299   |2017/10/17  |2019/04/09                 |N/A                  |
|服务器核心                      |长期        |1607   |14393   |2016/10/15  |2022/01/11                 |2027/01/11           |
|Nano Server                      |半年      |1607   |14393   |2016/10/15  |2018/10/09                 |N/A                  |

有关服务要求和其他附加信息，请参阅  [Windows 生命周期常见问题解答](https://support.microsoft.com/help/18581/lifecycle-faq-windows-products)、[Windows Server 版本信息](https://docs.microsoft.com/windows-server/get-started/windows-server-release-info)和 [Windows 基础操作系统映像 Docker Hub 存储库](https://hub.docker.com/_/microsoft-windows-base-os-images)。
