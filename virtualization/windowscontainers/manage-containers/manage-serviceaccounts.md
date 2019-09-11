---
title: 为 Windows 容器创建 gMSAs
description: 如何为 Windows 容器创建组托管服务帐户（gMSAs）。
keywords: docker、容器、active directory、gmsa、组托管服务帐户、组托管服务帐户
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 9ed9029e534d56bfe1830281d0bfd3ddde0cee9e
ms.sourcegitcommit: 5d4b6823b82838cb3b574da3cd98315cdbb95ce2
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/11/2019
ms.locfileid: "10079661"
---
# <a name="create-gmsas-for-windows-containers"></a>为 Windows 容器创建 gMSAs

基于 Windows 的网络通常使用 Active Directory （AD）来促进用户、计算机和其他网络资源之间的身份验证和授权。 企业应用程序开发人员通常将其应用设计为在加入域的服务器上进行广告集成和运行，以利用集成的 Windows 身份验证，从而使用户和其他服务能够轻松、透明地登录到应用程序与其标识。

虽然 Windows 容器不能加入域，但仍可使用 Active Directory 域标识支持各种身份验证方案。

若要实现此目的，你可以将 Windows 容器配置为使用[组托管服务帐户](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)（gMSA）运行，这是 windows Server 2012 中引入的一种特殊类型的服务帐户，旨在允许多台计算机共享标识而无需了解其密码。

当你使用 gMSA 运行容器时，容器主机从 Active Directory 域控制器检索 gMSA 密码，并将其提供给容器实例。 当容器的计算机帐户（系统）需要访问网络资源时，该容器将使用 gMSA 凭据。

本文介绍如何开始在 Windows 容器中使用 Active Directory 组托管服务帐户。

## <a name="prerequisites"></a>系统必备

若要运行具有 "组托管服务" 帐户的 Windows 容器，你将需要以下各项：

- 至少有一个运行 Windows Server 2012 或更高版本的域控制器的 Active Directory 域。 没有可使用 gMSAs 的林或域功能级别要求，但 gMSA 密码只能由运行 Windows Server 2012 或更高版本的域控制器分配。 有关详细信息，请参阅[gMSAs 的 Active Directory 要求](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts#BKMK_gMSA_Req)。
- 创建 gMSA 帐户的权限。 要创建 gMSA 帐户，您需要是域管理员或使用已被委派了*Create GroupManagedServiceAccount 对象*权限的帐户。
- 访问 internet 以下载 CredentialSpec PowerShell 模块。 如果你在断开连接的环境中工作，则可以[将该模块保存](https://docs.microsoft.com/powershell/module/powershellget/save-module?view=powershell-5.1)到具有 internet 访问的计算机上，并将其复制到开发计算机或容器主机。

## <a name="one-time-preparation-of-active-directory"></a>Active Directory 的一次性准备

如果你的域中尚未创建 gMSA，你将需要生成密钥分发服务（KDS）根密钥。 KDS 负责创建、旋转 gMSA 密码并将其发布到授权的主机。 当容器主机需要使用 gMSA 运行容器时，它将与 KDS 联系以检索当前密码。

若要检查 KDS 根密钥是否已创建，请在安装了广告 PowerShell 工具的域控制器或域成员上运行以下 PowerShell cmdlet 作为域管理员：

```powershell
Get-KdsRootKey
```

如果该命令返回键 ID，则 "全部" 已设置完毕，可向前跳到 "[创建组托管服务帐户](#create-a-group-managed-service-account)" 部分。 否则，请继续创建 KDS 根密钥。

在具有多个域控制器的生产环境或测试环境中，以域管理员身份在 PowerShell 中运行以下 cmdlet 以创建 KDS 根密钥。

```powershell
# For production environments
Add-KdsRootKey -EffectiveImmediately
```

尽管该命令意味着密钥将立即生效，但你需要等待10小时才能复制 KDS 根密钥并使其可在所有域控制器上使用。

如果你的域中仅有一个域控制器，你可以通过将该密钥设置为 "有效10小时前" 来加速此过程。

>[!IMPORTANT]
>不要在生产环境中使用此技术。

```powershell
# For single-DC test environments ONLY
Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
```

## <a name="create-a-group-managed-service-account"></a>创建组托管服务帐户

每个使用集成 Windows 身份验证的容器都需要至少一个 gMSA。 只要作为系统或网络服务运行的应用访问网络资源，就会使用主 gMSA。 无论分配给容器的主机名是什么，gMSA 的名称将成为网络上容器的名称。 也可以使用其他 gMSAs 配置容器，以防你希望在容器中以不同于容器计算机帐户的标识运行服务或应用程序。

创建 gMSA 时，你还会创建可在多台不同计算机之间同时使用的共享标识。 对 gMSA 密码的访问受 Active Directory 访问控制列表的保护。 我们建议为每个 gMSA 帐户创建一个安全组，并将相关的容器主机添加到安全组，以限制对该密码的访问权限。

最后，由于容器不会自动注册任何服务主体名称（SPN），因此你需要为你的 gMSA 帐户手动创建至少一个主机 SPN。

通常，使用与 gMSA 帐户相同的名称注册主机或 http SPN，但如果客户从负载平衡器后访问容器的应用程序或与 gMSA 名称不同的 DNS 名称，可能需要使用不同的服务名称。

例如，如果 gMSA 帐户名为 "WebApp01"，但你的用户在访问该网站`mysite.contoso.com`时，你应在`http/mysite.contoso.com` gMSA 帐户上注册 SPN。

某些应用程序可能需要其唯一协议的其他 Spn。 例如，SQL Server 需要`MSSQLSvc/hostname` SPN。

下表列出了创建 gMSA 所需的属性。

|gMSA 属性 | 必需值 | 示例 |
|--------------|----------------|--------|
|姓名 | 任何有效的帐户名称。 | `WebApp01` |
|DnsHostName | 附加到帐户名称的域名。 | `WebApp01.contoso.com` |
|ServicePrincipalNames | 至少设置主机 SPN，如有必要，请添加其他协议。 | `'host/WebApp01', 'host/WebApp01.contoso.com'` |
|PrincipalsAllowedToRetrieveManagedPassword | 包含容器托管的安全组。 | `WebApp01Hosts` |

确定 gMSA 的名称后，请在 PowerShell 中运行以下 cmdlet 以创建安全组和 gMSA。

> [!TIP]
> 您需要使用属于**域管理员**安全组或已被委派了 "**创建 GroupManagedServiceAccount 对象**" 权限的帐户来运行以下命令。
> [ADServiceAccount](https://docs.microsoft.com/powershell/module/addsadministration/new-adserviceaccount?view=win10-ps) cmdlet 是来自[远程服务器管理工具](https://aka.ms/rsat)的广告 PowerShell 工具的一部分。

```powershell
# Replace 'WebApp01' and 'contoso.com' with your own gMSA and domain names, respectively

# To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
# To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
# To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

# Create the security group
New-ADGroup -Name "WebApp01 Authorized Hosts" -SamAccountName "WebApp01Hosts" -GroupScope DomainLocal

# Create the gMSA
New-ADServiceAccount -Name "WebApp01" -DnsHostName "WebApp01.contoso.com" -ServicePrincipalNames "host/WebApp01", "host/WebApp01.contoso.com" -PrincipalsAllowedToRetrieveManagedPassword "WebApp01Hosts"

# Add your container hosts to the security group
Add-ADGroupMember -Identity "WebApp01Hosts" -Members "ContainerHost01", "ContainerHost02", "ContainerHost03"
```

我们建议你为你的开发、测试和生产环境创建单独的 gMSA 帐户。

## <a name="prepare-your-container-host"></a>准备容器主机

将运行具有 gMSA 的 Windows 容器的每个容器主机必须已加入域，并且具有检索 gMSA 密码的权限。

1. 将您的计算机加入到您的 Active Directory 域。
2. 确保你的主机属于控制对 gMSA 密码的访问的安全组。
3. 重新启动计算机，使其获取新的组成员身份。
4. 为[Windows Server 的](https://docs.docker.com/install/windows/docker-ee/)windows 10 或 Docker 设置[docker 桌面](https://docs.docker.com/docker-for-windows/install/)。
5. 建议您使用通过运行[Test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount)验证主机是否可以使用 gMSA 帐户。 如果该命令返回**False**，请按照[疑难解答说明进行](gmsa-troubleshooting.md#make-sure-the-host-can-use-the-gmsa)操作。

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

## <a name="create-a-credential-spec"></a>创建凭据规范

凭据规范文件是一个 JSON 文档，其中包含有关你希望容器使用的 gMSA 帐户的元数据。 通过将标识配置与容器映像分开，你可以更改容器使用的 gMSA，只需交换凭据规范文件，无需更改代码。

凭据规范文件是使用加入域的容器主机上的[CredentialSpec PowerShell 模块](https://aka.ms/credspec)创建的。
创建文件后，您可以将其复制到其他容器主机或容器 orchestrator。
凭据规范文件不包含任何机密（如 gMSA 密码），因为容器主机代表容器检索 gMSA。

Docker 希望在 Docker 数据目录中的**CredentialSpecs**目录下找到凭据规范文件。 在默认安装中，你可以在上`C:\ProgramData\Docker\CredentialSpecs`找到此文件夹。

要在容器主机上创建凭据规范文件，请执行以下操作：

1. 安装 RSAT AD PowerShell 工具
    - 对于 Windows Server，请运行**安装-add-windowsfeature 的 RSAT-AD-PowerShell**。
    - 对于 Windows 10 版本1809或更高版本，运行**Add-WindowsCapability-Name "Rsat. 0.0.1.0。 Tools ~**~ ~ ~ ~"。
    - 对于较早版本的 Windows 10， <https://aka.ms/rsat>请参阅。
2. 运行以下 cmdlet 以安装最新版本的[CredentialSpec PowerShell 模块](https://aka.ms/credspec)：

    ```powershell
    Install-Module CredentialSpec
    ```

    如果你的容器主机上没有 internet 访问权限，请`Save-Module CredentialSpec`在连接了 internet 的计算机上运行，并将模块`C:\Program Files\WindowsPowerShell\Modules`文件夹复制到容器`$env:PSModulePath`主机上的其他位置。

3. 运行以下 cmdlet 以创建新的凭据规范文件：

    ```powershell
    New-CredentialSpec -AccountName WebApp01
    ```

    默认情况下，cmdlet 将使用所提供的 gMSA 名称作为容器的计算机帐户创建凭据规范。 将使用文件名的 gMSA 域和帐户名称将文件保存在 Docker CredentialSpecs 目录中。

    如果你作为容器中的辅助 gMSA 运行服务或进程，你可以创建包含其他 gMSA 帐户的凭据规范。 若要执行此操作， `-AdditionalAccounts`请使用参数：

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -AdditionalAccounts LogAgentSvc, OtherSvc
    ```

    有关支持的参数的完整列表，请`Get-Help New-CredentialSpec`运行。

4. 你可以使用以下 cmdlet 显示所有凭据规范及其完整路径的列表：

    ```powershell
    Get-CredentialSpec
    ```

## <a name="next-steps"></a>后续步骤

既然已经设置了您的 gMSA 帐户，您就可以使用它：

- [配置应用](gmsa-configure-app.md)
- [运行容器](gmsa-run-container.md)
- [安排容器](gmsa-orchestrate-containers.md)

如果在安装过程中遇到任何问题，请查看我们的[故障排除指南](gmsa-troubleshooting.md)以了解可能的解决方案。

## <a name="additional-resources"></a>其他资源

- 若要了解有关 gMSAs 的详细信息，请参阅[组托管服务帐户概述](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)。
- 有关视频演示，请从 Ignite 2016 观看[录制的演示](https://youtu.be/cZHPz80I-3s?t=2672)。
