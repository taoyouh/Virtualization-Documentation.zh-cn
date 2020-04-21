---
title: 将嵌套 VM 配置为直接与 Azure 虚拟网络中的资源通信
description: 嵌套虚拟化
keywords: windows 10, hyper-v, Azure
author: mrajess
ms.date: 12/10/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1ecb85a6-d938-4c30-a29b-d18bd007ba08
ms.openlocfilehash: 63007d21fcc046f384405c7d85143bfc576ecc07
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/15/2020
ms.locfileid: "81395750"
---
# <a name="configure-nested-vms-to-communicate-with-resources-in-an-azure-virtual-network"></a>将嵌套 VM 配置为与 Azure 虚拟网络中的资源通信

有关如何在 Azure 中部署和配置嵌套虚拟机的原始指南要求通过 NAT 交换机访问这些 VM。 这会带来一些限制：

1. 嵌套 VM 无法访问本地资源或 Azure 虚拟网络中的资源。
2. 本地资源或 Azure 中的资源只能通过 NAT 访问嵌套 VM，这意味着多个来宾不能共享同一端口。

本文档将详述如何进行部署，以便通过 RRAS、用户定义的路由、专用于出站 NAT 的子网来启用来宾 Internet 访问，并提供“浮动”地址空间，使嵌套 VM 的行为和通信方式与任何其他在 Azure 中直接部署到 VNet 的虚拟机一样。

开始使用本指南之前，请完成以下任务：

1. 阅读[此处提供的指南](https://docs.microsoft.com/azure/virtual-machines/windows/nested-virtualization)，了解嵌套虚拟化。
2. 在实施之前阅读这整篇文章。

## <a name="high-level-overview-of-what-were-doing-and-why"></a>我们要做的事情及其原因的简要概述
* 我们将创建一个具有两个 NIC 且支持嵌套的 VM。 
* 一个 NIC 将用于提供可以通过 NAT 进行 Internet 访问的嵌套 VM，另一个 NIC 则用于将流量从内部交换机路由到虚拟机监控程序外部的资源。 每个 NIC 都需要位于不同的路由域（即不同的子网）中。
* 这意味着，我们需要一个至少包含三个子网的虚拟网络。 一个用于 NAT，一个用于 LAN 路由，一个不使用，而是为嵌套 VM 预留。 在本文档中，我们用于子网的名称为“NAT”、“Hyper-V-LAN”和“Ghosted”。
* 这些子网的大小由你自行决定，但有一些注意事项。 “Ghosted”子网的大小决定了你可以为嵌套 VM 设置多少个 IP。 此外，“NAT”和“Hyper-V-LAN”子网的大小决定了你可以为虚拟机监控程序设置多少个 IP。 因此，如果只是打算设置一个或两个虚拟机监控程序，则从技术上来讲，我们在这里可以创建真正的小型子网。
* 背景：即使配置了内部或外部交换机，嵌套 VM 也不会从其主机连接到的 VNet 接收 DHCP。 
  * 这意味着 Hyper-V 主机必须提供 DHCP。
* Hyper-V 主机无法感知 VNet 上当前分配的租约。因此，为了避免出现主机分配已存在的 IP 这样一种情况，我们必须分配一个仅供 Hyper-V 主机使用的 IP 块。 这样可以避免出现 IP 重复的情况。
  * 我们选择的 IP 块将对应于 Hyper-V 所在的 VNet 中的子网。
  * 我们希望它对应于现有子网的原因是，需要处理经 ExpressRoute 进行的回向 BGP 播发。 如果我们只是建立一个供 Hyper-V 主机使用的 IP 范围，则必须创建一系列静态路由，方便本地客户端与嵌套 VM 通信。 这就意味着它并不是一项硬性要求，因为我们可以为嵌套 VM 建立 IP 范围，然后创建将客户端定向到该范围的 Hyper-V 主机所需的所有路由。
* 我们会在 Hyper-V 中创建内部交换机，然后为新创建的接口分配一个 IP 地址，该地址在我们为 DHCP 留出的范围内。 该 IP 地址将成为嵌套 VM 的默认网关，用于在内部交换机与连接到 VNet 的主机的 NIC 之间进行路由。
* 我们会在主机上安装“路由和远程访问”角色，将主机变成路由器。  这是必需的，否则无法在主机外部资源与嵌套 VM 之间通信。
* 我们会告诉其他资源如何访问这些嵌套 VM。 这就要求我们创建一个用户定义的路由表，其中包含嵌套 VM 所在的 IP 范围的静态路由。 此静态路由将指向 Hyper-V 的 IP 地址。
* 然后，你将此 UDR 置于网关子网上，这样本地客户端就会知道如何访问我们的嵌套 VM。
* 还需将此 UDR 放在 Azure 中需要连接到嵌套 VM 的任何其他子网上。
* 如果有多个 Hyper-V 主机，则需创建更多的“浮动”子网，并向 UDR 添加一个额外的静态路由。
* 解除 Hyper-V 主机的授权时，会删除我们的“浮动”子网或重新调整其用途，并从我们的 UDR 中删除该静态路由，或者完全删除该 UDR（如果该 Hyper-V 主机是最后一个）。

## <a name="creating-the-host"></a>创建主机

我将快速讲解一下那些取决于个人偏好的配置值，例如 VM 名称、资源组等。

1. 导航到 portal.azure.com
2. 单击左上的“创建资源”
3. 从“常用”列中选择“Window Server 2016 VM”
4. 在“基本信息”选项卡上，请务必选择能够进行嵌套虚拟化的“VM 大小”
5. 转到“网络”选项卡
6. 使用以下配置创建新的虚拟网络
    * VNet 地址空间：10.0.0.0/22
    * 子网 1
        * 名称：NAT
        * 地址空间：10.0.0.0/24
    * 子网 2
        * 名称：Hyper-V-LAN
        * 地址空间：10.0.1.0/24
    * 子网 3
        * 名称：Ghosted
        * 地址空间：10.0.2.0/24
    * 子网 4
        * 名称：Azure-VMs
        * 地址空间：10.0.3.0/24
7. 确保为 VM 选择了 NAT 子网
8. 转到“查看 + 创建”，选择“创建”

## <a name="create-the-second-network-interface"></a>创建第二个网络接口
1. VM 完成预配后，请在 Azure 门户中浏览到该 VM
2. 停止 VM
3. 停止 VM 后，请转到“设置”下的“网络”
4. “附加网络接口”
5. “创建网络接口”
6. 为其指定一个名称（什么样的名称并不重要，但务必记住它）
7. 选择“Hyper-V-LAN”作为子网
8. 确保选择主机所在的“资源组”
9. “创建”
10. 此时会返回到上一屏幕。请确保选择新创建的“网络接口”，然后选择“确定”
11. 返回到“概览”窗格，在上一操作完成后重启 VM
12. 导航到刚创建的第二个 NIC，可以在之前选择的“资源组”中找到它
13. 转到“IP 配置”，将“IP 转发”切换到“启用”，然后保存所做的更改

## <a name="setting-up-hyper-v"></a>设置 Hyper-V
1. 以远程方式进入主机
2. 打开提升的 PowerShell 提示符
3. 运行以下命令：`Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart`
4. 此时会重启主机
5. 重新连接到主机，继续完成其余设置

## <a name="creating-our-virtual-switch"></a>创建虚拟交换机

1. 在管理模式下打开 PowerShell。
2. 创建内部交换机：`New-VMSwitch -Name "NestedSwitch" -SwitchType Internal`
3. 为新创建的接口分配 IP：`New-NetIPAddress –IPAddress 10.0.2.1 -PrefixLength 24 -InterfaceAlias "vEthernet (NestedSwitch)"`

## <a name="install-and-configure-dhcp"></a>安装并配置 DHCP

许多人在第一次尝试实现嵌套虚拟化时都错过了此组件。与本地不同（在本地，来宾 VM 会从主机所在的网络接收 DHCP），为 Azure 中的嵌套 VM 提供 DHCP 时，必须通过嵌套 VM 运行时所在的主机提供。因此，需要将一个 IP 地址静态分配给每个不可缩放的嵌套 VM。 

1. 安装 DHCP 角色：`Install-WindowsFeature DHCP -IncludeManagementTools`
2. 创建 DHCP 作用域：`Add-DhcpServerV4Scope -Name "Nested VMs" -StartRange 10.0.2.2 -EndRange 10.0.2.254 -SubnetMask 255.255.255.0`
3. 为作用域配置 DNS 和默认网关选项：`Set-DhcpServerV4OptionValue -DnsServer 168.63.129.16 -Router 10.0.2.1`
    * 如果想要正常进行名称解析，请务必输入有效的 DNS 服务器。 在此示例中，我使用 [Azure 的递归 DNS](https://docs.microsoft.com/azure/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances)。

## <a name="installing-remote-access"></a>安装远程访问

1. 打开“服务器管理器”，选择“添加角色和功能”。
2. 选择“下一步”，直到你到达“服务器角色”。
3. 选中“远程访问”，单击“下一步”，直到你到达“角色服务”。
4. 选中“路由”，选择“添加功能”，接着选择“下一步”，然后单击“安装”。 完成此向导的操作并等待安装完成。

## <a name="configuring-remote-access"></a>配置远程访问

1. 打开“服务器管理器”，选择“工具”，然后选择“路由和远程访问”。
2. 在“路由和远程访问”管理面板左侧会出现一个图标，其旁边是你的服务器名称。请右键单击此图标，然后选择“配置并启用路由和远程访问”。
3. 在向导中选择“下一步”，选择“自定义配置”，然后选择“下一步”。
4. 选中“NAT”和“LAN 路由”，接着选择“下一步”，然后选择“完成”。 如果它要求你启动服务，则执行该操作。
5. 现在，导航到“IPv4”节点并展开它，使“NAT”节点可用。
6. 右键单击“NAT”，选择“新建接口...”，接着选择“以太网”（这应该是你的 IP 为“10.0.0.4”的第一个 NIC），然后选择用于连接到 Internet 的“公共接口”并在该接口上启用 NAT。 
7. 现在，我们需要创建一些静态路由，强制将 LAN 流量发送到第二个 NIC。 若要执行此操作，请转到“IPv4”下的“静态路由”节点。
8. 然后，我们将创建以下路由。
    * 路由 1
        * 接口：以太网
        * 目标：10.0.0.0
        * 网络掩码：255.255.255.0
        * 网关：10.0.0.1
        * 指标：256
        * 注意：我们将其放在此处，这样主 NIC 就可以响应目标为其自己的接口的流量。 如果我们未在此处提供此项，则以下路由会导致目标为 NIC 1 的流量发送到 NIC 2。 这会创建一个非对称路由。 10.0.0.1 是 Azure 分配给 NAT 子网的 IP 地址。 Azure 使用某个范围中的第一个可用 IP 作为默认网关。 因此，如果将 192.168.0.0/24 用于 NAT 子网，则网关将为 192.168.0.1。 在进行路由时，系统会选择更具体的路由，这意味着此路由将取代以下路由。

    * 路由 2
        * 接口：以太网 2
        * 目标：10.0.0.0
        * 网络掩码：255.255.252.0
        * 网关：10.0.1.1
        * 指标：256
        * 注意：这是一个针对目标为 Azure VNet 的流量的“全部捕获”路由。 它会强制流量发往第二个 NIC。 对于其他范围，如果希望嵌套 VM 访问这些范围，则需为其添加其他路由。 因此，如果本地网络是 172.16.0.0/22，则需使用另一个路由将该流量发送到虚拟机监控程序的第二个 NIC。

## <a name="creating-a-route-table-within-azure"></a>在 Azure 中创建路由表

请参阅[此文](https://docs.microsoft.com/azure/virtual-network/tutorial-create-route-table-portal)，更深入地了解如何在 Azure 中创建和管理路由。

1. 导航到 https://portal.azure.com 。
2. 在左上角选择“创建资源”。
3. 在搜索字段中键入“路由表”，然后按 Enter。
4. 顶部结果将是路由表，选择它，然后选择“创建”
5. 命名路由表。在我的示例中，我将其命名为“Routes-for-nested-VMs”。
6. 确保选择 Hyper-V 主机所在的订阅。
7. 创建新的资源组或选择现有的资源组，确保创建路由表的区域是 Hyper-V 主机所在的区域。
8. 选择“创建”。

## <a name="configuring-the-route-table"></a>配置路由表

1. 导航到刚创建的路由表。 为此，可以从门户顶部正中的搜索栏中搜索路由表的名称。
2. 选择路由表后，请从边栏选项卡中转到“路由”。
3. 选择“添加”。
4. 为路由命名，我使用“Nested-VMs”作为名称。
5. 对于地址前缀，请输入“浮动”子网的 IP 范围。 在此示例中，它将是 10.0.2.0/24。
6. 对于“下一跃点类型”，请选择“虚拟设备”，输入 Hyper-V 主机的第二个 NIC 的 IP 地址 (10.0.1.4)，然后选择“确定”。
7. 现在，在边栏选项卡中选择“子网”（位于“路由”正下方）。
8. 依次选择“关联”、“Nested-Fun”VNet、“Azure-VMs”子网、“确定”。
9. 对于 Hyper-V 主机所在的子网以及需要访问嵌套 VM 的任何其他子网，请执行这个相同的过程。 如果已连接 

# <a name="end-state-configuration-reference"></a>结束状态配置参考
本指南中的环境有以下配置。 本部分用作参考。

1. Azure 虚拟网络信息。
    * VNet 高级配置。
        * 名称：Nested-Fun
        * 地址空间：10.0.0.0/22
        * 注意：此配置将包含四个子网。 另外，这些范围不是一成不变的， 可以根据环境按需调整。 

    * 第一个子网的高级配置。
        * 名称：NAT
        * 地址空间：10.0.0.0/24
        * 注意：这是我们的 Hyper-V 主机主 NIC 所在的位置。 这将用于处理嵌套 VM 的出站 NAT。 它将成为嵌套 VM 的 Internet 网关。

    * 第二个子网的高级配置。
        * 名称：Hyper-V-LAN
        * 地址空间：10.0.1.0/24
        * 注意：Hyper-V 主机会使用第二个 NIC 来处理嵌套 VM 与 Hyper-V 主机外部非 Internet 资源之间的路由。

    * 第三个子网的高级配置。
        * 名称：Ghosted
        * 地址空间：10.0.2.0/24
        * 注意：这将是一个“浮动”子网。 地址空间将由嵌套式 VM 使用，其存在是为了处理回到本地的路由播发。 实际上不会将 VM 部署到此子网中。

    * 第四个子网的高级配置。
        * 名称：Azure-VMs
        * 地址空间：10.0.3.0/24
        * 注意：包含 Azure VM 的子网。

1. Hyper-V 主机具有以下 NIC 配置。
    * 主 NIC 
        * IP 地址：10.0.0.4
        * 子网掩码：255.255.255.0
        * 默认网关：10.0.0.1
        * DNS：针对 DHCP 进行配置
        * 已启用 IP 转发：否

    * 辅助 NIC
        * IP 地址：10.0.1.4
        * 子网掩码：255.255.255.0
        * 默认网关：空
        * DNS：针对 DHCP 进行配置
        * 已启用 IP 转发：是

    * Hyper-V 为内部虚拟交换机创建了 NIC
        * IP 地址：10.0.2.1
        * 子网掩码：255.255.255.0
        * 默认网关：空

3. 路由表会有一个规则。
    * 规则 1
        * 名称：Nested-VMs
        * 目标：10.0.2.0/24
        * 下一个跃点：虚拟设备 - 10.0.1.4

## <a name="conclusion"></a>结论

现在应该能够将虚拟机（甚至包括 32 位 VM！）部署到 Hyper-V 主机，并且可以从本地和 Azure 中访问它。
