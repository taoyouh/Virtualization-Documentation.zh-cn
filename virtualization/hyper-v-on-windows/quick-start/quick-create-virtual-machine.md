---
title: 使用 Hyper-V 创建虚拟机
description: 在 Windows 10 创意者更新上使用 Hyper-V 创建新虚拟机
keywords: windows 10, hyper-v, 虚拟机
author: scooley
ms.date: 04/07/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: f1e75efa-8745-4389-b8dc-91ca931fe5ae
ms.openlocfilehash: 970e92def02e5386d38a2e72d5ef921aa8321fdf
ms.sourcegitcommit: 08cc38955faad26f075b912a64b8ffb6b36f190c
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/26/2019
ms.locfileid: "9578680"
---
# <a name="create-a-virtual-machine-with-hyper-v"></a>使用 Hyper-V 创建虚拟机

创建虚拟机并为其安装操作系统。

我们一直在构建用于创建虚拟机，因此，说明已显著发生在过去的三个版本的新工具。

选择你的操作系统以获取合适的一组说明：

* [Windows 10 Fall Creators Update (版本 1709) 及更高版本](quick-create-virtual-machine.md#windows-10-fall-creators-update-windows-10-version-1709)
* [Windows 10 创意者更新 (版本 1703)](quick-create-virtual-machine.md#windows-10-creators-update-windows-10-version-1703)
* [Windows 10 周年更新 (版本 1607) 及更早版本](quick-create-virtual-machine.md#before-windows-10-creators-update-windows-10-version-1607-and-earlier)

让我们开始吧。

## <a name="windows-10-fall-creators-update-windows-10-version-1709"></a>Windows 10 Fall Creators Update (Windows 10 版本 1709年)

在 Fall Creators Update 中，“快速创建”进行了扩展，以包括可以独立从 Hyper-V 管理器中启动的虚拟机库。

若要在 Fall Creators Update 中创建新虚拟机，请执行以下操作：

1. 从“开始”菜单中打开“Hyper-V Quick Create”。

    ![Windows“开始”菜单中的“快速创建”库](media/quick-create-start-menu.png)

1. 选择一个操作系统或者使用本地安装源选择你自己的操作系统。

    ![库视图](media/vmgallery.png)

    1. 如果你想要使用自己的映像创建虚拟机，请选择 **Local Installation Source**。
    1. 选择 **Change Installation Source**。
      ![用于使用本地安装源的按钮](media/change-source.png)
    1. 选择要转变为新虚拟机的 .iso 或 .vhdx。
    1. 如果映像为 Linux 映像，请取消选择“安全启动”选项。
      ![用于使用本地安装源的按钮](media/toggle-secure-boot.png)

1. 选择“创建虚拟机”

就这么简单！  “快速创建”将完成其余的工作。

## <a name="windows-10-creators-update-windows-10-version-1703"></a>Windows 10 创意者更新 (Windows 10 版本 1703年)

![“快速创建”UI 的屏幕截图](media/quickcreatesteps_inked.jpg)

1. 从“开始”菜单打开 Hyper-V 管理器。

1. 在 Hyper-V 管理器内右侧的**操作**菜单中查找**快速创建**。

1. 自定义你的虚拟机。

    * （可选）为虚拟机命名。
    * 选择虚拟机的安装媒体。 你可以从 .iso 或 .vhdx 文件中安装。
    如果要在虚拟机中安装 Windows，你可以启用 Windows 安全启动。 否则，请不要选中。
    * 设置网络。
    如果你有现成的虚拟交换机，则可以在网络下拉列表中进行选择。 如果你没有现成的交换机，你将看到一个用于设置自动网络的按钮，该按钮可以自动配置虚拟网络。

1. 单击**连接**以启动虚拟机。 无需担心编辑设置，你可以随时返回去更改设置。

    系统可能会提示你“按任意键以从 CD 或 DVD 启动”。 按照提示继续操作。  据了解，你将从 CD 安装。

恭喜，你有了新的虚拟机。  现在，你可以安装操作系统了。

你的虚拟机应如下所示：

![虚拟机启动屏幕](media/OSDeploy_upd.png)

> **注意：** 除非你运行的是批量许可版本的 Windows，否则需要为虚拟机内运行的 Windows 提供单独的许可证。 虚拟机的操作系统独立于主机操作系统。

## <a name="before-windows-10-creators-update-windows-10-version-1607-and-earlier"></a>Windows 10 创意者更新 (Windows 10 版本 1607 及更早版本) 之前

如果运行的不是 Windows 10 创意者更新或更高版本，请按照以下说明进行操作并改用新的虚拟机向导：

1. [创建虚拟网络](connect-to-network.md)
1. [创建新的虚拟机](create-virtual-machine.md)
