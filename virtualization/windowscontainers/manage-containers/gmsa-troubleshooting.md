---
title: Windows 容器的 gMSAs 疑难解答
description: 如何对 Windows 容器的组托管服务帐户（gMSAs）进行故障排除。
keywords: docker、容器、active directory、gmsa、组托管服务帐户、组托管服务帐户、疑难解答、疑难解答
author: Heidilohr
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 00a0d9b1367da55b7669fc26a3eca303272967ab
ms.sourcegitcommit: 5d4b6823b82838cb3b574da3cd98315cdbb95ce2
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/11/2019
ms.locfileid: "10079710"
---
# <a name="troubleshoot-gmsas-for-windows-containers"></a>Windows 容器的 gMSAs 疑难解答

## <a name="known-issues"></a>已知问题

### <a name="container-hostname-must-match-the-gmsa-name-for-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>容器主机名必须与 Windows Server 2016 和 Windows 10 版本1709和1803的 gMSA 名称相匹配

如果你运行的是 Windows Server 2016、版本1709或1803，则你的容器的主机名必须与你的 gMSA SAM 帐户名相匹配。

当主机名与 gMSA 名称不匹配时，入站 NTLM 身份验证请求和名称/SID 转换（由许多库（如 ASP.NET 成员身份角色提供程序）使用将失败。 即使主机名和 gMSA 名称不匹配，Kerberos 仍将继续正常工作。

此限制在 Windows Server 2019 中已修复，在此情况下，容器现在始终在网络上使用其 gMSA 名称，而不考虑分配的主机名。

### <a name="using-a-gmsa-with-more-than-one-container-simultaneously-leads-to-intermittent-failures-on-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>将 gMSA 与多个容器同时使用时，Windows Server 2016 和 Windows 10、版本1709和1803中将导致间歇性故障

由于所有容器都需要使用同一主机名，因此第二个问题会影响 Windows Server 2019 和 Windows 10 版本1809之前的 Windows 版本。 当为多个容器分配相同的标识和主机名时，当两个容器同时与同一域控制器对话时，可能会出现争用条件。 当另一个容器与同一域控制器通信时，它将取消与使用相同标识的任何以前的容器的通信。 这可能导致间歇性身份验证失败，有时在容器内运行`nltest /sc_verify:contoso.com`时可能会被视为信任失败。

我们更改了 Windows Server 2019 中的行为以将容器标识与计算机名称分开，从而允许多个容器同时使用相同的 gMSA。

### <a name="you-cant-use-gmsas-with-hyper-v-isolated-containers-on-windows-10-versions-1703-1709-and-1803"></a>在 Windows 10 版本1703、1709和1803上，不能将 gMSAs 与 Hyper-v 隔离容器配合使用

当你尝试在 Windows 10 和 Windows Server 版本1703、1709和1803上使用具有 Hyper-v 隔离容器的 gMSA 时，容器初始化将挂起或失败。

此错误已在 Windows Server 2019 和 Windows 10 版本1809中修复。 你还可以在 Windows Server 2016 和 Windows 10 版本1607的 gMSAs 上运行 Hyper-v 隔离容器。

## <a name="general-troubleshooting-guidance"></a>常规疑难解答指南

如果在使用 gMSA 运行容器时遇到错误，以下说明可帮助你识别根本原因。

### <a name="make-sure-the-host-can-use-the-gmsa"></a>请确保主机可以使用 gMSA

1. 验证主机是否已加入域以及是否可以访问域控制器。
2. 从 RSAT 安装广告 PowerShell 工具并运行[Test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount)以查看计算机是否有权检索 gMSA。 如果 cmdlet 返回**False**，则计算机无法访问 gMSA 密码。

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

3. 如果**ADServiceAccount**返回**False**，请验证主机属于可以访问 gMSA 密码的安全组。

    ```powershell
    # Get the current computer's group membership
    Get-ADComputer $env:computername | Get-ADPrincipalGroupMembership | Select-Object DistinguishedName

    # Get the groups allowed to retrieve the gMSA password
    # Change "WebApp01" for your own gMSA name
    (Get-ADServiceAccount WebApp01 -Properties PrincipalsAllowedToRetrieveManagedPassword).PrincipalsAllowedToRetrieveManagedPassword
    ```

4. 如果你的主机属于授权检索 gMSA 密码的安全组，但仍未通过**ADServiceAccount 的测试**，则你可能需要重新启动计算机才能获取反映其当前组成员身份的新票证。

#### <a name="check-the-credential-spec-file"></a>检查凭据规范文件

1. 从[CredentialSpec PowerShell 模块](https://aka.ms/credspec)中运行**CredentialSpec** ，以查找计算机上的所有凭据规范。 凭据规范必须存储在 Docker 根目录下的 "CredentialSpecs" 目录中。 你可以通过运行**docker 信息-f "{{} 找到 docker root 目录。DockerRootDir}} "**。
2. 打开 CredentialSpec 文件，确保以下字段正确填写：
    - **Sid**： gMSA 帐户的 sid
    - **MachineAccountName**： GMSA SAM 帐户名（不要包含完整的域名或美元符号）
    - **DnsTreeName**： Active Directory 林的 FQDN
    - **DnsName**： gMSA 所属的域的 FQDN
    - **NetBiosName**： gMSA 所属的域的 NETBIOS 名称
    - **GroupManagedServiceAccounts/Name**： gMSA SAM 帐户名（不包括完整的域名称或美元符号）
    - **GroupManagedServiceAccounts/作用域**：一个用于域 FQDN 的条目，一个用于 NETBIOS

    你的输入应类似于以下示例：完整的凭据规范：

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

3. 验证针对你的业务流程解决方案的凭据规范文件的路径是否正确。 如果您使用的是 Docker，请确保容器运行命令包括`--security-opt="credentialspec=file://NAME.json"`"CredentialSpec"，其中 "name. json" 使用名称输出替换为**Get**。 该名称是平面文件名，相对于 Docker 根目录下的 CredentialSpecs 文件夹。

### <a name="check-the-firewall-configuration"></a>检查防火墙配置

如果你在容器或主机网络上使用严格的防火墙策略，则可能会阻止与 Active Directory 域控制器或 DNS 服务器的连接。

| 协议和端口 | 用途 |
|-------------------|---------|
| TCP 和 UDP 53 | DNS |
| TCP 和 UDP 88 | Kerberos |
| TCP 139 | NetLogon |
| TCP 和 UDP 389 | 适用 |
| TCP 636 | LDAP SSL |

你可能需要允许访问其他端口，具体取决于你的容器发送到域控制器的流量类型。
有关 Active directory 使用的所有端口的完整列表，请参阅[Active directory 和 Active Directory 域服务端口要求](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd772723(v=ws.10)#communication-to-domain-controllers)。

### <a name="check-the-container"></a>检查容器

1. 如果你运行的是 windows Server 2019 或 Windows 10 版本1809之前的 Windows 版本，你的容器主机名必须匹配 gMSA 名称。 确保`--hostname`参数与 gMSA 短名称（不是域组件，例如 "webapp01"，而不是 "webapp01.contoso.com"）相匹配。

2. 检查容器网络配置以验证容器能否解析和访问 gMSA 的域的域控制器。 错误配置容器中的 DNS 服务器是身份问题的常见原因。

3. 通过在容器中运行以下 cmdlet 来检查容器是否有有效的连接（使用`docker exec`或等效）：

    ```powershell
    nltest /sc_verify:contoso.com
    ```

    如果 gMSA 可用且网络`NERR_SUCCESS`连接允许容器与域通信，则应返回信任验证。 如果失败，请验证主机和容器的网络配置。 两者都需要能够与域控制器进行通信。

4. 确保你的应用[配置为使用 gMSA](gmsa-configure-app.md)。 使用 gMSA 时，容器内的用户帐户不会更改。 相反，系统帐户在与其他网络资源交谈时使用 gMSA。 这意味着你的应用需要作为网络服务或本地系统运行，才能利用 gMSA 标识。

    > [!TIP]
    > 如果你运行`whoami`或使用其他工具来标识容器中的当前用户上下文，则不会看到 gMSA 名称本身。 这是因为你始终以本地用户（而不是域标识）的身份登录到容器。 GMSA 在与网络资源进行交谈时由计算机帐户使用，这就是你的应用需要作为网络服务或本地系统运行的原因。

5. 最后，如果你的容器似乎配置正确，但用户或其他服务无法自动对你的容器化应用进行身份验证，请检查你的 gMSA 帐户上的 Spn。 客户端将通过其访问你的应用程序的名称找到 gMSA 帐户。 这可能意味着，如果客户通过负载`host`平衡器或其他 DNS 名称连接到你的应用，你将需要 gMSA 的其他 spn。
