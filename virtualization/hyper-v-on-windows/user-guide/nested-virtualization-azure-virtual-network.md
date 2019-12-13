---
title: 将嵌套式 Vm 配置为直接与 Azure 虚拟网络中的资源进行通信
description: 嵌套虚拟化
keywords: windows 10，hyper-v，Azure
author: mrajess
ms.date: 12/10/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1ecb85a6-d938-4c30-a29b-d18bd007ba08
ms.openlocfilehash: efd180c458457da1cea6b379e21ba3a37083d15a
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910957"
---
# <a name="configure-nested-vms-to-communicate-with-resources-in-an-azure-virtual-network"></a>配置嵌套 Vm 以便与 Azure 虚拟网络中的资源通信

在 Azure 中部署和配置嵌套虚拟机的原始指导要求你通过 NAT 交换机访问这些 Vm。 这会带来一些限制：

1. 嵌套式 Vm 无法访问本地资源或 Azure 虚拟网络中的资源。
2. Azure 中的本地资源或资源只能通过 NAT 访问嵌套 Vm，这意味着多个来宾不能共享相同的端口。

本文档将指导你完成部署，其中我们将使用 RRAS、用户定义的路由、专用于出站 NAT 的子网来允许来宾 internet 访问，并提供 "浮动" 地址空间，以允许嵌套式 Vm 的行为和与其他虚拟机一样进行通信直接部署到 Azure 中的 VNet。

开始本指南之前，请执行以下操作：

1. 阅读[此处提供](https://docs.microsoft.com/azure/virtual-machines/windows/nested-virtualization)的有关嵌套虚拟化的指导。
2. 实现之前，请阅读本文。

## <a name="high-level-overview-of-what-were-doing-and-why"></a>我们正在做的事情以及原因的简要概述
* 我们将创建一个具有两个 Nic 的支持嵌套的 VM。 
* 一个 NIC 将用于通过 NAT 通过 internet 访问来提供嵌套式 Vm，而另一个 NIC 用于将来自内部交换机的流量路由到虚拟机监控程序外部的资源。 每个 NIC 都需要位于不同的路由域中，这意味着不同的子网。
* 这意味着，我们需要一个至少包含三个子网的虚拟网络。 一个适用于 NAT，一个用于 LAN 路由，另一个用于使用，但对于嵌套式 Vm 为 "reserved"。 此文档中使用的子网名称为 "NAT"、"Hyper-v-LAN" 和 "幻像"。
* 这些子网的大小由你自行决定，但有一些注意事项。 "幻像" 子网的大小决定了对于嵌套 Vm 有多少个 Ip。 此外，"NAT" 和 "Hyper-v-LAN" 子网的大小决定了虚拟机监控程序的 Ip 数目。 如果只是计划有一个或两个虚拟机监控程序，则从技术上说，您可以从技术上创建真正的小型子网。
* 背景：嵌套式 Vm 即使配置了内部或外部交换机，也不会从其主机连接到的 VNet 接收 DHCP。 
  * 这意味着 Hyper-v 主机必须提供 DHCP。
* Hyper-v 主机不能识别 VNet 上当前分配的租约，因此，为了避免主机分配 IP 已经存在的情况下，我们必须分配一个 ip 块，以便仅供 Hyper-v 主机使用。 这将允许我们避免重复的 IP 方案。
  * 我们选择的 Ip 块将对应于 Hyper-v 所在的同一 VNet 中的子网。
  * 我们希望此对应于现有子网的原因是通过 ExpressRoute 来处理 BGP 播发。 如果我们只是为 Hyper-v 主机提供了一个 IP 范围来使用，则必须创建一系列静态路由，以允许本地的客户端与嵌套的 Vm 通信。 这就意味着，这并不是一项硬性要求，因为你可以为嵌套 Vm 建立 IP 范围，然后创建将客户端定向到该范围的 Hyper-v 主机所需的所有路由。
* 我们将在 Hyper-v 中创建内部交换机，然后，我们会将新创建的接口分配给我们为 DHCP 设置的范围内的 IP 地址。 此 IP 地址将成为嵌套 Vm 的默认网关，并用于在内部交换机与连接到 VNet 的主机的 NIC 之间进行路由。
* 我们将在主机上安装 "路由和远程访问" 角色，这会将主机转到路由器。  这是允许主机外部的资源与嵌套的 Vm 之间的通信所必需的。
* 我们会告诉其他资源如何访问这些嵌套式 Vm。 这就要求创建一个用户定义的路由表，该表包含嵌套 Vm 所在的 IP 范围的静态路由。 此静态路由将指向 Hyper-v 的 IP 地址。
* 然后，将此 UDR 置于网关子网，以便来自本地的客户端知道如何访问我们的嵌套 Vm。
* 还需要将此 UDR 放在 Azure 中需要连接到嵌套 Vm 的任何其他子网上。
* 对于多个 Hyper-v 主机，你要创建其他 "浮动" 子网，并向 UDR 添加其他静态路由。
* 当你解除 Hyper-v 主机的授权时，你将删除/重新调整我们的 "浮动" 子网，并从我们的 UDR 中删除该静态路由，如果这是最后一个 Hyper-v 主机，请完全删除该 UDR。

## <a name="creating-the-host"></a>创建主机

我将在任何由个人首选项（例如 VM 名称、资源组等）中的配置值中进行重新启动。

1. 导航到 portal.azure.com
2. 单击左上角的 "创建资源"
3. 从 "常用" 列中选择 "Window Server 2016 VM"
4. 在 "基本信息" 选项卡上，请务必选择能够进行嵌套虚拟化的 VM 大小
5. 转到 "网络" 选项卡
6. 使用以下配置创建新的虚拟网络
    * VNet 地址空间： 10.0.0.0/22
    * 子网 1
        * 名称： NAT
        * 地址空间： 10.0.0.0/24
    * 子网 2
        * 名称： Hyper-v-LAN
        * 地址空间： 10.0.1.0/24
    * 子网 3
        * 名称：幻像
        * 地址空间： 10.0.2.0/24
    * 子网4
        * 名称： Azure Vm
        * 地址空间： 10.0.3.0/24
7. 确保为 VM 选择了 NAT 子网
8. 请参阅 "查看 + 创建" 并选择 "创建"

## <a name="create-the-second-network-interface"></a>创建第二个网络接口
1. VM 完成预配后，请在 Azure 门户中浏览到该 VM
2. 停止 VM
3. 停止后，请跳到 "设置" 下的 "网络"
4. "附加网络接口"
5. "创建网络接口"
6. 为该名称指定一个名称（这并不重要，但请务必记住它）
7. 选择子网的 "Hyper-v-LAN"
8. 确保你选择的主机所在的资源组
9. 创建
10. 这会使你返回到上一个屏幕，请确保选择新创建的网络接口，然后选择 "确定"
11. 返回到 "概述" 窗格，并在上一操作完成后重新启动 VM
12. 导航到刚刚创建的第二个 NIC，可以在之前选择的资源组中找到它
13. 转到 "IP 配置"，将 "IP 转发" 切换到 "已启用"，然后保存更改

## <a name="setting-up-hyper-v"></a>设置 Hyper-v
1. 远程进入主机
2. 打开提升权限的 PowerShell 提示符
3. 运行以下命令 `Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart`
4. 这将重新启动主机
5. 重新连接到主机以继续执行其余的设置

## <a name="creating-our-virtual-switch"></a>创建我们的虚拟交换机

1. 在管理模式下打开 PowerShell。
2. 创建内部交换机： `New-VMSwitch -Name "NestedSwitch" -SwitchType Internal`
3. 为新创建的接口分配 IP： `New-NetIPAddress –IPAddress 10.0.2.1 -PrefixLength 24 -InterfaceAlias "vEthernet (NestedSwitch)"`

## <a name="install-and-configure-dhcp"></a>安装和配置 DHCP

*许多人在第一次尝试获取嵌套虚拟化工作时都错过了此组件。与你的来宾 Vm 将从你的主机所在的网络接收 DHCP 的本地不同，Azure 中的嵌套 Vm 必须通过其运行所在的主机提供。这或需要静态地将一个 IP 地址分配给每个嵌套 VM，这是不可缩放的。*

1. 安装 DHCP 角色： `Install-WindowsFeature DHCP -IncludeManagementTools`
2. 创建 DHCP 作用域： `Add-DhcpServerV4Scope -Name "Nested VMs" -StartRange 10.0.2.2 -EndRange 10.0.2.254 -SubnetMask 255.255.255.0`
3. 为作用域配置 DNS 和默认网关选项： `Set-DhcpServerV4OptionValue -DnsServer 168.63.129.16 -Router 10.0.2.1`
    * 如果希望名称解析正常工作，请务必输入有效的 DNS 服务器。 在此示例中，我使用[Azure 的递归 DNS](https://docs.microsoft.com/azure/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances)。

## <a name="installing-remote-access"></a>安装远程访问

1. 打开服务器管理器，然后选择 "添加角色和功能"。
2. 选择 "下一步"，直到到达 "服务器角色"。
3. 选中 "远程访问"，然后单击 "下一步"，直到进入 "角色服务"。
4. 选中 "路由"，选择 "添加功能"，然后选择 "下一步"，然后单击 "安装"。 完成向导并等待安装完成。

## <a name="configuring-remote-access"></a>配置远程访问

1. 打开服务器管理器，选择 "工具"，然后选择 "路由和远程访问"。
2. 在 "路由和远程访问管理" 面板的左侧，你将看到一个图标，其中包含你的服务器名称，右键单击此图标，然后选择 "配置并启用路由和远程访问"。
3. 在向导中，选择 "下一步"，检查 "自定义配置" 的 "放射" 按钮，然后选择 "下一步"。
4. 选中 "NAT" 和 "LAN 路由"，然后选择 "下一步" 和 "完成"。 如果它要求您启动服务，则执行此操作。
5. 现在，导航到 "IPv4" 节点并展开它，以便使 "NAT" 节点可用。
6. 右键单击 "NAT"，然后选择 "新建接口 ..."然后选择 "以太网"，这应该是 IP 为 "10.0.0.4" 的第一个 NIC
7. 现在，我们需要创建一些静态路由，强制 LAN 流量从第二个 NIC 传出。 要执行此操作，请转到 "IPv4" 下的 "静态路由" 节点。
8. 接下来，我们将创建以下路由。
    * 路由1
        * 接口：以太网
        * 目标：10.0.0。0
        * 网络掩码：255.255.255。0
        * 网关：10.0.0。1
        * 指标：256
        * 注意：我们将其放在此处，以允许主 NIC 响应目标为其自己的接口的流量。 如果此处未提供此项，以下路由将导致 NIC 1 的流量发送到 NIC 2。 这将创建一个非对称路由。 10.0.0.1 是 Azure 分配给 NAT 子网的 IP 地址。 Azure 使用范围中的第一个可用 IP 作为默认网关。 因此，如果你已将 192.168.0.0/24 用于 NAT 子网，则网关将为192.168.0.1。 在路由更具体的路由入选时，这意味着此路由将取代以下路由。

    * 路由2
        * 接口：以太网2
        * 目标：10.0.0。0
        * 网络掩码：255.255.252。0
        * 网关：10.0.1。1
        * 指标：256
        * 注意：这是适用于 Azure VNet 的流量的全部捕获路由。 它将强制执行第二个 NIC 的流量。 需要为要嵌套的 Vm 访问的其他范围添加其他路由。 因此，如果本地的网络是 172.16.0.0/22，则需要使用另一个路由将该流量发送到虚拟机监控程序的第二个 NIC。

## <a name="creating-a-route-table-within-azure"></a>在 Azure 中创建路由表

有关在 Azure 中创建和管理路由的详细信息，请参阅[此文](https://docs.microsoft.com/azure/virtual-network/tutorial-create-route-table-portal)。

1. 导航到 https://portal.azure.com 。
2. 在左上角选择 "创建资源"。
3. 在搜索字段中键入 "路由表"，然后按 enter。
4. 顶部结果将是路由表，选择此表，然后选择 "创建"
5. 命名路由表，在我的示例中，我将其命名为 "路由嵌套型 Vm"。
6. 请确保选择 Hyper-v 主机所在的同一订阅。
7. 创建新的资源组或选择现有的资源组，并确保在其中创建路由表的区域是 Hyper-v 主机所在的同一区域。
8. 选择“创建”。

## <a name="configuring-the-route-table"></a>配置路由表

1. 导航到刚刚创建的路由表。 为此，可以从门户顶部中心的搜索栏中搜索路由表的名称。
2. 选择路由表后，请从边栏选项卡中选择 "路由"。
3. 选择“添加”。
4. 为路由命名，并使用 "嵌套 Vm"。
5. 对于 "地址前缀"，请输入 "浮动" 子网的 IP 范围。 在这种情况下，它会是 10.0.2.0/24。
6. 对于 "下一跃点类型"，请选择 "虚拟设备"，然后输入 Hyper-v 主机的 IP 地址（10.0.1.4），然后选择 "确定"。
7. 现在，在边栏选项卡中选择 "子网"，这将直接位于 "路由" 下面。
8. 选择 "关联"，选择我们的 "嵌入乐趣" VNet，然后选择 "Azure Vm" 子网，然后选择 "确定"。
9. 对于 Hyper-v 主机所在的子网以及需要访问嵌套 Vm 的任何其他子网，请执行此过程。 如果已连接 

# <a name="end-state-configuration-reference"></a>结束状态配置参考
本指南中的环境具有以下配置。 本部分 inteded 要用作参考。

1. Azure 虚拟网络信息。
    * VNet 高级配置。
        * 名称：嵌套趣味
        * 地址空间： 10.0.0.0/22
        * 注意：这将由四个子网组成。 此外，这些范围并不是在石头中设置的。 但可以随意满足你的环境要求。 

    * 第一级子网高级配置。
        * 名称： NAT
        * 地址空间： 10.0.0.0/24
        * 注意：这是我们的 Hyper-v 主机主要 NIC 所在的位置。 这将用于处理嵌套 Vm 的出站 NAT。 它将成为嵌套 Vm 的 internet 网关。

    * 第二子网高级别配置。
        * 名称： Hyper-v-LAN
        * 地址空间： 10.0.1.0/24
        * 注意： Hyper-v 主机将使用第二个 NIC 来处理嵌套 Vm 与 Hyper-v 主机外部的非 internet 资源之间的路由。

    * 第三级子网高级配置。
        * 名称：幻像
        * 地址空间： 10.0.2.0/24
        * 注意：这将是一个 "浮动" 子网。 地址空间将由嵌套式 Vm 使用，并存在于将路由播发发回到本地。 实际上不会将 Vm 部署到此子网中。

    * 第四个子网高级别配置。
        * 名称： Azure Vm
        * 地址空间： 10.0.3.0/24
        * 注意：包含 Azure Vm 的子网。

1. Hyper-v 主机具有以下 NIC 配置。
    * 主 NIC 
        * IP 地址：10.0.0。4
        * 子网掩码：255.255.255.0
        * 默认网关：10.0.0。1
        * DNS：为 DHCP 配置
        * 已启用 IP 转发：否

    * 辅助 NIC
        * IP 地址：10.0.1。4
        * 子网掩码：255.255.255.0
        * 默认网关：空
        * DNS：为 DHCP 配置
        * 已启用 IP 转发：是

    * Hyper-v 为内部虚拟交换机创建了 NIC
        * IP 地址：10.0.2。1
        * 子网掩码：255.255.255.0
        * 默认网关：空

3. 路由表将有一个规则。
    * 规则 1
        * 名称：嵌套式 Vm
        * 目标： 10.0.2.0/24
        * 下一个跃点：虚拟设备-10.0.1。4

## <a name="conclusion"></a>结论

现在应能够将虚拟机（甚至是32位 VM！）部署到 Hyper-v 主机，并使其可以从本地和 Azure 中访问。
