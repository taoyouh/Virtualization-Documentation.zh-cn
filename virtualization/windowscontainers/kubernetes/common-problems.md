---
title: Kubernetes 疑难解答
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: troubleshooting
ms.prod: containers
description: 关于部署 Kubernetes 和加入 Windows 节点的常见问题的解决方案。
keywords: kubernetes，1.14，linux，编译
ms.openlocfilehash: eb8162a55eb1a639cde40faed7b01a48f0c50ad3
ms.sourcegitcommit: fed735dafbe40179b1e1c46840655248b52617b0
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/09/2020
ms.locfileid: "84614871"
---
# <a name="troubleshooting-kubernetes"></a>Kubernetes 疑难解答 #
此页面逐一介绍 Kubernetes 设置、网络和部署的一些常见问题。

> [!tip]
> 通过向[我们的文档存储库](https://github.com/MicrosoftDocs/Virtualization-Documentation/)提出 PR 来建议常见问题解答项目。

本页分为以下几类：
1. [一般问题](#general-questions)
2. [常见网络错误](#common-networking-errors)
3. [常见 Windows 错误](#common-windows-errors)
4. [常见的 Kubernetes 主服务器错误](#common-kubernetes-master-errors)

## <a name="general-questions"></a>一般问题 ##

### <a name="how-do-i-know-startps1-on-windows-completed-successfully"></a>如何实现知道，Windows 上的 ps1 已成功完成？ ###
你应看到 kubelet、kube 和（如果选择 Flannel 作为网络解决方案） flanneld 你的节点上运行的主机代理进程，并且运行的日志显示在单独的 PoSh 窗口中。 除此之外，Windows 节点在 Kubernetes 群集中应作为 "Ready" 列出。

### <a name="can-i-configure-to-run-all-of-this-in-the-background-instead-of-posh-windows"></a>是否可以将配置为在后台而不是 PoSh 窗口中运行所有此项？ ###
从 Kubernetes 1.11 版开始，kubelet & kube-proxy 可以作为本机[Windows 服务](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services)运行。 你还可以始终使用替代服务管理器（如[nssm](https://nssm.cc/) ）在后台始终运行这些进程（flanneld、kubelet & kube-proxy）。 有关示例步骤，请参阅[Kubernetes 上的 Windows 服务](./kube-windows-services.md)。

### <a name="i-have-problems-running-kubernetes-processes-as-windows-services"></a>将 Kubernetes 进程作为 Windows 服务运行时出现问题 ###
对于初始故障排除，可以使用[nssm](https://nssm.cc/)中的以下标志将 stdout 和 stderr 重定向到输出文件：
```
nssm set <Service Name> AppStdout C:\k\mysvc.log
nssm set <Service Name> AppStderr C:\k\mysvc.log
```
有关更多详细信息，请参阅官方[nssm 使用](https://nssm.cc/usage)文档。

## <a name="common-networking-errors"></a>常见网络错误 ##

### <a name="load-balancers-are-plumbed-inconsistently-across-the-cluster-nodes"></a>负载均衡器在群集节点之间不一致 ###
在 Windows 上，kube-proxy 为群集中的每个 Kubernetes 服务创建一个 HNS 负载均衡器。 在（默认值） kube 配置中，包含许多（通常为 100 +）负载均衡器的群集中的节点可能会耗尽可用的暂时 TCP 端口（也称为 动态端口范围，默认情况下涵盖端口49152到65535）。 这是因为每个（非 DSR）负载均衡器的每个节点上保留了大量端口。 此问题可能会在 kube 中出现错误，例如：
```
Policy creation failed: hcnCreateLoadBalancer failed in Win32: The specified port already exists.
```

用户可以通过运行[CollectLogs](https://github.com/microsoft/SDN/blob/master/Kubernetes/windows/debug/collectlogs.ps1)脚本并查阅文件来识别此问题 `*portrange.txt` 。

`CollectLogs.ps1`还将模拟 HNS 分配逻辑，以便在临时 TCP 端口范围内测试端口池分配可用性，并在中报告成功/失败 `reservedports.txt` 。 此脚本保留10个 64 TCP 临时端口范围（用于模拟 HNS 行为），计算保留成功 & 失败数，然后释放已分配的端口范围。 如果成功率小于10，则表明暂时池的可用空间不足。 还将在中生成大约有多少64块端口保留的 heuristical 摘要 `reservedports.txt` 。

若要解决此问题，可以采取几个步骤：
1.  对于永久解决方案，kube 负载平衡应设置为[DSR 模式](https://techcommunity.microsoft.com/t5/Networking-Blog/Direct-Server-Return-DSR-in-a-nutshell/ba-p/693710)。 DSR 模式完全实现，并且仅在较新的[Windows Server 有问必答版本 18945](https://blogs.windows.com/windowsexperience/2019/07/30/announcing-windows-server-vnext-insider-preview-build-18945/#o1bs7T2DGPFpf7HM.97) （或更高版本）上可用。
2. 作为一种解决方法，用户还可以使用命令（如）增加可用的临时端口的默认 Windows 配置 `netsh int ipv4 set dynamicportrange TCP <start_port> <port_count>` 。 *警告：* 覆盖默认的动态端口范围可能会对主机上依赖于非暂时范围可用的 TCP 端口的其他进程/服务产生影响，因此应仔细选择此范围。
3. 使用累积更新[KB4551853](https://support.microsoft.com/en-us/help/4551853)中包含的智能端口池共享来实现非 DSR 模式负载均衡器的可伸缩性增强功能。

### <a name="hostport-publishing-is-not-working"></a>HostPort 发布不起作用 ###
若要使用 HostPort 功能，请确保 CNI 插件为[0.8.6 以上](https://github.com/containernetworking/plugins/releases/tag/v0.8.6)版本或更高版本，并且 CNI 配置文件具有 `portMappings` 以下功能：
```
"capabilities": {
    "portMappings":  true
}
```

### <a name="i-am-seeing-errors-such-as-hnscall-failed-in-win32-the-wrong-diskette-is-in-the-drive"></a>我看到一些错误，如 "Win32 中的 hnsCall 失败：驱动器中有错误的软盘"。 ###
对 HNS 对象进行自定义修改或安装新的 Windows 更新时，会发生此错误，而无需撕裂旧的 HNS 对象。 它指示之前在更新之前创建的 HNS 对象与当前安装的 HNS 版本不兼容。

在 Windows Server 2019 （及更低）上，用户可以通过删除 HNS. 数据文件来删除 HNS 对象 
```
Stop-Service HNS
rm C:\ProgramData\Microsoft\Windows\HNS\HNS.data
Start-Service HNS
```

用户应能够直接删除任何不兼容的 HNS 终结点或网络：
```
hnsdiag list endpoints
hnsdiag delete endpoints <id>
hnsdiag list networks 
hnsdiag delete networks <id>
Restart-Service HNS
```

Windows Server 上的用户，版本1903可从以下注册表位置中删除任何以网络名称开头的 Nic （例如 `vxlan0` 或 `cbr0` ）：
```
\\Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\NicList
```

### <a name="containers-on-my-flannel-host-gw-deployment-on-azure-cannot-reach-the-internet"></a>我的 Flannel 主机上的容器-Azure 上的 gw 部署无法访问 internet ###
在 Azure 上的主机-gw 模式下部署 Flannel 时，数据包必须通过 Azure 物理主机 vSwitch。 用户应为分配给节点的每个子网为 "虚拟设备" 类型的[用户定义的路由](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview#user-defined)。 这可以通过 Azure 门户来完成（请参阅[此处](https://docs.microsoft.com/en-us/azure/virtual-network/tutorial-create-route-table-portal)的示例），也可以通过 Azure CLI 完成此操作 `az` 。 下面是一个名为 "MyRoute" 的示例 UDR，其中使用了包含 IP 10.0.0.4 和相应 pod 子网 10.244.0.0/24 的节点的 az 命令：
```
az network route-table create --resource-group <my_resource_group> --name BridgeRoute 
az network route-table route create  --resource-group <my_resource_group> --address-prefix 10.244.0.0/24 --route-table-name BridgeRoute  --name MyRoute --next-hop-type VirtualAppliance --next-hop-ip-address 10.0.0.4 
```

>[!TIP]
> 如果你自己在 Azure 或其他云提供商的 IaaS Vm 上部署 Kubernetes，则还可以改用[覆盖网络](./network-topologies.md#flannel-in-vxlan-mode)。

### <a name="my-windows-pods-cannot-ping-external-resources"></a>Windows pod 无法对外部资源进行 ping 操作 ###
Windows pod 目前没有针对 ICMP 协议编程的出站规则。 但是，支持 TCP/UDP。 尝试演示与群集外部的资源的连接时，请替换为 `ping <IP>` 相应的 `curl <IP>` 命令。

如果仍面临问题，则很可能是[cni](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf)中的网络配置需要特别注意。 您始终可以编辑此静态文件，配置将应用于所有新创建的 Kubernetes 资源。

为什么？
Kubernetes 网络要求（请参阅[Kubernetes 模型](https://kubernetes.io/docs/concepts/cluster-administration/networking/)）之一用于在不内部 NAT 的情况下进行群集通信。 为满足此要求，我们对所有不希望出站 NAT 发生的通信提供了[例外](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf#L20)。 但是，这也意味着你需要排除尝试从例外例外中查询的外部 IP。 只有这样，来自 Windows pod 的流量才会 SNAT'ed 正确地接收来自外部世界的响应。 在这种情况下，中的例外项 `cni.conf` 应如下所示：
```conf
"ExceptionList": [
  "10.244.0.0/16",  # Cluster subnet
  "10.96.0.0/12",   # Service subnet
  "10.127.130.0/24" # Management (host) subnet
]
```

### <a name="my-windows-node-cannot-access-a-nodeport-service"></a>我的 Windows 节点无法访问 NodePort 服务 ###
由于 Windows Server 2019 上的设计限制，从节点本身的本地 NodePort 访问会失败。 NodePort access 适用于其他节点或外部客户端。

### <a name="my-windows-node-stops-routing-thourgh-nodeports-after-i-scaled-down-my-pods"></a>我的 Windows 节点在缩放到我的箱后停止路由 thourgh NodePorts ###
由于设计限制，在 Windows 节点上至少需要有一个 pod 才能运行 NodePort 转发。

### <a name="after-some-time-vnics-and-hns-endpoints-of-containers-are-being-deleted"></a>经过一段时间后，将删除容器的 Vnic 和 HNS 终结点 ###
如果 `hostname-override` 未将参数传递给[kube](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/)，则可能会导致此问题。 若要解决此问题，用户需要将主机名传递到 kube-proxy，如下所示：
```
C:\k\kube-proxy.exe --hostname-override=$(hostname)
```

### <a name="on-flannel-vxlan-mode-my-pods-are-having-connectivity-issues-after-rejoining-the-node"></a>在 Flannel （vxlan）模式下，在重新加入节点后，我的 pod 出现连接问题 ###
每当将以前删除的节点重新加入到群集中时，flannelD 将尝试为该节点分配一个新的 pod 子网。 用户应删除以下路径中的旧 pod 子网配置文件：
```powershell
Remove-Item C:\k\SourceVip.json
Remove-Item C:\k\SourceVipRequest.json
```

### <a name="after-launching-startps1-flanneld-is-stuck-in-waiting-for-the-network-to-be-created"></a>启动 start. ps1 后，Flanneld 停滞在 "正在等待创建网络" ###
正在调查此问题的许多报告;这很可能是 flannel 网络的管理 IP 的设置时间问题。 解决方法是只重新启动 start. ps1 或手动重新启动，如下所示：
```
PS C:> [Environment]::SetEnvironmentVariable("NODE_NAME", "<Windows_Worker_Hostname>")
PS C:> C:\flannel\flanneld.exe --kubeconfig-file=c:\k\config --iface=<Windows_Worker_Node_IP> --ip-masq=1 --kube-subnet-mgr=1
```

还有一个可用于解决当前正在审核的问题的[PR](https://github.com/coreos/flannel/pull/1042) 。


### <a name="my-windows-pods-cannot-launch-because-of-missing-runflannelsubnetenv"></a>由于缺少/run/flannel/subnet.env，无法启动我的 Windows pod ###
这表示 Flannel 未正确启动。 你可以尝试重新启动 flanneld，或者可以在 `/run/flannel/subnet.env` Windows 辅助角色节点上的 Kubernetes 主机上手动将文件复制到 `C:\run\flannel\subnet.env` ，然后将 `FLANNEL_SUBNET` 该行修改为分配的子网。 例如，如果已分配节点子网 10.244.4.1/24：
```
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.4.1/24
FLANNEL_MTU=1500
FLANNEL_IPMASQ=true
```
更安全的做法是让 flanneld 为你生成此文件。


### <a name="pod-to-pod-connectivity-between-hosts-is-broken-on-my-kubernetes-cluster-running-on-vsphere"></a>在 vSphere 上运行的 Kubernetes 群集上，主机之间的 pod 到 pod 连接中断 
由于 vSphere 和 Flannel 都为覆盖网络预留端口4789（默认 VXLAN 端口），因此包最终会被拦截。 如果将 vSphere 用于覆盖网络，则应该将其配置为使用不同的端口，以便释放4789。  


### <a name="my-endpointsips-are-leaking"></a>我的终结点/Ip 正在泄漏 ###
存在2个可能导致终结点泄露的已知问题。 
1.  第一个[已知问题](https://github.com/kubernetes/kubernetes/issues/68511)是 Kubernetes 版本1.11 中的问题。 请避免使用 Kubernetes 版本 1.11.0-1.11.2。
2. 可能导致终结点泄露的第二个[已知问题](https://github.com/docker/libnetwork/issues/1950)是终结点存储中的并发问题。 若要接收此修补程序，你必须使用 Docker EE 18.09 或更高版本。

### <a name="my-pods-cannot-launch-due-to-network-failed-to-allocate-for-range-errors"></a>由于 "网络：未能分配范围" 错误，无法启动我的 pod ###
这表示节点上的 IP 地址空间已用完。 若要清理任何[泄露的终结点](#my-endpointsips-are-leaking)，请在受影响的节点上迁移任何资源 & 运行以下命令：
```
c:\k\stop.ps1
Get-HNSEndpoint | Remove-HNSEndpoint
Remove-Item -Recurse c:\var
```

### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>我的 Windows 节点无法使用服务 IP 访问我的服务 ###
这是 Windows 上当前网络堆栈的已知限制。 但*Windows pod 可以访问* **are**服务 IP。

### <a name="no-network-adapter-is-found-when-starting-kubelet"></a>启动 Kubelet 时未发现任何网络适配器 ###
Windows 网络堆栈需要虚拟适配器以使 Kubernetes 网络工作。 如果以下命令未返回任何结果（在管理员 shell 中），则表示虚拟网络创建 &mdash; Kubelet 工作的必要先决条件 &mdash; 已失败：

```powershell
Get-HnsNetwork | ? Name -ieq "cbr0"
Get-NetAdapter | ? Name -Like "vEthernet (Ethernet*"
```

通常，如果主机的网络适配器不是 "以太网"，则可以修改 InterfaceName 脚本的[InterfaceName](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/start.ps1#L6)参数。 否则，请参考脚本的输出， `start-kubelet.ps1` 查看虚拟网络创建过程中是否存在错误。 

### <a name="pods-stop-resolving-dns-queries-successfully-after-some-time-alive"></a>Pod 在处于活动状态一段时间后成功停止解析 DNS 查询 ###
Windows Server、版本1803及更低版本的网络堆栈中存在一个已知的 DNS 缓存问题，有时可能会导致 DNS 请求失败。 若要解决此问题，可以使用以下注册表项将最大 TTL 缓存值设置为零：

```Dockerfile
FROM microsoft/windowsservercore:<your-build>
SHELL ["powershell', "-Command", "$ErrorActionPreference = 'Stop';"]
New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxCacheTtl -Value 0 -Type DWord 
New-ItemPropery -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxNegativeCacheTtl -Value 0 -Type DWord
```

### <a name="i-am-still-seeing-problems-what-should-i-do"></a>我仍然看到问题。 我该怎么办？ ### 
你的网络或主机上可能设置了其他限制，以阻止在节点之间进行某些类型的通信。 请确保：
  - 已正确配置所选[网络拓扑](./network-topologies.md)
  - 允许可能来自 Pod 的流量
  - 允许 HTTP 流量（如果要部署 Web 服务）
  - 不会删除来自不同协议的数据包（ie ICMP 与 TCP/UDP）

>[!TIP]
> 对于其他自助资源，[此处也提供](https://techcommunity.microsoft.com/t5/Networking-Blog/Troubleshooting-Kubernetes-Networking-on-Windows-Part-1/ba-p/508648)了适用于 Windows 的 Kubernetes 故障排除指南。

## <a name="common-windows-errors"></a>常见 Windows 错误 ##

### <a name="my-kubernetes-pods-are-stuck-at-containercreating"></a>我的 Kubernetes Pod 会停滞在“ContainerCreating” ###
很多原因可导致此问题，但最常见原因之一是暂停映像配置不正确。 这极有可能进一步导致其他问题。


### <a name="when-deploying-docker-containers-keep-restarting"></a>在进行部署时，Docker 容器不断重启 ###
检查暂停映像是否与你的操作系统版本兼容。 [说明](./deploying-resources.md)假定 OS 和容器均为1803版。 如果你有更高版本的 Windows（如会员版本），将需要相应地调整映像。 请参阅 Microsoft 用于映像的 [Docker 存储库](https://hub.docker.com/u/microsoft/)。 无论如何，暂停映像 Dockerfile 和示例服务都会预期映像标记为 `:latest`。


## <a name="common-kubernetes-master-errors"></a>常见的 Kubernetes 主服务器错误 ##
调试 Kubernetes 主机分为三个主要类别（按可能性顺序）：

  - Kubernetes 系统容器出现问题。
  - `kubelet` 的运行方式出现问题。
  - 系统出现问题。

运行 `kubectl get pods -n kube-system` 以查看 Kubernetes 正在创建的 Pod；这样可以深入了解哪些特定项将会崩溃或无法正常启动。 然后，运行 `docker ps -a` 以查看支持这些 Pod 的所有原始容器。 最后，针对怀疑会导致问题的容器运行 `docker logs [ID]`，以查看进程的原始输出。


### <a name="cannot-connect-to-the-api-server-at-httpsaddressport"></a>无法在 `https://[address]:[port]` 连接到 API 服务器 ###
通常，此错误表示证书问题。 请确保你已经正确生成了配置文件，其中的 IP 地址与主机的 IP 地址匹配，并且你已将其复制到 API 服务器装载的目录中。

如果遵循[我们的说明](./creating-a-linux-master.md)，可以找到以下内容：   
* `~/kube/kubelet/`
* `$HOME/.kube/config`
*  `/etc/kubernetes/admin.conf`

 否则，请参阅 API 服务器的清单文件以检查装入点。
