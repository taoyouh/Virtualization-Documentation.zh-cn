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
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998534"
---
# <a name="network-isolation-and-security"></a>网络隔离和安全

## <a name="isolation-with-network-namespaces"></a>与网络命名空间隔离

每个容器终结点都放在其自己的__网络命名空间__中。 管理主机 vNIC 和主机网络堆栈都位于默认的网络命名空间中。 为了在同一主机上的容器之间强制执行网络隔离, 为每个 Windows Server 容器和容器创建了一个网络命名空间, 该容器是在安装了容器的网络适配器的 Hyper-v 隔离下运行的。 Windows Server 容器使用主机 vNIC 连接到虚拟交换机。 Hyper-v 隔离使用合成 VM NIC (未公开到实用工具 VM) 附加到虚拟交换机。

![文本](media/network-compartment-visual.png)

```powershell
Get-NetCompartment
```

## <a name="network-security"></a>网络安全

根据所使用的容器和网络驱动程序，系统会结合 Windows 防火墙和 [VFP](https://www.microsoft.com/research/project/azure-virtual-filtering-platform/) 来强制执行端口 ACL。

### <a name="windows-server-containers"></a>Windows Server 容器

这些容器使用 Windows 主机的防火墙（使用网络命名空间来阐明）以及 VFP

* 默认出站：全部允许
* 默认入站：允许所有（TCP、UDP、ICMP、IGMP）未经请求的网络流量
  * 拒绝不是来自这些协议的所有其他网络流量

  >[!NOTE]
  >在 Windows Server 版本1709和 Windows 10 秋季创意者更新之前, 默认入站规则为 "全部拒绝"。 运行这些较旧版本的用户可以创建带有 ( ``docker run -p``端口转发) 的入站允许规则。

### <a name="hyper-v-isolation"></a>Hyper-V 隔离

在 Hyper-v 隔离中运行的容器拥有自己的独立内核, 因此使用以下配置运行其自己的 Windows 防火墙实例:

* Windows 防火墙（在实用工具虚拟机中运行）和 VFP 中的默认“全部允许”

![文本](media/windows-firewall-containers.png)

### <a name="kubernetes-pods"></a>Kubernetes 箱

在[Kubernetes pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/)中, 首先创建终结点所连接的基础结构容器。 属于同一 pod 的容器 (包括基础结构和工作容器) 共享一个通用网络命名空间 (相同 IP 和端口空间)。

![文本](media/pod-network-compartment.png)

### <a name="customizing-default-port-acls"></a>自定义默认端口 ACL

如果想要修改默认的端口 Acl, 请先阅读我们的主机网络服务文档 (即将添加的链接)。 您需要在以下组件内更新策略:

>[!NOTE]
>对于透明和 NAT 模式中的 Hyper-v 隔离, 当前无法 reprogram 默认的端口 Acl。 这在表中用“X”来反映。

| 网络驱动程序 | Windows Server 容器 | Hyper-V 隔离  |
| -------------- |-------------------------- | ------------------- |
| Transparent | Windows 防火墙 | X |
| NAT | Windows 防火墙 | X |
| L2Bridge | 两者 | VFP |
| L2Tunnel | 两者 | VFP |
| Overlay  | 两者 | VFP |