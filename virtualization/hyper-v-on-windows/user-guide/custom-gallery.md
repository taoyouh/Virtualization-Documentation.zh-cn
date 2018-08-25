---
title: 创建自定义虚拟机库
description: 在 Windows 10 创意者更新和更高版本中将自己的条目构建到虚拟机库中。
keywords: windows 10, hyper-v, 快速创意, 虚拟机, 库
ms.author: scooley
ms.date: 05/04/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: d9238389-7028-4015-8140-27253b156f37
ms.openlocfilehash: c7a6462b331f469148eb4cf5a0a2740c9929fa29
ms.sourcegitcommit: 2b5d806fc978e60fb71ce33ef491d4cfd6fc4456
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/03/2018
ms.locfileid: "2596065"
---
# <a name="create-a-custom-virtual-machine-gallery"></a>创建自定义虚拟机库

> Windows 10 Fall Creators Update 和更高版本。

在 Fall Creators Update 中，“快速创建”为包括虚拟机库而进行了扩展。

![包含自定义映像的快速创建虚拟机库](media/vmgallery.png)

虽然 Microsoft 和 Microsoft 合作伙伴提供了一组映像，但该库也会列出你自己的映像。

本文详细介绍：

* 构建与库兼容的虚拟机。
* 创建新库源。
* 将自定义库源添加到库中。

## <a name="gallery-architecture"></a>库体系结构

虚拟机库是 Windows 注册表中定义的一组虚拟机源的图形视图。  每个虚拟机源都是一个 JSON 文件路径（本地路径或 URI），并且以虚拟机作为列表项。

你在库中看到的虚拟机列表是第一个源的完整内容，接着是第二个源的内容，以此类推，直到列出所有可用的虚拟机为止。  每次启动库时都会动态创建列表。

![库体系结构](media/vmgallery-architecture.png)

注册表项： `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization`

值名称： `GalleryLocations`

类型： `REG_MULTI_SZ`

## <a name="create-gallery-compatible-virtual-machines"></a>创建与库兼容的虚拟机

库中的虚拟机可以是磁盘映像 (.iso) 或虚拟硬盘驱动器 (.vhdx)。

由虚拟硬盘驱动器制作的虚拟机具有一些配置要求：

1. 以支持 UEFI 固件为构建目的。 如果它们是使用 Hyper-V 创建的，那么这是第 2 代虚拟机。
1. 应该至少为虚拟硬盘分配 20GB 的空间 - 请记住，这是最大大小。  Hyper-V 将不会占用虚拟机当前未使用的空间。

### <a name="testing-a-new-vm-image"></a>测试新虚拟机映像

虚拟机库使用与从本地安装源安装相同的机制创建虚拟机。

若要验证虚拟机映像是否将启动并运行，请执行以下操作：

1. 打开虚拟机库（Hyper-V 快速创建），然后选择 **Local Installation Source**。
  ![用于使用本地安装源的按钮](media/use-local-source.png)
1. 选择 **Change Installation Source**。
  ![用于使用本地安装源的按钮](media/change-source.png)
1. 选择将在库中使用的 .iso 或 .vhdx。
1. 如果映像为 Linux 映像，请取消选择“安全启动”选项。
  ![用于使用本地安装源的按钮](media/toggle-secure-boot.png)
1. 创建虚拟机。  如果虚拟机正常启动，则可以入库。

## <a name="build-a-new-gallery-source"></a>构建新库源

下一步是创建新库源。  这是列出虚拟机并添加你在库中看到的所有额外信息的 JSON 文件。

文本信息：

![标记的库文本位置](media/gallery-text.png)

* **名称** - 必填 - 这是显示在左列和虚拟机视图顶部的名称。
* **发布者** - 必填
* **说明** - 必填 - 描述虚拟机的字符串列表。
* **版本** - 必填
* lastUpdated - 默认为 0001 年 1 月 1 日（星期一）。

  格式应为：yyyy-mm-ddThh:mm:ssZ

  以下 PowerShell 命令将以正确格式提供今天的日期并将其放在剪贴板上：

  ``` PowerShell
  Get-Date -UFormat "%Y-%m-%dT%TZ" | clip.exe
  ```

* 区域设置 - 默认为空白。

图片：

![标记的库图片位置](media/gallery-pictures.png)

* **徽标** - 必需
* 符号
* 缩略图

当然，还有你的虚拟机（.iso 或 .vhdx）。

若要生成的哈希值，可以使用以下 powershell 命令：

  ``` PowerShell
  Get-FileHash -Path .\TMLogo.jpg -Algorithm SHA256
  ```

以下 JSON 模板有简易版项目和库的架构。  如果你在 VSCode 中进行编辑，它将自动提供 IntelliSense 功能。

[!code-json[main](../../../hyperv-tools/vmgallery/vm-gallery-template.json)]

## <a name="connect-your-gallery-to-the-vm-gallery-ui"></a>将你的库连接到虚拟机库 UI

将自定义库源添加到虚拟机库的最简单方法是在 REGEDIT 中添加它。

1. 打开 **regedit.exe**
1. 导航到 `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\`
1. 查找 `GalleryLocations` 项。

    如果此项已经存在，请转到**编辑**菜单并进行**修改**。

    如果此项尚不存在，请转到**编辑**菜单，从**新建**浏览到**多字符串值**

1. 将你的库添加到 `GalleryLocations` 注册表项。

    ![包含新项的库注册表项](media/new-gallery-uri.png)

## <a name="troubleshooting"></a>疑难解答

### <a name="check-for-errors-loading-gallery"></a>检查库加载错误

虚拟机库在 Windows 事件查看器中会提供错误报告。  若要检查错误，请执行以下操作：

1. 打开事件查看器
1. 导航到 **Windows 日志** -> **应用程序**
1. 查找源 VMCreate 中的事件。

## <a name="resources"></a>资源

GitHub [链接](https://github.com/MicrosoftDocs/Virtualization-Documentation/tree/live/hyperv-tools/vmgallery)中提供一部分库脚本和帮助程序。

请在[此处](https://go.microsoft.com/fwlink/?linkid=851584)参阅示例库条目。  这是定义收件箱库的 JSON 文件。
