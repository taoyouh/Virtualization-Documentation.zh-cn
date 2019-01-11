---
title: 配置嵌套虚拟机直接与 Azure 虚拟网络中的资源进行通信
description: 嵌套虚拟化
keywords: windows 10 的 hyper-v Azure
author: mrajess
ms.date: 12/10/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1ecb85a6-d938-4c30-a29b-d18bd007ba08
ms.openlocfilehash: c39a2e5639d013a0aba15150117b7ab18ea32f97
ms.sourcegitcommit: 1aef193cf56dd0870139b5b8f901a8d9808ebdcd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/11/2019
ms.locfileid: "9001653"
---
# <a name="configure-nested-vms-to-communicate-with-resources-in-an-azure-virtual-network"></a>配置嵌套虚拟机与 Azure 虚拟网络中的资源进行通信

部署和配置 Azure 内的嵌套的虚拟机上的原始指南，必须通过 NAT 交换机访问这些虚拟机。 这样便提供了几个限制：

1. 嵌套的虚拟机无法访问本地资源的或在 Azure 虚拟网络内。
2. 在本地资源或在 Azure 中的资源只能访问嵌套的虚拟机通过一个 NAT，这意味着多个来宾不能共享同一个端口。

本文档将指导完成部署，借此，我们将使用 RRAS，某些用户定义的路由，以及"浮动"的地址空间，以允许嵌套的虚拟机的行为，并像直接向内 Azure VNet 部署的任何其他虚拟机进行通信。

在开始本指南中，请： 之前

1. 读取[指南提供了](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/nested-virtualization)嵌套虚拟化、 创建嵌套支持虚拟机，并安装这些虚拟机中的 HYPER-V 角色。 请勿继续通过设置 HYPER-V 角色。
2. 阅读本文整个之前实现。

本指南假定有关目标环境：

1. 我们都在操作[中心和分支拓扑](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/hybrid-networking/hub-spoke)中, 连接到 ExpressRoute 我们集线器。
1. 我们分支网络分配 10.0.0.0/23，为两个/24 子网雕刻的地址空间。
    * 10.0.0.0/24-我们 HYPER-V 主机所在的子网。
    * 10.0.1.0/24 – 这将是"浮动"的子网。 此地址空间可供应用我们嵌套的虚拟机和，用于处理返回到本地路由公布。
    * 分支 VNet 巧妙名为"分支"。

1. 我们 hub 网络 IP 范围不相关，但知道其名称是"中心"。
1. 我们 HYPER-V 被分配 10.0.0.4/24 的地址。
1. 我们在 10.0.0.10/24 有 DNS 服务器，这不是一项要求，但我们演练的假设。

## <a name="high-level-overview-of-what-were-doing-and-why"></a>我们的高级别概述执行和原因

* 背景： 嵌套的虚拟机不会从其主机连接到，即使你配置的内部或外部交换机 VNet 接收 DHCP。 
  * 这意味着在 HYPER-V 主机必须提供 DHCP。
* 我们将使用由 HYPER-V 主机只需分配的 Ip 的块。  在 HYPER-V 主机不知道 VNet 上的当前已分配租约，因此为了避免在其中宿主已分配 IP 存在的情况下我们必须为 HYPER-V 主机只需使用分配的 Ip 的块。 这将允许我们可以避免重复的 IP 方案。
  * 我们选择的 Ip 的块将对应于在 HYPER-V 主机驻留在 VNet 内的子网。
  * 我们希望它对应于现有的子网的原因是返回到 ExpressRoute 处理 BGP 广告。 如果我们只需组成要使用的 HYPER-V 主机的 IP 范围，则我们将需要创建一系列静态路由以使客户端在本地与嵌套的虚拟机进行通信。 这意味着，这不是硬性要求为你无法使嵌套的虚拟机的 IP 范围，然后创建指引客户端与该范围的 HYPER-V 主机所需的所有路由。
* 我们将创建内部交换机内 HYPER-V，然后我们会将新创建的接口我们留出的 DHCP 范围内的 IP 地址。 此 IP 地址将成为默认网关我们嵌套的虚拟机的且在用于内部交换机和的主机的连接到我们 VNet NIC 之间的路线。
* 我们将在会变为路由器我们主机的主机上安装的路由和远程访问的角色。  这是需要允许对主机外部的资源和我们嵌套的虚拟机之间的通信。
* 我们将告诉其他资源如何访问这些嵌套的虚拟机。 这需要我们创建一个用户定义的路由表包含针对嵌套的虚拟机驻留在 IP 范围的静态路由。 该静态路由将指向 HYPER-V 的 IP 地址。
* 然后会将此 UDR 放置在网关子网，以便客户端来自本地知道如何到达我们嵌套的虚拟机。
* 你还将此 UDR 放在需要连接到嵌套的虚拟机的 Azure 中的任何其他子网。
* 为多个 HYPER-V 主机您可以创建其他"浮动"子网，并向 UDR 添加额外的静态路由。
* 解除授权 HYPER-V 主机时你将删除/重新调整其用途我们"浮动"的子网和删除该静态路由从我们 UDR，或如果这是最后一个 HYPER-V 主机，完全删除 UDR。

## <a name="creating-our-virtual-switch"></a>创建我们虚拟交换机

1. 在管理模式下打开 PowerShell。
2. 创建内部交换机： `New-VMSwitch -Name "NestedSwitch" -SwitchType Internal`
3. 将新创建的接口分配 IP: `New-NetIPAddress –IPAddress 10.0.1.1 -PrefixLength 24 -InterfaceAlias "vEthernet (NestedSwitch)"`

## <a name="install-and-configure-dhcp"></a>安装和配置 DHCP

*当他们首次尝试获取嵌套虚拟化工作时，许多用户忽略此组件。 与不同本地其中你来宾虚拟机将从你的主机所在，网络接收 DHCP 在 Azure 中的嵌套虚拟机必须提供 DHCP 通过主机运行。 这也需要静态分配 IP 地址为每个嵌套的虚拟机，这种方法不可扩展。*

1. 安装 DHCP 角色： `Install-WindowsFeature DHCP -IncludeManagementTools`
2. 创建 DHCP 作用域： `Add-DhcpServerV4Scope -Name "Nested VMs" -StartRange 10.0.1.2 -EndRange 10.0.1.254 -SubnetMask 255.255.255.0`
3. 配置 DNS 和默认网关为作用域选项： `Set-DhcpServerV4OptionValue -DnsServer 10.0.0.10 -Router 10.0.1.1`
    * 请务必输入有效的 DNS 服务器。 在此情况下我恰好提供 Windows DNS 10.0.0.0/24 网络上有一台服务器。

## <a name="installing-remote-access"></a>安装远程访问

* 打开服务器管理器，然后选择"添加角色和功能"。
* 选择"下一步"，直到到达"服务器角色"。
* 检查"远程访问"并单击"下一步"，直到到达"角色服务"。
* 检查"路由"，选择"添加功能"，然后选择"下一步"，然后再"安装"。 完成向导，并等待，以完成安装。

## <a name="configuring-remote-access"></a>配置远程访问

* 打开服务器管理器并选择"工具"，然后选择"路由和远程访问"。
* 在路由和远程访问管理面板的右侧你将看到与你的服务器名称在它旁边的图标，右键单击此并选择"配置并启用路由和远程访问"。
* 在向导中，选择"下一步"、"自定义配置"，检查径向按钮，然后选择"下一步"。
* 检查"NAT""和 LAN 路由"，然后选择"下一步"，然后"完成"。 如果它询问你是否要启动该服务，则执行此操作。
* 现在导航到"IPv4"节点，并将其展开，以便"NAT"节点将可用。
* 右键单击"NAT"、 选择"...新接口"，然后你应该看到三个选项。 
* 这些选项的两个是 HYPER-V 虚拟交换机，你应该会看到"Ethernet"选项，选择此选项。 简而言之，选择直接连接到 Azure VNet 的接口。

## <a name="creating-a-route-table-within-azure"></a>创建 Azure 内的路由表

请参阅[本文](https://docs.microsoft.com/en-us/azure/virtual-network/tutorial-create-route-table-portal)有关深入阅读有关创建和管理 Azure 内的路由。

* 导航到https://portal.azure.com。
* 在左上角中选择"创建资源"。
* 在搜索字段中键入"路由表"，然后点击输入。
* 顶部的结果将是路由表，选择此项，然后依次选择"创建"
* 命名路由表，在本例中我其名为"路由-为-嵌套的虚拟机"。
* 请确保你选择在 HYPER-V 主机驻留在同一个订阅。
* 创建新的资源组或选择一个现有并确保你创建路由表中的区域是在 HYPER-V 主机驻留在同一个区域。
* 选择"创建"。

## <a name="configuring-the-route-table"></a>配置路由表

* 导航到刚创建的路由表。 你可以通过搜索门户的顶部中心在搜索栏的路由表的名称来执行此操作。
* 选择后路由表从转到"路线"边栏选项卡内。
* 选择"添加"。
* 为给定名称的路线，我与"嵌套的虚拟机"出现问题。
* 地址前缀我们"浮动"的子网的输入的 IP 范围。 在此情况下，它将 10.0.1.0/24。
* 对于"下一跃点类型"选择"虚拟设备"，然后输入可能 10.0.0.4，，然后选择"确定"的 HYPER-V 主机的 IP 地址。
* 现在从内边栏选项卡中选择"子网"，这将是"路线"的正下方。
* 选择"关联"，然后选择我们英寸 Hub"虚拟网络，然后选择"GatewaySubnet"，然后选择"确定"。
* 我们 HYPER-V 主机驻留在以及与需要访问嵌套的虚拟机的任何其他子网的子网中执行此相同的过程。

## <a name="conclusion"></a>总结

你现在应该能够将虚拟机 (甚至是 32 位 VM ！) 部署到 HYPER-V 主机并使其可以从本地和 Azure 内访问。
