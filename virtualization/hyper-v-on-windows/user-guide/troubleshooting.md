---
title: Windows 10 上的 Hyper-V 疑难解答
description: Windows 10 上的 Hyper-V 疑难解答
keywords: windows 10, hyper-v
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: f0ec8eb4-ffc4-4bf1-9a19-7a8c3975b359
ms.openlocfilehash: 4d1b7b310d0df7c198d5446b339a9c38279c72db
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/26/2019
ms.locfileid: "9575138"
---
# <a name="troubleshoot-hyper-v-on-windows-10"></a>Windows 10 上的 Hyper-V 疑难解答

## <a name="i-updated-to-windows-10-and-now-i-cant-connect-to-my-downlevel-windows-81-or-server-2012-r2-host"></a>我已更新到 Windows 10，但现在我无法连接到下层（Windows 8.1 或 Server 2012 R2）主机。
在 Windows 10 中，Hyper-V 管理器已移动到 WinRM 进行远程管理。  这意味着现在必须在远程主机上启用远程管理，才能使用 Hyper-V 管理器管理它。

有关详细信息，请参阅[管理远程 Hyper-V 主机](https://technet.microsoft.com/windows-server-docs/compute/hyper-v/manage/Remotely-manage-Hyper-V-hosts)

## <a name="i-changed-the-checkpoint-type-but-it-is-still-taking-the-wrong-type-of-checkpoint"></a>我已更改检查点类型，但它还是采用错误类型的检查点
如果你采用 VMConnect 的检查点，并且在 Hyper-V 管理器中更改检查点类型，所采用的检查点会是打开 VMConnect 时指定的任意检查点类型。

关闭 VMConnect 并重新打开，以使其采用正确类型的检查点。

## <a name="when-i-try-to-create-a-virtual-hard-disk-on-a-flash-drive-an-error-message-is-displayed"></a>当我尝试在 U 盘上创建虚拟硬盘时，将显示一条错误消息。
Hyper-V 不支持 FAT/FAT32 格式化的磁盘驱动器，因为这些文件系统不提供访问控制列表 (ACL)，并且不支持大于 4GB 的文件。 ExFAT 格式化的磁盘仅提供有限的 ACL 功能，因此出于安全原因，这些磁盘不受支持。
在 PowerShell 中显示的错误消息为“系统无法创建‘\[VHD 路径\]’：由于文件系统限制，所请求的操作无法完成 (0x80070299)。”

改为使用 NTFS 格式化的驱动器。 

## <a name="i-get-this-message-when-i-try-to-install-hyper-v-cannot-be-installed-the-processor-does-not-support-second-level-address-translation-slat"></a>在尝试安装时收到此消息：“无法安装 Hyper-V：处理器不支持二级地址转换 (SLAT)。”
Hyper-V 需要使用 SLAT 才能运行虚拟机。 如果你的计算机不支持 SLAT，则它无法成为虚拟机的主机。

如果仅尝试安装管理工具，请在“**程序和功能**” > “**打开或关闭 Windows 功能**”中取消选择“**Hyper-V 平台**”。
