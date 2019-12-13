---
title: Windows 上容器中的设备
description: 适用于 Windows 上的容器的设备支持
keywords: docker，容器，设备，硬件
author: cwilhit
ms.openlocfilehash: 1ad63c158a42f116882c949b242274dde8d893fc
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910597"
---
# <a name="devices-in-containers-on-windows"></a>Windows 上容器中的设备

默认情况下，为 Windows 容器提供对主机设备的最小访问权限，就像 Linux 容器一样。 某些工作负荷有好处，甚至是强制性的，可以访问主机硬件设备并与之进行通信。 本指南介绍容器中支持的设备以及入门方式。

## <a name="requirements"></a>要求

要使此功能正常工作，你的环境必须满足以下要求：
- 容器主机必须运行 Windows Server 2019 或 Windows 10，版本1809或更高版本。
- 容器基本映像版本必须是1809或更高版本。
- 容器必须是在进程隔离模式下运行的 Windows 容器。
- 容器主机必须运行 Docker 引擎19.03 或更高版本。

## <a name="run-a-container-with-a-device"></a>使用设备运行容器

若要使用设备启动容器，请使用以下命令：

```shell
docker run --isolation=process --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

必须将 `{interface class guid}` 替换为适当的[设备接口类 GUID](https://docs.microsoft.com/windows-hardware/drivers/install/overview-of-device-interface-classes)，可在以下部分中找到该 GUID。

若要启动具有多个设备的容器，请使用以下命令并将多个 `--device` 参数一起使用：

```shell
docker run --isolation=process --device="class/{interface class GUID}" --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

在 Windows 中，所有设备都声明其实现的接口类的列表。 通过将此命令传递到 Docker，可以确保将标识为实现请求的类的所有设备都查明到容器中。

这意味着你**不**会从主机中分配设备。 而是将该主机与容器共享。 同样，因为您指定的是类 GUID，所以，实现该 GUID 的_所有_设备将与该容器共享。

## <a name="what-devices-are-supported"></a>支持哪些设备

目前支持以下设备（及其设备接口类 Guid）：
  
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
> 设备支持依赖于驱动程序。 尝试传递上表中未定义的类 Guid 可能会导致未定义的行为。

## <a name="hyper-v-isolated-windows-container-support"></a>Hyper-v 隔离的 Windows 容器支持

Hyper-v 中的工作负荷的设备分配和设备共享目前不支持 Windows 容器中的工作负荷。

## <a name="hyper-v-isolated-linux-container-support"></a>Hyper-v 隔离 Linux 容器支持

Hyper-v 中的工作负荷的设备分配和设备共享目前不支持隔离 Linux 容器。
