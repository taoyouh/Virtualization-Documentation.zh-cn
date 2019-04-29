---
title: 与 Windows 虚拟机共享设备
description: 本节将引导你与 Hyper-V 虚拟机共享设备（USB、音频、麦克风和加载的驱动器）
keywords: windows 10, hyper-v
ms.author: scooley
ms.date: 10/20/2017
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: d1aeb9cb-b18f-43cb-a568-46b33346a188
ms.openlocfilehash: 52d51fca03f454a311a123f20e5aeda9376fdc3d
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/26/2019
ms.locfileid: "9578518"
---
# <a name="share-devices-with-your-virtual-machine"></a>与你的虚拟机共享设备

> 仅适用于 Windows 虚拟机。

增强会话模式可通过 RDP（远程桌面协议）将 Hyper-V 与虚拟机连接起来。  这不仅会改善你的整体虚拟机查看体验，而且使用 RDP 连接还可以使虚拟机与你的计算机共享设备。  由于 RDP 在 Windows 10 中默认打开，所以与 Windows 虚拟机连接时，你可能已经在使用 RDP。  本文着重介绍了一些好处和连接设置对话框中的隐藏选项。

RDP/增强会话模式：

* 使虚拟机实现可调整大小和高 DPI 感知。
* 改进虚拟机集成
  * 共享的剪贴板
  * 通过拖放和复制粘贴进行文件共享
* 允许设备共享
  * 麦克风/扬声器
  * USB 设备
  * 数据磁盘（包括 C:）
  * 打印机

本文介绍了如何查看会话类型、进入增强会话模式和配置会话设置。

## <a name="share-drives-and-devices"></a>共享驱动器和设备

与虚拟机连接时会弹出一个连接窗口，增强会话模式的设备共享功能就隐藏在这个不显眼的连接窗口里面：

![](media/esm-default-view.png)

默认情况下，使用增强会话模式的虚拟机将共享剪贴板和打印机。  此外，它们还默认配置为将音频从虚拟机传递回计算机的扬声器。

如果要与虚拟机共享设备或者要更改这些默认设置：

1. 显示更多选项

  ![](media/esm-show-options.png)

1. 查看本地资源

  ![](media/esm-local-resources.png)

### <a name="share-storage-and-usb-devices"></a>共享存储和 USB 设备

默认情况下，使用增强会话模式的虚拟机将共享打印机、剪贴板，并将智能卡和其他安全设备转接到虚拟机，以便你可以在虚拟机上使用更多安全登录工具。

如果要共享其他设备，比如 USB 设备或 C: 驱动器，请选择“更多...”菜单：  
![](media/esm-more-devices.png)

在那里可以选择你想要同虚拟机共享的设备。  系统驱动器 (Windows C:) 对文件共享十分有用。  
![](media/esm-drives-usb.png)

### <a name="share-audio-devices-speakers-and-microphones"></a>共享音频设备（扬声器和麦克风）

默认情况下，使用增强会话模式的虚拟机可以传递音频，因此你可以在虚拟机上听到音频。  虚拟机将使用当前在主机上选择的音频设备。

如果要更改这些设置或者要添加麦克风传递（以便你可以在虚拟机上录制音频）：

选择“设置...”菜单以配置远程音频设置  
![](media/esm-audio.png)

现在来配置音频和麦克风设置  
![](media/esm-audio-settings.png)

由于你的虚拟机可能正在本地运行，“在此计算机上播放”和“在远程计算机上播放”选项将产生相同结果。

## <a name="re-launching-the-connection-settings"></a>重新启动连接设置

如果分辨率和设备共享对话框未出现，请尝试从 Windows 菜单或者以管理员身份从命令行单独启动 VMConnect。  

``` Powershell
vmconnect.exe
```

## <a name="check-session-type"></a>查看会话类型

你可以使用虚拟机连接工具 (VMConnect) 顶部的增强会话模式图标来查看连接的类型。  你还可以通过此按钮在基本会话和增强会话模式之间进行切换。

![](media/esm-button-location.png)

| 图标 | 连接状态 |
|:-----|:---------|
|![](media/esm-basic.png)| 你当前正以增强会话模式运行。  单击此图标将以基本模式重新连接到虚拟机。 |
|![](media/esm-connect.png)| 你当前正以基本会话模式运行，但增强会话模式现在可用。  单击此图标将以增强会话模式重新连接到虚拟机。  |
|![](media/esm-stop.png)| 你当前正以基本模式运行。  增强会话模式不适用于此虚拟机。 |