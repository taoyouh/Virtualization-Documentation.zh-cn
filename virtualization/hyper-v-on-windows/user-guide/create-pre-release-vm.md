---
title: 尝试 Hyper-V 的预发行功能
description: 尝试 Hyper-V 的预发行功能
keywords: windows 10, hyper-v
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 426c87cc-fa50-4b8d-934e-0b653d7dea7d
ms.openlocfilehash: 8f1c1b96fe88f46a24b8ebb46d4f387c9717f6ba
ms.sourcegitcommit: 34d8b2ca5eebcbdb6958560b1f4250763bee5b48
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "9620705"
---
# <a name="try-pre-release-features-for-hyper-v"></a>尝试 Hyper-V 的预发行功能

> 这是初步内容，可能还会更改。  
  预发行版虚拟机用于开发或测试环境，仅因为它不受 Microsoft 支持。

提前获得 Windows Server 2016 Technical Preview 上 Hyper-V 的预发行功能，在你的开发或测试环境中试用。 你可以率先了解最新的 Hyper-V 功能，通过提供早期反馈帮助产品定型。

作为预发行版创建的虚拟机不具有内部版本到内部版本的兼容性或将来的支持。  请勿在生产环境中使用。

以下是为何它们仅适用于非生产环境的其他原因：

* 预发行版虚拟机没有向前兼容性。 无法将这些虚拟机升级到新的配置版本。
* 预发行版虚拟机的内部版本间的定义不一致。 如果更新主机操作系统，则现有的预发行版虚拟机可能与主机不兼容。 这些虚拟机可能无法启动，或可能最初看起来工作正常，但后来会遇到重大的兼容性问题。
* 如果将预发行版虚拟机导入到具有其他内部版本的主机，则结果不可预知。 可以将预发行版虚拟机移动到另一台主机。 但只有两台主机运行相同的内部版本，这种情况才应该会正常工作。

## <a name="create-a-pre-release-virtual-machine"></a>创建预发行版虚拟机

可以在运行 Windows Server 2016 Technical Preview 的 Hyper-V 主机上创建预发行版虚拟机。

1. 在 Windows 桌面上，单击“开始”按钮并键入名称 **Windows PowerShell** 的任一部分。
2. 右键单击**Windows PowerShell**，然后选择**以管理员身份运行**。
3. 将 [NEW-VM](https://docs.microsoft.com/powershell/module/hyper-v/new-vm?view=win10-ps) cmdlet 与 -Prerelease 标志配合使用，以创建预发行版虚拟机。 例如，运行以下命令，其中 VM 名称是你想要创建的虚拟机的名称。

``` PowerShell
New-VM -Name <VM Name> -Prerelease
```
使用 -Prerelease 标志的其他示例：
 - 若要创建使用现有的虚拟硬盘或新硬盘的虚拟机，请参阅 [Create a virtual machine in Hyper-V on Windows Server 2016 Technical Preview](https://docs.microsoft.com/windows-server/virtualization/hyper-v/get-started/Create-a-virtual-machine-in-Hyper-V#BKMK_PowerShell)（在 Windows Server 2016 Technical Preview 上的 Hyper-V 中创建虚拟机）中的 PowerShell 示例。
 - 若要创建引导到操作系统映像的新虚拟硬盘，请参阅 [Deploy a Windows Virtual Machine in Hyper-V on Windows 10](https://docs.microsoft.com/virtualization/hyper-v-on-windows/quick-start/create-virtual-machine)（在 Windows 10 上的 Hyper-V 中部署 Windows 虚拟机）中的 PowerShell 示例。

 这些文章中介绍的示例适用于运行 Windows 10 或 Windows Server 2016 Technical Preview 的 Hyper-V 主机。 但现在，你只能使用 -Prerelease 标志在运行 Windows Server 2016 Technical Preview 的 Hyper-V 主机上创建预发行版虚拟机。

## <a name="see-also"></a>另请参阅
-  [虚拟化博客](https://techcommunity.microsoft.com/t5/Virtualization/bg-p/Virtualization) - 了解可用的预发行版功能以及如何进行尝试。
- [Supported virtual machine configuration versions](https://docs.microsoft.com/windows-server/virtualization/hyper-v/deploy/Upgrade-virtual-machine-version-in-Hyper-V-on-Windows-or-Windows-Server#BKMK_SupportedConfigVersions)（受支持的虚拟机配置版本）-了解如何检查虚拟机配置版本以及 Microsoft 支持哪些版本。
