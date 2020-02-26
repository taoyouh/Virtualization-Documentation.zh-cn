---
title: Windows 容器网络
description: Windows 容器内的网络隔离和安全。
keywords: docker, 容器
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: d5081104f1614a91d6441a5e879a439f1df1bf77
ms.sourcegitcommit: 16744984ede5ec94cd265b6bff20aee2f782ca88
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/18/2020
ms.locfileid: "77439284"
---
# <a name="network-isolation-and-security"></a>网络隔离和安全

## <a name="isolation-with-network-namespaces"></a>与网络命名空间隔离

每个容器终结点都放在其自己的__网络命名空间__中。 管理主机 vNIC 和主机网络堆栈都位于默认的网络命名空间中。 为了在同一主机上的容器之间强制实施网络隔离，将为每个 Windows Server 容器创建一个网络命名空间，并在 Hyper-v 隔离下运行容器的网络适配器。 Windows Server 容器使用主机 vNIC 连接到虚拟交换机。 Hyper-v 隔离使用合成 VM NIC （不公开给实用工具 VM）附加到虚拟交换机。

![text](media/network-compartment-visual.png)

```powershell
Get-NetCompartment
```

## <a name="network-security"></a>网络安全性

根据所使用的容器和网络驱动程序，系统会结合 Windows 防火墙和 [VFP](https://www.microsoft.com/research/project/azure-virtual-filtering-platform/) 来强制执行端口 ACL。

### <a name="windows-server-containers"></a>Windows Server 容器

这些容器使用 Windows 主机的防火墙（使用网络命名空间来阐明）以及 VFP

* 默认出站：全部允许
* 默认入站：允许所有（TCP、UDP、ICMP、IGMP）未经请求的网络流量
  * 拒绝不是来自这些协议的所有其他网络流量

  >[!NOTE]
  >在 Windows Server 版本1709和 Windows 10 秋季创建者更新之前，默认入站规则为 "全部拒绝"。 运行这些旧版本的用户可以创建包含 ``docker run -p`` （端口转发）的入站允许规则。

### <a name="hyper-v-isolation"></a>Hyper-V 隔离

在 Hyper-v 隔离中运行的容器具有其自己的独立内核，因此使用以下配置运行其自己的 Windows 防火墙实例：

* Windows 防火墙（在实用工具虚拟机中运行）和 VFP 中的默认“全部允许”

![text](media/windows-firewall-containers.png)

### <a name="kubernetes-pods"></a>Kubernetes pod

在[Kubernetes pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/)中，首先创建了一个终结点所附加到的基础结构容器。 属于同一 pod （包括基础结构和辅助角色容器）的容器共享公共网络命名空间（相同 IP 和端口空间）。

![text](media/pod-network-compartment.png)

### <a name="customizing-default-port-acls"></a>自定义默认端口 ACL

如果要修改默认端口 Acl，请首先阅读主机网络服务文档（即将添加的链接）。 需要在以下组件中更新策略：

>[!NOTE]
>对于透明模式和 NAT 模式下的 Hyper-v 隔离，当前无法重新编程默认端口 Acl。 这在表中用“X”来反映。

| 网络驱动程序 | Windows Server 容器 | Hyper-V 隔离  |
| -------------- |-------------------------- | ------------------- |
| 透明 | Windows 防火墙 | X |
| NAT | Windows 防火墙 | X |
| L2Bridge | 两者同时 | VFP |
| L2Tunnel | 两者同时 | VFP |
| 覆盖  | 两者同时 | VFP |