---
title: "从头开始创建 Kubernetes 主机"
author: gkudra-msft
ms.author: gekudray
ms.date: 11/16/2017
ms.topic: get-started-article
ms.prod: containers
description: "从头开始创建 Kubernetes 群集主机。"
keywords: "kubernetes, 1.9, 主机, linux"
ms.openlocfilehash: d5251b1a2dc06bef396820e324fb240eed04acc8
ms.sourcegitcommit: b0e21468f880a902df63ea6bc589dfcff1530d6e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/17/2018
---
# <a name="kubernetes-master--from-scratch"></a>从头开始创建 Kubernetes 主机 #
此页面将从头到尾逐步介绍 Kubernetes 主机的手动部署。

若要按步骤操作，需要具有一台最近更新的类似于 Ubuntu 的 Linux 计算机。 这与 Windows 根本没有关系；二进制文件通过 Linux 进行交叉编译。


> [!Warning]  
> 由于 Kubernetes 版本之间的易变性，本指南做出的假设将来可能并不会实现。


## <a name="preparing-the-master"></a>准备主机 ##
首先，安装所有必备项：

```bash
sudo apt-get install curl git build-essential docker.io conntrack
```


[本存储库](https://github.com/Microsoft/SDN/tree/master/Kubernetes/linux)中有一个脚本集合，这些脚本有助于完成设置过程。 请将它们签出到 `~/kube/`；在将来的步骤中，将会为很多 Docker 容器装载此整个目录，所以请使其结构与指南中概述的结构保持相同。

```bash
mkdir ~/kube
mkdir ~/kube/bin
git clone https://github.com/Microsoft/SDN /tmp/k8s 
cd /tmp/k8s/Kubernetes/linux
chmod -R +x *.sh
chmod +x manifest/generate.py
mv * ~/kube/
```


### <a name="installing-the-linux-binaries"></a>安装 Linux 二进制文件 ###

> [!Note]  
> 若要包括修补程序或者使用非常先进的 Kubernetes 代码，而不是下载预生成的二进制文件，请参阅[本页面](./compiling-kubernetes-binaries.md)。

从 [Kubernetes 主线](https://github.com/kubernetes/kubernetes/releases/tag/v1.9.1)下载并安装官方 Linux 二进制文件，安装如下所示：

```bash
wget -O kubernetes.tar.gz https://github.com/kubernetes/kubernetes/releases/download/v1.9.1/kubernetes.tar.gz
tar -vxzf kubernetes.tar.gz 
cd kubernetes/cluster 
# follow the prompts from this command, the defaults are generally fine:
./get-kube-binaries.sh
cd ../server
tar -vxzf kubernetes-server-linux-amd64.tar.gz 
cd kubernetes/server/bin
cp hyperkube kubectl ~/kube/bin/
```

将二进制文件添加到 `$PATH` 中，以便从任何位置运行它们。 请注意，这只针对会话设置路径；请将此行添加到 `~/.profile` 中以进行永久设置。

```bash
$ PATH="$HOME/kube/bin:$PATH"
```

### <a name="install-cni-plugins"></a>安装 CNI 插件 ###
Kubernetes 网络需要基本的 CNI 插件。 可以从[此处](https://github.com/containernetworking/plugins/releases)下载这些插件，并且应将其解压缩到 `/opt/cni/bin/`：

```bash
DOWNLOAD_DIR="${HOME}/kube/cni-plugins"
CNI_BIN="/opt/cni/bin/"
mkdir ${DOWNLOAD_DIR}
cd $DOWNLOAD_DIR
curl -L $(curl -s https://api.github.com/repos/containernetworking/plugins/releases/latest | grep browser_download_url | grep 'amd64.*tgz' | head -n 1 | cut -d '"' -f 4) -o cni-plugins-amd64.tgz
tar -xvzf cni-plugins-amd64.tgz
sudo mkdir -p ${CNI_BIN}
sudo cp -r !(*.tgz) ${CNI_BIN}
ls ${CNI_BIN}
```


### <a name="certificates"></a>证书 ###
首先，通过 `ifconfig` 或以下内容获取本地 IP 地址：

```bash
$ ip addr show dev eth0
```

前提是接口名称已知。 整个过程中会多次引用该地址；将其设置为环境变量可以简化该过程。 以下代码段会暂时对其进行设置；如果会话结束或 Shell 关闭，则需要重新设置。

```bash
$ MASTER_IP=10.123.45.67   # example! replace
```

准备节点在群集中通信将使用的证书：

```bash
cd ~/kube/certs
./generate-certs.sh $MASTER_IP
```

### <a name="prepare-manifests--addons"></a>准备清单和加载项 ###
将主 IP 地址和*完整*群集 CIDR 传递到 `manifest` 文件夹中的 Python 脚本，以生成一组指定 Kubernetes 系统 Pod 的 YAML 文件：

```bash
cd ~/kube/manifest
./generate.py $MASTER_IP --cluster-cidr 192.168.0.0/16
```

（删除）移动 Python 脚本，以防 Kubernetes 将它误当作清单；如果未执行此操作，稍后将会导致问题。

> [!Important]  
> 如果 Kubernetes 版本与本指南中的版本不同，请在脚本上使用各种版本控制标记（例如 `--api-version`）以[自定义 Pod 部署的映像](https://console.cloud.google.com/gcr/images/google-containers/GLOBAL/hyperkube-amd64)。 并非所有清单都使用相同的映像且具有不同的版本控制架构（特别是 `etcd` 和加载项管理器）。


#### <a name="manifest-customization"></a>清单自定义 ####
此时，可能需要进行特定于设置的更改。 例如，可能需要将子网手动分配给节点，而不是让 Kubernetes 自动进行管理。 此特定配置在脚本中具有一个选项（有关 `--im-sure` 参数的说明，请参阅 `--help`）：

```bash
./generate.py $MASTER_IP --im-sure
```

任何其他自定义配置选项都将需要手动修改生成的清单。


### <a name="configure--run-kubernetes"></a>配置和运行 Kubernetes ###
将 Kubernetes 配置为使用生成的证书。 这将在 `~/.kube/config` 中创建配置：

```bash
./configure-kubectl.sh $MASTER_IP
```

现在，将此文件复制到 Pod 以后所需的文件位置中：

```bash
mkdir ~/kube/kubelet
sudo cp ~/.kube/config ~/kube/kubelet/
```

Kubernetes“客户端”`kubelet` 可以启动了。 以下脚本都将无限期运行；在每个终端会话之后打开另一个终端会话以继续工作：

```bash
cd ~/kube
sudo ./start-kubelet.sh
```

运行 Kubeproxy 脚本，并传递部分群集 CIDR：

```bash
cd ~/kube
sudo ./start-kubeproxy.sh 192.168
```


> [!Important]  
> 这将是节点将来所在的预期*完整* /16 CIDR，*即使该 CIDR 上存在非 Kubernetes 流量也不例外。* Kubeproxy *仅*适用于到*服务*子网的 Kubernetes 流量，因此它不会干扰其他主机的流量。

> [!Note]  
> 这些脚本可以转入后台作为守护程序运行。 本指南仅介绍如何手动运行脚本，因为这在设置过程中可以最有效地捕获错误。


## <a name="verifying-the-master"></a>验证主机 ##
几分钟后，系统应处于以下状态：

  - 在 `docker ps` 下面将有 ~23 个工作线程和 Pod 容器。
  - 如果调用 `kubectl cluster-info`，则将显示有关 Kubernetes 主 API 服务器以及 DNS 和 Heapster 加载项的信息。
  - `ifconfig` 将显示一个新接口 `cbr0` 与所选择的群集 CIDR。

