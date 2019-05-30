---
title: 将嵌套的 Vm 配置为直接与 Azure 虚拟网络中的资源进行通信
description: 嵌套虚拟化
keywords: windows 10、hyper-v、Azure
author: mrajess
ms.date: 12/10/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1ecb85a6-d938-4c30-a29b-d18bd007ba08
ms.openlocfilehash: 2f1c6a124ba4f2f9d199d3cc5bb38c9082f72b3d
ms.sourcegitcommit: a7f9ab96be359afb37783bbff873713770b93758
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/28/2019
ms.locfileid: "9681137"
---
# <a name="configure-nested-vms-to-communicate-with-resources-in-an-azure-virtual-network"></a>将嵌套的 Vm 配置为与 Azure 虚拟网络中的资源进行通信

有关在 Azure 中部署和配置嵌套虚拟机的原始指南必须通过 NAT 交换机访问这些虚拟机。 这带来了一些限制:

1. 嵌套虚拟机无法访问本地资源或在 Azure 虚拟网络内访问资源。
2. Azure 中的本地资源或资源只能通过 NAT 访问嵌套的 Vm, 这意味着多个来宾无法共享同一个端口。

本文档将指导你使用 RRAS、用户定义的路由、专用于出站 NAT 以允许来宾 internet 访问的子网以及 "浮动" 地址空间, 以便允许嵌套的 Vm 与任何其他虚拟机的行为和通信已直接部署到 Azure 中的 VNet。

开始本指南之前, 请:

1. 阅读[此处提供的有关](https://docs.microsoft.com/azure/virtual-machines/windows/nested-virtualization)嵌套虚拟化的指南。
2. 在实现之前, 请先阅读本文的完整文章。

## <a name="high-level-overview-of-what-were-doing-and-why"></a>高级别概述我们正在做的事情以及原因
* 我们将创建一个具有两个 Nic 的支持嵌套的 VM。 
* 一个 NIC 将用于通过 NAT 向我们的嵌套 Vm 提供 internet 访问, 而另一个 NIC 将用于将来自内部交换机外部的资源的流量路由到虚拟机监控程序外部的资源。 每个 NIC 都需要位于不同的路由域中, 这意味着不同的子网。
* 这意味着我们将需要至少有三个子网的虚拟网络。 一个用于 NAT, 一个用于 LAN 路由, 另一个用于 LAN 路由, 另一个用于 "保留" 以供嵌套的 Vm 使用。 我们在此文档中使用子网的名称为 "NAT"、"Hyper-v-LAN" 和 "幻像"。
* 这些子网的大小由你决定, 但有一些注意事项。 "幻影" 子网的大小决定了你对嵌套 Vm 有多少个 Ip。 此外, "NAT" 和 "Hyper-v-LAN" 子网的大小决定了虚拟机监控程序有多少个 Ip。 因此, 如果你仅计划拥有一个或两个虚拟机监控程序, 则从技术上讲, 你可以在此处创建真正的小型子网
* 背景: 即使配置了内部或外部交换机, 嵌套的 Vm 也不会从其主机连接到的 VNet 接收 DHCP。 
  * 这意味着 Hyper-v 主机必须提供 DHCP。
* Hyper-v 主机不知道 VNet 中当前分配的租约, 因此为了避免主机分配已存在的 IP 的情况, 我们必须分配一个由 Hyper-v 主机仅使用的 IP 块的主机。 这将允许我们避免使用重复的 IP 方案。
  * 我们选择的 IPs 块将对应于您的 Hyper-v 所在的同一 VNet 中的子网。
  * 我们希望此对应于现有子网的原因是通过 ExpressRoute 处理 BGP 播发。 如果我们刚刚为 Hyper-v 主机创建了一个 IP 范围, 那么我们必须创建一系列静态路由以允许客户本地与嵌套的 Vm 通信。 这意味着, 这不是硬要求, 因为你可以为嵌套的 Vm 建立 IP 范围, 然后创建将客户端定向到该范围的 Hyper-v 主机所需的所有路由。
* 我们将在 Hyper-v 中创建一个内部开关, 然后我们会将新创建的接口分配给我们为 DHCP 设置的范围内的 IP 地址。 此 IP 地址将成为我们的嵌套 Vm 的默认网关, 并用于在内部交换机和连接到 VNet 的主机的 NIC 之间路由。
* 我们将在主机上安装路由和远程访问角色, 这将把主机转化为路由器。  这是允许主机外部资源与我们的嵌套 Vm 之间的通信所必需的。
* 我们将告诉其他资源如何访问这些嵌套的 Vm。 这需要创建一个用户定义的路由表, 该表包含嵌套的 Vm 所在的 IP 范围的静态路由。 此静态路由将指向 Hyper-v 的 IP 地址。
* 然后, 你将在网关子网上放置此 UDR, 以使来自本地的客户知道如何访问我们的嵌套 Vm。
* 你还可以将此 UDR 放置在 Azure 中任何其他需要连接到嵌套虚拟机的子网上。
* 对于多个 Hyper-v 主机, 你可以创建其他 "浮动" 子网, 并向 UDR 添加其他静态路由。
* 当您停止使用 Hyper-v 主机时, 您将删除/重新使用我们的 "浮动" 子网并从我们的 UDR 中删除该静态路由, 或者如果这是最后一个 Hyper-v 主机, 请完全删除 UDR。

## <a name="creating-the-host"></a>创建主机

我将通过任何高达个人首选项的配置值 (如 VM 名称、资源组等) 进行光泽。

1. 导航到 portal.azure.com
2. 单击左上方的 "创建资源"
3. 从 "常用" 列中选择 "Window Server 2016 VM"
4. 在 "基础" 选项卡上, 确保选择支持嵌套虚拟化的 VM 大小
5. 移动到 "网络" 选项卡
6. 使用以下配置创建新的虚拟网络
    * VNet 地址空间: 10.0.0.0/22
    * 子网1
        * 名称: NAT
        * 地址空间: 10.0.0.0/24
    * 子网2
        * 名称: Hyper-v-LAN
        * 地址空间: 10.0.1.0/24
    * 子网3
        * 名称: 幻像
        * 地址空间: 10.0.2.0/24
    * 子网4
        * 名称: Azure-Vm
        * 地址空间: 10.0.3.0/24
7. 确保已为 VM 选择了 NAT 子网
8. 转到 "审阅 + 创建", 然后选择 "创建"

## <a name="create-the-second-network-interface"></a>创建第二个网络接口
1. 在 VM 完成预配后, 在 Azure 门户中浏览到它
2. 停止 VM
3. 停止后, 请转到 "设置" 下的 "网络"
4. "附加网络接口"
5. "创建网络接口"
6. 为其命名 (不考虑命名内容), 但一定要记住它。
7. 为子网选择 "Hyper-v-LAN"
8. 确保选择主机所在的同一资源组
9. 造成
10. 这将返回到上一个屏幕, 请确保选择新创建的网络接口, 然后选择 "确定"
11. 返回到 "概述" 窗格, 并在上一个操作完成后再次启动 VM
12. 导航到刚才创建的第二个 NIC, 可以在之前选择的资源组中找到它
13. 转到 "IP 配置" 并将 "IP 转发" 切换到 "已启用", 然后保存更改

## <a name="setting-up-hyper-v"></a>设置 Hyper-v
1. 远程进入您的主机
2. 打开提升的 PowerShell 提示
3. 运行以下命令 `Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart`
4. 这将重新启动主机
5. 重新连接到主机以继续其余设置

## <a name="creating-our-virtual-switch"></a>创建虚拟交换机

1. 在 "管理模式" 下打开 PowerShell。
2. 创建内部开关: `New-VMSwitch -Name "NestedSwitch" -SwitchType Internal`
3. 将新创建的接口分配给 IP: `New-NetIPAddress –IPAddress 10.0.2.1 -PrefixLength 24 -InterfaceAlias "vEthernet (NestedSwitch)"`

## <a name="install-and-configure-dhcp"></a>安装和配置 DHCP

*许多人首次尝试获取嵌套的虚拟化时, 会错过此组件。 与你的来宾 Vm 将从你的主机所在的网络接收 DHCP 的内部部署不同, Azure 中的嵌套 Vm 必须通过其运行所在的主机提供 DHCP。 或者, 你需要将 IP 地址静态分配给每个嵌套 VM, 这是不可伸缩的。*

1. 安装 DHCP 角色: `Install-WindowsFeature DHCP -IncludeManagementTools`
2. 创建 DHCP 作用域: `Add-DhcpServerV4Scope -Name "Nested VMs" -StartRange 10.0.2.2 -EndRange 10.0.2.254 -SubnetMask 255.255.255.0`
3. 为作用域配置 DNS 和默认网关选项: `Set-DhcpServerV4OptionValue -DnsServer 168.63.129.16 -Router 10.0.2.1`
    * 如果希望名称解析正常工作, 请确保输入有效的 DNS 服务器。 在这种情况下, 我使用的是[Azure 的递归 DNS](https://docs.microsoft.com/azure/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances)。

## <a name="installing-remote-access"></a>安装远程访问

1. 打开服务器管理器, 然后选择 "添加角色和功能"。
2. 选择 "下一步", 直到到达 "服务器角色"。
3. 选中 "远程访问", 然后单击 "下一步", 直到到达 "角色服务"。
4. 选中 "路由", 选择 "添加功能", 然后选择 "下一步", 然后选择 "安装"。 完成向导并等待安装完成。

## <a name="configuring-remote-access"></a>配置远程访问

1. 打开服务器管理器, 选择 "工具", 然后选择 "路由和远程访问"。
2. 在 "路由和远程访问管理" 面板的左侧, 你将看到一个带有你的 "服务器名称" 的图标, 右键单击此项并选择 "配置并启用路由和远程访问"。
3. 在向导中选择 "下一步", 检查 "自定义配置" 的 "径向" 按钮, 然后选择 "下一步"。
4. 选中 "NAT" 和 "局域网路由", 然后选择 "下一步", 然后选择 "完成"。 如果它要求您启动服务, 请执行此操作。
5. 现在导航到 "IPv4" 节点并展开它, 以使 "NAT" 节点可用。
6. 右键单击 "NAT", 选择 "新建接口 ..."然后选择 "以太网", 这应该是 IP 为 "10.0.0.4" 的第一个 NIC
7. 现在, 我们需要创建一些静态路由来强制从第二个 NIC 出 LAN 流量。 通过转到 "IPv4" 下的 "静态路由" 节点来执行此操作。
8. 我们将创建以下路由。
    * 路由1
        * 接口: 以太网
        * 目标: 10.0.0。0
        * 网络掩码: 255.255.255。0
        * 网关: 10.0.0。1
        * 公制: 256
        * 注意: 我们将此处放入此处, 让主要 NIC 能够响应其自身接口的通信量。 如果我们在此处没有此内容, 以下路由将导致使用 NIC 1 的通信流出 NIC 2。 这将创建一个非对称路由。 10.0.0.1 是 Azure 分配给 NAT 子网的 IP 地址。 Azure 使用范围中的第一个可用 IP 作为默认网关。 因此, 如果你对 NAT 子网使用了 192.168.0.0/24, 网关将为192.168.0.1。 在路由更具体的路由是 wins 的, 这种路由将取代以下路由。

    * 路由2
        * 接口: 以太网2
        * 目标: 10.0.0。0
        * 网络掩码: 255.255.252。0
        * 网关: 10.0.1。1
        * 公制: 256
        * 注意: 这是适用于我们的 Azure VNet 的流量的 "捕获所有路由"。 它将强制从第二个 NIC 出流量。 你需要为你希望嵌套的 Vm 访问的其他区域添加其他路由。 因此, 如果你是本地网络是 172.16.0.0/22, 则你希望使用另一个路由将该流量发送到我们的虚拟机监控程序的第二个 NIC。

## <a name="creating-a-route-table-within-azure"></a>在 Azure 中创建路由表

有关在 Azure 中创建和管理路由的详细信息, 请参阅[本文](https://docs.microsoft.com/azure/virtual-network/tutorial-create-route-table-portal)。

1. 导航到https://portal.azure.com。
2. 在左上角选择 "创建资源"。
3. 在搜索字段中键入 "路由表", 然后按 enter。
4. 顶部结果将为 "路由表", 选择此列表, 然后选择 "创建"
5. 对路由表进行命名, 在我的事例中, 我将其命名为 "嵌套的-Vm"。
6. 请确保选择您的 Hyper-v 主机所驻留的同一订阅。
7. 创建新的资源组或选择现有的资源组, 并确保你在中创建路由表的区域与你的 Hyper-v 主机所在的区域相同。
8. 选择 "创建"。

## <a name="configuring-the-route-table"></a>配置路由表

1. 导航到刚刚创建的路由表。 你可以通过从门户顶部中心的搜索栏中搜索路由表的名称来执行此操作。
2. 选择路由表后, 请转到刀片式服务器内的 "路由"。
3. 选择 "添加"。
4. 为您的路线指定一个名称, 我使用的是 "嵌套-Vm"。
5. 对于地址前缀, 输入我们的 "浮动" 子网的 IP 范围。 在这种情况下, 它将是 10.0.2.0/24。
6. 对于 "下一跃点类型", 请选择 "虚拟设备", 然后输入 Hyper-v 主机的 IP 地址, 第二个 NIC 是 10.0.1.4, 然后选择 "确定"。
7. 现在从刀片式服务器内选择 "子网", 这将直接位于 "路由" 下方。
8. 选择 "关联", 然后选择 "嵌套趣味" VNet, 然后选择 "Azure-Vm" 子网, 然后选择 "确定"。
9. 对我们的 Hyper-v 主机所在的子网以及需要访问嵌套虚拟机的任何其他子网执行此相同的操作。 如果已连接 

# <a name="end-state-configuration-reference"></a>结束状态配置参考
本指南中的环境具有以下配置。 本部分 inteded 用作参考。

1. Azure 虚拟网络信息。
    * VNet 高级别配置。
        * 名称: 嵌套趣味
        * 地址空间: 10.0.0.0/22
        * 注意: 这将由四个子网组成。 此外, 这些范围不会在石头中设置。 您可以随时随意解决您的环境。 

    * 第一级子网高级配置。
        * 名称: NAT
        * 地址空间: 10.0.0.0/24
        * 注意: 这是我们的 Hyper-v 主机主要 NIC 所在的位置。 这将用于处理嵌套的 Vm 的出站 NAT。 它将是您的嵌套 Vm 的 internet 网关。

    * 第二级子网高级配置。
        * 名称: Hyper-v-LAN
        * 地址空间: 10.0.1.0/24
        * 注意: 我们的 Hyper-v 主机将具有第二个 NIC, 该 NIC 将用于处理位于 Hyper-v 主机外部的嵌套 Vm 和非 internet 资源之间的路由。

    * 第三级子网高级配置。
        * 名称: 幻像
        * 地址空间: 10.0.2.0/24
        * 注意: 这将是一个 "浮动" 子网。 地址空间将由我们的嵌套 Vm 占用, 并存在于将路由播发重新处理到本地。 不会实际将任何 Vm 部署到此子网中。

    * 第四子网高级别配置。
        * 名称: Azure-Vm
        * 地址空间: 10.0.3.0/24
        * 注意: 包含 Azure Vm 的子网。

1. 我们的 Hyper-v 主机具有以下 NIC 配置。
    * 主要 NIC 
        * IP 地址: 10.0.0。4
        * 子网掩码: 255.255.255。0
        * 默认网关: 10.0.0。1
        * DNS: 已针对 DHCP 配置
        * 已启用 IP 转发: 否

    * 辅助 NIC
        * IP 地址: 10.0.1。4
        * 子网掩码: 255.255.255。0
        * 默认网关: 空
        * DNS: 已针对 DHCP 配置
        * 已启用 IP 转发: 是

    * Hyper-v 为内部虚拟交换机创建了 NIC
        * IP 地址: 10.0.2。1
        * 子网掩码: 255.255.255。0
        * 默认网关: 空

3. 我们的路由表将有一个规则。
    * 规则1
        * 名称: 嵌套的 Vm
        * 目标: 10.0.2.0/24
        * 下一跃点: 虚拟设备-10.0.1。4

## <a name="conclusion"></a>总结

现在, 你应该能够将虚拟机 (甚至32位 VM!) 部署到 Hyper-v 主机, 并且可以从本地和 Azure 内访问它。
