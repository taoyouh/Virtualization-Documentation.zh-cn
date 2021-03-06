---
title: 为 Windows 容器创建 gMSA
description: 如何为 Windows 容器创建组托管服务帐户 (gMSA)。
keywords: docker, 容器, active directory, gmsa, 组托管服务帐户
author: rpsqrd
ms.date: 01/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 36061cfc491dd9dd581d1e6bce92a29e4a6f217d
ms.sourcegitcommit: 530712469552a1ef458883001ee748bab2c65ef7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/26/2020
ms.locfileid: "77628932"
---
# <a name="create-gmsas-for-windows-containers"></a>为 Windows 容器创建 gMSA

基于 Windows 的网络通常使用 Active Directory (AD) 为用户、计算机和其他网络资源之间的身份验证和授权提供便利。 企业应用程序开发人员通常将其应用设计为在其中集成 AD，并在加入域的服务器上运行，以利用集成 Windows 身份验证，这使得用户和其他服务可以轻松地使用其标识自动透明地登录到应用程序。

尽管 Windows 容器无法加入域，但它们仍可使用 Active Directory 域标识来支持各种身份验证方案。

若要实现此目的，你可以将 Windows 容器配置为通过[组托管服务帐户](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview) (gMSA) 来运行。此帐户是 Windows Server 2012 中引入的一种特殊类型的服务帐户，旨在允许多台计算机共享某个标识而无需知道该标识的密码。

通过 gMSA 运行容器时，容器主机将从 Active Directory 域控制器中检索 gMSA 密码，并将其提供给容器实例。 每当容器的计算机帐户 (SYSTEM) 需要访问网络资源时，容器就会使用 gMSA 凭据。

本文介绍了如何开始将 Active Directory 组托管服务帐户用于 Windows 容器。

## <a name="prerequisites"></a>必备条件

若要通过组托管服务帐户运行 Windows 容器，你需要具有以下项：

- 一个 Active Directory 域，其中至少有一台运行 Windows Server 2012 或更高版本的域控制器。 若要使用 gMSA，在林或域级别没有功能要求，但 gMSA 密码只能由运行 Windows Server 2012 或更高版本的域控制器分发。 有关详细信息，请参阅 [gMSA 的 Active Directory 要求](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts#BKMK_gMSA_Req)。
- 创建 gMSA 帐户所需的权限。 若要创建 gMSA 帐户，你需要是域管理员，或者使用已被委派了“创建 msDS-GroupManagedServiceAccount 对象”权限的帐户。
- 访问 Internet，下载 CredentialSpec PowerShell 模块。 如果在无连接的环境中工作，则可先在能够访问 Internet 的计算机上[保存此模块](https://docs.microsoft.com/powershell/module/powershellget/save-module?view=powershell-5.1)，然后将其复制到开发计算机或容器主机。

## <a name="one-time-preparation-of-active-directory"></a>一次性 Active Directory 准备工作

如果尚未在域中创建 gMSA，则需生成密钥分发服务 (KDS) 根密钥。 KDS 负责创建、轮换 gMSA 密码并将其发布到经授权的主机。 当容器主机需要使用 gMSA 来运行容器时，它将与 KDS 联系以检索当前密码。

若要检查是否已创建 KDS 根密钥，请在安装了 AD PowerShell 工具的域控制器或域成员上以域管理员身份运行以下 PowerShell cmdlet：

```powershell
Get-KdsRootKey
```

如果该命令返回了密钥 ID，则一切都已设置完毕，可跳到[创建组托管服务帐户](#create-a-group-managed-service-account)部分。 否则，请继续创建 KDS 根密钥。

在包含多个域控制器的生产环境或测试环境中，以域管理员身份在 PowerShell 中运行以下 cmdlet，以创建 KDS 根密钥。

```powershell
# For production environments
Add-KdsRootKey -EffectiveImmediately
```

尽管该命令暗示密钥会立即生效，但你需要等待 10 小时，KDS 根密钥才会被复制并可供在所有域控制器上使用。

如果域中只有一台域控制器，则可以通过将该密钥设置为在 10 小时前生效来加速此过程。

>[!IMPORTANT]
>请勿在生产环境中使用此方法。

```powershell
# For single-DC test environments ONLY
Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
```

## <a name="create-a-group-managed-service-account"></a>创建组托管服务帐户

使用集成 Windows 身份验证的每个容器都需要至少一个 gMSA。 当以“系统”或“网络服务”身份运行的应用访问网络上的资源时，将使用主要 gMSA。 gMSA 的名称将成为网络上的容器名称，不管分配给该容器的主机名是什么。 如果你希望以不同于容器计算机帐户的身份在容器中运行服务或应用程序，还可以为容器配置其他 gMSA。

创建 gMSA 时，还会创建一个可在许多不同计算机上同时使用的共享标识。 对 gMSA 密码的访问受 Active Directory 访问控制列表保护。 建议为每个 gMSA 帐户创建一个安全组，并将相关容器主机添加到该安全组，以限制对密码的访问。

最后，由于容器不会自动注册任何服务主体名称 (SPN)，因此你需要为 gMSA 帐户手动创建至少一个主机 SPN。

通常，主机或 http SPN 使用与 gMSA 帐户相同的名称进行注册，但如果客户端从负载平衡器后面访问容器化应用程序，或者使用与 gMSA 名称不同的 DNS 名称，则你可能需要使用不同的服务名称。

例如，如果 gMSA 帐户命名为“WebApp01”，但你的用户访问 `mysite.contoso.com` 处的站点，则应在 gMSA 帐户上注册一个 `http/mysite.contoso.com` SPN。

某些应用程序可能需要为其独特的协议使用额外的 SPN。 例如，SQL Server 需要 `MSSQLSvc/hostname` SPN。

下表列出了创建 gMSA 所需的属性。

|gMSA 属性 | 必需的值 | 示例 |
|--------------|----------------|--------|
|名称 | 任何有效的帐户名称。 | `WebApp01` |
|DnsHostName | 追加到帐户名称的域名。 | `WebApp01.contoso.com` |
|ServicePrincipalNames | 至少设置主机 SPN，并根据需要添加其他协议。 | `'host/WebApp01', 'host/WebApp01.contoso.com'` |
|PrincipalsAllowedToRetrieveManagedPassword | 包含你的容器主机的安全组。 | `WebApp01Hosts` |

确定 gMSA 的名称后，在 PowerShell 中运行以下 cmdlet 来创建安全组和 gMSA。

> [!TIP]
> 你需要使用属于**域管理员**安全组的帐户或已被委派了“创建 msDS-GroupManagedServiceAccount 对象”权限的帐户来运行以下命令。
> [New-ADServiceAccount](https://docs.microsoft.com/powershell/module/addsadministration/new-adserviceaccount?view=win10-ps) cmdlet 是[远程服务器管理工具](https://aka.ms/rsat)中的 AD PowerShell 工具的一部分。

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
Add-ADGroupMember -Identity "WebApp01Hosts" -Members "ContainerHost01$", "ContainerHost02$", "ContainerHost03$"
```

建议分别为开发环境、测试环境和生产环境创建单独的 gMSA 帐户。

## <a name="prepare-your-container-host"></a>准备容器主机

将通过 gMSA 运行 Windows 容器的每个容器主机都必须加入域，并且必须有权检索 gMSA 密码。

1. 将你的计算机加入 Active Directory 域。
2. 确保你的主机属于控制着 gMSA 密码访问的安全组。
3. 重启计算机，使其获得新的组成员身份。
4. 安装 [Docker Desktop for Windows 10](https://docs.docker.com/docker-for-windows/install/) 或 [Docker for Windows Server](https://docs.docker.com/install/windows/docker-ee/)。
5. （建议）运行 [Test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount) 来验证主机是否可以使用 gMSA 帐户。 如果该命令返回了 **False**，请遵循[故障排除说明](gmsa-troubleshooting.md#make-sure-the-host-can-use-the-gmsa)进行操作。

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

## <a name="create-a-credential-spec"></a>创建凭据规范

凭据规范文件是一个 JSON 文档，其中包含有关你希望容器使用的 gMSA 帐户的元数据。 通过将标识配置与容器映像分离，你可以对容器使用哪个 gMSA 进行更改，只需简单地交换凭据规范文件即可，无需更改代码。

凭据规范文件是使用已加入域的容器主机上的 [CredentialSpec PowerShell 模块](https://aka.ms/credspec)创建的。
创建该文件后，可以将其复制到其他容器主机或容器业务流程协调程序。
凭据规范文件不包含任何机密（例如 gMSA 密码），因为容器主机代表容器来检索 gMSA。

Docker 会在 Docker 数据目录中的 **CredentialSpecs** 目录下查找凭据规范文件。 在默认安装中，可以在 `C:\ProgramData\Docker\CredentialSpecs` 中找到此文件夹。

若要在容器主机上创建凭据规范文件，请执行以下操作：

1. 安装 RSAT AD PowerShell 工具
    - 对于 Windows Server，请运行 **Install-WindowsFeature RSAT-AD-PowerShell**。
    - 对于 Windows 10 版本 1809 或更高版本，请运行 **Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'** 。
    - 对于较旧版本的 Windows 10，请参阅 <https://aka.ms/rsat>。
2. 运行以下 cmdlet 来安装最新版本的 [CredentialSpec PowerShell 模块](https://aka.ms/credspec)：

    ```powershell
    Install-Module CredentialSpec
    ```

    如果你的容器主机上没有 Internet 访问权限，请在连接到 Internet 的计算机上运行 `Save-Module CredentialSpec`，并将模块文件夹复制到 `C:\Program Files\WindowsPowerShell\Modules` 或容器主机上 `$env:PSModulePath` 中的其他位置。

3. 运行以下 cmdlet 来创建新的凭据规范文件：

    ```powershell
    New-CredentialSpec -AccountName WebApp01
    ```

    默认情况下，该 cmdlet 将使用提供的 gMSA 名称作为容器的计算机帐户来创建凭据规范。 该文件将使用 gMSA 域和帐户名称作为文件名保存在 Docker CredentialSpecs 目录中。

    如果要将文件保存到其他目录，请使用 `-Path` 参数：

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -Path "C:\MyFolder\WebApp01_CredSpec.json"
    ```

    如果要在容器中以另一个 gMSA 的身份运行服务或进程，则还可以创建包含其他 gMSA 帐户的凭据规范。 若要执行该操作，请使用 `-AdditionalAccounts` 参数：

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -AdditionalAccounts LogAgentSvc, OtherSvc
    ```

    若要获取支持的参数的完整列表，请运行 `Get-Help New-CredentialSpec -Full`。

4. 可以使用以下 cmdlet 显示所有凭据规范的列表及其完整路径：

    ```powershell
    Get-CredentialSpec
    ```

## <a name="next-steps"></a>后续步骤

设置 gMSA 帐户后，可以使用它来执行以下操作：

- [配置应用](gmsa-configure-app.md)
- [运行容器](gmsa-run-container.md)
- [协调容器](gmsa-orchestrate-containers.md)

如果在设置过程中遇到任何问题，请查看我们的[故障排除指南](gmsa-troubleshooting.md)，了解可能的解决方案。

## <a name="additional-resources"></a>其他资源

- 若要详细了解 gMSA，请参阅[组托管服务帐户概述](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)。
- 有关视频演示，请观看 Ignite 2016 中的[录制的演示](https://youtu.be/cZHPz80I-3s?t=2672)。
