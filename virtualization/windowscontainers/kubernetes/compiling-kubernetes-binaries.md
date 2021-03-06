---
title: 编译 Kubernetes 二进制文件
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: how-to
ms.prod: containers
description: 编译和交叉编译源中的 Kubernetes 二进制文件。
keywords: kubernetes，1.12，linux，编译
ms.openlocfilehash: a0c9ed6ef0872e19de49fa97f4727b6e0e09ed43
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192344"
---
# <a name="compiling-kubernetes-binaries"></a>编译 Kubernetes 二进制文件 #
编译 Kubernetes 需要有效的 Go 环境。 此页面将逐一介绍编译 Linux 二进制文件和交叉编译 Windows 二进制文件的一些方法。
> [!NOTE]
> 此页面完全是自愿的，仅包含在想要试验最新 & 最新源代码的相关 Kubernetes 开发人员。

> [!tip]
> 接收有关可订阅的最新开发的通知 [@kubernetes-announce](https://groups.google.com/forum/#!forum/kubernetes-announce) 。

## <a name="installing-go"></a>安装 Go ##
为简单起见，Go 将安装在临时的自定义位置：

```bash
cd ~
wget https://redirector.gvt1.com/edgedl/go/go1.11.1.linux-amd64.tar.gz -O go1.11.1.tar.gz
tar -vxzf go1.11.1.tar.gz
mkdir gopath
export GOROOT="$HOME/go"
export GOPATH="$HOME/gopath"
export PATH="$GOROOT/bin:$PATH"
```

> [!Note]
> 这将设置你的会话环境变量。 将 `export` 添加到你的 `~/.profile` 中以进行永久设置。

运行 `go env` 以确保已正确设置路径。 以下是一些用于生成 Kubernetes 二进制文件的选项：

  - [本地](#build-locally)生成。
  - 使用 [Vagrant](#build-with-vagrant) 生成二进制文件。
  - 在 Kubernetes 项目中利用[标准容器化生成脚本](https://github.com/kubernetes/kubernetes/tree/master/build#key-scripts)。 为此，请执行[本地生成](#build-locally)步骤，一直执行到 `make` 步骤，然后使用链接的说明进行操作。

若要将 Windows 二进制文件复制到它们各自的节点，请使用类似于 [WinSCP](https://winscp.net/eng/download.php) 的可视化工具或类似于 [pscp](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) 的命令行工具将它们传输到 `C:\k` 目录。


## <a name="building-locally"></a>本地生成 ##
> [!Tip]
> 如果遇到 "权限被拒绝" 错误，可以根据 `kubelet` 中的说明，通过构建 Linux 来避免这些错误 [`acs-engine`](https://github.com/Azure/acs-engine/blob/master/scripts/build-windows-k8s.sh#L176) ：
>
> _由于似乎是 Kubernetes Windows 生成系统中的 bug，因此必须先生成 Linux 二进制文件才能生成 `_output/bin/deepcopy-gen` 。如果生成到 Windows，则执行此操作将生成一个空的 `deepcopy-gen` 。_

首先，检索 Kubernetes 存储库：

```bash
KUBEREPO="k8s.io/kubernetes"
go get -d $KUBEREPO
# Note: the above command may spit out a message about
#       "no Go files in...", but it can be safely ignored!
cd $GOPATH/src/$KUBEREPO
```

现在，检查要从中进行生成的分支并生成 Linux `kubelet` 二进制文件。 若要避免上述 Windows 生成错误，则必须执行此步骤。 在此，我们将使用 `v1.12.2`。 在 `git checkout` 后，你可以应用待处理的 PR、修补程序或对自定义二进制文件进行其他修改。

```bash
git checkout tags/v1.12.2
make clean && make WHAT=cmd/kubelet
```

最后，生成必需的 Windows 客户端二进制文件（最后一步可能会有所不同，具体取决于以后应从哪里检索 Windows 二进制文件）：

```bash
KUBE_BUILD_PLATFORMS=windows/amd64 make WHAT=cmd/kubectl
KUBE_BUILD_PLATFORMS=windows/amd64 make WHAT=cmd/kubelet
KUBE_BUILD_PLATFORMS=windows/amd64 make WHAT=cmd/kube-proxy
cp _output/local/bin/windows/amd64/kube*.exe ~/kube-win/
```

生成 Linux 二进制文件的步骤完全相同；只需去掉命令的 `KUBE_BUILD_PLATFORMS=windows/amd64` 前缀。 输出目录将改为 `_output/.../linux/amd64`。


## <a name="build-with-vagrant"></a>使用 Vagrant 构建 ##
在[此处](https://github.com/Microsoft/SDN/tree/master/Kubernetes/linux/vagrant)可以获得 Vagrant 安装文件。 使用它来准备 Vagrant VM，然后在其中执行以下命令：

```bash
DIST_DIR="${HOME}/kube/"
SRC_DIR="${HOME}/src/k8s-main/"
mkdir ${DIST_DIR}
mkdir -p "${SRC_DIR}"

git clone https://github.com/kubernetes/kubernetes.git ${SRC_DIR}

cd ${SRC_DIR}
git checkout tags/v1.12.2
KUBE_BUILD_PLATFORMS=linux/amd64   build/run.sh make WHAT=cmd/kubelet
KUBE_BUILD_PLATFORMS=windows/amd64 build/run.sh make WHAT=cmd/kubelet
KUBE_BUILD_PLATFORMS=windows/amd64 build/run.sh make WHAT=cmd/kube-proxy
cp _output/dockerized/bin/windows/amd64/kube*.exe ${DIST_DIR}

ls ${DIST_DIR}
```

