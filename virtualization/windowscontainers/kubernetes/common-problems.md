---
title: "Kubernetes 疑难解答"
author: gkudra-msft
ms.author: gekudray
ms.date: 11/16/2017
ms.topic: troubleshooting
ms.prod: containers
description: "关于部署 Kubernetes 和加入 Windows 节点的常见问题的解决方案。"
keywords: "kubernetes，1.9，linux，编译"
ms.openlocfilehash: 73b44ffd12fba58ac4ef38352c012061a6817945
ms.sourcegitcommit: ad5f6344230c7c4977adf3769fb7b01a5eca7bb9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/05/2017
---
# <a name="troubleshooting-kubernetes"></a>Kubernetes 疑难解答 #
此页面逐一介绍 Kubernetes 设置、网络和部署的一些常见问题。

> [!tip]
> 通过向[我们的文档存储库](https://github.com/MicrosoftDocs/Virtualization-Documentation/)提出 PR 来建议常见问题解答项目。


## <a name="common-deployment-errors"></a>常见部署错误 ##
调试 Kubernetes 主机分为三个主要类别（按可能性顺序）：

  - Kubernetes 系统容器出现问题。
  - `kubelet` 的运行方式出现问题。
  - 系统出现问题。


运行 `kubectl get pods -n kube-system` 以查看 Kubernetes 正在创建的 Pod；这样可以深入了解哪些特定项将会崩溃或无法正常启动。 然后，运行 `docker ps -a` 以查看支持这些 Pod 的所有原始容器。 最后，针对怀疑会导致问题的容器运行 `docker logs [ID]`，以查看进程的原始输出。


### <a name="permission-denied-errors"></a>_“权限被拒绝”_错误 ###
确保脚本具有可执行权限：

```bash
chmod +x [script name]
```

此外，某些脚本（如 `kubelet`）必须以管理员权限运行，并且应该带有前缀 `sudo`。


### <a name="cannot-connect-to-the-api-server-at-httpsaddressport"></a>无法在 `https://[address]:[port]` 连接到 API 服务器 ###
通常，此错误表示证书问题。 请确保你已经正确生成了配置文件，其中的 IP 地址与主机的 IP 地址匹配，并且你已将其复制到 API 服务器装载的目录中。

如果按照[我们的说明](./creating-a-linux-master)进行操作，这将在 `~/kube/kubelet/` 中；否则，请参阅 API 服务器的清单文件以检查装入点。


## <a name="common-networking-errors"></a>常见网络错误 ##
你的网络或主机上可能设置了其他限制，以阻止在节点之间进行某些类型的通信。 请确保：

  - 允许可能来自 Pod 的流量
  - 允许 HTTP 流量（如果要部署 Web 服务）
  - ICMP 数据包不会被丢弃


<!-- ### My Linux node cannot ping my Windows pods ### -->

## <a name="common-windows-errors"></a>常见 Windows 错误 ##


### <a name="my-windows-pods-cannot-access-the-linux-master-or-vice-versa"></a>我的 Windows Pod 无法访问 Linux 主机，反之亦然。 ###
如果你使用的是 Hyper-V 虚拟机，请确保在网络适配器上启用 MAC 欺骗。


### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>我的 Windows 节点无法使用服务 IP 访问我的服务。 ###
这是 Windows 上当前网络堆栈的已知限制。
