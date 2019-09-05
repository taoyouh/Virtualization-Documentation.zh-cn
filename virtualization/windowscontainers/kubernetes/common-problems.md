---
title: Kubernetes 疑难解答
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: troubleshooting
ms.prod: containers
description: 关于部署 Kubernetes 和加入 Windows 节点的常见问题的解决方案。
keywords: kubernetes、1.14、linux、compile
ms.openlocfilehash: b6e4e648ff050e13a0930f2834949867e44ce895
ms.sourcegitcommit: d252f356a3de98f224e1550536810dfc75345303
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/04/2019
ms.locfileid: "10069931"
---
# <a name="troubleshooting-kubernetes"></a>Kubernetes 疑难解答 #
此页面逐一介绍 Kubernetes 设置、网络和部署的一些常见问题。

> [!tip]
> 通过向[我们的文档存储库](https://github.com/MicrosoftDocs/Virtualization-Documentation/)提出 PR 来建议常见问题解答项目。

此页面分为以下几类:
1. [一般问题](#general-questions)
2. [常见网络错误](#common-networking-errors)
3. [常见的 Windows 错误](#common-windows-errors)
4. [常见 Kubernetes 主错误](#common-kubernetes-master-errors)

## <a name="general-questions"></a>一般问题 ##

### <a name="how-do-i-know-startps1-on-windows-completed-successfully"></a>如何知道 "开始"。 Windows 上的 ps1 已成功完成？ ###
你应该看到 kubelet、kube-proxy 和 (如果选择 Flannel 作为网络解决方案) flanneld 你的节点上运行的主机代理进程, 并且在单独的 PoSh 窗口中显示运行的日志。 此外, 你的 Windows 节点在 Kubernetes 群集中应列为 "就绪"。

### <a name="can-i-configure-to-run-all-of-this-in-the-background-instead-of-posh-windows"></a>是否可以配置为在后台运行所有这些操作, 而不是 PoSh windows？ ###
从 Kubernetes 版本1.11 开始, kubelet & kube-proxy 可以作为本机[Windows 服务](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services)运行。 你还可以始终使用备用服务管理器 (如[nssm](https://nssm.cc/) ), 在后台始终运行这些进程 (flanneld、kubelet & kube-proxy)。 有关示例的步骤, 请参阅[Windows 服务 Kubernetes](./kube-windows-services.md) 。

### <a name="i-have-problems-running-kubernetes-processes-as-windows-services"></a>将 Kubernetes 进程作为 Windows 服务运行时遇到问题 ###
对于初始疑难解答, 可以使用[nssm](https://nssm.cc/)中的以下标志将 stdout 和 stderr 重定向到输出文件:
```
nssm set <Service Name> AppStdout C:\k\mysvc.log
nssm set <Service Name> AppStderr C:\k\mysvc.log
```
有关其他详细信息, 请参阅官方[nssm 使用](https://nssm.cc/usage)文档。

## <a name="common-networking-errors"></a>常见网络错误 ##

### <a name="hostport-publishing-is-not-working"></a>HostPort 发布不起作用 ###
目前不能使用 Kubernetes `containers.ports.hostPort`字段发布端口, 因为 Windows CNI 插件不会遵守此字段。 在节点上发布端口时, 请使用 NodePort 发布。

### <a name="i-am-seeing-errors-such-as-hnscall-failed-in-win32-the-wrong-diskette-is-in-the-drive"></a>我在 Win32 中看到错误, 例如 "hnsCall 失败: 驱动器中有错误的软盘"。 ###
在对 HNS 对象进行自定义修改或安装新的 Windows 更新 (将更改引入到 HNS, 而不会断开旧的 HNS 对象) 时, 可能会发生此错误。 它指示以前在更新之前创建的 HNS 对象与当前安装的 HNS 版本不兼容。

在 Windows Server 2019 (和更低) 上, 用户可以通过删除 HNS 数据文件来删除 HNS 对象。 
```
Stop-Service HNS
rm C:\ProgramData\Microsoft\Windows\HNS\HNS.data
Start-Service HNS
```

用户应能够直接删除任何不兼容的 HNS 终结点或网络:
```
hnsdiag list endpoints
hnsdiag delete endpoints <id>
hnsdiag list networks 
hnsdiag delete networks <id>
Restart-Service HNS
```

Windows Server 上的用户 (版本 1903) 可以转到以下注册表位置并删除以网络名称开头的任何 Nic (例如`vxlan0`或`cbr0`):
```
\\Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\NicList
```

### <a name="containers-on-my-flannel-host-gw-deployment-on-azure-cannot-reach-the-internet"></a>我的 Flannel 主机上的容器-Azure 上的 gw 部署无法连接到 internet ###
在 Azure 上的主机-gw 模式下部署 Flannel 时, 数据包必须经历 Azure 物理主机 vSwitch。 用户应针对分配给节点的每个子网为[用户定义](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview#user-defined)的 "虚拟装置" 类型的路由编程。 这可以通过 Azure 门户 (请参阅[此处](https://docs.microsoft.com/en-us/azure/virtual-network/tutorial-create-route-table-portal)的示例) 或通过`az` azure CLI 执行此操作。 下面是名为 "MyRoute" 的一个示例 UDR, 其中包含 IP 10.0.0.4 和各自的 pod 子网 10.244.0.0/24 的节点的 az 命令:
```
az network route-table create --resource-group <my_resource_group> --name BridgeRoute 
az network route-table route create  --resource-group <my_resource_group> --address-prefix 10.244.0.0/24 --route-table-name BridgeRoute  --name MyRoute --next-hop-type VirtualAppliance --next-hop-ip-address 10.0.0.4 
```

### <a name="my-windows-pods-cannot-ping-external-resources"></a>我的 Windows 箱无法 ping 外部资源 ###
Windows 盒目前没有为 ICMP 协议预先设定的出站规则。 但是, 支持 TCP/UDP。 当尝试演示与群集外的资源的连接时, 请用`ping <IP>`对应`curl <IP>`的命令替换。

如果你仍面临问题, 很可能是[cni](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf)中的网络配置值得格外注意。 始终可以编辑此静态文件, 配置将应用于任何新创建的 Kubernetes 资源。

为什么？
Kubernetes 网络要求 (请参阅[Kubernetes 模型](https://kubernetes.io/docs/concepts/cluster-administration/networking/)) 之一用于在没有 NAT 的情况下发生群集通信。 为满足此要求, 我们有一个可供我们不希望出站 NAT 出现的所有通信的[例外](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf#L20)项。 但是, 这还意味着你需要从例外类型中排除你试图查询的外部 IP。 只有在您的 Windows SNAT'ed 中发起的通信量才会被正确地接收到来自外部世界的响应。 在这方面, 您的例外`cni.conf`项应如下所示:
```conf
"ExceptionList": [
  "10.244.0.0/16",  # Cluster subnet
  "10.96.0.0/12",   # Service subnet
  "10.127.130.0/24" # Management (host) subnet
]
```

### <a name="my-windows-node-cannot-access-a-nodeport-service"></a>我的 Windows 节点无法访问 NodePort 服务 ###
从节点本身的本地 NodePort 访问将失败。 这是一个已知限制。 NodePort access 将从其他节点或外部客户端工作。

### <a name="after-some-time-vnics-and-hns-endpoints-of-containers-are-being-deleted"></a>在某段时间后, 将删除容器的 vNICs 和 HNS 终结点 ###
当参数未传递到`hostname-override` [kube-proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/)时, 可能会导致此问题。 若要解决此问题, 用户需要将主机名传递到 kube-代理, 如下所示:
```
C:\k\kube-proxy.exe --hostname-override=$(hostname)
```

### <a name="on-flannel-vxlan-mode-my-pods-are-having-connectivity-issues-after-rejoining-the-node"></a>在 Flannel (vxlan) 模式下, 我的箱在 rejoining 节点后出现连接问题 ###
当以前删除的节点重新加入群集时, flannelD 将尝试为该节点分配新的 pod 子网。 用户应删除以下路径中的旧 pod 子网配置文件:
```powershell
Remove-Item C:\k\SourceVip.json
Remove-Item C:\k\SourceVipRequest.json
```

### <a name="after-launching-startps1-flanneld-is-stuck-in-waiting-for-the-network-to-be-created"></a>启动启动后, Flanneld 将停留在 "正在等待创建网络" 中 ###
正在调查的此问题有很多报告;最有可能是 flannel 网络的管理 IP 设置时的计时问题。 解决方法是简单地重新启动启动。 ps1 或手动重新启动, 如下所示:
```
PS C:> [Environment]::SetEnvironmentVariable("NODE_NAME", "<Windows_Worker_Hostname>")
PS C:> C:\flannel\flanneld.exe --kubeconfig-file=c:\k\config --iface=<Windows_Worker_Node_IP> --ip-masq=1 --kube-subnet-mgr=1
```

目前还有一个可解决此问题的[PR](https://github.com/coreos/flannel/pull/1042) 。


### <a name="on-flannel-host-gw-my-windows-pods-do-not-have-network-connectivity"></a>在 Flannel (host-gw) 上, 我的 Windows 箱没有网络连接 ###
如果您希望使用 l2bridge 进行网络连接 (亦[即 flannel 主机网关](./network-topologies.md#flannel-in-host-gateway-mode)), 则应确保为 Windows 容器主机虚拟机 (来宾) 启用 MAC 地址欺骗。 若要实现此目的, 你应该在托管 Vm 的计算机上以管理员身份运行以下内容 (例如, Hyper-v 提供的示例):

```powershell
Get-VMNetworkAdapter -VMName "<name>" | Set-VMNetworkAdapter -MacAddressSpoofing On
```

> [!TIP]
> 如果您使用基于 VMware 的产品满足您的虚拟化需要, 请考虑为 MAC 欺骗要求启用[混杂模式](https://kb.vmware.com/s/article/1004099)。

>[!TIP]
> 如果你要在其他云提供商的 Azure 或 IaaS Vm 上部署 Kubernetes, 你也可以改用[覆盖网络](./network-topologies.md#flannel-in-vxlan-mode)。

### <a name="my-windows-pods-cannot-launch-because-of-missing-runflannelsubnetenv"></a>由于缺少/run/flannel/subnet.env, 我的 Windows 盒无法启动 ###
这表示 Flannel 未正确启动。 你可以尝试重新启动 flanneld, 也可以从`/run/flannel/subnet.env` Kubernetes master 上手动将文件复制到`C:\run\flannel\subnet.env` Windows worker 节点上, 并将`FLANNEL_SUBNET`该行修改为分配的子网。 例如, 如果分配了节点子网 10.244.4.1/24:
```
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.4.1/24
FLANNEL_MTU=1500
FLANNEL_IPMASQ=true
```
更安全的做法是让 flanneld 为你生成此文件。

### <a name="pod-to-pod-connectivity-between-hosts-is-broken-on-my-kubernetes-cluster-running-on-vsphere"></a>在 vSphere 上运行的 Kubernetes 群集上, 主机间的 pod 到 pod 连接中断 
由于 vSphere 和 Flannel 都保留端口 4789 (默认 VXLAN 端口) 用于覆盖网络, 因此数据包最终会被截取。 如果 vSphere 用于覆盖网络, 则应将其配置为使用不同的端口, 以便释放4789。  


### <a name="my-endpointsips-are-leaking"></a>我的终结点/Ip 泄漏 ###
存在2个当前已知问题, 这些问题可能会导致终结点泄漏。 
1.  第一个[已知问题](https://github.com/kubernetes/kubernetes/issues/68511)是 Kubernetes 版本1.11 中的一个问题。 请避免使用 Kubernetes 版本 1.11.0-1.11.2。
2. 可能导致终结点泄漏的第二个[已知问题](https://github.com/docker/libnetwork/issues/1950)是终结点存储中的并发问题。 若要接收修补程序, 必须使用 Docker EE 18.09 或更高版本。

### <a name="my-pods-cannot-launch-due-to-network-failed-to-allocate-for-range-errors"></a>由于 "网络: 无法为区域分配" 错误, 无法启动我的箱 ###
这表示你的节点上的 IP 地址空间已用完。 若要清理任何[泄漏的终结点](#my-endpointsips-are-leaking), 请在受影响的节点上迁移任何资源 & 运行以下命令:
```
c:\k\stop.ps1
Get-HNSEndpoint | Remove-HNSEndpoint
Remove-Item -Recurse c:\var
```

### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>我的 Windows 节点无法使用服务 IP 访问我的服务 ###
这是 Windows 上当前网络堆栈的已知限制。 但是, Windows*盒* **** 可以访问服务 IP。

### <a name="no-network-adapter-is-found-when-starting-kubelet"></a>启动 Kubelet 时未发现任何网络适配器 ###
Windows 网络堆栈需要虚拟适配器以使 Kubernetes 网络工作。 如果以下命令未返回任何结果（在管理员 shell 中），则表示虚拟网络创建 &mdash; Kubelet 工作的必要先决条件 &mdash; 已失败：

```powershell
Get-HnsNetwork | ? Name -ieq "cbr0"
Get-NetAdapter | ? Name -Like "vEthernet (Ethernet*"
```

通常, 在主机的网络适配器不是 "以太网" 的情况下, 修改 InterfaceName 脚本的[](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/start.ps1#L6)参数是值得的。 否则, 请参阅`start-kubelet.ps1`脚本的输出以查看在虚拟网络创建期间是否存在错误。 

### <a name="pods-stop-resolving-dns-queries-successfully-after-some-time-alive"></a>Pod 在处于活动状态一段时间后成功停止解析 DNS 查询 ###
Windows Server、版本1803和更低版本的网络堆栈中存在已知的 DNS 缓存问题, 有时可能会导致 DNS 请求失败。 要解决此问题, 您可以使用以下注册表项将最大 TTL 缓存值设置为零:

```Dockerfile
FROM microsoft/windowsservercore:<your-build>
SHELL ["powershell', "-Command", "$ErrorActionPreference = 'Stop';"]
New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxCacheTtl -Value 0 -Type DWord 
New-ItemPropery -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxNegativeCacheTtl -Value 0 -Type DWord
```

### <a name="i-am-still-seeing-problems-what-should-i-do"></a>我仍在看到问题。 我该怎么办？ ### 
你的网络或主机上可能设置了其他限制，以阻止在节点之间进行某些类型的通信。 请确保：
  - 您已正确配置所选的[网络拓扑](./network-topologies.md)
  - 允许可能来自 Pod 的流量
  - 允许 HTTP 流量（如果要部署 Web 服务）
  - 不会丢弃来自不同协议 (ie ICMP 与 TCP/UDP) 的数据包


## <a name="common-windows-errors"></a>常见的 Windows 错误 ##

### <a name="my-kubernetes-pods-are-stuck-at-containercreating"></a>我的 Kubernetes Pod 会停滞在“ContainerCreating” ###
很多原因可导致此问题，但最常见原因之一是暂停映像配置不正确。 这极有可能进一步导致其他问题。


### <a name="when-deploying-docker-containers-keep-restarting"></a>在进行部署时，Docker 容器不断重启 ###
检查暂停映像是否与你的操作系统版本兼容。 [说明](./deploying-resources.md)假设操作系统和容器都是版本1803。 如果你有更高版本的 Windows（如会员版本），将需要相应地调整映像。 请参阅 Microsoft 用于映像的 [Docker 存储库](https://hub.docker.com/u/microsoft/)。 无论如何，暂停映像 Dockerfile 和示例服务都会预期映像标记为 `:latest`。


## <a name="common-kubernetes-master-errors"></a>常见 Kubernetes 主错误 ##
调试 Kubernetes 主机分为三个主要类别（按可能性顺序）：

  - Kubernetes 系统容器出现问题。
  - `kubelet` 的运行方式出现问题。
  - 系统出现问题。

运行 `kubectl get pods -n kube-system` 以查看 Kubernetes 正在创建的 Pod；这样可以深入了解哪些特定项将会崩溃或无法正常启动。 然后，运行 `docker ps -a` 以查看支持这些 Pod 的所有原始容器。 最后，针对怀疑会导致问题的容器运行 `docker logs [ID]`，以查看进程的原始输出。


### <a name="cannot-connect-to-the-api-server-at-httpsaddressport"></a>无法在 `https://[address]:[port]` 连接到 API 服务器 ###
通常，此错误表示证书问题。 请确保你已经正确生成了配置文件，其中的 IP 地址与主机的 IP 地址匹配，并且你已将其复制到 API 服务器装载的目录中。

如果按照[我们的说明](./creating-a-linux-master.md)进行操作, 请在以下位置找到:   
* `~/kube/kubelet/`
* `$HOME/.kube/config`
*  `/etc/kubernetes/admin.conf`

 否则, 请参阅 API 服务器的清单文件以检查装入点。