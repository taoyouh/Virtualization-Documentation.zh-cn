---
title: 加入 Linux 节点
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: 在混合 OS Kubernetes 群集上部署 Kubernetes resoureces。
keywords: kubernetes、1.14、windows、入门
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: e6c569ae8d5bf50e24ea0fc7a6dd04734b60a863
ms.sourcegitcommit: d252f356a3de98f224e1550536810dfc75345303
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/04/2019
ms.locfileid: "10069941"
---
# <a name="deploying-kubernetes-resources"></a>部署 Kubernetes 资源 #
假设你有一个包含至少1个主服务器和1个工作人员的 Kubernetes 群集, 则可以部署 Kubernetes 资源。
> [!TIP] 
> 想知道目前在 Windows 上支持哪些 Kubernetes 资源？ 有关详细信息, 请参阅[有关 Windows 的](https://github.com/orgs/kubernetes/projects/8)[正式支持功能](https://kubernetes.io/docs/setup/production-environment/windows/intro-windows-in-kubernetes/#supported-functionality-and-limitations)和 Kubernetes 指南。


## <a name="running-a-sample-service"></a>运行示例服务 ##
你将部署非常简单的[基于 PowerShell 的 Web 服务](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml)，以确保成功加入群集并正确配置网络。

在执行此操作之前, 最好确保所有节点保持良好状态。
```bash
kubectl get nodes
```

如果一切正常, 您可以下载并运行以下服务:
> [!Important] 
> 之前`kubectl apply`, 请确保将示例文件中的`microsoft/windowsservercore`图像仔细检查/修改为可[由你的节点运行的容器图像](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#choosing-container-os-versions)!

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/simpleweb.yml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

这将创建一个部署和一个服务。 最后一个 "监视" 命令将无限期地查询箱, 以跟踪其状态;只需`Ctrl+C`按一下即可`watch`在完成观察时退出该命令。

如果一切顺利，你将可能：

  - 在 Windows 节点上的命令`docker ps`下, 查看每个 pod 2 个容器
  - 在 Linux 主机的 `kubectl get pods` 命令下看到 2 个 Pod 容器
  - `curl` （针对 Linux 主机中端口 80 上的 *Pod* IP）可以获取 Web 服务器响应；这表明网络中节点到 Pod 的通信正常。
  - 可以通过 `docker exec` *在 Pod 之间*（如果拥有多个 Windows 节点，也可以跨主机）执行 ping 操作；这表明 Pod 之间的通信正常。
  - `curl` 从 Linux 主服务器和单个盒`kubectl get services`中的虚拟*服务 IP* (见下);此示例演示了对 pod 通信的正确服务。
  - `curl` 具有 Kubernetes[默认 DNS 后缀](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services)的*服务名称*, 演示了正确的服务发现。
  - `curl` 来自群集外部的 Linux 主机或计算机的*NodePort* ;这将演示入站连接。
  - `curl` 盒内的外部 Ip;这将演示出站连接。

> [!Note]  
> Windows*容器主机*将**无法**从计划的服务访问服务 IP。 这是一个[已知的平台限制](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip), 将在未来版本向 Windows Server 中改进。 但是, Windows*盒* **** 可以访问服务 IP。

## <a name="next-steps"></a>后续步骤 ##
在本部分中, 我们介绍了如何在 Windows 节点上安排 Kubernetes 资源。 本指南结束。 如果遇到任何问题, 请查看疑难解答部分:

> [!div class="nextstepaction"]
> [故障排除](./common-problems.md)

否则, 你可能还希望将 Kubernetes 组件作为 Windows 服务运行:
> [!div class="nextstepaction"]
> [Windows 服务](./kube-windows-services.md)
