---
title: Windows 容器网络
description: Windows 容器网络体系结构简单介绍。
keywords: docker, 容器
author: jmesser81
ms.date: 03/27/2018
ms.topic: overview
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: 2235ae8b48828535facaa9b2be3dfc7fde450516
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192624"
---
# <a name="windows-container-networking"></a>Windows 容器网络

>[!IMPORTANT]
>请参阅[Docker Container 网络](https://docs.docker.com/engine/userguide/networking/)以获取一般的 docker 网络命令、选项和语法。 * * * 除了[不受支持的功能和网络选项](#unsupported-features-and-network-options)中所述的任何情况外，在 Windows 上支持的所有 Docker 网络命令都与 Linux 上的语法相同。 但是，Windows 和 Linux 网络堆栈有所不同，因此你会发现 Windows 不支持某些 Linux 网络命令（例如，ifconfig）。

## <a name="basic-networking-architecture"></a>基本网络体系结构

本主题概述 Docker 如何在 Windows 上创建和管理主机网络。 关于网络，Windows 容器的作用类似于虚拟机。 每个容器都有一个连接到 Hyper-V 虚拟交换机 (vSwitch) 的虚拟网络适配器 (vNIC)。 Windows 支持可通过 Docker 创建的以下五个不同的[网络驱动程序或模式](./network-drivers-topologies.md)：*nat*、*overlay*、*transparent*、*l2bridge* 和 *l2tunnel*。 根据物理网络基础结构和单 VS 多主机网络要求，应该选择最符合你需要的网络驱动程序。

![text](media/windowsnetworkstack-simple.png)

首次运行 docker 引擎时，它将创建默认的 NAT 网络“nat”，该网络使用内部 vSwitch 和一个名为 `WinNAT` 的 Windows 组件。 如果主机上有通过 PowerShell 或 Hyper-V 管理器创建的任何预先存在的外部 vSwitch，则也可以使用 *transparent* 网络驱动程序将它们提供给 Docker，并且在你运行 ``docker network ls`` 命令时可以看到它们。

![text](media/docker-network-ls.png)

- **内部**vSwitch 是指未直接连接到容器主机上的网络适配器的 vSwitch。
- **外部**vSwitch 是直接连接到容器主机上的网络适配器的一个。

![text](media/get-vmswitch.png)

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

### <a name="policy-creation"></a>创建策略

- 默认 NAT 网络：HNS 使用相应的 Windows 防火墙允许规则创建 WinNAT 端口转移规则/映射
- 所有其他网络：HNS 利用虚拟筛选平台 (VFP) 来创建策略
    - 这包括：负载均衡、ACL、封装等。
    - [在此处](https://docs.microsoft.com/windows-server/networking/technologies/hcn/hcn-top)查找你的 HNS api 和已发布的架构

![text](media/HNS-Management-Stack.png)

## <a name="unsupported-features-and-network-options"></a>不受支持的功能和网络选项

当前在 Windows 上**不**支持以下网络选项：

- 附加到 l2bridge、NAT 和覆盖网络的 Windows 容器不支持通过 IPv6 堆栈进行通信。
- 通过 IPsec 进行加密的容器通信。
- 容器的 HTTP 代理支持。
- [主机模式](https://docs.docker.com/ee/ucp/interlock/config/host-mode-networking/)网络
- 通过透明网络驱动程序在虚拟化的 Azure 基础结构上联网。

| Command        | 不支持的选项   |
|---------------|:--------------------:|
| ``docker run``|   ``--ip6``, ``--dns-option`` |
| ``docker network create``| ``--aux-address``, ``--internal``, ``--ip-range``, ``--ipam-driver``, ``--ipam-opt``, ``--ipv6``, ``--opt encrypted`` |
