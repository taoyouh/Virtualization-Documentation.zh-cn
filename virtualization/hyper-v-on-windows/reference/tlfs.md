---
title: 虚拟机监控程序规范
description: 虚拟机监控程序规范
keywords: windows 10, hyper-v
author: allenma
ms.date: 06/26/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: aee64ad0-752f-4075-a115-2d6b983b4f49
ms.openlocfilehash: f1b33bf805f7868bb42ee1f49965c2ea9a45916f
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/06/2020
ms.locfileid: "78853971"
---
# <a name="hypervisor-specifications"></a>虚拟机监控程序规范

## <a name="hypervisor-top-level-functional-specification"></a>虚拟机监控程序顶层功能规范

Hyper-V 虚拟机监控程序顶层功能规范 (TLFS) 描述了虚拟机监控程序对其他操作系统组件的外部可见的行为。 此规范对来宾操作系统开发人员很有用。
  
> 此规范根据 Microsoft 开放规范承诺书而提供。  阅读以下内容，进一步了解有关 [Microsoft 开放规范承诺书](https://docs.microsoft.com/openspecs/dev_center/ms-devcentlp/51a0d3ff-9f77-464c-b83f-2de08ed28134)的详细信息。  

#### <a name="download"></a>下载
版本 | 文档
--- | ---
Windows Server Standard 2012 R2 | [Hypervisor Top Level Functional Specification v6.0.pdf](https://github.com/MicrosoftDocs/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v6.0.pdf)
Windows Server 2016（修订版 C） | [Hypervisor Top Level Functional Specification v5.0c.pdf](https://github.com/MicrosoftDocs/Virtualization-Documentation/raw/live/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v5.0C.pdf)
Windows Server 2012 R2（修订版 B） | [Hypervisor Top Level Functional Specification v4.0b.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v4.0b.pdf)
Windows Server 2012 | [Hypervisor Top Level Functional Specification v3.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v3.0.pdf)
Windows Server 2008 R2 | [Hypervisor Top Level Functional Specification v2.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v2.0.pdf)

## <a name="requirements-for-implementing-the-microsoft-hypervisor-interface"></a>实现 Microsoft 虚拟机管理程序接口的要求

TLFS 全面介绍了 Microsoft 指定虚拟机监控程序体系结构，该体系结构对来宾虚拟机来说即“HV#1”接口。  然而，TLFS 中描述的部分接口不需要通过声明遵循 Microsoft HV#1 虚拟机监控程序规范的第三方虚拟机监控程序实现。 若要了解最少有多少虚拟机监控程序接口必须通过声明遵循 Microsoft HV#1 接口规范的虚拟机监控程序实现，请参阅《实现 Microsoft 虚拟机监控程序接口的要求》文档。

#### <a name="download"></a>下载

[Requirements for Implementing the Microsoft Hypervisor Interface.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Requirements%20for%20Implementing%20the%20Microsoft%20Hypervisor%20Interface.pdf)
