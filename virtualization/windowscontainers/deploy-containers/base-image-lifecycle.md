---
title: 基本图像服务生命周期
description: 有关 Windows 容器基本映像生命周期的信息。
keywords: windows 容器、容器、生命周期、发布信息、基本图像、容器基图像
author: Heidilohr
ms.author: helohr
ms.date: 06/17/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: bb5e5fabadde421de9d420edd2fc921457432930
ms.sourcegitcommit: e61db4d98d9476a622e6cc8877650d9e7a6b4dd9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/13/2019
ms.locfileid: "10288135"
---
# <a name="base-image-servicing-lifecycles"></a>基本图像服务生命周期

Windows 容器基本映像基于半年度频道发布或 Windows Server 长期服务通道版本。 本文将告诉你对两个频道中不同版本的基本图像的支持时间有多长。

半年频道是每年两次的功能更新版本，每个版本有十八个月的服务时间线。 这使客户能够以更快的速度利用新的操作系统功能，无论是在应用程序中（尤其是在容器和 microservices 上构建的，也在软件定义的混合数据中心中）。 有关详细信息，请参阅[Windows Server 半年度频道概述](https://docs.microsoft.com/windows-server/get-started/semi-annual-channel-overview)。

对于服务器核心映像，客户还可以使用长期服务通道，每隔两至三年释放 Windows Server 的新主要版本。 长期服务频道发布接收五年的主流支持和五年的延长支持。 此通道适用于需要较长服务选项和功能稳定性的系统。

下表列出了每种类型的基本映像、其服务通道以及其支持的持续时间。

|基本图像                       |服务渠道|版本|操作系统内部版本|可用性|主要支持结束日期|延长的支持日期|
|---------------------------------|-----------------|-------|--------|------------|---------------------------|---------------------|
|服务器核心版、Nano Server、Windows|半年度      |1909   |18363   |11/12/2019  |05/11/2021                 |不适用                  |
|服务器核心版、Nano Server、Windows|半年度      |1903   |18362   |05/21/2019  |12/08/2020                 |不适用                  |
|服务器核心                      |长期        |1809   |17763   |2018/11/13  |2024/01/09                 |2029/01/09           |
|服务器核心版、Nano Server、Windows|半年度      |1809   |17763   |2018/11/13  |05/12/2020                 |不适用                  |
|服务器 Core、Nano Server         |半年度      |1803   |17134   |04/30/2018  |11/12/2019                 |不适用                  |
|服务器 Core、Nano Server         |半年度      |1709   |16299   |10/17/2017  |04/09/2019                 |不适用                  |
|服务器核心                      |长期        |1607   |14393   |10/15/2016  |01/11/2022                 |2027/01/11           |
|Nano Server                      |半年度      |1607   |14393   |10/15/2016  |10/09/2018                 |不适用                  |

有关服务要求和其他其他信息，请参阅 [Windows 生命周期常见问题解答](https://support.microsoft.com/help/18581/lifecycle-faq-windows-products)、 [windows Server 发布信息](https://docs.microsoft.com/windows-server/get-started/windows-server-release-info)和[windows 基本操作系统映像 Docker 中心存储库](https://hub.docker.com/_/microsoft-windows-base-os-images)。
