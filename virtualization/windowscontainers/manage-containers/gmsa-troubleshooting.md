---
title: 对 Windows 容器的 gMSA 进行故障排除
description: 如何对 Windows 容器的组托管服务帐户 (gMSA) 进行故障排除。
keywords: docker, 容器, active directory, gmsa, 组托管服务帐户, 故障排除
author: rpsqrd
ms.date: 10/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 89f255e307c2a48fd743d5abd1a49bba7703aaf3
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910237"
---
# <a name="troubleshoot-gmsas-for-windows-containers"></a>对 Windows 容器的 gMSA 进行故障排除

## <a name="known-issues"></a>已知问题

### <a name="container-hostname-must-match-the-gmsa-name-for-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>容器主机名必须与 Windows Server 2016 和 Windows 10 版本 1709 和 1803 的 gMSA 名称匹配

如果你运行的是 Windows Server 2016 版本 1709 或 1803，则容器的主机名必须与 gMSA SAM 帐户名称匹配。

当主机名与 gMSA 名称不匹配时，入站 NTLM 身份验证请求和名称/SID 转换（由许多库使用，例如 ASP.NET 成员身份角色提供程序）会失败。 即使主机名与 gMSA 名称不匹配，Kerberos 也会正常运行。

此限制在 Windows Server 2019 中已修复。目前，容器在网络上将始终使用其 gMSA 名称，不管分配的主机名是什么。

### <a name="using-a-gmsa-with-more-than-one-container-simultaneously-leads-to-intermittent-failures-on-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>在 Windows Server 2016 和 Windows 10 版本 1709 和 1803 上，将一个 gMSA 同时用于多个容器会导致出现间歇性故障

因为所有容器都需要使用相同的主机名，所以有另一个问题影响 Windows Server 2019 和 Windows 10 版本 1809 之前的 Windows 版本。 如果为多个容器分配了相同的标识和主机名，则当两个容器同时与同一域控制器通信时，可能会出现争用情况。 当另一个容器与同一域控制器通信时，它会取消与使用相同标识的任何先前容器的通信。 这可能会导致间歇性的身份验证失败。当你在容器内运行 `nltest /sc_verify:contoso.com` 时，这有时可能会表现为信任失败。

我们在 Windows Server 2019 中更改了此行为，将容器标识与计算机名称分开，允许多个容器同时使用同一 gMSA。

### <a name="you-cant-use-gmsas-with-hyper-v-isolated-containers-on-windows-10-versions-1703-1709-and-1803"></a>在 Windows 10 版本 1703、1709 和 1803 上，不能将 gMSA 用于 Hyper-V 隔离容器

在 Windows 10 和 Windows Server 版本 1703、1709 和 1803 上，尝试将 gMSA 用于 Hyper-V 隔离容器时，容器初始化会挂起或失败。

此 bug 在 Windows Server 2019 和 Windows 10 版本 1809 中已修复。 在 Windows Server 2016 和 Windows 10 版本 1607 上，你也可以通过 gMSA 运行 Hyper-V 隔离容器。

## <a name="general-troubleshooting-guidance"></a>常规故障排除指南

如果通过 gMSA 运行容器时遇到错误，则以下说明可能有助于你查明根本原因。

### <a name="make-sure-the-host-can-use-the-gmsa"></a>确保主机可以使用 gMSA

1. 验证主机是否已加入域，以及是否可以访问域控制器。
2. 从 RSAT 安装 AD PowerShell 工具并运行 [Test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount)，以查看计算机是否有权检索 gMSA。 如果该 cmdlet 返回 **False**，则计算机无权访问 gMSA 密码。

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

3. 如果 **Test-ADServiceAccount** 返回 **False**，请验证主机是否属于可以访问 gMSA 密码的安全组。

    ```powershell
    # Get the current computer's group membership
    Get-ADComputer $env:computername | Get-ADPrincipalGroupMembership | Select-Object DistinguishedName

    # Get the groups allowed to retrieve the gMSA password
    # Change "WebApp01" for your own gMSA name
    (Get-ADServiceAccount WebApp01 -Properties PrincipalsAllowedToRetrieveManagedPassword).PrincipalsAllowedToRetrieveManagedPassword
    ```

4. 如果你的主机属于有权检索 gMSA 密码的安全组但执行 **Test-ADServiceAccount** 时仍然失败，则可能需要重启计算机以获取反映其当前组成员身份的新票证。

#### <a name="check-the-credential-spec-file"></a>检查凭据规范文件

1. 从 [CredentialSpec PowerShell 模块](https://aka.ms/credspec)运行 **Get-CredentialSpec**，找到计算机上的所有凭据规范。 凭据规范必须存储在 Docker 根目录下的“CredentialSpecs”目录中。 可以通过运行 **docker info -f "{{.DockerRootDir}}"** 找到 Docker 根目录。
2. 打开 CredentialSpec 文件，确保已正确填写以下字段：
    - **Sid**：gMSA 帐户的 SID
    - **MachineAccountName**：gMSA SAM 帐户名称（不包括完整域名，也不包括美元符号）
    - **DnsTreeName**：Active Directory 林的 FQDN
    - **DnsName**：gMSA 所属域的 FQDN
    - **NetBiosName**：gMSA 所属域的 NETBIOS 名称
    - **GroupManagedServiceAccounts/Name**：gMSA SAM 帐户名称（不包括完整域名，也不包括美元符号）
    - **GroupManagedServiceAccounts/Scope**：一个条目用于域 FQDN，一个条目用于 NETBIOS

    你的输入应当类似于以下完整凭据规范示例：

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

3. 验证凭据规范文件的路径对于你的业务流程解决方案来说是否正确。 如果使用的是 Docker，请确保容器运行命令包括 `--security-opt="credentialspec=file://NAME.json"`，其中，“NAME.json”将替换为 **Get-CredentialSpec** 输出的名称。 该名称是一个平面文件名，相对于 Docker 根目录下的 CredentialSpecs 文件夹。

### <a name="check-the-firewall-configuration"></a>检查防火墙配置

如果你在容器或主机网络上使用严格的防火墙策略，该策略可能会阻止到 Active Directory 域控制器或 DNS 服务器的必需连接。

| 协议和端口 | 用途 |
|-------------------|---------|
| TCP 和 UDP 53 | DNS |
| TCP 和 UDP 88 | Kerberos |
| TCP 139 | NetLogon |
| TCP 和 UDP 389 | LDAP |
| TCP 636 | LDAP SSL |

你可能需要根据容器发送到域控制器的流量的类型来允许对其他端口的访问。
有关 Active Directory 使用的端口的完整列表，请参阅 [Active Directory 和 Active Directory 域服务端口要求](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd772723(v=ws.10)#communication-to-domain-controllers)。

### <a name="check-the-container"></a>检查容器

1. 如果运行的是 Windows Server 2019 或 Windows 10 版本 1809 之前的 Windows 版本，则容器主机名必须与 gMSA 名称匹配。 确保 `--hostname` 参数与 gMSA 短名称（不含域组件，例如“webapp01”而非“webapp01.contoso.com”）匹配。

2. 检查容器网络配置，验证容器能否解析和访问 gMSA 域的域控制器。 容器中配置不当的 DNS 服务器是导致标识问题的常见罪魁祸首。

3. 通过在容器中运行以下 cmdlet（使用 `docker exec` 或等效项）来检查容器是否具有到域的有效连接：

    ```powershell
    nltest /sc_verify:contoso.com
    ```

    如果 gMSA 可用并且网络连接允许容器与域进行通信，则信任验证应返回 `NERR_SUCCESS`。 如果该验证失败，请验证主机和容器的网络配置。 两者都需要能够与域控制器进行通信。

4. 检查容器是否可以获取有效的 Kerberos 票证授予票证 (TGT)：

    ```powershell
    klist get krbtgt
    ```

    此命令应返回“A ticket to krbtgt has been retrieved successfully”（已成功检索 krbtgt 的票证）并列出用于检索票证的域控制器。 如果能够获取 TGT，但上一步中 `nltest` 失败，则可能表示 gMSA 帐户配置不当。 有关详细信息，请参阅[检查 gMSA 帐户](#check-the-gmsa-account)。

    如果无法在容器中获取 TGT，则可能表示存在 DNS 或网络连接问题。 请确保容器可以使用域 DNS 名称来解析域控制器，并且可以从该容器路由到域控制器。

5. 确保你的应用已[配置为使用 gMSA](gmsa-configure-app.md)。 使用 gMSA 时，容器中的用户帐户不会更改。 相反，当系统帐户与其他网络资源通信时，它将使用 gMSA。 这意味着应用将需要作为“网络服务”或“本地系统”运行以利用 gMSA 标识。

    > [!TIP]
    > 如果运行 `whoami` 或使用其他工具在容器中标识当前用户上下文，则不会看到 gMSA 名称本身。 这是因为你始终作为本地用户而不是域标识登录到容器。 计算机帐户在与网络资源通信时都使用 gMSA，这就是应用需要作为“网络服务”或“本地系统”运行的原因。

### <a name="check-the-gmsa-account"></a>检查 gMSA 帐户

1. 如果容器看起来已正确配置，但用户或其他服务无法自动向容器化应用进行身份验证，请检查你的 gMSA 帐户上的 SPN。 客户端将根据它们用来访问应用程序的名称来查找 gMSA 帐户。 这可能意味着在特定情况下（例如，当客户端通过负载均衡器或其他 DNS 名称连接到你的应用时），你的 gMSA 需要额外的 `host` SPN。

2. 确保 gMSA 和容器主机属于同一个 Active Directory 域。 如果 gMSA 属于其他域，则容器主机将无法检索 gMSA 密码。

3. 确保域中只有一个帐户与你的 gMSA 同名。 gMSA 对象在其 SAM 帐户名后面附加了美元符号 ($)，因此 gMSA 可以命名为“myaccount$”，而不相关的用户帐户在同一域中可以命名为“myaccount”。 如果域控制器或应用程序必须按名称查找 gMSA，这可能会导致问题。 可以通过以下命令在 AD 中搜索具有类似名称的对象：

    ```powershell
    # Replace "GMSANAMEHERE" with your gMSA account name (no trailing dollar sign)
    Get-ADObject -Filter 'sAMAccountName -like "GMSANAMEHERE*"'
    ```

4. 如果对 gMSA 帐户启用了无约束委托，请确保 [UserAccountControl 属性](https://support.microsoft.com/en-us/help/305144/how-to-use-useraccountcontrol-to-manipulate-user-account-properties)的 `WORKSTATION_TRUST_ACCOUNT` 标志仍处于启用状态。 若要使容器中的 NETLOGON 与域控制器通信，此标志是必需的。在应用必须将名称解析为 SID 的情况下就是如此，反之亦然。 可以通过以下命令来检查是否正确配置了标志：

    ```powershell
    $gMSA = Get-ADServiceAccount -Identity 'yourGmsaName' -Properties UserAccountControl
    ($gMSA.UserAccountControl -band 0x1000) -eq 0x1000
    ```

    如果上述命令返回 `False`，请使用以下命令将 `WORKSTATION_TRUST_ACCOUNT` 标志添加到 gMSA 帐户的 UserAccountControl 属性。 此命令还将清除 UserAccountControl 属性中的 `NORMAL_ACCOUNT`、`INTERDOMAIN_TRUST_ACCOUNT` 和 `SERVER_TRUST_ACCOUNT` 标志。

    ```powershell
    Set-ADObject -Identity $gMSA -Replace @{ userAccountControl = ($gmsa.userAccountControl -band 0x7FFFC5FF) -bor 0x1000 }
    ```
