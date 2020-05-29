---
title: Windows 上的容器中的设备
description: 为 Windows 上的容器提供的设备支持
keywords: docker, 容器, 设备, 硬件
author: cwilhit
ms.openlocfilehash: 1ad63c158a42f116882c949b242274dde8d893fc
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910597"
---
# <a name="devices-in-containers-on-windows"></a>Windows 上的容器中的设备

默认情况下，Windows 容器被授予对主机设备的最小访问权限，就像 Linux 容器一样。 在某些工作负荷中，访问主机硬件设备并与之通信是有益的，甚至是必要的。 本指南介绍了容器中支持哪些设备以及如何开始使用。

## <a name="requirements"></a>要求

要使此功能生效，你的环境必须满足以下要求：
- 容器主机必须运行 Windows Server 2019 或 Windows 10 版本 1809 或更高版本。
- 容器基础映像版本必须是 1809 或更高版本。
- 容器必须是在进程隔离模式下运行的 Windows 容器。
- 容器主机必须运行 Docker 引擎 19.03 或更高版本。

## <a name="run-a-container-with-a-device"></a>运行包含设备的容器

若要启动包含设备的容器，请使用以下命令：

```shell
docker run --isolation=process --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

必须将 `{interface class guid}` 替换为合适的[设备接口类 GUID](https://docs.microsoft.com/windows-hardware/drivers/install/overview-of-device-interface-classes)，这可以在以下部分找到。

若要启动包含多个设备的容器，请使用以下命令并将多个 `--device` 参数串在一起：

```shell
docker run --isolation=process --device="class/{interface class GUID}" --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

在 Windows 中，所有设备都声明其实现的接口类的列表。 将此命令传递给 Docker 即可确保所有确定用于实现所请求类的设备都将插入到容器中。

这意味着**不**会将设备分配到主机之外， 而是让主机与容器共享设备。 同样，因为指定了类 GUID，所以，将与容器共享所有实现了该 GUID 的设备。

## <a name="what-devices-are-supported"></a>支持哪些设备

目前支持以下设备（及其设备接口类 GUID）：
  
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
<td><center>请参阅 <a href="https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/gpu-acceleration">GPU 加速</a>文档</center></td>
</tr>
</tbody>
</table>

> [!IMPORTANT]
> 设备支持依赖于驱动程序。 尝试传递上表中未定义的类 GUID 可能会导致未定义的行为。

## <a name="hyper-v-isolated-windows-container-support"></a>Hyper-V 隔离 Windows 容器支持

目前不支持为 Hyper-V 隔离 Windows 容器中的工作负荷进行设备分配和设备共享。

## <a name="hyper-v-isolated-linux-container-support"></a>Hyper-V 隔离 Linux 容器支持

目前不支持为 Hyper-V 隔离 Linux 容器中的工作负荷进行设备分配和设备共享。
