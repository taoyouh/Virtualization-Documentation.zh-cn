---
title: 加入 Linux 节点
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: 部署混合操作系统 Kubernetes 群集上的 Kubernetes resoureces。
keywords: kubernetes，1.13，windows，入门
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 7d2f1dd789a96a3ee4898ef196f872e574d6321f
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/26/2019
ms.locfileid: "9574898"
---
# <a name="deploying-kubernetes-resources"></a>部署 Kubernetes 资源 #
假设你有 Kubernetes 群集包含至少 1 个大纲和 1 个工作线程，现在可以部署 Kubernetes 资源。
> [!TIP] 
> 想知道在 Windows 上现在支持哪些 Kubernetes 资源？ 请有关更多详细信息，参阅[正式受支持的功能](https://kubernetes.io/docs/getting-started-guides/windows/#supported-features)和[Windows 路线图上的 Kubernetes](https://trello.com/b/rjTqrwjl/windows-k8s-roadmap) 。


## <a name="running-a-sample-service"></a>运行示例服务 ##
你将部署非常简单的[基于 PowerShell 的 Web 服务](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml)，以确保成功加入群集并正确配置网络。

之前执行此操作，它最好始终以确保我们的所有节点都都正常运行。
```bash
kubectl get nodes
```

如果一切正常，你可以下载并运行以下服务：
> [!Important] 
> 之前`kubectl apply`，请确保为双击的 check/修改`microsoft/windowsservercore`中的示例文件[是由你节点可运行的容器映像](https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility#choosing-container-os-versions)的图像 ！

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/simpleweb.yml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

这将创建一项部署和一项服务。 最后，请观看命令查询 pod 无限期以跟踪其状态;只需按下`Ctrl+C`退出`watch`观察完后的命令。

如果一切顺利，你将可能：

  - 通过每 pod 下的 2 个容器，请参阅`docker ps`命令在 Windows 节点
  - 在 Linux 主机的 `kubectl get pods` 命令下看到 2 个 Pod 容器
  - `curl` （针对 Linux 主机中端口 80 上的 *Pod* IP）可以获取 Web 服务器响应；这表明网络中节点到 Pod 的通信正常。
  - 可以通过 `docker exec` *在 Pod 之间*（如果拥有多个 Windows 节点，也可以跨主机）执行 ping 操作；这表明 Pod 之间的通信正常。
  - `curl` 虚拟*服务 IP* (下看到`kubectl get services`) 在 Linux 主机和个别 pod;此演示了正确的服务到 pod 的通信。
  - `curl` 与 Kubernetes[默认 DNS 后缀](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services)，演示正确的服务发现*服务名称*。
  - `curl` *NodePort*从 Linux 主机或群集; 之外的计算机此示例演示入站的连接。
  - `curl` 来自 pod; 内的外部 Ip此示例演示出站连接。

> [!Note]  
> Windows*容器主机*将**不**能从服务计划对其访问服务 IP。 这是[已知平台限制](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip)，将在将来版本到 Windows Server 中得到改进。 Windows *pod* **都**能够但是访问服务 IP。

### <a name="port-mapping"></a>端口映射 ### 
也可以通过映射节点上的端口来访问分别通过其各自节点托管于 Pod 中的服务。 另一个[可用的 YAML 示例](https://github.com/Microsoft/SDN/blob/master/Kubernetes/PortMapping.yaml)，也可以用于说明此功能，该示例涉及从节点上端口 4444 到 Pod 上端口 80 的映射。 要进行部署，请按照前述的相同步骤操作：

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/PortMapping.yaml -O win-webserver-port-mapped.yaml
kubectl apply -f win-webserver-port-mapped.yaml
watch kubectl get pods -o wide
```

现在应该可以 `curl` 端口 4444 上的*节点* IP 并收到 Web 服务器响应。 请注意，由于必须执行一对一映射，按节点向单个 Pod 的扩展将受到限制。


## <a name="next-steps"></a>后续步骤 ##
在此部分中，我们将介绍如何安排 Windows 节点上的 Kubernetes 资源。 以上就是本指南。 如果出现任何问题，请查看疑难解答部分：

> [!div class="nextstepaction"]
> [疑难解答](./common-problems.md)

否则，你可能也有兴趣作为 Windows 服务运行 Kubernetes 组件：
> [!div class="nextstepaction"]
> [Windows 服务](./kube-windows-services.md)