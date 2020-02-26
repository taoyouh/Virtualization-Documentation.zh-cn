---
title: 在 Windows 10 上启用 Hyper-V
description: 在 Windows 10 上安装 Hyper-V
keywords: windows 10, hyper-v
author: scooley
ms.date: 02/15/2019
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: 752dc760-a33c-41bb-902c-3bb2ecd9ac86
ms.openlocfilehash: bad59fcc65bf66ab3c6dc940a17111e46a9bc226
ms.sourcegitcommit: 16744984ede5ec94cd265b6bff20aee2f782ca88
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/18/2020
ms.locfileid: "77439694"
---
# <a name="install-hyper-v-on-windows-10"></a>在 Windows 10 上安装 Hyper-V

启用 Hyper-V 以在 Windows 10 上创建虚拟机。  
可以通过多种方式启用 hyper-v，包括使用 Windows 10 控制面板、PowerShell 或使用部署映像服务和管理工具（DISM）。 本文档将逐一介绍每个选项。

> **注意**：Hyper-V 作为可选功能内置于 Windows -- 无需下载 Hyper-V。

## <a name="check-requirements"></a>检查要求

* Windows 10 企业版、专业版或教育版
* 具有二级地址转换 (SLAT) 的 64 位处理器。
* 对 VM 监视器模式扩展（Intel Cpu 上的 VT）的 CPU 支持。
* 最少 4 GB 内存。

**请勿**在 Windows 10 家庭版上安装 Hyper-V 角色。

通过打开 > **更新和安全** > **激活**的**设置**，从 Windows 10 家庭版升级到 windows 10 专业版。

有关详细信息和疑难解答，请参阅 [Windows 10 Hyper-V 系统要求](../reference/hyper-v-requirements.md)。

## <a name="enable-hyper-v-using-powershell"></a>使用 PowerShell 启用 Hyper-V

1. 以管理员身份打开 PowerShell 控制台。

2. 运行以下命令：

  ```powershell
  Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
  ```

  如果无法找到此命令，请确保你以管理员身份运行 PowerShell。

安装完成后，请重启。

## <a name="enable-hyper-v-with-cmd-and-dism"></a>使用 CMD 和 DISM 启用 Hyper-V

部署映像服务和管理工具 (DISM) 可帮助配置 Windows 和 Windows 映像。  在众多应用程序中，DISM 可以在操作系统运行时启用 Windows 功能。

使用 DISM 启用 Hyper-V 角色：

1. 以管理员身份打开 PowerShell 或 CMD 会话。

1. 键入下列命令：

  ```powershell
  DISM /Online /Enable-Feature /All /FeatureName:Microsoft-Hyper-V
  ```

  ![显示正在启用 Hyper-V 的控制台窗口。](media/dism_upd.png)

有关 DISM 的详细信息，请参阅 [DISM 技术参考](<https://docs.microsoft.com/previous-versions/windows/it-pro/windows-8.1-and-8/hh824821(v=win.10)>)。

## <a name="enable-the-hyper-v-role-through-settings"></a>通过“设置”启用 Hyper-V 角色

1. 右键单击 Windows 按钮并选择“应用和功能”。

2. 在 "相关设置" 下的右侧选择 "**程序和功能**"。 

3. 选择“打开或关闭 Windows 功能”。

4. 选择“Hyper-V”，然后单击“确定”。

![Windows 程序和功能对话框](media/enable_role_upd.png)

安装完成后，系统会提示你重新启动计算机。

## <a name="make-virtual-machines"></a>创建虚拟机

[创建第一个虚拟机](quick-create-virtual-machine.md)
