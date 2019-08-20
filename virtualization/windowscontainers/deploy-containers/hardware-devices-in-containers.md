---
title: Windows 上的容器中的设备
description: Windows 上的容器存在哪些设备支持
keywords: docker、容器、设备、硬件
author: cwilhit
ms.openlocfilehash: 1ad63c158a42f116882c949b242274dde8d893fc
ms.sourcegitcommit: 2f8fd4b2e7113fbb7c323d89f3c72df5e1a4437e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/20/2019
ms.locfileid: "10045027"
---
# <a name="devices-in-containers-on-windows"></a>Windows 上的容器中的设备

默认情况下, Windows 容器具有对主机设备的最小访问权限, 就像 Linux 容器一样。 某些工作负荷有好处-甚至是强制性的, 可与主机硬件设备进行访问和通信。 本指南介绍容器中支持哪些设备以及如何开始使用。

## <a name="requirements"></a>要求

若要使用此功能, 你的环境必须满足以下要求:
- 容器主机必须运行 Windows Server 2019 或 Windows 10 版本1809或更高版本。
- 你的容器基映像版本必须为1809或更高版本。
- 你的容器必须是在进程隔离模式下运行的 Windows 容器。
- 容器主机必须运行 Docker 引擎19.03 或更高版本。

## <a name="run-a-container-with-a-device"></a>使用设备运行容器

若要启动带有设备的容器, 请使用以下命令:

```shell
docker run --isolation=process --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

必须将`{interface class guid}`具有相应[设备接口类 GUID 的设备](https://docs.microsoft.com/windows-hardware/drivers/install/overview-of-device-interface-classes)替换为以下部分中的可用。

若要启动具有多个设备的容器, 请将以下命令和字符串`--device`一起使用多个参数:

```shell
docker run --isolation=process --device="class/{interface class GUID}" --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

在 Windows 中, 所有设备都声明了它们所实现的接口类的列表。 通过将此命令传递到 Docker, 它将确保标识为实现所请求的类的所有设备都将被查明到容器中。

这意味着您**不**会将设备分配给主机。 而是由主机与容器共享。 同样, 由于你要指定类 GUID, 因此实现该 GUID 的_所有_设备都将与容器共享。

## <a name="what-devices-are-supported"></a>支持哪些设备

今天支持以下设备 (及其设备接口类 Guid):
  
<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>设备类型</center></th>
<th><center>接口类 GUID</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>GPIO</center></td>
<td><center>916EF1CB-8426-468D-A6F7-9AE8076881B3</center></td>
</tr>
<tr valign="top">
<td><center>I2C 总线</center></td>
<td><center>A11EE3C6-8421-4202-A3E7-B91FF90188E4</center></td>
</tr>
<tr valign="top">
<td><center>COM 端口</center></td>
<td><center>86E0D1E0-8089-11D0-9CE4-08003E301F73</center></td>
</tr>
<tr valign="top">
<td><center>SPI 总线</center></td>
<td><center>DCDE6AF9-6610-4285-828F-CAAF78C424CC</center></td>
</tr>
<tr valign="top">
<td><center>DirectX GPU 加速</center></td>
<td><center>请参阅<a href="https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/gpu-acceleration">GPU 加速</a>文档</center></td>
</tr>
</tbody>
</table>

> [!IMPORTANT]
> 设备支持依赖于驱动程序。 尝试传递未在上面的表中定义的类 Guid 可能会导致未定义的行为。

## <a name="hyper-v-isolated-windows-container-support"></a>Hyper-v-隔离的 Windows 容器支持

当前不支持在 Hyper-v 中对工作负荷进行设备分配和设备共享。

## <a name="hyper-v-isolated-linux-container-support"></a>Hyper-v-隔离的 Linux 容器支持

目前不支持设备分配和 Hyper-v 中的工作负荷共享-隔离的 Linux 容器。
