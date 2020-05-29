---
title: 将应用配置为使用组托管服务帐户
description: 如何将应用配置为对 Windows 容器使用组托管服务帐户 (gMSA)。
keywords: docker, 容器, active directory, gmsa, 应用, 应用程序, 组托管服务帐户, 配置
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 6635381d5f7ddbebf7bdea4624af241b9f6a6864
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909787"
---
# <a name="configure-your-app-to-use-a-gmsa"></a>将应用配置为使用 gMSA

在典型配置中，只会为容器提供一个组托管服务帐户 (gMSA)。每当容器计算机帐户尝试向网络资源进行身份验证时，都会使用该帐户。 这意味着，如果你的应用需要使用 gMSA 标识，则它需要作为“本地系统”或“网络服务”运行。

## <a name="run-an-iis-app-pool-as-network-service"></a>将 IIS 应用池作为“网络服务”运行

如果将 IIS 网站托管在容器中，则若要利用 gMSA，只需将应用池标识设置为“网络服务”即可。 可以通过在 Dockerfile 中添加以下命令来完成此操作：

```dockerfile
RUN %windir%\system32\inetsrv\appcmd.exe set AppPool DefaultAppPool -processModel.identityType:NetworkService
```

如果以前为 IIS 应用池使用了静态用户凭据，请考虑使用 gMSA 替换那些凭据。 你可以在开发环境、测试环境和生产环境之间更改 gMSA，IIS 会自动选取当前标识，无需更改容器映像。

## <a name="run-a-windows-service-as-network-service"></a>将 Windows 服务作为“网络服务”运行

如果你的容器化应用作为 Windows 服务运行，则可在 Dockerfile 中将该服务设置为作为“网络服务”运行：

```dockerfile
RUN sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""
```

## <a name="run-arbitrary-console-apps-as-network-service"></a>将任意控制台应用作为“网络服务”运行

对于未托管在 IIS 或服务管理器中的常规控制台应用，通常最简单的方式是将容器作为“网络服务”运行，以便应用自动继承 gMSA 上下文。 此功能从 Windows Server 版本 1709 开始提供。

将以下行添加到 Dockerfile，使其在默认情况下作为“网络服务”运行：

```dockerfile
USER "NT AUTHORITY\NETWORK SERVICE"
```

还可以每次使用 `docker exec` 连接到作为“网络服务”的容器。 当容器不像通常那样作为“网络服务”运行时，如果你对正在运行的容器中的连接问题进行排查，这将特别有用。

```powershell
# Opens an interactive PowerShell console in the container (id = 85d) as the Network Service account
docker exec -it --user "NT AUTHORITY\NETWORK SERVICE" 85d powershell
```

## <a name="next-steps"></a>后续步骤

除了配置应用外，还可以使用 gMSA 来执行以下操作：

- [运行容器](gmsa-run-container.md)
- [协调容器](gmsa-orchestrate-containers.md)

如果在设置过程中遇到任何问题，请查看我们的[故障排除指南](gmsa-troubleshooting.md)，了解可能的解决方案。
