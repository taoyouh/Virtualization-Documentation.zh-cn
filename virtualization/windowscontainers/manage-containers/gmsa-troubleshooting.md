---
title: 排查 Windows 容器的 Gmsa 问题
description: 如何对 Windows 容器的组托管服务帐户（Gmsa）进行故障排除。
keywords: docker，容器，active directory，gmsa，组托管服务帐户，组托管服务帐户，故障排除，故障排除
author: rpsqrd
ms.date: 10/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 89f255e307c2a48fd743d5abd1a49bba7703aaf3
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910237"
---
# <a name="troubleshoot-gmsas-for-windows-containers"></a>排查 Windows 容器的 Gmsa 问题

## <a name="known-issues"></a>已知问题

### <a name="container-hostname-must-match-the-gmsa-name-for-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>容器主机名必须与 Windows Server 2016 和 Windows 10 版本1709和1803的 gMSA 名称匹配

如果运行的是 Windows Server 2016，版本1709或1803，则容器的主机名必须与 gMSA SAM 帐户名称匹配。

当主机名与 gMSA 名称不匹配时，入站 NTLM 身份验证请求和名称/SID 转换（由许多库（如 ASP.NET 成员身份角色提供程序）使用会失败。 即使 hostname 和 gMSA 名称不匹配，Kerberos 仍将继续正常工作。

此限制已在 Windows Server 2019 中得到解决，其中容器现在始终在网络上使用其 gMSA 名称，而不考虑分配的主机名。

### <a name="using-a-gmsa-with-more-than-one-container-simultaneously-leads-to-intermittent-failures-on-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>同时使用具有多个容器的 gMSA 会导致 Windows Server 2016 和 Windows 10 版本1709和1803出现间歇性故障

由于所有容器都需要使用同一主机名，因此第二个问题会影响 Windows Server 2019 和 Windows 10 版本1809之前的 Windows 版本。 如果为多个容器分配相同的标识和主机名，则当两个容器同时与同一域控制器通信时，可能会出现争用条件。 当另一个容器与同一域控制器通信时，它将取消与使用相同标识的任何以前的容器的通信。 这可能导致间歇的身份验证失败，有时在容器中运行 `nltest /sc_verify:contoso.com` 时，可能会被视为信任失败。

我们更改了 Windows Server 2019 中的行为，以便将容器标识与计算机名称分离，从而允许多个容器同时使用同一 gMSA。

### <a name="you-cant-use-gmsas-with-hyper-v-isolated-containers-on-windows-10-versions-1703-1709-and-1803"></a>在 Windows 10 版本1703、1709和1803上，不能将 Gmsa 与 Hyper-v 独立容器一起使用

尝试将 gMSA 与 Windows 10 和 Windows Server 版本1703、1709和1803上的 Hyper-v 隔离容器一起使用时，容器初始化会挂起或失败。

此错误是在 Windows Server 2019 和 Windows 10 版本1809中修复的。 你还可以在 Windows Server 2016 上的 Gmsa 和 Windows 10 版本1607上运行 Hyper-v 独立容器。

## <a name="general-troubleshooting-guidance"></a>一般故障排除指南

如果在运行具有 gMSA 的容器时遇到错误，以下说明可帮助你确定根本原因。

### <a name="make-sure-the-host-can-use-the-gmsa"></a>请确保主机可以使用 gMSA

1. 验证主机是否已加入域，并且是否可以访问域控制器。
2. 从 RSAT 安装 AD PowerShell 工具并运行[uninstall-adserviceaccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount)以查看计算机是否有权检索 gMSA。 如果该 cmdlet 返回**False**，则计算机无权访问 gMSA 密码。

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

3. 如果**uninstall-adserviceaccount**返回**False**，请验证主机是否属于可以访问 gMSA 密码的安全组。

    ```powershell
    # Get the current computer's group membership
    Get-ADComputer $env:computername | Get-ADPrincipalGroupMembership | Select-Object DistinguishedName

    # Get the groups allowed to retrieve the gMSA password
    # Change "WebApp01" for your own gMSA name
    (Get-ADServiceAccount WebApp01 -Properties PrincipalsAllowedToRetrieveManagedPassword).PrincipalsAllowedToRetrieveManagedPassword
    ```

4. 如果主机属于授权检索 gMSA 密码的安全组，但仍未通过**Uninstall-adserviceaccount 测试**，则可能需要重新启动计算机以获取反映其当前组成员身份的新票证。

#### <a name="check-the-credential-spec-file"></a>检查凭据规范文件

1. 从[CredentialSpec PowerShell 模块](https://aka.ms/credspec)中运行**CredentialSpec**以查找计算机上的所有凭据规范。 凭据规范必须存储在 Docker 根目录下的 "CredentialSpecs" 目录中。 可以通过运行**docker info-f "{{。DockerRootDir}} "** 。
2. 打开 CredentialSpec 文件，并确保正确填写以下字段：
    - **Sid**： gMSA 帐户的 sid
    - **MachineAccountName**： GMSA SAM 帐户名称（不包括完整的域名或美元符号）
    - **DnsTreeName**： Active Directory 林的 FQDN
    - **DnsName**： gMSA 所属域的 FQDN
    - **NetBiosName**： gMSA 所属的域的 NETBIOS 名称
    - **GroupManagedServiceAccounts/Name**： gMSA SAM 帐户名称（不包括完整的域名或美元符号）
    - **GroupManagedServiceAccounts/作用域**：一项用于域 FQDN，一个用于 NETBIOS

    输入应类似于以下完整凭据规范示例：

    ```json
    {
        "CmsPlugins": [
            "ActiveDirectory"
        ],
        "DomainJoinConfig": {
            "Sid": "S-1-5-21-702590844-1001920913-2680819671",
            "MachineAccountName": "webapp01",
            "Guid": "56d9b66c-d746-4f87-bd26-26760cfdca2e",
            "DnsTreeName": "contoso.com",
            "DnsName": "contoso.com",
            "NetBiosName": "CONTOSO"
        },
        "ActiveDirectoryConfig": {
            "GroupManagedServiceAccounts": [
                {
                    "Name": "webapp01",
                    "Scope": "contoso.com"
                },
                {
                    "Name": "webapp01",
                    "Scope": "CONTOSO"
                }
            ]
        }
    }
    ```

3. 验证凭据规范文件的路径对于你的业务流程解决方案是正确的。 如果使用的是 Docker，请确保容器运行命令包含 `--security-opt="credentialspec=file://NAME.json"`，其中 " **CredentialSpec**" 被替换为名称输出。 该名称是平面文件名，相对于 Docker 根目录下的 CredentialSpecs 文件夹。

### <a name="check-the-firewall-configuration"></a>检查防火墙配置

如果在容器或主机网络上使用严格的防火墙策略，则可能会阻止与 Active Directory 域控制器或 DNS 服务器的连接。

| 协议和端口 | 用途 |
|-------------------|---------|
| TCP 和 UDP 53 | DNS |
| TCP 和 UDP 88 | Kerberos |
| TCP 139 | NetLogon |
| TCP 和 UDP 389 | LDAP |
| TCP 636 | LDAP SSL |

你可能需要根据容器发送到域控制器的流量类型允许访问其他端口。
有关 Active Directory 使用的端口的完整列表，请参阅[Active Directory 和 Active Directory 域服务端口要求](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd772723(v=ws.10)#communication-to-domain-controllers)。

### <a name="check-the-container"></a>检查容器

1. 如果在 Windows Server 2019 或 Windows 10 版本1809之前运行 Windows 版本，则容器主机名必须与 gMSA 名称匹配。 确保 `--hostname` 参数匹配 gMSA 短名称（无域组件; 例如，"webapp01" 而不是 "webapp01.contoso.com"）。

2. 请检查容器网络配置，验证容器能否解析和访问 gMSA 域的域控制器。 在容器中配置错误的 DNS 服务器是标识问题的常见问题。

3. 通过在容器中运行以下 cmdlet 来检查容器是否具有到域的有效连接（使用 `docker exec` 或等效项）：

    ```powershell
    nltest /sc_verify:contoso.com
    ```

    如果 gMSA 可用并且网络连接允许容器与域通信，则信任验证应返回 `NERR_SUCCESS`。 如果该操作失败，请验证主机和容器的网络配置。 两者都需要能够与域控制器进行通信。

4. 检查容器是否可以获取有效的 Kerberos 票证授予票证（TGT）：

    ```powershell
    klist get krbtgt
    ```

    此命令应返回 "已成功检索到 krbtgt 的票证"，并列出用于检索票证的域控制器。 如果能够获取 TGT，但上一步中 `nltest` 失败，则这可能表示 gMSA 帐户配置错误。 有关详细信息，请参阅[检查 gMSA 帐户](#check-the-gmsa-account)。

    如果无法在容器中获取 TGT，这可能表示 DNS 或网络连接问题。 确保容器可以使用域 DNS 名称解析域控制器，并且域控制器可以从容器中路由。

5. 确保将应用[配置为使用 gMSA](gmsa-configure-app.md)。 当你使用 gMSA 时，容器中的用户帐户不会更改。 相反，当系统帐户与其他网络资源通信时，将使用 gMSA。 这意味着你的应用程序将需要作为网络服务或本地系统运行以利用 gMSA 标识。

    > [!TIP]
    > 如果运行 `whoami` 或使用其他工具在容器中标识当前用户上下文，则不会看到 gMSA 名称本身。 这是因为你始终以本地用户而不是域标识登录到容器。 计算机帐户在与网络资源通信时使用 gMSA，这就是应用需要作为网络服务或本地系统运行的原因。

### <a name="check-the-gmsa-account"></a>检查 gMSA 帐户

1. 如果你的容器似乎已正确配置，但用户或其他服务无法自动向容器化应用进行身份验证，请检查你的 gMSA 帐户上的 Spn。 客户端将按其访问应用程序的名称查找 gMSA 帐户。 例如，如果客户端通过负载平衡器或其他 DNS 名称连接到你的应用，则可能需要 gMSA 的其他 `host` Spn。

2. 确保 gMSA 和容器主机属于同一个 Active Directory 域。 如果 gMSA 属于其他域，则容器主机将无法检索 gMSA 密码。

3. 确保域中只有一个帐户与 gMSA 同名。 gMSA 对象在其 SAM 帐户名称后追加了美元符号（$），因此，在同一域中，可以将 gMSA 命名为 "我的帐户 $"，并将其命名为 "我的帐户"。 如果域控制器或应用程序必须按名称查找 gMSA，这可能会导致问题。 可以通过以下命令在 AD 中搜索具有类似名称的对象：

    ```powershell
    # Replace "GMSANAMEHERE" with your gMSA account name (no trailing dollar sign)
    Get-ADObject -Filter 'sAMAccountName -like "GMSANAMEHERE*"'
    ```

4. 如果对 gMSA 帐户启用了无约束委托，请确保[UserAccountControl 属性](https://support.microsoft.com/en-us/help/305144/how-to-use-useraccountcontrol-to-manipulate-user-account-properties)仍启用了 `WORKSTATION_TRUST_ACCOUNT` 标志。 要使容器中的 NETLOGON 与域控制器通信，此标志是必需的，因为应用程序必须将名称解析为 SID，反之亦然。 可以通过以下命令来检查是否正确配置了标志：

    ```powershell
    $gMSA = Get-ADServiceAccount -Identity 'yourGmsaName' -Properties UserAccountControl
    ($gMSA.UserAccountControl -band 0x1000) -eq 0x1000
    ```

    如果上述命令返回 `False`，请使用以下命令将 `WORKSTATION_TRUST_ACCOUNT` 标志添加到 gMSA 帐户的 UserAccountControl 属性。 此命令还将清除 UserAccountControl 属性中的 `NORMAL_ACCOUNT`、`INTERDOMAIN_TRUST_ACCOUNT`和 `SERVER_TRUST_ACCOUNT` 标志。

    ```powershell
    Set-ADObject -Identity $gMSA -Replace @{ userAccountControl = ($gmsa.userAccountControl -band 0x7FFFC5FF) -bor 0x1000 }
    ```
