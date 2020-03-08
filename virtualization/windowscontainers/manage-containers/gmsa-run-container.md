---
title: 使用 gMSA 运行容器
description: 如何使用组托管服务帐户（gMSA）运行 Windows 容器。
keywords: docker，容器，active directory，gmsa，组托管服务帐户，组托管服务帐户
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: b997cf79cdf7f1782b6299198859714563c45f8c
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/06/2020
ms.locfileid: "78853931"
---
# <a name="run-a-container-with-a-gmsa"></a>使用 gMSA 运行容器

若要使用组托管服务帐户（gMSA）运行容器，请将凭据规范文件提供给[docker 运行](https://docs.docker.com/engine/reference/run)的 `--security-opt` 参数：

```powershell
# For Windows Server 2016, change the image name to mcr.microsoft.com/windows/servercore:ltsc2016
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

>[!IMPORTANT]
>在 Windows Server 2016 版本1709和1803上，容器的主机名必须与 gMSA 短名称匹配。

在上面的示例中，gMSA SAM 帐户名称为 "webapp01"，因此容器主机名也称为 "webapp01"。

在 Windows Server 2019 和更高版本中，主机名字段不是必需的，但该容器仍将通过 gMSA 名称而不是主机名进行标识，即使您显式提供不同的主机名也是如此。

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

如果未 `NERR_Success`受信任的 DC 连接状态和信任验证状态，请按照[故障排除说明](gmsa-troubleshooting.md#check-the-container)来调试问题。

你可以通过运行以下命令并检查客户端名称来验证容器中的 gMSA 标识：

```powershell
PS C:\> klist get webapp01

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

若要打开 PowerShell 或另一个控制台应用作为 gMSA 帐户，你可以要求容器在 Network Service 帐户下运行，而不是在 normal ContainerAdministrator （或 ContainerUser for NanoServer）帐户下运行：

```powershell
# NOTE: you can only run as Network Service or SYSTEM on Windows Server 1709 and later
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 --user "NT AUTHORITY\NETWORK SERVICE" -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

作为网络服务运行时，可以通过尝试连接到域控制器上的 SYSVOL 来测试网络身份验证作为 gMSA：

```powershell
# This command should succeed if you're successfully running as the gMSA
PS C:\> dir \\contoso.com\SYSVOL


    Directory: \\contoso.com\sysvol


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d----l        2/27/2019   8:09 PM                contoso.com
```

## <a name="next-steps"></a>后续步骤

除了运行容器外，还可以使用 Gmsa 来执行以下操作：

- [配置应用](gmsa-configure-app.md)
- [协调容器](gmsa-orchestrate-containers.md)

如果在安装过程中遇到任何问题，请查看[故障排除指南](gmsa-troubleshooting.md)，了解可能的解决方法。
