---
title: 将应用配置为使用组托管服务帐户
description: 如何将应用配置为对 Windows 容器使用组托管服务帐户（Gmsa）。
keywords: docker，容器，active directory，gmsa，应用，应用程序，组托管服务帐户，组托管服务帐户，配置
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 6635381d5f7ddbebf7bdea4624af241b9f6a6864
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909787"
---
# <a name="configure-your-app-to-use-a-gmsa"></a>将应用配置为使用 gMSA

在典型配置中，仅在容器计算机帐户尝试对网络资源进行身份验证时，才为容器提供一个组托管服务帐户（gMSA）。 这意味着，如果应用需要使用 gMSA 标识，则需要将其作为**本地系统**或**网络服务**运行。

## <a name="run-an-iis-app-pool-as-network-service"></a>作为网络服务运行 IIS 应用池

如果要在容器中托管 IIS 网站，则需要执行的所有操作都是将应用程序池标识设置为**网络服务**。 可以通过添加以下命令在 Dockerfile 中执行此操作：

```dockerfile
RUN %windir%\system32\inetsrv\appcmd.exe set AppPool DefaultAppPool -processModel.identityType:NetworkService
```

如果以前对 IIS 应用程序池使用了静态用户凭据，请考虑 gMSA 作为这些凭据的替换。 你可以在开发环境、测试环境和生产环境之间更改 gMSA，IIS 将自动选取当前标识，而无需更改容器映像。

## <a name="run-a-windows-service-as-network-service"></a>将 Windows 服务作为网络服务运行

如果容器化应用作为 Windows 服务运行，则可以将服务设置为在 Dockerfile 中作为**网络服务**运行：

```dockerfile
RUN sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""
```

## <a name="run-arbitrary-console-apps-as-network-service"></a>作为网络服务运行任意控制台应用

对于未在 IIS 或 Service Manager 中承载的通用控制台应用程序，通常将容器作为**网络服务**运行，以便应用自动继承 gMSA 上下文。 此功能在 Windows Server 版本1709中提供。

将以下行添加到 Dockerfile，以使其在默认情况下作为网络服务运行：

```dockerfile
USER "NT AUTHORITY\NETWORK SERVICE"
```

你还可以使用 `docker exec`一次性连接到容器作为网络服务。 如果在容器通常不能作为网络服务运行的情况下正在运行的容器中的连接问题进行故障排除，这会特别有用。

```powershell
# Opens an interactive PowerShell console in the container (id = 85d) as the Network Service account
docker exec -it --user "NT AUTHORITY\NETWORK SERVICE" 85d powershell
```

## <a name="next-steps"></a>后续步骤

除了配置应用外，还可以使用 Gmsa 来执行以下操作：

- [运行容器](gmsa-run-container.md)
- [协调容器](gmsa-orchestrate-containers.md)

如果在安装过程中遇到任何问题，请查看[故障排除指南](gmsa-troubleshooting.md)，了解可能的解决方法。
