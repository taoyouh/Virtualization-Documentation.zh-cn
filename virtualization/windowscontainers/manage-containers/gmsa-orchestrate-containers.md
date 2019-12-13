---
title: 使用 gMSA 协调容器
description: 如何使用组托管服务帐户（gMSA）来协调 Windows 容器。
keywords: docker，容器，active directory，gmsa，orchestration，kubernetes，组托管服务帐户，组托管服务帐户
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 3d102aac45a1becf1879a718bb255d753b215006
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910257"
---
# <a name="orchestrate-containers-with-a-gmsa"></a>使用 gMSA 协调容器

在生产环境中，你通常会使用容器协调器来部署和管理你的应用程序和服务。 每个业务流程协调器都有其自己的管理范例，并负责接受凭据规范以提供给 Windows 容器平台。

在将容器与组托管服务帐户（Gmsa）协调时，请确保：

> [!div class="checklist"]
> * 可以计划用 Gmsa 运行容器的所有容器主机均已加入域
> * 容器主机有权检索容器使用的所有 Gmsa 的密码
> * 将创建凭据规范文件并将其上载到 orchestrator，或将其复制到每个容器主机上，具体取决于 orchestrator 如何处理它们。
> * 容器网络允许容器与 Active Directory 域控制器进行通信，以检索 gMSA 票证

## <a name="how-to-use-gmsa-with-service-fabric"></a>如何将 gMSA 与 Service Fabric 配合使用

在应用程序清单中指定凭据规范位置时，Service Fabric 支持使用 gMSA 运行 Windows 容器。 需要创建凭据规范文件，并将其放入每个主机上 Docker data 目录的**CredentialSpecs**子目录中，以便 Service Fabric 可以找到它。 你可以运行**CredentialSpec** Cmdlet （ [CredentialSpec PowerShell 模块](https://aka.ms/credspec)的一部分）来验证你的凭据规范是否位于正确的位置。

有关如何配置应用程序的详细信息，请参阅[快速入门：将 windows 容器部署到 Service Fabric](https://docs.microsoft.com/azure/service-fabric/service-fabric-quickstart-containers) ，并[设置在 Service Fabric 上运行的 Windows 容器的 gMSA](https://docs.microsoft.com/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers) 。

## <a name="how-to-use-gmsa-with-docker-swarm"></a>如何将 gMSA 与 Docker Swarm 配合使用

若要将 gMSA 与 Docker Swarm 管理的容器一起使用，请使用 `--credential-spec` 参数运行[Docker service create](https://docs.docker.com/engine/reference/commandline/service_create/)命令：

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

请参阅[Docker Swarm 示例](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only)，了解有关如何将凭据规格用于 Docker 服务的详细信息。

## <a name="how-to-use-gmsa-with-kubernetes"></a>如何将 gMSA 与 Kubernetes 配合使用

使用 Kubernetes 中的 Gmsa 对 Windows 容器进行计划的支持以 Kubernetes 1.14 中的 alpha 功能形式提供。 有关此功能的最新信息以及如何在 Kubernetes 分发中测试此功能的信息，请参阅[Configure gMSA For Windows pod and 容器](https://kubernetes.io/docs/tasks/configure-pod-container/configure-gmsa)。

## <a name="next-steps"></a>后续步骤

除了协调容器外，还可以使用 Gmsa 来执行以下操作：

- [配置应用](gmsa-configure-app.md)
- [运行容器](gmsa-run-container.md)

如果在安装过程中遇到任何问题，请查看[故障排除指南](gmsa-troubleshooting.md)，了解可能的解决方法。
