---
title: "Windows 10 Hyper-V 系统要求"
description: "Windows 10 Hyper-V 系统要求"
keywords: windows 10, hyper-v
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 6e5e6b01-7a9d-4123-8cc7-f986e10cd372
ms.openlocfilehash: 36f95049c227b1498540e1ee5c9194a09fbfeb83
ms.sourcegitcommit: d5f30aa1bdfb34dd9e1909d73b5bd9f4153d6b46
ms.translationtype: HT
ms.contentlocale: zh-CN
---
# <a name="windows-10-hyper-v-system-requirements"></a>Windows 10 Hyper-V 系统要求

Hyper-V 可用于 Windows 8 及更高版本的 64 位 Windows 专业版、企业版和教育版。  Hyper-V 需要二级地址转换 (SLAT) - 存在于 Intel 和 AMD 最新一代的 64 位处理器中。

你可以在具有 4GB RAM 的主机上运行 3 或 4 台基本虚拟机，但如果要运行更多的虚拟机则需要更多的资源。 另外，你可能还需要创建有 32 个处理器和 512GB RAM 的较大虚拟机，具体取决于你的物理硬件。

## <a name="operating-system-requirements"></a>操作系统要求

可以在以下版本的 Windows 10 上启用 Hyper-V 角色：

- Windows 10 企业版
- Windows 10 专业版
- Windows 10 教育版

**不能**在以下版本上安装 Hyper-V 角色：

- Windows 10 家庭版
- Windows 10 移动版
- Windows 10 移动企业版

>Windows 10 家庭版可以升级到 Windows 10 专业版。 若要执行此操作，请依次打开“**设置**” > “**更新和安全**” > “**激活**”。 可以在此处访问应用商店并购买升级。

## <a name="hardware-requirements"></a>硬件要求

虽然本文档未提供兼容 Hyper-V 的硬件完整列表，但需要具备以下各项：
    
- 具有二级地址转换 (SLAT) 的 64 位处理器。
- CPU 支持 VM 监视器模式扩展（Intel CPU 上的 VT-c）。
- 最少 4 GB 内存。 由于虚拟机与 Hyper-V 主机共享内存，因此将需要提供足够的内存来处理预期虚拟工作负荷。

需要在系统 BIOS 中启用以下各项：
- 虚拟化技术 - 可能具有不同标记，具体取决于主板制造商。
- 硬件强制实施的数据执行保护。

## <a name="verify-hardware-compatibility"></a>验证硬件兼容性

若要验证兼容性，请打开 PowerShell 或命令提示符 (cmd.exe)，然后键入 **systeminfo**。 如果列出的所有 Hyper-V 要求都具有值 **Yes**，则你的系统可以运行 Hyper-V 角色。 如果任一项返回**No**，请查看本文档中列出的要求并进行调整（如果可能）。

![](media/SystemInfo-upd.png)

在现有 Hyper-V 主机上运行 **systeminfo** 时，Hyper-V 要求部分读取如下内容：

```
Hyper-V Requirements: A hypervisor has been detected. Features required for Hyper-V are not be displayed.
```