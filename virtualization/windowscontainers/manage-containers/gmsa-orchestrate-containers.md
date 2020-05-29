---
title: 使用 gMSA 协调容器
description: 如何使用组托管服务帐户 (gMSA) 来协调 Windows 容器。
keywords: docker, 容器, active directory, gmsa, 业务流程, kubernetes, 组托管服务帐户
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 3d102aac45a1becf1879a718bb255d753b215006
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910257"
---
# <a name="orchestrate-containers-with-a-gmsa"></a>使用 gMSA 协调容器

在生产环境中，你通常会使用容器业务流程协调程序来部署和管理应用和服务。 每个业务流程协调程序都有其自己的管理范例，并负责接受要提供给 Windows 容器平台的凭据规范。

在使用组托管服务帐户 (gMSA) 来协调容器时，请确保：

> [!div class="checklist"]
> * 可以按计划使用 gMSA 来运行容器的所有容器主机均已加入域
> * 容器主机有权检索容器使用的所有 gMSA 的密码
> * 会创建凭据规范文件并将其上传到业务流程协调程序，或将其复制到每个容器主机上，具体取决于业务流程协调程序希望如何处理它们。
> * 容器网络允许容器与 Active Directory 域控制器通信，以便检索 gMSA 票证

## <a name="how-to-use-gmsa-with-service-fabric"></a>如何将 gMSA 与 Service Fabric 配合使用

当你在应用程序清单中指定了凭据规范位置时，Service Fabric 支持使用 gMSA 运行 Windows 容器。 你需要创建凭据规范文件，并将其置于每个主机上 Docker 数据目录的 **CredentialSpecs** 子目录中，以便 Service Fabric 可以找到它。 可以运行 **Get-CredentialSpec**（[CredentialSpec PowerShell 模块](https://aka.ms/credspec)的一部分）来验证凭据规范是否位于正确的位置。

请参阅[快速入门：将 Windows 容器部署到 Service Fabric](https://docs.microsoft.com/azure/service-fabric/service-fabric-quickstart-containers) 和[为在 Service Fabric 上运行的 Windows 容器设置 gMSA](https://docs.microsoft.com/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers)，详细了解如何配置应用程序。

## <a name="how-to-use-gmsa-with-docker-swarm"></a>如何将 gMSA 与 Docker Swarm 配合使用

若要将 gMSA 用于由 Docker Swarm 管理的容器，请运行参数为 `--credential-spec` 的 [docker service create](https://docs.docker.com/engine/reference/commandline/service_create/) 命令：

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

若要详细了解如何将凭据规范用于 Docker 服务，请参阅 [Docker Swarm 示例](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only)。

## <a name="how-to-use-gmsa-with-kubernetes"></a>如何将 gMSA 与 Kubernetes 配合使用

支持在 Kubernetes 中使用 gMSA 来调度 Windows 容器是 Kubernetes 1.14 中的一项 alpha 功能。 请参阅[为 Windows Pod 和容器配置 gMSA](https://kubernetes.io/docs/tasks/configure-pod-container/configure-gmsa)，了解此功能的最新信息以及如何在你的 Kubernetes 发行版中测试此功能。

## <a name="next-steps"></a>后续步骤

除了协调各个容器之外，还可以使用 gMSA 来执行以下操作：

- [配置应用](gmsa-configure-app.md)
- [运行容器](gmsa-run-container.md)

如果在设置过程中遇到任何问题，请查看我们的[故障排除指南](gmsa-troubleshooting.md)，了解可能的解决方案。
