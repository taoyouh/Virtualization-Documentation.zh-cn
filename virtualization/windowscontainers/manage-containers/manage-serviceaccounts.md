---
title: 组托管服务帐户转换为 Windows 容器
description: 组托管服务帐户转换为 Windows 容器
keywords: docker，容器，active directory gmsa
author: rpsqrd
ms.date: 05/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: d4a59f351cad36219e8289f9d58b55250c99fc6e
ms.sourcegitcommit: 34d8b2ca5eebcbdb6958560b1f4250763bee5b48
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "9620895"
---
# <a name="group-managed-service-accounts-for-windows-containers"></a>组托管服务帐户转换为 Windows 容器

基于 Windows 的网络通常使用 Active Directory (AD) 来促进身份验证和授权之间用户、 计算机和其他网络资源。 企业应用程序开发人员通常设计为 AD-集成且已加入域的服务器，以充分利用集成 Windows 身份验证，轻松地为用户和其他服务来自动且透明登录到上运行其应用具有其标识的应用程序。

虽然 Windows 容器不能加入域，他们仍然可以使用 Active Directory 域标识，以支持各种身份验证方案。

若要实现此目的，你可以配置若要使用[组托管服务帐户](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)(gMSA)，这是一种特殊类型的引入 Windows Server 2012 中设计为允许多台计算机而无需共享标识的服务帐户运行的 Windows 容器若要了解其密码。

使用 gMSA 运行容器时，容器主机将从 Active Directory 域控制器中检索的 gMSA 密码，并赋予其到容器实例。 每当其计算机帐户 （系统） 需要访问网络资源时，该容器将使用 gMSA 凭据。

本文介绍了如何开始使用 Windows 容器的 Active Directory 组托管服务帐户。

## <a name="prerequisites"></a>系统必备

若要使用组托管服务帐户运行 Windows 的容器，你将需要：

- 带有至少一个域控制器运行 Windows Server 2012 或更高版本的 Active Directory 域。 没有林或域功能级别要求使用 Gmsa，但 gMSA 密码仅可以通过运行 Windows Server 2012 域控制器分布式或更高版本。 有关详细信息，请参阅[Gmsa 的 Active Directory 要求](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts#BKMK_gMSA_Req)。
- 创建 gMSA 帐户的权限。 若要创建 gMSA 帐户，你将需要是域管理员或使用的帐户是否已*创建了 GroupManagedServiceAccount 对象*权限的委派。
- 访问 internet 下载的 CredentialSpec PowerShell 模块。 如果你正在断开连接的环境中，你可以在 internet 的计算机上[保存模块](https://docs.microsoft.com/powershell/module/powershellget/save-module?view=powershell-5.1)访问并将其复制到你的开发计算机或容器主机。

## <a name="one-time-preparation-of-active-directory"></a>Active directory 的一次性准备工作

如果你尚未创建 gMSA 域中，你将需要生成密钥分发服务 (KDS) 根密钥。 KDS 负责创建、 旋转和释放授权主机对 gMSA 密码。 当容器主机需要使用 gMSA 运行的容器时，它将与 KDS 来检索当前的密码。

若要检查是否 KDS 根密钥已创建，使用安装了 AD PowerShell 工具为上一个域控制器或域成员的域管理员运行以下 PowerShell cmdlet:

```powershell
Get-KdsRootKey
```

如果命令返回一个键是所有组 ID，并且可以跳到[创建组托管服务帐户](#create-a-group-managed-service-account)部分。 否则，继续创建 KDS 根密钥。

在生产环境或测试环境中的使用多个域控制器中，运行以下 cmdlet 在 PowerShell 中为域管理员创建 KDS 根密钥。

```powershell
# For production environments
Add-KdsRootKey -EffectiveImmediately
```

尽管命令键将立即生效，你将需要等待 10 小时才能 KDS 根密钥是复制和适用于所有域控制器上的使用。

如果你的域中只有一个域控制器，你可以通过设置生效前 10 小时的密钥加速该过程。

>[!IMPORTANT]
>请勿在生产环境中使用此方法。

```powershell
# For single-DC test environments ONLY
Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
```

## <a name="create-a-group-managed-service-account"></a>创建组托管服务帐户

使用集成 Windows 身份验证每个容器需要至少一个 gMSA。 作为一种系统或网络服务运行的应用访问网络上的资源时，使用主 gMSA。 GMSA 的名称会在网络上，而不考虑分配给容器的主机名的容器的名称。 在你想要在容器中运行的服务或应用作为容器计算机帐户从不同的标识的情况下，也可以与其他 Gmsa，配置容器。

创建 gMSA 时，你还创建可以跨许多不同的计算机的同时使用的共享的标识。 访问 gMSA 密码保护的 Active Directory 访问控制列表。 我们建议创建每个 gMSA 帐户的安全组并添加到安全组相关的容器主机，以限制访问的密码。

最后，由于容器不自动注册任何服务主体名称 (SPN)，你将需要手动创建 gMSA 帐户至少的主机 SPN。

通常情况下，主机或 http SPN 注册使用相同的名称作为 gMSA 帐户，但你可能需要使用不同的服务名称，如果客户端访问后面负载平衡器或 DNS 名称不同于 gMSA 名称的容器化应用程序。

例如，如果 gMSA 帐户的名称为"WebApp01"，但你的用户访问该站点在`mysite.contoso.com`，你应该注册`http/mysite.contoso.com`SPN gMSA 帐户上的。

某些应用程序可能需要其他 Spn 其唯一的协议。 例如，SQL Server 需要`MSSQLSvc/hostname`SPN。

下表列出了用于创建 gMSA 所需的属性。

|gMSA 属性 | 所需的值 | 示例 |
|--------------|----------------|--------|
|姓名 | 任何有效的帐户的名称。 | `WebApp01` |
|仅 | 附加到帐户名称的域名。 | `WebApp01.contoso.com` |
|ServicePrincipalNames | 至少设置主机 SPN，根据需要添加其他协议。 | `'host/WebApp01', 'host/WebApp01.contoso.com'` |
|PrincipalsAllowedToRetrieveManagedPassword | 包含在容器主机的安全组。 | `WebApp01Hosts` |

一旦确定你 gMSA，以创建安全组和 gMSA 的 PowerShell 中运行以下 cmdlet 的名称。

> [!TIP]
> 你将需要使用的帐户是否属于**域管理员**安全组或已委派运行以下命令**创建了 GroupManagedServiceAccount 对象**权限。
> [新建 ADServiceAccount](https://docs.microsoft.com/powershell/module/addsadministration/new-adserviceaccount?view=win10-ps) cmdlet 是从[远程服务器管理工具](https://aka.ms/rsat)AD PowerShell 工具的一部分。

```powershell
# Replace 'WebApp01' and 'contoso.com' with your own gMSA and domain names, respectively

# To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
# To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
# To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

# Create the security group
New-ADGroup -Name "WebApp01 Authorized Hosts" -SamAccountName "WebApp01Hosts" -Scope DomainLocal

# Create the gMSA
New-ADServiceAccount -Name "WebApp01" -DnsHostName "WebApp01.contoso.com" -ServicePrincipalNames "host/WebApp01", "host/WebApp01.contoso.com" -PrincipalsAllowedToRetrieveManagedPassword "WebApp01Hosts"

# Add your container hosts to the security group
Add-ADGroupMember -Identity "WebApp01Hosts" -Members "ContainerHost01", "ContainerHost02", "ContainerHost03"
```

我们建议为开发人员、 测试和生产环境中创建单独的 gMSA 帐户。

## <a name="prepare-your-container-host"></a>准备容器主机

将使用 gMSA 运行 Windows 容器的每个容器主机必须加入域，并有权将检索 gMSA 密码。

1. 将计算机加入到 Active Directory 域。
2. 确保你的主机属于控制访问 gMSA 密码的安全组。
3. 重新启动计算机，因此它获取其新的组成员身份。
4. 设置[Docker 桌面适用于 Windows 10](https://docs.docker.com/docker-for-windows/install/)或[适用于 Windows Server 的 Docker](https://docs.docker.com/install/windows/docker-ee/)。
5. （推荐）验证主机可以通过运行[测试 ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount)使用 gMSA 帐户。 如果命令返回**False**，请参阅诊断步骤[疑难解答](#troubleshooting)部分。

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

## <a name="create-a-credential-spec"></a>创建凭据规范

凭据规格文件是包含你想要使用的容器的 gMSA 帐户相关的元数据的 JSON 文档。 通过将标识配置分开的容器映像，你可以不更改容器使用通过只需更换凭据规范文件，需要更改任何代码的 gMSA。

已加入域的容器主机上使用[CredentialSpec PowerShell 模块](https://aka.ms/credspec)创建凭据规范文件。
创建该文件后，你可以将其复制到其他容器主机或容器 orchestrator。
凭据规范文件不包含任何密钥，如 gMSA 密码，因为在容器主机检索 gMSA 代表该容器。

Docker 希望 Docker 数据目录中找到**CredentialSpecs**目录下的凭据规格文件。 在默认安装中，你会发现此文件夹`C:\ProgramData\Docker\CredentialSpecs`。

若要创建容器主机上的凭据规格文件：

1. 安装 RSAT AD PowerShell 工具
    - 适用于 Windows Server，运行**安装 WindowsFeature RSAT AD PowerShell**。
    - 对于 Windows 10 版本 1809年或更高版本，运行**安装 WindowsCapability-联机 Rsat.ActiveDirectory.DS-LDS.Tools~~~0.0.1.0**。
    - 适用于较旧版本的 Windows 10，请参阅<https://aka.ms/rsat>。
2. 运行以下 cmdlet 安装最新版本的[CredentialSpec PowerShell 模块](https://aka.ms/credspec)：

    ```powershell
    Install-Module CredentialSpec
    ```

    如果容器主机上没有 internet 访问权限，运行`Save-Module CredentialSpec`在 internet 连接的计算机上，并将复制到的模块文件夹`C:\Program Files\WindowsPowerShell\Modules`或在其他位置`$env:PSModulePath`在容器主机上。

3. 运行以下 cmdlet 创建新的凭据规格文件：

    ```powershell
    New-CredentialSpec -AccountName WebApp01
    ```

    默认情况下，该 cmdlet 将创建的容器用提供的 gMSA 的名称作为计算机帐户的凭据规范。 将使用文件名的 gMSA 域和帐户名称的 Docker CredentialSpecs 目录中保存该文件。

    你可以创建包含其他 gMSA 帐户，如果你正在运行的服务或进程作为辅助 gMSA 容器中的凭据规范。 若要执行该操作，请使用`-AdditionalAccounts`参数：

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -AdditionalAccounts LogAgentSvc, OtherSvc
    ```

    有关支持的参数的完整列表，运行`Get-Help New-CredentialSpec`。

4. 你可以显示所有凭据规格和使用以下 cmdlet 其完整路径的列表：

    ```powershell
    Get-CredentialSpec
    ```

## <a name="configure-your-application-to-use-the-gmsa"></a>配置应用程序使用 gMSA

在典型的配置中，容器只会获得一个 gMSA 帐户进行身份验证到网络资源的容器计算机帐户时使用。 这意味着你的应用将需要作为**本地系统**或**网络服务**运行，是否需要使用 gMSA 标识。

### <a name="run-an-iis-app-pool-as-network-service"></a>作为网络服务运行 IIS 应用程序池

如果你正在托管你的容器中的 IIS 网站，只需要怎么做才能利用 gMSA 设置你的应用程序池标识为**网络服务**。 你可以在 Dockerfile 中执行，通过添加以下命令：

```dockerfile
RUN (Get-IISAppPool DefaultAppPool).ProcessModel.IdentityType = "NetworkService"
```

如果你的 IIS 应用池以前使用静态的用户凭据，考虑 gMSA 作为这些凭据的替代。 你可以更改开发人员、 测试和生产环境之间 gMSA，IIS 将自动选择设置的当前标识而无需更改的容器映像。

### <a name="run-a-windows-service-as-network-service"></a>作为网络服务运行 Windows 服务

如果容器化的应用作为 Windows 服务在运行，你可以设置要作为你 Dockerfile 中的**网络服务**运行的服务：

```dockerfile
RUN sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""
```

### <a name="run-arbitrary-console-apps-as-network-service"></a>作为网络服务运行任意控制台应用

对于未托管 IIS 或服务管理器中的通用控制台应用，它通常是最简单的方法作为**网络服务**运行容器，以便应用会自动继承 gMSA 上下文。 此功能是自 Windows Server 版本 1709年起可用。

将以下行添加到你的 Dockerfile，以使其默认为网络服务运行：

```dockerfile
USER "NT AUTHORITY\NETWORK SERVICE"
```

也可以连接到容器为网络服务分别为一次性`docker exec`。 这在如果正在运行的容器的连接问题疑难解答作为网络服务通常不运行容器时十分有用。

```powershell
# Opens an interactive PowerShell console in the container (id = 85d) as the Network Service account
docker exec -it --user "NT AUTHORITY\NETWORK SERVICE" 85d powershell
```

## <a name="run-a-container-with-a-gmsa"></a>使用 gMSA 运行的容器

若要使用 gMSA 运行的容器，提供的凭据规格文件的`--security-opt`参数的[docker 运行](https://docs.docker.com/engine/reference/run)：

```powershell
# For Windows Server 2016, change the image name to mcr.microsoft.com/windows/servercore:ltsc2016
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

>[!IMPORTANT]
>在 Windows Server 2016 版本 1709年和 1803年，该容器的主机名必须匹配的 gMSA 短名称。

在上一示例中，gMSA SAM 帐户名是"webapp01，"，因此容器主机名也称为"webapp01。"

在 Windows Server 2019 上及更高版本，主机名字段不是必需的但容器将仍然标识其本身的 gMSA 名称而不是主机名，即使你显式提供一个不同。

若要检查 gMSA 正常工作，请在容器中运行以下 cmdlet:

```powershell
# Replace contoso.com with your own domain
PS C:\> nltest /sc_verify:contoso.com

Flags: b0 HAS_IP  HAS_TIMESERV
Trusted DC Name \\dc01.contoso.com
Trusted DC Connection Status Status = 0 0x0 NERR_Success
Trust Verification Status = 0 0x0 NERR_Success
The command completed successfully
```

如果受信任 DC 连接状态和信任验证状态不是`NERR_Success`，查看有关如何调试问题的提示[疑难解答](#troubleshooting)部分。

你可以通过运行以下命令并检查客户端名称验证从容器内的 gMSA 标识：

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

若要打开 PowerShell 或作为 gMSA 帐户的另一个控制台应用，你可以询问容器，以在网络服务帐户而不是普通 ContainerAdministrator （或为 NanoServer ContainerUser） 帐户下运行：

```powershell
# NOTE: you can only run as Network Service or SYSTEM on Windows Server 1709 and later
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 --user "NT AUTHORITY\NETWORK SERVICE" -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

当你为网络服务运行时，你可以通过尝试连接到 SYSVOL 域控制器上作为 gMSA 测试网络身份验证：

```powershell
# This command should succeed if you're successfully running as the gMSA
PS C:\> dir \\contoso.com\SYSVOL


    Directory: \\contoso.com\sysvol


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d----l        2/27/2019   8:09 PM                contoso.com
```

## <a name="orchestrate-containers-with-gmsa"></a>协调容器与 gMSA

在生产环境中，你将经常使用容器 orchestrator 部署和管理你的应用和服务。 每个 orchestrator 具有其自己的管理模式，并且负责接受凭据规格，以向 Windows 容器平台。

当你在协调容器与 Gmsa 时，请确保：

> [!div class="checklist"]
> * 可被安排为使用 Gmsa 运行容器的所有容器主机都是加入域
> * 容器主机有权将检索的所有 Gmsa 容器使用的密码
> * 凭据规格创建并上传到 orchestrator 或文件复制到每个容器主机，具体取决于 orchestrator 的首选方式来处理它们。
> * 容器网络允许容器与 Active Directory 域控制器来检索 gMSA 票证进行通信

### <a name="how-to-use-gmsa-with-service-fabric"></a>如何使用 Service Fabric gMSA

Service Fabric 支持使用 gMSA 运行 Windows 容器时应用程序清单中指定的凭据规格位置。 你将需要创建凭据规格文件，并将放置在**CredentialSpecs**子目录的每个主机上的 Docker 数据目录，以便 Service Fabric 可以找到它。 你可以运行**获取 CredentialSpec** cmdlet，一部分的[CredentialSpec PowerShell 模块](https://aka.ms/credspec)，以验证凭据规范是否正确的位置。

请参阅[快速入门： 将 Windows 容器部署到 Service Fabric](https://docs.microsoft.com/azure/service-fabric/service-fabric-quickstart-containers)为并[设置 Windows 上运行的容器 Service Fabric gMSA](https://docs.microsoft.com/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers)有关如何配置你的应用程序的详细信息。

### <a name="how-to-use-gmsa-with-docker-swarm"></a>使用 Docker 群的 gMSA

若要使用由 Docker 群的容器使用 gMSA，运行[docker 服务创建](https://docs.docker.com/engine/reference/commandline/service_create/)命令`--credential-spec`参数：

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

请参阅有关如何使用凭据规格与 Docker 服务的详细信息的[Docker 群的示例](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only)。

### <a name="how-to-use-gmsa-with-kubernetes"></a>如何与 Kubernetes 使用 gMSA

作为 alpha 功能在 Kubernetes 1.14 提供了有关计划与 Gmsa Kubernetes 中的 Windows 容器的支持。 有关此功能以及如何在 Kubernetes 分发测试它的最新信息，请参阅[Windows pod 和容器配置 gMSA](https://kubernetes.io/docs/tasks/configure-pod-container/configure-gmsa) 。

## <a name="example-uses"></a>示例用法

### <a name="sql-connection-strings"></a>SQL 连接字符串

当服务作为容器中的本地系统或网络服务运行时，它可以使用 Windows 集成身份验证连接到 Microsoft SQL Server。

下面是一个使用容器标识对 SQL Server 进行身份验证的连接字符串的示例：

```sql
Server=sql.contoso.com;Database=MusicStore;Integrated Security=True;MultipleActiveResultSets=True;Connect Timeout=30
```

在 Microsoft SQL Server上，使用域和 gMSA 名称（以 $ 结尾）创建一个登录名。 创建登录后，你可以将其添加到数据库上的用户，并向其提供适当的访问权限。

示例：

```sql
CREATE LOGIN "DEMO\WebApplication1$"
    FROM WINDOWS
    WITH DEFAULT_DATABASE = "MusicStore"
GO

USE MusicStore
GO
CREATE USER WebApplication1 FOR LOGIN "DEMO\WebApplication1$"
GO

EXEC sp_addrolemember 'db_datareader', 'WebApplication1'
EXEC sp_addrolemember 'db_datawriter', 'WebApplication1'
```

若要了解实际操作，请查看[录制演示](https://youtu.be/cZHPz80I-3s?t=2672)可从会话中的 Microsoft Ignite 2016"遍历路径到此-转换到容器的工作负荷。"

## <a name="troubleshooting"></a>疑难解答

### <a name="known-issues"></a>已知问题

#### <a name="container-hostname-must-match-the-gmsa-name-for-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>容器主机名必须匹配适用于 Windows Server 2016 和 Windows 10 版本 1709年和 1803年的 gMSA 名称

如果你运行的 Windows Server 2016 中，版本 1709年或 1803、 你的容器的主机名必须匹配你 gMSA SAM 帐户名称。

当主机名与 gMSA 名称不匹配时，入站 NTLM 身份验证请求和名称/SID 翻译 （使用很多库，如 ASP.NET 成员资格角色提供程序） 将失败。 Kerberos 将继续正常工作，即使的主机名和 gMSA 名称不匹配。

在 Windows Server 2019，其中该容器将现在始终使用其 gMSA 名称而不考虑分配的主机名在网络已修复此限制。

#### <a name="using-a-gmsa-with-more-than-one-container-simultaneously-leads-to-intermittent-failures-on-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>在 Windows Server 2016 和 Windows 10 版本 1709年和 1803年上同时使用多个容器使用 gMSA 导致的间歇性故障

由于使用的相同主机名所需的所有容器，第二个问题将影响在 Windows Server 2019 之前的 Windows 和 Windows 10 版本 1809年的版本。 当多个容器都分配了相同的标识和主机名时，两个容器会同时交谈相同的域控制器，可能会出现争用情况。 当另一个容器交谈相同的域控制器时，它将与使用相同的标识任何之前容器取消通信。 这可能导致间歇性身份验证失败，并且有时可以在运行时观察为信任失败`nltest /sc_verify:contoso.com`容器内。

我们将更改 Windows Server 2019，可以从计算机名称，分离的容器标识中的行为允许多个容器，同时使用同一个 gMSA。

#### <a name="you-cant-use-gmsas-with-hyper-v-isolated-containers-on-windows-10-versions-1703-1709-and-1803"></a>不能与 Windows 10 版本 1703年、 1709年和 1803年上的 HYPER-V 隔离容器使用 Gmsa

容器初始化将挂起或失败时尝试使用 Windows 10 和 Windows Server 版本 1703年、 1709年和 1803年上的 HYPER-V 隔离容器 gMSA。

在 Windows Server 2019 和 Windows 10 版本 1809年中已修复此 bug。 你还可以在 Windows Server 2016 和 Windows 10 版本 1607年上，使用 Gmsa 运行 HYPER-V 隔离容器。

### <a name="general-troubleshooting-guidance"></a>一般疑难解答指南

如果你正在使用 gMSA 运行容器时遇到错误，按照以下说明可能有助于你确定的根本原因。

#### <a name="make-sure-the-host-can-use-the-gmsa"></a>请确保在主机可以使用 gMSA

1. 验证主机是加入域，并可以访问域控制器。
2. 从 RSAT 安装 AD PowerShell 工具和运行[测试 ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount)查看计算机是否具有检索 gMSA 的访问权限。 该 cmdlet 将返回**False**，如果计算机没有访问 gMSA 密码。

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

3. 如果**测试 ADServiceAccount**返回**False**，验证主机所属可以访问 gMSA 密码的安全组。

    ```powershell
    # Get the current computer's group membership
    Get-ADComputer $env:computername | Get-ADPrincipalGroupMembership | Select-Object DistinguishedName

    # Get the groups allowed to retrieve the gMSA password
    # Change "WebApp01" for your own gMSA name
    (Get-ADServiceAccount WebApp01 -Properties PrincipalsAllowedToRetrieveManagedPassword).PrincipalsAllowedToRetrieveManagedPassword
    ```

4. 如果你的主机属于一个安全组授权检索 gMSA 密码，但仍然失败的**测试 ADServiceAccount**，你可能需要重新启动计算机以获取新票证反映其当前的组成员身份。

#### <a name="check-the-credential-spec-file"></a>检查凭据规范文件

1. 要在计算机上找到完整的凭据规格的[CredentialSpec PowerShell 模块](https://aka.ms/credspec)中运行**获取 CredentialSpec** 。 凭据规格必须存储在 Docker 根目录下的"CredentialSpecs"目录中。 通过运行**docker 信息-f，可以在根目录查找 Docker"{{。DockerRootDir}}"**。
2. 打开 CredentialSpec 文件，请确保正确填写以下字段：
    - **Sid**: gMSA 帐户的 SID
    - **MachineAccountName**: gMSA SAM 帐户名称 （不包括完整域名或美元符号）
    - **DnsTreeName**： 你的 Active Directory 林中的 FQDN
    - **DnsName**: gMSA 所属的域的 FQDN
    - **NetBiosName**: gMSA 所属的域的 NETBIOS 名称
    - **GroupManagedServiceAccounts/名称**： gMSA SAM 帐户名称 （不包括完整域名或美元符号）
    - **GroupManagedServiceAccounts/范围**： 一个条目域 FQDN，一个用于 NETBIOS

    输入应如下所示的完整凭据规范下面的示例：

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

3. 验证凭据规格文件的路径正确编排解决方案。 如果你使用 Docker，请确保运行命令的容器包括`--security-opt="credentialspec=file://NAME.json"`、"NAME.json"的名称输出替换通过**获取 CredentialSpec**。 该名称是平面文件名称，相对于 Docker 根目录下 CredentialSpecs 文件夹。

#### <a name="check-the-container"></a>检查容器

1. 如果你运行的版本的 Windows Server 2019 之前的 Windows 或 Windows 10 版本 1809，容器主机名必须匹配的 gMSA 名称。 确保`--hostname`参数匹配的 gMSA 短名称 （没有域组件; 例如，"webapp01"而不是"webapp01.contoso.com"）。

2. 检查要验证容器的容器网络配置可以解决和访问 gMSA 的域的域控制器。 在容器中的错误配置的 DNS 服务器是标识问题的常见原因。

3. 检查是否在容器具有有效的连接到域通过在容器中运行以下 cmdlet (使用`docker exec`或等效项):

    ```powershell
    nltest /sc_verify:contoso.com
    ```

    信任验证应返回`NERR_SUCCESS`如果提供 gMSA 并且网络连接允许容器到域通信。 如果失败，验证主机和容器的网络配置。 二者都需要能够与域控制器通信。

4. 确保你的应用[配置为使用 gMSA](#configure-your-application-to-use-the-gmsa)。 当你使用 gMSA，不会改变容器内的用户帐户。 相反，在系统帐户使用 gMSA 交谈其他网络资源。 这意味着你的应用将需要为网络服务或本地系统利用 gMSA 标识运行。

    > [!TIP]
    > 如果你运行`whoami`或使用其他工具来确定你当前的用户上下文在容器中，你将不会看到 gMSA 名称本身。 这是因为你始终登录到容器而不是域标识的本地用户。 每当它交谈网络资源，这正是你的应用需要作为网络服务或本地系统运行的计算机帐户使用 gMSA。

5. 最后，如果你的容器似乎正确配置，但用户或其他服务无法自动验证到容器化应用，查看你 gMSA 帐户上的 Spn。 客户端将它们达到你的应用程序的名称查找 gMSA 帐户。 这可能意味着，你需要其他`host`Spn 你 gMSA 如果，例如，客户端连接到你的应用通过负载平衡器或不同的 DNS 名称。

## <a name="additional-resources"></a>其他资源

- [组托管服务帐户概述](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)
