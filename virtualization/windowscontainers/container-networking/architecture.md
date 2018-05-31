---
title: Windows 容器网络
description: Windows 容器网络体系结构简单介绍。
keywords: docker, 容器
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: 0ade6677a8cd07f21cd00d019f167685e0ba5e7e
ms.sourcegitcommit: ec186664e76d413d3bf75f2056d5acb556f4205d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/11/2018
ms.locfileid: "1876019"
---
# <a name="windows-container-networking"></a>Windows 容器网络
> ***免责声明：请参考 [Docker 容器网络](https://docs.docker.com/engine/userguide/networking/)，以了解一般 docker 网络命令、选项和语法。*** 除了[下面](#unsupported-features-and-network-options)描述的任何情况之外，所有 Docker 网络命令在 Windows 上都受支持，并且所用语法与 Linux 上的语法相同。 但请注意，Windows 和 Linux 网络堆栈不同，因此你会发现某些 Linux 网络命令（如 ifconfig）在 Windows 上不受支持。


## <a name="basic-networking-architecture"></a>基本网络体系结构
本主题概述 Docker 如何在 Windows 上创建和管理主机网络。 对于网络，Windows 容器的作用类似于虚拟机。 每个容器都有一个连接到 Hyper-V 虚拟交换机 (vSwitch) 的虚拟网络适配器 (vNIC)。 Windows 支持可通过 Docker 创建的以下五个不同的[网络驱动程序或模式](./network-drivers-topologies.md)：*nat*、*overlay*、*transparent*、*l2bridge* 和 *l2tunnel*。 根据物理网络基础结构和单 VS 多主机网络要求，应该选择最符合你需要的网络驱动程序。


![文本](media/windowsnetworkstack-simple.png)


首次运行 docker 引擎时，它将创建默认的 NAT 网络“nat”，该网络使用内部 vSwitch 和一个名为 `WinNAT` 的 Windows 组件。 如果主机上有通过 PowerShell 或 Hyper-V 管理器创建的任何预先存在的外部 vSwitch，则也可以使用 *transparent* 网络驱动程序将它们提供给 Docker，并且在你运行 ``docker network ls`` 命令时可以看到它们。  


![文本](media/docker-network-ls.png)


> - ***内部*** vSwitch 是未直接连接到容器主机上的网络适配器的 vSwitch 

> - ***外部*** vSwitch 是_已_直接连接到容器主机上的网络适配器的 vSwitch  


![文本](media/get-vmswitch.png)


“Nat”网络是在 Windows 上运行的容器的默认网络。 任何在 Windows 上运行但未使用任何标志或参数来实现特定网络配置的容器都将连接到默认的“nat”网络，并且将被自动分配一个“nat”网络内部前缀 IP 范围内的 IP 地址。 用于“nat”的默认内部 IP 前缀是 172.16.0.0/16。 


## <a name="container-network-management-with-host-network-service"></a>容器网络管理与主机网络服务

主机网络服务 (HNS) 和主机计算服务 (HCS) 共同创建容器并将终结点连接到网络。

### <a name="network-creation"></a>网络的创建
  - HNS 为每个网络创建一个 Hyper-V 虚拟交换机
  - HNS 根据需要创建 NAT 和 IP 池

### <a name="endpoint-creation"></a>终结点创建
  - HNS 为每个容器终结点创建网络命名空间
  - HNS/HCS 将 v(m)NIC 放入网络命名空间内
  - HNS 创建 (vSwitch) 端口
  - HNS 将 IP 地址、DNS 信息、路由等（受制于网络模式）分配给终结点

### <a name="policy-creation"></a>策略创建
  - 默认 NAT 网络：HNS 使用相应的 Windows 防火墙允许规则创建 WinNAT 端口转移规则/映射
  - 所有其他网络：HNS 利用虚拟筛选平台 (VFP) 来创建策略
    - 这包括：负载均衡、ACL、封装等。
    - 请查找我们**不久将发布**的 HNS API 和架构。


![文本](media/HNS-Management-Stack.png)


 ## <a name="unsupported-features-and-network-options"></a>不受支持的功能和网络选项
 Windows 当前**不**支持以下网络选项：

 | 命令        | 不受支持的选项   |
 | ---------------|:--------------------:|
 | ``docker run``|   ``--ip6``, ``--dns-option`` |
 | ``docker network create``| ``--aux-address``, ``--internal``, ``--ip-range``, ``--ipam-driver``, ``--ipam-opt``, ``--ipv6``, ``--opt encrypted`` |