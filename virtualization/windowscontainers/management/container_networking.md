# 容器网络

**这是初步内容，可能还会更改。**

关于网络，Windows 容器的作用类似于虚拟机。 每个容器都有一个连接到虚拟交换机的虚拟网络适配器，通过该适配器可转发入站和出站流量。 两种类型的网络配置都可用。

- **网络地址转换模式** - 每个容器均已连接到内部虚拟交换机，并且将接收内部 IP 地址。 NAT 配置会将此内部地址转换为容器主机的外部地址。

- **透明模式** - 每个容器均已连接到外部虚拟交换机，并且将从 DHCP 服务器接收 IP 地址。

本文档将详细介绍每种模式的优点和配置。

## NAT 网络模式

**网络地址转换** - 此配置包含类型为 NAT 和 WinNat 的内部网络交换机。 在此配置中，容器主机拥有一个在网络上可访问的“外部”IP 地址。 所有容器都分配了在网络上无法访问的“内部”地址。 为了使容器在此配置中可访问，需要将主机的外部端口映射到容器端口的内部端口。 这些映射存储在 NAT 端口映射表中。 通过主机的 IP 地址和外部端口可以访问容器，该容器可将流量转发到容器的内部 IP 地址和端口。 NAT 的优点是容器主机可以扩展到数百个容器，尽管仅使用一个外部可用的 IP 地址。 此外，NAT 允许多个容器托管可能需要相同通信端口的应用程序。

### 主机配置

若要配置网络地址转换的容器主机，请按照下列步骤进行操作。

创建一个类型为“NAT”的虚拟交换机，并通过内部子网对其进行配置。 有关 **New-VMSwitch** 命令的详细信息，请参阅 [New-VMSwitch 参考](https://technet.microsoft.com/en-us/library/hh848455.aspx)。

```powershell
New-VMSwitch -Name "NAT" -SwitchType NAT -NATSubnetAddress "172.16.0.0/12"
```
创建网络地址转换对象。 此对象将负责 NAT 地址转换。 有关 **New-NetNat** 命令的详细信息，请参阅 [New-NetNat 参考](https://technet.microsoft.com/en-us/library/dn283361.aspx)。

```powershell
New-NetNat -Name NAT -InternalIPInterfaceAddressPrefix "172.16.0.0/12" 
```

### 容器配置

创建 Windows 容器时，可以为该容器选择一个虚拟交换机。 当容器连接到配置为使用 NAT 的虚拟交换机时，该容器将接收已转换的地址。

此示例将创建连接到支持 NAT 的虚拟交换机的容器。

```powershell
New-Container -Name DemoNAT -ContainerImageName WindowsServerCore -SwitchName "NAT"
```

在容器启动后，可以从容器内查看 IP 地址。

```powershell
[DemoNAT]: PS C:\> ipconfig

Windows IP Configuration
Ethernet adapter vEthernet (Virtual Switch-527ED2FB-D56D-4852-AD7B-E83732A032F5-0):
   Connection-specific DNS Suffix  . : contoso.com
   Link-local IPv6 Address . . . . . : fe80::384e:a23d:3c4b:a227%16
   IPv4 Address. . . . . . . . . . . : 172.16.0.2
   Subnet Mask . . . . . . . . . . . : 255.240.0.0
   Default Gateway . . . . . . . . . : 172.16.0.1
```

有关启动和连接到 Windows 容器的详细信息，请参阅[管理容器](./manage_containers.md)。

### 端口映射

若要访问“支持 NAT”的容器内部的应用程序，需要在容器和容器主机之间创建端口映射。 若要创建映射，需要容器的 IP 地址、“内部”容器端口和“外部”主机端口。

此示例将通过 IP 地址 **172.16.0.2** 创建一个主机的端口 **80** 到容器的端口 **80** 之间的映射。

```powershell
Add-NetNatStaticMapping -NatName "Nat" -Protocol TCP -ExternalIPAddress 0.0.0.0 -InternalIPAddress 172.16.0.2 -InternalPort 80 -ExternalPort 80
```

此示例将通过 IP 地址 **172.16.0.3** 创建一个容器主机的端口 **82** 到容器的端口 **80** 之间的映射。

```powershell
Add-NetNatStaticMapping -NatName "Nat" -Protocol TCP -ExternalIPAddress 0.0.0.0 -InternalIPAddress 172.16.0.3 -InternalPort 80 -ExternalPort 82
```
> 每个外部端口都将需要相应的防火墙规则。 此规则可使用 `New-NetFirewallRule` 创建。 有关详细信息，请参阅 [New-NetFirewallRule 参考](https://technet.microsoft.com/en-us/library/jj554908.aspx)。

在创建端口映射后，可通过容器主机（物理或虚拟）的 IP 地址和公开的外部端口访问容器应用程序。 例如，下图描述了 NAT 配置，具有面向容器主机的外部端口 **82** 的请求。 根据端口映射，此请求将返回在容器 2 中托管的应用程序。

![](./media/nat1.png)

来自 Internet 浏览器的请求视图。

![](./media/portmapping.png)

## 透明网络模式

**透明网络** - 此配置包含一个外部网络交换机。 在此配置中，每个容器都从 DHCP 服务器接收 IP 地址，并且在此 IP 地址上可访问。 其优点是不保留端口映射表。

### 主机配置

若要配置容器系统，以便容器可以从 DHCP 服务器接收 IP 地址，需要创建可连接到物理或虚拟网络适配器的虚拟交换机。

以下示例使用名为“以太网”的网络适配器创建名为 DHCP 的虚拟交换机。

```powershell
New-VMSwitch -Name DHCP -NetAdapterName Ethernet
```

如果容器主机本身就是虚拟机，则需要在与容器交换机一起使用的网络适配器上启用 MacAddressSpoofing。 以下示例在名为 `DemoVm` 的 VM 上完成此操作。

```powershell
Get-VMNetworkAdapter -VMName DemoVM | Set-VMNetworkAdapter -MacAddressSpoofing On
```
外部虚拟交换机现在可以连接到容器，随后可以从 DHCP 服务器接收 IP 地址。 在此配置中，可以在分配给容器的 IP 地址上访问容器内部托管的应用程序。

## Docker 配置

启动 Docker 守护程序时，可以选择一个网桥。 在 Windows 上运行 Docker 时，此守护程序为 External 或 NAT 虚拟交换机。 以下示例启动 Docker 守护程序，并指定一个名为 `Virtual Switch` 的虚拟交换机。

```powershell
Docker daemon –D –b “Virtual Switch” -H 0.0.0.0:2375
```

如果你已使用 Windows 容器快速入门中提供的脚本部署容器主机和 Docker，将创建类型为 NAT 的内部虚拟交换机，并创建预配置为使用此交换机的 Docker 服务。 若要更改 Docker 服务所使用的虚拟交换机，需要停止 Docker 服务、修改配置文件并再次启动该服务。

若要停止该服务，请运行以下 PowerShell 命令。

```powershell
Stop-Service docker
```

可在“c:\programdata\docker\runDockerDaemon.cmd”上找到配置文件。 编辑以下行，从而将 `Virtual Switch` 替换为 Docker 服务要使用的虚拟交换机的名称。

```powershell
docker daemon -D -b “New Switch Name"
```
最后启动该服务。

```powershell
Start-Service docker
```

## 管理容器网络适配器

无论是哪种网络配置（NAT 或不透明），可以使用多个 PowerShell 命令来管理容器网络适配器和虚拟交换机连接。

管理容器网络适配器

- Add-ContainerNetworkAdapter - 将网络适配器添加到容器。
- Set-ContainerNetworkAdapter - 修改容器网络适配器。
- Remove-ContainerNetworkAdapter - 删除容器网络适配器。
- Remove-ContainerNetworkAdapter - 返回有关容器网络适配器的数据。

管理容器网络适配器和虚拟交换机之间的连接。

- Connect-ContainerNetworkAdapter - 将容器连接到虚拟交换机。
- Disconect-ContainerNetworkAdapter - 断开容器与虚拟交换机的连接。

有关其中每个命令的详细信息，请参阅[容器 PowerShell 参考](https://technet.microsoft.com/en-us/library/mt433069.aspx)。




