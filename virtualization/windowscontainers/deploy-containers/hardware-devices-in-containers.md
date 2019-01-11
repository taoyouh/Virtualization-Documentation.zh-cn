---
title: 在 Windows 上的容器中的设备
description: 在 Windows 上的容器存在哪些设备支持
keywords: docker，容器，设备硬件
author: cwilhit
ms.openlocfilehash: f70388bf3724af7cb92f20e2053aa4ddb1f953a3
ms.sourcegitcommit: 5cbaef0806db21d7bbcc99964837f10f4207a51f
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/11/2019
ms.locfileid: "9001749"
---
# <a name="devices-in-containers-on-windows"></a>在 Windows 上的容器中的设备

默认情况下，Windows 容器提供最少访问主机设备--就像 Linux 容器。 有某些工作负载很有用-或甚至是强制性-访问和与主机硬件设备进行通信。 本指南介绍在容器中支持哪些设备以及如何开始。

## <a name="requirements"></a>要求

- 你必须运行 Windows Server 2019 或更高版本或 Windows 10 专业版/企业版与 2018 年 10 月更新
- 你必须运行 Docker 版本 18.09 或更高版本。
- 在容器映像版本必须为 1809年或更高版本。
- 容器都必须在进程隔离模式下运行的 Windows 容器。

## <a name="run-a-container-with-a-device"></a>与设备运行的容器

若要与设备启动容器，使用以下命令：

```shell
docker run --isolation=process --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

你必须替换`{interface class guid}`与相应的[设备接口类 GUID](https://docs.microsoft.com/en-us/windows-hardware/drivers/install/overview-of-device-interface-classes)，可找到下面部分中。

若要使用多个设备启动容器，使用以下命令并组合在一起多个`--device`参数：

```shell
docker run --isolation=process --device="class/{interface class GUID}" --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

在 Windows 中，所有设备都声明它们实现的接口类的列表。 通过将此命令传递给 Docker，它将确保所有设备标识为实现请求的类将进行都检测到容器。

这意味着你**不**会指定设备远离主机。 相反，主机将它与共享容器。 同样，因为你要指定一个类 GUID，将与容器共享实现该 GUID 的_所有_设备。

## <a name="what-devices-are-supported"></a>什么支持设备

以下设备 （和接口类 Guid 其设备） 目前支持：
  
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
</tbody>
</table>

> [!TIP]
> 上面列出的设备是现在支持在 Windows 容器中的_仅_设备。 尝试通过任何其他类 Guid 将导致无法启动容器。

## <a name="hyper-v-container-device-support"></a>HYPER-V 容器设备支持

设备分配和设备共享中不支持 HYPER-V 隔离容器今天。

## <a name="linux-containers-on-windows-lcow-device-support"></a>在 Windows (LCOW) 设备支持的 Linux 容器

设备分配和设备共享中不支持 LCOW 今天。