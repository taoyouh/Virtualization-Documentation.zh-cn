---
title: 具有 gMSA 的协调容器
description: 如何使用组托管服务帐户（gMSA）协调 Windows 容器。
keywords: docker、容器、active directory、gmsa、orchestration、kubernetes、组托管服务帐户、组托管服务帐户
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 3d102aac45a1becf1879a718bb255d753b215006
ms.sourcegitcommit: 22dcc1400dff44fb85591adf0fc443360ea92856
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/12/2019
ms.locfileid: "10209837"
---
# <a name="orchestrate-containers-with-a-gmsa"></a>具有 gMSA 的协调容器

在生产环境中，你将经常使用容器 orchestrator 部署和管理你的应用和服务。 每个 orchestrator 都有其自己的管理范例，并负责接受凭据规范以提供给 Windows 容器平台。

当你通过组托管服务帐户（gMSAs）协调容器时，请确保：

> [!div class="checklist"]
> * 可以计划运行具有 gMSAs 的容器的所有容器主机都已加入域
> * 容器主机具有检索容器使用的所有 gMSAs 的密码的访问权限
> * 将创建凭据规范文件并将其上载到 orchestrator，或复制到每个容器主机，具体取决于 orchestrator 如何首选处理它们。
> * 容器网络允许容器与 Active Directory 域控制器通信以检索 gMSA 票证

## <a name="how-to-use-gmsa-with-service-fabric"></a>如何将 gMSA 与 Service Fabric 配合使用

当你在应用程序清单中指定凭据规范位置时，Service Fabric 支持使用 gMSA 运行 Windows 容器。 你需要创建凭据规范文件并将其放在每个主机上 Docker 数据目录的**CredentialSpecs**子目录中，以便服务结构可以找到它。 你可以运行**CredentialSpec** Cmdlet （ [CredentialSpec PowerShell 模块](https://aka.ms/credspec)的一部分）验证凭据规范是否位于正确的位置。

有关如何配置你的应用程序的详细信息，请参阅[快速入门：将 windows 容器部署到 Service fabric](https://docs.microsoft.com/azure/service-fabric/service-fabric-quickstart-containers)和[设置适用于 Service fabric 的 Windows 容器的 gMSA](https://docs.microsoft.com/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers) 。

## <a name="how-to-use-gmsa-with-docker-swarm"></a>如何将 gMSA 与 Docker 配合使用 Swarm

若要将 gMSA 与由 Docker Swarm 托管的容器结合使用，请使用该`--credential-spec`参数运行[Docker 服务 create](https://docs.docker.com/engine/reference/commandline/service_create/)命令：

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

有关如何将凭据规范与 Docker 服务配合使用的详细信息，请参阅[Docker Swarm 示例](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only)。

## <a name="how-to-use-gmsa-with-kubernetes"></a>如何将 gMSA 与 Kubernetes 结合使用

对使用 Kubernetes 中的 gMSAs 的 Windows 容器进行计划的支持在 Kubernetes 1.14 中以 alpha 功能的形式提供。 请参阅为[Windows 箱和容器配置 gMSA](https://kubernetes.io/docs/tasks/configure-pod-container/configure-gmsa) ，了解有关此功能的最新信息以及如何在 Kubernetes 分发中进行测试。

## <a name="next-steps"></a>后续步骤

除了 "协调容器" 外，你还可以使用 gMSAs 执行以下操作：

- [配置应用](gmsa-configure-app.md)
- [运行容器](gmsa-run-container.md)

如果在安装过程中遇到任何问题，请查看我们的[故障排除指南](gmsa-troubleshooting.md)以了解可能的解决方案。
