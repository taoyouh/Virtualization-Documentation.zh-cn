---
title: 将你的应用配置为使用组托管服务帐户
description: 如何将应用配置为使用 Windows 容器的组托管服务帐户（gMSAs）。
keywords: docker、容器、active directory、gmsa、应用、应用程序、组托管服务帐户、组托管服务帐户、配置
author: Heidilohr
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 234909d7f0cb0f30ee7fbf4796dd0381bfbff89f
ms.sourcegitcommit: 5d4b6823b82838cb3b574da3cd98315cdbb95ce2
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/11/2019
ms.locfileid: "10079711"
---
# <a name="configure-your-app-to-use-a-gmsa"></a>将你的应用配置为使用 gMSA

在典型配置中，仅当容器计算机帐户尝试对网络资源进行身份验证时，容器才会使用一个组托管服务帐户（gMSA）。 这意味着，如果应用需要使用 gMSA 标识，你的应用将需要作为**本地系统**或**网络服务**运行。

## <a name="run-an-iis-app-pool-as-network-service"></a>将 IIS 应用池作为网络服务运行

如果你在容器中托管 IIS 网站，你需要执行的所有操作都将你的应用池标识设置为**网络服务**。 你可以通过添加以下命令在 Dockerfile 中执行此操作：

```dockerfile
RUN %windir%\system32\inetsrv\appcmd.exe set AppPool DefaultAppPool -processModel.identityType:NetworkService
```

如果以前为 IIS 应用池使用了静态用户凭据，请考虑 gMSA 作为这些凭据的替换项。 你可以更改开发环境、测试和生产环境之间的 gMSA，IIS 将自动获取当前标识，而无需更改容器映像。

## <a name="run-a-windows-service-as-network-service"></a>作为网络服务运行 Windows 服务

如果你的容器化应用作为 Windows 服务运行，你可以将该服务设置为在 Dockerfile 中作为**网络服务**运行：

```dockerfile
RUN sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""
```

## <a name="run-arbitrary-console-apps-as-network-service"></a>将任意控制台应用作为网络服务运行

对于未在 IIS 或服务管理器中托管的泛型控制台应用，通常最简单的做法是将容器作为**网络服务**运行，以便应用自动继承 gMSA 上下文。 此功能可从 Windows Server 版本1709获得。

将以下行添加到你的 Dockerfile，以使其在默认情况下作为网络服务运行：

```dockerfile
USER "NT AUTHORITY\NETWORK SERVICE"
```

你还可以通过`docker exec`一次性地将容器作为网络服务连接到。 如果当容器通常不作为网络服务运行时，如果正在运行的容器中出现连接问题，此功能将非常有用。

```powershell
# Opens an interactive PowerShell console in the container (id = 85d) as the Network Service account
docker exec -it --user "NT AUTHORITY\NETWORK SERVICE" 85d powershell
```

## <a name="next-steps"></a>后续步骤

除了配置应用外，你还可以使用 gMSAs 执行以下操作：

- [运行容器](gmsa-run-container.md)
- [安排容器](gmsa-orchestrate-containers.md)

如果在安装过程中遇到任何问题，请查看我们的[故障排除指南](gmsa-troubleshooting.md)以了解可能的解决方案。
