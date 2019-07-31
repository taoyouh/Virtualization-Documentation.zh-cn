---
title: 加入 Windows 节点
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: 将 Windows 节点加入带有 v 1.14 的 Kubernetes 群集。
keywords: kubernetes、1.14、windows、入门
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: c9dbfec968d52d9fbc528892f0e3749270e3ff70
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/31/2019
ms.locfileid: "9882980"
---
# <a name="joining-windows-server-nodes-to-a-cluster"></a>将 Windows Server 节点加入群集 #
[设置 Kubernetes 主节点](./creating-a-linux-master.md)并[选择所需的网络解决方案](./network-topologies.md)后, 即可加入 Windows Server 节点以形成群集。 这需要先[准备一些 Windows 节点,](#preparing-a-windows-node)然后才能加入。

## <a name="preparing-a-windows-node"></a>准备 Windows 节点 ##
> [!NOTE]  
> Windows 部分中的所有代码段都将在_提升的_ PowerShell 中运行。

### <a name="install-docker-requires-reboot"></a>安装 Docker (需要重启) ###
Kubernetes 使用[Docker](https://www.docker.com/)作为其容器引擎, 因此我们需要安装它。 你可以按照[官方文档说明](../manage-docker/configure-docker-daemon.md#install-docker)、[Docker 说明](https://store.docker.com/editions/enterprise/docker-ee-server-windows)进行操作，或者尝试以下步骤：

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
Install-Package -Name Docker -ProviderName DockerMsftProvider
Restart-Computer -Force
```

如果你在代理的后面，则必须定义以下 PowerShell 环境变量：
```powershell
[Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://proxy.example.com:80/", [EnvironmentVariableTarget]::Machine)
[Environment]::SetEnvironmentVariable("HTTPS_PROXY", "http://proxy.example.com:443/", [EnvironmentVariableTarget]::Machine)
```

如果重新启动后, 您会看到以下错误:

![文本](media/docker-svc-error.png)

然后手动启动 docker 服务:

```powershell
Start-Service docker
```

### <a name="create-the-pause-infrastructure-image"></a>创建 "暂停" (基础结构) 映像 ###
> [!Important]
> 如果容器映像冲突, 请务必小心;没有所需的标记可能会导致`docker pull`不兼容的容器图像, 从而导致诸如未确定的`ContainerCreating`状态之类的[部署问题](./common-problems.md#when-deploying-docker-containers-keep-restarting)。

现在，`docker` 已安装，你需要准备“暂停”映像，以供 Kubernetes 准备基础结构 Pod。 有三个步骤: 
  1. [提取图像](#pull-the-image)
  2. [将其标记](#tag-the-image)为 microsoft/nanoserver: 最新
  3. 并[运行它](#run-the-container)


#### <a name="pull-the-image"></a>提取图像 ####     
 获取特定 Windows 版本的图像。 例如, 如果您运行的是 Windows Server 2019:

 ```powershell
docker pull mcr.microsoft.com/windows/nanoserver:1809
 ```

#### <a name="tag-the-image"></a>标记图像 ####
本指南稍后将使用的 Dockerfiles 查找`:latest`图像标记。 标记你刚刚提取的 nanoserver 图像, 如下所示:

```powershell
docker tag mcr.microsoft.com/windows/nanoserver:1809 microsoft/nanoserver:latest
```

#### <a name="run-the-container"></a>运行容器 ####
仔细检查容器实际在您的计算机上运行:

```powershell
docker run microsoft/nanoserver:latest
```

你应看到如下所示的内容:

![文本](./media/docker-run-sample.png)

> [!tip]
> 如果无法运行容器, 请参阅:[用容器映像匹配容器主机版本](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#matching-container-host-version-with-container-image-versions)


#### <a name="prepare-kubernetes-for-windows-directory"></a>为 Windows 目录准备 Kubernetes ####
创建 "Kubernetes for Windows" 目录以存储 Kubernetes 二进制文件以及任何部署脚本和配置文件。

```powershell
mkdir c:\k
```

#### <a name="copy-kubernetes-certificate"></a>复制 Kubernetes 证书 #### 
将 Kubernetes 证书文件 (`$HOME/.kube/config`)[从 master](./creating-a-linux-master.md#collect-cluster-information)复制到此新`C:\k`目录。

> [!tip]
> 你可以使用[xcopy](https://docs.microsoft.com/windows-server/administration/windows-commands/xcopy)或[WinSCP](https://winscp.net/eng/download.php)之类的工具在节点之间传输配置文件。

#### <a name="download-kubernetes-binaries"></a>下载 Kubernetes 二进制文件 ####
若要能够运行 Kubernetes, 首先需要下载`kubectl`、 `kubelet`和`kube-proxy`二进制文件。 你可以从`CHANGELOG.md` [最新版本](https://github.com/kubernetes/kubernetes/releases/)的文件中的链接下载这些内容。
 - 例如, 下面是[v 1.14 节点二进制文件](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.14.md#node-binaries)。
 - 使用 "[展开-存档](https://docs.microsoft.com/powershell/module/microsoft.powershell.archive/expand-archive?view=powershell-6)" 之类的工具解压缩存档并将二进制文件放`C:\k\`入其中。

#### <a name="optional-setup-kubectl-on-windows"></a>可选在 Windows 上设置 kubectl ####
如果您希望从 Windows 控制群集, 则可以使用`kubectl`命令执行此操作。 首先, 要使`kubectl`其在`C:\k\`目录外可用, 请修改`PATH`环境变量:

```powershell
$env:Path += ";C:\k"
```

若要使此更改永久有效，请在计算机目标中修改变量：

```powershell
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\k", [EnvironmentVariableTarget]::Machine)
```

接下来, 我们将验证[群集证书](#copy-kubernetes-certificate)是否有效。 若要设置`kubectl`查找配置文件的位置, 可以传递`--kubeconfig`参数或修改`KUBECONFIG`环境变量。 例如，如果该配置位于 `C:\k\config`：

```powershell
$env:KUBECONFIG="C:\k\config"
```

为让此设置对于当前用户范围永久有效：

```powershell
[Environment]::SetEnvironmentVariable("KUBECONFIG", "C:\k\config", [EnvironmentVariableTarget]::User)
```

最后, 若要检查是否已正确发现配置, 您可以使用:

```powershell
kubectl config view
```

如果你收到连接错误，

```
Unable to connect to the server: dial tcp [::1]:8080: connectex: No connection could be made because the target machine actively refused it.
```

你应该仔细检查 kubeconfig 位置或尝试再次复制它。

如果未看到任何错误, 则节点现在已准备好加入群集。

## <a name="joining-the-windows-node"></a>加入 Windows 节点 ##
根据[所选网络解决方案](./network-topologies.md), 您可以:
1. [将 Windows Server 节点加入 Flannel (vxlan 或 host-gw) 群集](#joining-a-flannel-cluster)
2. [使用 ToR 开关将 Windows Server 节点加入群集](#joining-a-tor-cluster)

### <a name="joining-a-flannel-cluster"></a>加入 Flannel 群集 ###
[此 Microsoft 存储库](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/overlay)上有一个 Flannel 部署脚本集合, 可帮助你将此节点加入到群集。

下载[Flannel start. ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1)脚本, 应提取到`C:\k`的内容:

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/start.ps1 -o c:\k\start.ps1
```

假设你已[准备好 Windows 节点](#preparing-a-windows-node), 并且`c:\k`你的目录如下所示, 你已准备好加入该节点。

![文本](./media/flannel-directory.png)

#### <a name="join-node"></a>加入节点 #### 
若要简化加入 Windows 节点的过程, 只需运行一个 Windows 脚本即可启动`kubelet`、 `kube-proxy`、 `flanneld`和加入该节点。

> [!Note]
> [启动. ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1)引用[install](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/install.ps1), 它将下载其他文件, 例如`flanneld`可执行文件和[Dockerfile for 基础结构箱](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/Dockerfile),*并为你安装*。 对于覆盖网络模式, 将打开本地 UDP 端口4789的[防火墙](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/helper.psm1#L111)。 在首次创建新的 pod 网络的外部 vSwitch 时, 可能会打开/关闭多个 powershell 窗口以及网络中断时间。

```powershell
cd c:\k
.\start.ps1 -ManagementIP <Windows Node IP> -NetworkMode <network mode>  -ClusterCIDR <Cluster CIDR> -ServiceCIDR <Service CIDR> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Log directory>
```
# [<a name="managementip"></a>ManagementIP](#tab/ManagementIP)
分配给 Windows 节点的 IP 地址。 可以使用`ipconfig`查找此文件。

|  |  | 
|---------|---------|
|参数     | `-ManagementIP`        |
|默认值    | n.A. **required**        |

# [<a name="networkmode"></a>NetworkMode](#tab/NetworkMode)
网络模式`l2bridge` (flannel host-gw) 或`overlay` (flannel vxlan) 选择为[网络解决方案](./network-topologies.md)。

> [!Important] 
> `overlay` 网络模式 (flannel vxlan) 需要 Kubernetes v 1.14 二进制文件 (或更高版本) 和[KB4489899](https://support.microsoft.com/help/4489899)。

|  |  | 
|---------|---------|
|参数     | `-NetworkMode`        |
|默认值    | `l2bridge`        |


# [<a name="clustercidr"></a>ClusterCIDR](#tab/ClusterCIDR)
[群集子网范围](./getting-started-kubernetes-windows.md#cluster-subnet-def)。

|  |  | 
|---------|---------|
|参数     | `-ClusterCIDR`        |
|默认值    | `10.244.0.0/16`        |


# [<a name="servicecidr"></a>ServiceCIDR](#tab/ServiceCIDR)
[服务子网范围](./getting-started-kubernetes-windows.md#service-subnet-def)。

|  |  | 
|---------|---------|
|参数     | `-ServiceCIDR`        |
|默认值    | `10.96.0.0/12`        |


# [<a name="kubednsserviceip"></a>KubeDnsServiceIP](#tab/KubeDnsServiceIP)
[KUBERNETES DNS 服务 IP](./getting-started-kubernetes-windows.md#plan-ip-addressing-for-your-cluster)。

|  |  | 
|---------|---------|
|参数     | `-KubeDnsServiceIP`        |
|默认值    | `10.96.0.10`        |


# [<a name="interfacename"></a>InterfaceName](#tab/InterfaceName)
Windows 主机的网络接口的名称。 可以使用`ipconfig`查找此文件。

|  |  | 
|---------|---------|
|参数     | `-InterfaceName`        |
|默认值    | `Ethernet`        |


# [<a name="logdir"></a>LogDir](#tab/LogDir)
Kubelet 和 kube-代理日志在其各自的输出文件中被重定向到其各自的输出文件的目录。

|  |  | 
|---------|---------|
|参数     | `-LogDir`        |
|默认值    | `C:\k`        |


---

> [!tip]
> 您已记下群集子网、服务子网和 kube-[以前](./creating-a-linux-master.md#collect-cluster-information)Linux 主服务器中的 DNS IP

运行后, 你应该能够:
  * 使用查看加入的 Windows 节点 `kubectl get nodes`
  * 请参阅打开3个 powershell 窗口, `kubelet`一个用于, `flanneld`一个用于, 另一个用于 `kube-proxy`
  * 查看`flanneld`、 `kubelet`和`kube-proxy`运行的节点上的主机代理进程

如果成功, 请继续执行[后续步骤](#next-steps)。

## <a name="joining-a-tor-cluster"></a>加入 ToR 群集 ##
> [!NOTE]
> 如果您[以前](./network-topologies.md#flannel-in-host-gateway-mode)选择 "Flannel" 作为网络解决方案, 则可以跳过此部分。

若要执行此操作, 您需要按照有关[在 Kubernetes 上为上游 L3 路由拓扑设置 Windows Server 容器](https://kubernetes.io/docs/getting-started-guides/windows/#for-1-upstream-l3-routing-topology-and-2-host-gateway-topology)的说明进行操作。 这包括确保配置上游路由器, 以使分配给节点的 pod CIDR 前缀映射到各自的节点 IP。

假设新节点按`kubectl get nodes`"就绪" 列出, kubelet + kube 代理正在运行, 并且您已配置了上游 ToR 路由器, 您已准备好执行后续步骤。

## <a name="next-steps"></a>后续步骤 ##
在本部分中, 我们介绍了如何将 Windows 工作者加入我们的 Kubernetes 群集。 现在, 你已准备好执行第5步:

> [!div class="nextstepaction"]
> [加入 Linux 工作人员](./joining-linux-workers.md)

或者, 如果您没有任何 Linux 工作人员可以直接跳到步骤 6, 请执行以下操作:

> [!div class="nextstepaction"]
> [部署 Kubernetes 资源](./deploying-resources.md)
