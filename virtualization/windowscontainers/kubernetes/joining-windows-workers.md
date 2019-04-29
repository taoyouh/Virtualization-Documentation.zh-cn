---
title: 加入 Windows 节点
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: 将 Windows 节点加入到 Kubernetes 群集与 v1.13。
keywords: kubernetes，1.13，windows，入门
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: ed0f13bd429e88f05469f91c3fc691bf0188b0a2
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/26/2019
ms.locfileid: "9578238"
---
# <a name="joining-windows-server-nodes-to-a-cluster"></a>Windows Server 节点加入群集 #
[设置 Kubernetes 主节点](./creating-a-linux-master.md)并[选择所需的网络解决方案](./network-topologies.md)后，你就可以加入 Windows Server 节点群集。 这在加入之前需要某些[准备 Windows 节点上](#preparing-a-windows-node)。

## <a name="preparing-a-windows-node"></a>准备 Windows 节点 ##
> [!NOTE]  
> Windows 部分中的所有代码段都将在_提升的_ PowerShell 中运行。

### <a name="install-docker-requires-reboot"></a>安装的 Docker （需要重启） ###
Kubernetes 使用[Docker](https://www.docker.com/)作为其容器引擎，，因此我们需要安装它。 你可以按照[官方文档说明](../manage-docker/configure-docker-daemon.md#install-docker)、[Docker 说明](https://store.docker.com/editions/enterprise/docker-ee-server-windows)进行操作，或者尝试以下步骤：

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

如果重启后，你将看到以下错误：

![文本](media/docker-svc-error.png)

然后手动启动 docker 服务：

```powershell
Start-Service docker
```

### <a name="create-the-pause-infrastructure-image"></a>创建"暂停"（基础结构） 映像 ###
> [!Important]
> 请务必小心容器映像冲突;没有预期的标记可能会导致`docker pull`不兼容容器映像，从而导致[部署问题](./common-problems.md#when-deploying-docker-containers-keep-restarting)如无限`ContainerCreating`状态。

现在，`docker` 已安装，你需要准备“暂停”映像，以供 Kubernetes 准备基础结构 Pod。 有以下三个步骤： 
  1. [拉取映像](#pull-the-image)
  2. 作为 microsoft[标记它](#tag-the-image)/ nanoserver:latest
  3. [运行](#run-the-container)


#### <a name="pull-the-image"></a>拉取映像 ####     
 拉取映像针对特定的 Windows 版本。 例如，如果你运行的 Windows Server 2019:

 ```powershell
docker pull mcr.microsoft.com/windows/nanoserver:1809
 ```

#### <a name="tag-the-image"></a>标记图像 ####
将稍后在本指南中使用 Dockerfile 查找`:latest`标记的图像。 标记 nanoserver 图像你只需提取，如下所示：

```powershell
docker tag mcr.microsoft.com/windows/nanoserver:1809 microsoft/nanoserver:latest
```

#### <a name="run-the-container"></a>运行该容器 ####
仔细检查容器实际上计算机上运行：

```powershell
docker run microsoft/nanoserver:latest
```

你应该会看到如下所示：

![文本](./media/docker-run-sample.png)

> [!tip]
> 如果你不能运行该容器请请参阅：[匹配容器主机版本与容器映像](https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility#matching-container-host-version-with-container-image-versions)


#### <a name="prepare-kubernetes-for-windows-directory"></a>准备 Kubernetes Windows 目录 ####
创建存储 Kubernetes 二进制文件的"Kubernetes Windows"目录以及任何部署脚本和配置文件。

```powershell
mkdir c:\k
```

#### <a name="copy-kubernetes-certificate"></a>将 Kubernetes 证书复制 #### 
将 Kubernetes 证书文件复制 (`$HOME/.kube/config`)[从主机](./creating-a-linux-master.md#collect-cluster-information)到此新`C:\k`目录。

> [!tip]
> 你可以使用[xcopy](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/xcopy)或[WinSCP](https://winscp.net/eng/download.php)等工具节点之间传输的配置文件。

#### <a name="download-kubernetes-binaries"></a>下载 Kubernetes 二进制文件 ####
若要能够运行 Kubernetes，首先需要下载`kubectl`， `kubelet`，并`kube-proxy`二进制文件。 你可以下载这些中的链接`CHANGELOG.md`文件的[最新版本](https://github.com/kubernetes/kubernetes/releases/)。
 - 例如，下面是[v1.13 节点二进制文件](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.13.md#node-binaries)。
 - 使用[展开存档](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.archive/expand-archive?view=powershell-6)等工具解压缩存档并将放置到的二进制文件`C:\k\`。

#### <a name="optional-setup-kubectl-on-windows"></a>（可选）在 Windows 上的安装程序 kubectl ####
你想要控制从 Windows 群集应，你可以使用完成`kubectl`命令。 首先，以使`kubectl`可用外`C:\k\`目录中，修改`PATH`环境变量：

```powershell
$env:Path += ";C:\k"
```

若要使此更改永久有效，请在计算机目标中修改变量：

```powershell
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\k", [EnvironmentVariableTarget]::Machine)
```

接下来，我们将验证[群集证书](#copy-kubernetes-certificate)有效。 若要设置的位置，`kubectl`寻找配置文件，你可以传递`--kubeconfig`参数或修改`KUBECONFIG`环境变量。 例如，如果该配置位于 `C:\k\config`：

```powershell
$env:KUBECONFIG="C:\k\config"
```

为让此设置对于当前用户范围永久有效：

```powershell
[Environment]::SetEnvironmentVariable("KUBECONFIG", "C:\k\config", [EnvironmentVariableTarget]::User)
```

最后，若要查看配置是否被正确发现，你可以使用：

```powershell
kubectl config view
```

如果你收到连接错误，

```
Unable to connect to the server: dial tcp [::1]:8080: connectex: No connection could be made because the target machine actively refused it.
```

你应该仔细检查 kubeconfig 位置或尝试将其复制重新。

如果你不看到任何错误现已准备好加入群集节点。

## <a name="joining-the-windows-node"></a>加入 Windows 节点 ##
具体取决于[所选的网络解决方案](./network-topologies.md)，你可以：
1. [将 Windows Server 节点加入到 Flannel （vxlan 或主机网关） 群集](#joining-a-flannel-cluster)
2. [将 Windows Server 节点加入到具有 ToR 开关的群集](#joining-a-tor-cluster)

### <a name="joining-a-flannel-cluster"></a>加入 Flannel 群集 ###
没有[此 Microsoft 存储库](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/overlay)，可帮助你将此节点加入到群集中的 Flannel 部署脚本的集合。

下载[Flannel start.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1)脚本，其中的内容应将其解压缩到`C:\k`:

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/start.ps1 -o c:\k\start.ps1
```

假设你[准备 Windows 节点](#preparing-a-windows-node)，以及`c:\k`目录的外观所示，你可以随时将节点加入。

![文本](./media/flannel-directory.png)

#### <a name="join-node"></a>将节点加入 #### 
若要简化加入 Windows 节点的过程，只需运行一个 Windows 脚本来启动`kubelet`， `kube-proxy`， `flanneld`，并加入节点。

> [!Note]
> [start.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1)引用[install.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/install.ps1)，如下载其他文件的`flanneld`可执行文件和[基础结构 pod 的 Dockerfile](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/Dockerfile) *和安装这些为你*。 对于覆盖网络模式，将为本地 UDP 端口 4789 打开[防火墙](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/helper.psm1#L111)。 可能有多个 powershell 窗口中正在以及几秒钟的网络中断而打开/关闭第一次创建新的外部 vSwitch pod 网络的过程。

```powershell
cd c:\k
.\start.ps1 -ManagementIP <Windows Node IP> -NetworkMode <network mode>  -ClusterCIDR <Cluster CIDR> -ServiceCIDR <Service CIDR> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Log directory>
```
# [<a name="managementip"></a>ManagementIP](#tab/ManagementIP)
分配给 Windows 节点的 IP 地址。 你可以使用`ipconfig`查找此。

|  |  | 
|---------|---------|
|参数     | `-ManagementIP`        |
|默认值    | n.A. **required**        |

# [<a name="networkmode"></a>NetworkMode](#tab/NetworkMode)
网络模式`l2bridge`(flannel 主机网关) 或`overlay`(flannel vxlan) 选择作为[网络解决方案](./network-topologies.md)。

> [!Important] 
> `overlay` 网络模式 (flannel vxlan) 需要 Kubernetes v1.14 二进制文件或以上。

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
[Kubernetes DNS 服务 IP](./getting-started-kubernetes-windows.md#kube-dns-def)。

|  |  | 
|---------|---------|
|参数     | `-KubeDnsServiceIP`        |
|默认值    | `10.96.0.10`        |


# [<a name="interfacename"></a>InterfaceName](#tab/InterfaceName)
Windows 主机的网络接口的名称。 你可以使用`ipconfig`查找此。

|  |  | 
|---------|---------|
|参数     | `-InterfaceName`        |
|默认值    | `Ethernet`        |


# [<a name="logdir"></a>LogDir](#tab/LogDir)
在其中 kubelet 和 kube 代理日志将重定向到其各自的输出文件的目录。

|  |  | 
|---------|---------|
|参数     | `-LogDir`        |
|默认值    | `C:\k`        |


---

> [!tip]
> 你已注意到的下群集子网和服务子网，并在 Linux 主机 KUBE-DNS IP[更早版本](./creating-a-linux-master.md#collect-cluster-information)

运行此之后，你应该能够：
  * 查看已加入的 Windows 节点上使用 `kubectl get nodes`
  * 请参阅 3 个 powershell 窗口中打开，一个用于`kubelet`、 一个用于`flanneld`，另一个用于 `kube-proxy`
  * 主机代理的过程，请参阅`flanneld`， `kubelet`，并`kube-proxy`的节点上运行

如果成功，继续执行[下一步骤](#next-steps)。

## <a name="joining-a-tor-cluster"></a>加入 ToR 群集 ##
> [!NOTE]
> 如果你选择 Flannel 作为你网络解决方案[之前](./network-topologies.md#flannel-in-host-gateway-mode)，你可以跳过此部分。

若要执行此操作，你需要按照[上游 L3 路由拓扑在 Kubernetes 上的 Windows Server 容器设置](https://kubernetes.io/docs/getting-started-guides/windows/#for-1-upstream-l3-routing-topology-and-2-host-gateway-topology)的说明操作。 这包括确保配置上游路由器，例如，荚 CIDR 前缀分配到一个节点映射到其各自节点 IP。

假设新节点通过列为"就绪" `kubectl get nodes`，kubelet + kube 代理正在运行，且你已配置上游 ToR 路由器，你可以随时后续步骤。

## <a name="next-steps"></a>后续步骤 ##
在此部分中，我们将介绍如何将 Windows 工作者加入到我们 Kubernetes 群集。 现在，你可以随时执行第 5 步：

> [!div class="nextstepaction"]
> [加入 Linux 工作人员](./joining-linux-workers.md)

或者，如果你没有任何 Linux 工作人员可随意跳到步骤 6:

> [!div class="nextstepaction"]
> [部署 Kubernetes 资源](./deploying-resources.md)
