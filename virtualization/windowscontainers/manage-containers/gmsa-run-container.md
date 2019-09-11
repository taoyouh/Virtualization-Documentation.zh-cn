---
title: 使用 gMSA 运行容器
description: 如何使用组托管服务帐户（gMSA）运行 Windows 容器。
keywords: docker、容器、active directory、gmsa、组托管服务帐户、组托管服务帐户
author: Heidilohr
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: b9c0406b5fe9527d88365dabf0cfd10114c34c74
ms.sourcegitcommit: 5d4b6823b82838cb3b574da3cd98315cdbb95ce2
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/11/2019
ms.locfileid: "10079707"
---
# <a name="run-a-container-with-a-gmsa"></a>使用 gMSA 运行容器

若要使用组托管服务帐户（gMSA）运行容器，请将凭据规范文件提供给 docker `--security-opt`的参数[运行](https://docs.docker.com/engine/reference/run)：

```powershell
# For Windows Server 2016, change the image name to mcr.microsoft.com/windows/servercore:ltsc2016
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

>[!IMPORTANT]
>在 Windows Server 2016 版本1709和1803上，容器的主机名必须匹配 gMSA 短名称。

在前面的示例中，gMSA SAM 帐户名为 "webapp01"，因此容器主机名也被命名为 "webapp01"。

在 Windows Server 2019 及更高版本中，不需要主机名字段，但容器仍将通过 gMSA 名称而不是主机名来标识自己，即使你显式提供了不同的名称也是如此。

若要检查 gMSA 是否正常工作，请在容器中运行以下 cmdlet：

```powershell
# Replace contoso.com with your own domain
PS C:\> nltest /sc_verify:contoso.com

Flags: b0 HAS_IP  HAS_TIMESERV
Trusted DC Name \\dc01.contoso.com
Trusted DC Connection Status Status = 0 0x0 NERR_Success
Trust Verification Status = 0 0x0 NERR_Success
The command completed successfully
```

如果受信任的 DC 连接状态和信任验证状态不`NERR_Success`是，请按照[疑难解答说明](gmsa-troubleshooting.md#check-the-container)调试问题。

你可以通过运行以下命令并检查客户端名称来验证容器内的 gMSA 标识：

```powershell
PS C:\> klist get krbtgt

Current LogonId is 0:0xaa79ef8
A ticket to krbtgt has been retrieved successfully.

Cached Tickets: (2)

#0>     Client: webapp01$ @ CONTOSO.COM
        Server: krbtgt/webapp01 @ CONTOSO.COM
        KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
        Ticket Flags 0x40a10000 -> forwardable renewable pre_authent name_canonicalize
        Start Time: 3/21/2019 4:17:53 (local)
        End Time:   3/21/2019 14:17:53 (local)
        Renew Time: 3/28/2019 4:17:42 (local)
        Session Key Type: AES-256-CTS-HMAC-SHA1-96
        Cache Flags: 0
        Kdc Called: dc01.contoso.com

[...]
```

若要将 PowerShell 或其他控制台应用打开为 gMSA 帐户，可以让容器在网络服务帐户下运行，而不是在普通 ContainerAdministrator （或 ContainerUser NanoServer）帐户下运行：

```powershell
# NOTE: you can only run as Network Service or SYSTEM on Windows Server 1709 and later
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 --user "NT AUTHORITY\NETWORK SERVICE" -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

当你作为网络服务运行时，你可以通过尝试连接到域控制器上的 SYSVOL 来测试网络身份验证，gMSA：

```powershell
# This command should succeed if you're successfully running as the gMSA
PS C:\> dir \\contoso.com\SYSVOL


    Directory: \\contoso.com\sysvol


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d----l        2/27/2019   8:09 PM                contoso.com
```

## <a name="next-steps"></a>后续步骤

除了运行的容器外，你还可以使用 gMSAs 执行以下操作：

- [配置应用](gmsa-configure-app.md)
- [安排容器](gmsa-orchestrate-containers.md)

如果在安装过程中遇到任何问题，请查看我们的[故障排除指南](gmsa-troubleshooting.md)以了解可能的解决方案。
