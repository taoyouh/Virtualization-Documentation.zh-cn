---
title: 加入 Linux 节点
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: how-to
ms.prod: containers
description: 在混合 OS Kubernetes 群集上部署 Kubernetes resoureces。
keywords: kubernetes，1.14，windows，入门
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 13f2168bc2e731b65768a73bbb04ffe9363ded59
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192334"
---
# <a name="deploying-kubernetes-resources"></a>部署 Kubernetes 资源 #
假设有一个 Kubernetes 群集，其中包含至少1个主节点和1个辅助角色，则可以部署 Kubernetes 资源。
> [!TIP]
> 在 Windows 上了解目前支持哪些 Kubernetes 资源？ 有关更多详细信息，请参阅[正式支持的功能](https://kubernetes.io/docs/setup/production-environment/windows/intro-windows-in-kubernetes/#supported-functionality-and-limitations)和[Windows 上的 Kubernetes 路线图](https://github.com/orgs/kubernetes/projects/8)。


## <a name="running-a-sample-service"></a>运行示例服务 ##
你将部署非常简单的[基于 PowerShell 的 Web 服务](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml)，以确保成功加入群集并正确配置网络。

在执行此操作之前，最好确保所有节点都运行正常。
```bash
kubectl get nodes
```

如果一切正常，您可以下载并运行以下服务：
> [!Important]
> 在之前 `kubectl apply` ，请务必仔细检查/修改示例文件中的映像，并将其修改 `microsoft/windowsservercore` 为[节点可运行的容器映像](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#choosing-container-os-versions)！

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/simpleweb.yml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

这将创建一个部署和一个服务。 最后一个 watch 命令会无限期地查询箱，以跟踪其状态;只需按一下 `Ctrl+C` 即可 `watch` 在完成观察后退出命令。

如果一切顺利，你将可能：

  - 在 `docker ps` Windows 节点上的命令下查看每个 pod 2 个容器
  - 在 Linux 主机的 `kubectl get pods` 命令下看到 2 个 Pod 容器
  - `curl`在 Linux 主端口80上的*pod* ip 上，获取 web 服务器响应;这说明了正确的节点，可通过网络进行通信。
  - 可以通过 `docker exec`*在 Pod 之间*（如果拥有多个 Windows 节点，也可以跨主机）执行 ping 操作；这表明 Pod 之间的通信正常。
  - `curl`Linux 主节点和来自各个 pod 的虚拟*服务 IP* （如下所示 `kubectl get services` ）; 这说明了正确的服务到 pod 通信。
  - `curl`带有 Kubernetes[默认 DNS 后缀](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services)的*服务名称*，用于演示正确的服务发现。
  - `curl`来自群集之外的 Linux 主机或计算机的*NodePort* ;这说明入站连接。
  - `curl`pod 内的外部 Ip;这说明出站连接。

> [!Note]
> Windows*容器主机*将**无法**从计划的服务访问服务 IP。 这是一个[已知的平台限制](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip)，将在未来版本的 Windows Server 中得到改进。 但*Windows pod 可以访问* **are**服务 IP。

## <a name="next-steps"></a>后续步骤 ##
在本部分中，我们介绍了如何在 Windows 节点上计划 Kubernetes 资源。 本指南将结束。 如果有任何问题，请查看故障排除部分：

> [!div class="nextstepaction"]
> [疑难解答](./common-problems.md)

否则，你可能还想要将 Kubernetes 组件作为 Windows 服务运行：
> [!div class="nextstepaction"]
> [Windows 服务](./kube-windows-services.md)
