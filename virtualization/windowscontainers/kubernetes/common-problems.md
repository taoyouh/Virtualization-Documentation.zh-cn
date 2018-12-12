---
title: Kubernetes 疑难解答
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: troubleshooting
ms.prod: containers
description: 关于部署 Kubernetes 和加入 Windows 节点的常见问题的解决方案。
keywords: kubernetes，1.12，linux，编译
ms.openlocfilehash: a5e9369b000aa83aa7ec6ec9bb147f0fd844c820
ms.sourcegitcommit: 8e9252856869135196fd054e3cb417562f851b51
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/08/2018
ms.locfileid: "6178910"
---
# <a name="troubleshooting-kubernetes"></a>Kubernetes 疑难解答 #
此页面逐一介绍 Kubernetes 设置、网络和部署的一些常见问题。

> [!tip]
> 通过向[我们的文档存储库](https://github.com/MicrosoftDocs/Virtualization-Documentation/)提出 PR 来建议常见问题解答项目。

此页面将细分为以下类别：
1. [一般问题](#general-questions)
2. [常见网络错误](#common-networking-errors)
3. [常见 Windows 错误](#common-windows-errors)
4. [常见的 Kubernetes 主错误](#common-kubernetes-master-errors)

## <a name="general-questions"></a>一般问题 ##

### <a name="how-do-i-know-startps1-on-windows-completed-successfully"></a>如何知道已成功完成的 Windows 上的 start.ps1？ ###
你应该会看到 kubelet，kube 代理 （如果你选择 Flannel 作为你的网络解决方案），并使用运行日志显示在你的节点上运行的 flanneld 主机代理进程分离 PoSh windows。 除此之外，你的 Windows 节点应列为"就绪"Kubernetes 群集中。

### <a name="can-i-configure-to-run-all-of-this-in-the-background-instead-of-posh-windows"></a>可以配置而不是 PoSh windows 在后台运行所有这些？ ###
从开始 Kubernetes 版本 1.11，kubelet 和 kube 代理可以作为运行本机[Windows 服务](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services)。 如[nssm.exe](https://nssm.cc/)替代服务管理器也始终可用于始终为你在后台运行这些进程 （flanneld、 kubelet 和 kube 代理）。


## <a name="common-networking-errors"></a>常见网络错误 ##

### <a name="my-windows-pods-do-not-have-network-connectivity"></a>我的 Windows pod 无法通过网络连接 ###
如果你使用的任何虚拟机，请确保所有虚拟机网络适配器上启用 MAC 欺骗。 请参阅[反欺骗保护](./getting-started-kubernetes-windows.md#disable-anti-spoofing-protection)更多详细信息。


### <a name="my-windows-pods-cannot-ping-external-resources"></a>我的 Windows pod 无法 ping 外部资源 ###
Windows pod 没有今天编程 ICMP 协议的出站规则。 但是，TCP/UDP 是受支持。 当试图演示连接到群集之外的资源，请替换`ping <IP>`具有相应`curl <IP>`命令。

如果你仍然面临的问题，最有可能你的网络配置中[cni.conf](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf)值得一些额外关注。 你始终可以编辑此静态文件，该配置将应用于任何新创建的 Kubernetes 资源。

为什么？
Kubernetes 网络要求之一 （请参阅[Kubernetes 模型](https://kubernetes.io/docs/concepts/cluster-administration/networking/)） 是群集通信，以在内部 NAT 不会发生。 若要服从此要求，我们必须为所有通信[ExceptionList](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf#L20)我们不希望发生的出站 NAT。 但是，这也意味着你需要将尝试从 ExceptionList 查询的外部 IP 中排除。 仅然后将来自你的 Windows pod 的流量进行 SNAT'ed 正确外界接收响应。 此问题，在你 ExceptionList`cni.conf`应该如下所示：
```
                "ExceptionList": [
                    "10.244.0.0/16",  # Cluster subnet
                    "10.96.0.0/12",   # Service subnet
                    "10.127.130.0/24" # Management (host) subnet
                ]
```

### <a name="after-launching-startps1-flanneld-is-stuck-in-waiting-for-the-network-to-be-created"></a>启动 start.ps1、 后 Flanneld 停滞在"等待要创建的网络" ###
有许多此问题报告它正在调查;很可能是计时问题时设置 flannel 网络管理 IP。 解决方法是只需重新启动 start.ps1 或，如下所示手动重新启动它：
```
PS C:> [Environment]::SetEnvironmentVariable("NODE_NAME", "<Windows_Worker_Hostname>")
PS C:> C:\flannel\flanneld.exe --kubeconfig-file=c:\k\config --iface=<Windows_Worker_Node_IP> --ip-masq=1 --kube-subnet-mgr=1
```

此外，还有[PR](https://github.com/coreos/flannel/pull/1042)当前解决此问题正在审查。

### <a name="my-windows-pods-cannot-launch-because-of-missing-runflannelsubnetenv"></a>我的 Windows pod 无法启动由于缺少 /run/flannel/subnet.env ###
这表示 Flannel 未正常启动。 你可以尝试重启 flanneld.exe 或者你可以将文件复制通过手动从`/run/flannel/subnet.env`在开始创建 Kubernetes 主机到`C:\run\flannel\subnet.env`Windows 工作者节点上和修改`FLANNEL_SUBNET`到不同数量的行。 例如，如果需要节点的子网 10.244.4.1/24:
```
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.4.1/24
FLANNEL_MTU=1500
FLANNEL_IPMASQ=true
```

### <a name="my-endpointsips-are-leaking"></a>我的终结点/Ip 泄漏 ###
存在 2 当前已知的问题可能会导致终结点泄漏。 
1.  第一个[已知问题](https://github.com/kubernetes/kubernetes/issues/68511)是 Kubernetes 版本 1.11 中的问题。 请避免使用 Kubernetes 版本 1.11.0-1.11.2。
2. 第二个[已知问题](https://github.com/docker/libnetwork/issues/1950)可能会导致终结点泄漏是并发问题终结点的存储区中。 若要接收该修补程序，你必须使用 Docker EE 18.09 或更高。

### <a name="my-pods-cannot-launch-due-to-network-failed-to-allocate-for-range-errors"></a>我的 pod 无法启动由于"网络： 范围分配失败"错误 ###
这表示你的节点上的 IP 地址空间用于。 若要清理[泄漏终结点](#my-endpointsips-are-leaking)，请迁移受影响的节点上的任何资源，并运行以下命令：
```
c:\k\stop.ps1
Get-HNSEndpoint | Remove-HNSEndpoint
Remove-Item -Recurse c:\var
```

### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>我的 Windows 节点无法使用服务 IP 访问我的服务 ###
这是 Windows 上当前网络堆栈的已知限制。 Windows *pod* **都**能够但是访问服务 IP。

### <a name="no-network-adapter-is-found-when-starting-kubelet"></a>启动 Kubelet 时未发现任何网络适配器 ###
Windows 网络堆栈需要虚拟适配器以使 Kubernetes 网络工作。 如果以下命令未返回任何结果（在管理员 shell 中），则表示虚拟网络创建 &mdash; Kubelet 工作的必要先决条件 &mdash; 已失败：

```powershell
Get-HnsNetwork | ? Name -ieq "cbr0"
Get-NetAdapter | ? Name -Like "vEthernet (Ethernet*"
```

请查询 `start-kubelet.ps1` 脚本的输出以查看虚拟网络创建期间是否发生错误。

### <a name="pods-stop-resolving-dns-queries-successfully-after-some-time-alive"></a>Pod 在处于活动状态一段时间后成功停止解析 DNS 查询 ###
已知的 DNS 缓存的 Windows Server 网络堆栈中的问题，版本 1803年及下面的有时可能会导致 DNS 请求失败。 若要解决此问题，你可以设置的最大的 TTL 缓存值为零使用以下注册表项：

```Dockerfile
FROM microsoft/windowsservercore:<your-build>
SHELL ["powershell', "-Command", "$ErrorActionPreference = 'Stop';"]
New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxCacheTtl -Value 0 -Type DWord 
New-ItemPropery -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxNegativeCacheTtl -Value 0 -Type DWord
```

### <a name="i-am-still-seeing-problems-what-should-i-do"></a>我仍然能看到问题。 应做什么？ ### 
你的网络或主机上可能设置了其他限制，以阻止在节点之间进行某些类型的通信。 请确保：
  - 你已正确配置你所选的[网络拓扑](./network-topologies.md)
  - 允许可能来自 Pod 的流量
  - 允许 HTTP 流量（如果要部署 Web 服务）
  - 来自不同协议 (ie ICMP 与 TCP/UDP) 数据包不会被丢弃


## <a name="common-windows-errors"></a>常见 Windows 错误 ##

### <a name="my-kubernetes-pods-are-stuck-at-containercreating"></a>我的 Kubernetes Pod 会停滞在“ContainerCreating” ###
很多原因可导致此问题，但最常见原因之一是暂停映像配置不正确。 这极有可能进一步导致其他问题。


### <a name="when-deploying-docker-containers-keep-restarting"></a>在进行部署时，Docker 容器不断重启 ###
检查暂停映像是否与你的操作系统版本兼容。 [说明](./deploying-resources.md)假定操作系统和容器都为版本 1803年。 如果你有更高版本的 Windows（如会员版本），将需要相应地调整映像。 请参阅 Microsoft 用于映像的 [Docker 存储库](https://hub.docker.com/u/microsoft/)。 无论如何，暂停映像 Dockerfile 和示例服务都会预期映像标记为 `:latest`。


## <a name="common-kubernetes-master-errors"></a>常见的 Kubernetes 主错误 ##
调试 Kubernetes 主机分为三个主要类别（按可能性顺序）：

  - Kubernetes 系统容器出现问题。
  - `kubelet` 的运行方式出现问题。
  - 系统出现问题。

运行 `kubectl get pods -n kube-system` 以查看 Kubernetes 正在创建的 Pod；这样可以深入了解哪些特定项将会崩溃或无法正常启动。 然后，运行 `docker ps -a` 以查看支持这些 Pod 的所有原始容器。 最后，针对怀疑会导致问题的容器运行 `docker logs [ID]`，以查看进程的原始输出。


### <a name="cannot-connect-to-the-api-server-at-httpsaddressport"></a>无法在 `https://[address]:[port]` 连接到 API 服务器 ###
通常，此错误表示证书问题。 请确保你已经正确生成了配置文件，其中的 IP 地址与主机的 IP 地址匹配，并且你已将其复制到 API 服务器装载的目录中。

如果遵循[我们的说明](./creating-a-linux-master.md)，良好的位置发现这是：   
* `~/kube/kubelet/`
* `$HOME/.kube/config`
*  `/etc/kubernetes/admin.conf`

 否则，请参阅 API 服务器的清单文件，以检查装入点。