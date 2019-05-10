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
ms.openlocfilehash: b39ec17ac04995e8e1ce8795b5721df7a291e31c
ms.sourcegitcommit: 34d8b2ca5eebcbdb6958560b1f4250763bee5b48
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "9620815"
---
# <a name="network-isolation-and-security"></a>网络隔离和安全

## <a name="isolation-with-network-namespaces"></a>隔离与网络命名空间

每个容器终结点都放在其自己的__网络命名空间__中。 管理主机 vNIC 和主机网络堆栈都位于默认的网络命名空间中。 若要强制执行在同一个主机上的容器之间的网络隔离，为每个 Windows Server 容器创建网络命名空间和容器 HYPER-V 隔离容器的网络适配器安装到下运行。 Windows Server 容器使用主机 vNIC 连接到虚拟交换机。 HYPER-V 隔离使用合成 VM NIC （不公开到实用工具 VM） 连接到虚拟交换机。

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
  >在 Windows Server 版本 1709年和 Windows 10 Fall Creators Update 之前, 默认入站的规则已全部拒绝。 运行这些较旧版本的用户可以创建入站的允许规则``docker run -p``（端口转移）。

### <a name="hyper-v-isolation"></a>Hyper-V 隔离

在 HYPER-V 隔离运行的容器具有其自己的独立的内核，因此通过以下配置运行自己的 Windows 防火墙实例：

* Windows 防火墙（在实用工具虚拟机中运行）和 VFP 中的默认“全部允许”

![文本](media/windows-firewall-containers.png)

### <a name="kubernetes-pods"></a>Kubernetes pod

在[Kubernetes pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/)，终结点连接到首次创建基础结构容器。 属于相同 pod，包括基础结构和工作线程容器的容器共享一个公用网络命名空间 （相同的 IP 和端口空间）。

![文本](media/pod-network-compartment.png)

### <a name="customizing-default-port-acls"></a>自定义默认端口 ACL

如果你想要修改默认端口 Acl，请首先阅读我们主机网络服务的文档 （不久将添加的链接）。 你将需要更新以下组件内的策略：

>[!NOTE]
>为透明和 NAT 模式中的 HYPER-V 隔离，你当前不能重新编程默认端口 Acl。 这在表中用“X”来反映。

| 网络驱动程序 | Windows Server 容器 | Hyper-V 隔离  |
| -------------- |-------------------------- | ------------------- |
| Transparent | Windows 防火墙 | X |
| NAT | Windows 防火墙 | X |
| L2Bridge | 两者 | VFP |
| L2Tunnel | 两者 | VFP |
| Overlay  | 两者 | VFP |