---
title: Windows 容器的组托管服务帐户
description: Windows 容器的组托管服务帐户
keywords: docker、容器、active directory、gmsa
author: rpsqrd
ms.date: 08/02/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: ec57152cf077f5007f4bf44a9ec902941c3bc749
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998354"
---
# <a name="group-managed-service-accounts-for-windows-containers"></a>Windows 容器的组托管服务帐户

基于 Windows 的网络通常使用 Active Directory (AD) 来促进用户、计算机和其他网络资源之间的身份验证和授权。 企业应用程序开发人员通常将其应用设计为在加入域的服务器上进行广告集成和运行, 以利用集成的 Windows 身份验证, 从而使用户和其他服务能够轻松、透明地登录到应用程序与其标识。

虽然 Windows 容器不能加入域, 但仍可使用 Active Directory 域标识支持各种身份验证方案。

若要实现此目的, 你可以将 Windows 容器配置为使用[组托管服务帐户](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)(gMSA) 运行, 这是 windows Server 2012 中引入的一种特殊类型的服务帐户, 旨在允许多台计算机共享标识而无需了解其密码。

当你使用 gMSA 运行容器时, 容器主机从 Active Directory 域控制器检索 gMSA 密码, 并将其提供给容器实例。 当容器的计算机帐户 (系统) 需要访问网络资源时, 该容器将使用 gMSA 凭据。

本文介绍如何开始在 Windows 容器中使用 Active Directory 组托管服务帐户。

## <a name="prerequisites"></a>系统必备

若要运行具有 "组托管服务" 帐户的 Windows 容器, 你将需要以下各项:

- 至少有一个运行 Windows Server 2012 或更高版本的域控制器的 Active Directory 域。 没有可使用 gMSAs 的林或域功能级别要求, 但 gMSA 密码只能由运行 Windows Server 2012 或更高版本的域控制器分配。 有关详细信息, 请参阅[gMSAs 的 Active Directory 要求](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts#BKMK_gMSA_Req)。
- 创建 gMSA 帐户的权限。 要创建 gMSA 帐户, 您需要是域管理员或使用已被委派了*Create GroupManagedServiceAccount 对象*权限的帐户。
- 访问 internet 以下载 CredentialSpec PowerShell 模块。 如果你在断开连接的环境中工作, 则可以[将该模块保存](https://docs.microsoft.com/powershell/module/powershellget/save-module?view=powershell-5.1)到具有 internet 访问的计算机上, 并将其复制到开发计算机或容器主机。

## <a name="one-time-preparation-of-active-directory"></a>Active Directory 的一次性准备

如果你的域中尚未创建 gMSA, 你将需要生成密钥分发服务 (KDS) 根密钥。 KDS 负责创建、旋转 gMSA 密码并将其发布到授权的主机。 当容器主机需要使用 gMSA 运行容器时, 它将与 KDS 联系以检索当前密码。

若要检查 KDS 根密钥是否已创建, 请在安装了广告 PowerShell 工具的域控制器或域成员上运行以下 PowerShell cmdlet 作为域管理员:

```powershell
Get-KdsRootKey
```

如果该命令返回键 ID, 则 "全部" 已设置完毕, 可向前跳到 "[创建组托管服务帐户](#create-a-group-managed-service-account)" 部分。 否则, 请继续创建 KDS 根密钥。

在具有多个域控制器的生产环境或测试环境中, 以域管理员身份在 PowerShell 中运行以下 cmdlet 以创建 KDS 根密钥。

```powershell
# For production environments
Add-KdsRootKey -EffectiveImmediately
```

尽管该命令意味着密钥将立即生效, 但你需要等待10小时才能复制 KDS 根密钥并使其可在所有域控制器上使用。

如果你的域中仅有一个域控制器, 你可以通过将该密钥设置为 "有效10小时前" 来加速此过程。

>[!IMPORTANT]
>不要在生产环境中使用此技术。

```powershell
# For single-DC test environments ONLY
Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
```

## <a name="create-a-group-managed-service-account"></a>创建组托管服务帐户

每个使用集成 Windows 身份验证的容器都需要至少一个 gMSA。 只要作为系统或网络服务运行的应用访问网络资源, 就会使用主 gMSA。 无论分配给容器的主机名是什么, gMSA 的名称将成为网络上容器的名称。 也可以使用其他 gMSAs 配置容器, 以防你希望在容器中以不同于容器计算机帐户的标识运行服务或应用程序。

创建 gMSA 时, 你还会创建可在多台不同计算机之间同时使用的共享标识。 对 gMSA 密码的访问受 Active Directory 访问控制列表的保护。 我们建议为每个 gMSA 帐户创建一个安全组, 并将相关的容器主机添加到安全组, 以限制对该密码的访问权限。

最后, 由于容器不会自动注册任何服务主体名称 (SPN), 因此你需要为你的 gMSA 帐户手动创建至少一个主机 SPN。

通常, 使用与 gMSA 帐户相同的名称注册主机或 http SPN, 但如果客户从负载平衡器后访问容器的应用程序或与 gMSA 名称不同的 DNS 名称, 可能需要使用不同的服务名称。

例如, 如果 gMSA 帐户名为 "WebApp01", 但你的用户在访问该网站`mysite.contoso.com`时, 你应在`http/mysite.contoso.com` gMSA 帐户上注册 SPN。

某些应用程序可能需要其唯一协议的其他 Spn。 例如, SQL Server 需要`MSSQLSvc/hostname` SPN。

下表列出了创建 gMSA 所需的属性。

|gMSA 属性 | 必需值 | 示例 |
|--------------|----------------|--------|
|姓名 | 任何有效的帐户名称。 | `WebApp01` |
|DnsHostName | 附加到帐户名称的域名。 | `WebApp01.contoso.com` |
|ServicePrincipalNames | 至少设置主机 SPN, 如有必要, 请添加其他协议。 | `'host/WebApp01', 'host/WebApp01.contoso.com'` |
|PrincipalsAllowedToRetrieveManagedPassword | 包含容器托管的安全组。 | `WebApp01Hosts` |

确定 gMSA 的名称后, 请在 PowerShell 中运行以下 cmdlet 以创建安全组和 gMSA。

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

将运行具有 gMSA 的 Windows 容器的每个容器主机必须已加入域, 并且具有检索 gMSA 密码的权限。

1. 将您的计算机加入到您的 Active Directory 域。
2. 确保你的主机属于控制对 gMSA 密码的访问的安全组。
3. 重新启动计算机, 使其获取新的组成员身份。
4. 为[Windows Server 的](https://docs.docker.com/install/windows/docker-ee/)windows 10 或 Docker 设置[docker 桌面](https://docs.docker.com/docker-for-windows/install/)。
5. 建议您使用通过运行[Test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount)验证主机是否可以使用 gMSA 帐户。 如果该命令返回**False**, 请参阅[疑难解答](#troubleshooting)部分了解诊断步骤。

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

## <a name="create-a-credential-spec"></a>创建凭据规范

凭据规范文件是一个 JSON 文档, 其中包含有关你希望容器使用的 gMSA 帐户的元数据。 通过将标识配置与容器映像分开, 你可以更改容器使用的 gMSA, 只需交换凭据规范文件, 无需更改代码。

凭据规范文件是使用加入域的容器主机上的[CredentialSpec PowerShell 模块](https://aka.ms/credspec)创建的。
创建文件后, 您可以将其复制到其他容器主机或容器 orchestrator。
凭据规范文件不包含任何机密 (如 gMSA 密码), 因为容器主机代表容器检索 gMSA。

Docker 希望在 Docker 数据目录中的**CredentialSpecs**目录下找到凭据规范文件。 在默认安装中, 你可以在上`C:\ProgramData\Docker\CredentialSpecs`找到此文件夹。

要在容器主机上创建凭据规范文件, 请执行以下操作:

1. 安装 RSAT AD PowerShell 工具
    - 对于 Windows Server, 请运行**安装-add-windowsfeature 的 RSAT-AD-PowerShell**。
    - 对于 Windows 10 版本1809或更高版本, 运行**Add-WindowsCapability-Name "Rsat. 0.0.1.0。 Tools ~**~ ~ ~ ~"。
    - 对于较早版本的 Windows 10, <https://aka.ms/rsat>请参阅。
2. 运行以下 cmdlet 以安装最新版本的[CredentialSpec PowerShell 模块](https://aka.ms/credspec):

    ```powershell
    Install-Module CredentialSpec
    ```

    如果你的容器主机上没有 internet 访问权限, 请`Save-Module CredentialSpec`在连接了 internet 的计算机上运行, 并将模块`C:\Program Files\WindowsPowerShell\Modules`文件夹复制到容器`$env:PSModulePath`主机上的其他位置。

3. 运行以下 cmdlet 以创建新的凭据规范文件:

    ```powershell
    New-CredentialSpec -AccountName WebApp01
    ```

    默认情况下, cmdlet 将使用所提供的 gMSA 名称作为容器的计算机帐户创建凭据规范。 将使用文件名的 gMSA 域和帐户名称将文件保存在 Docker CredentialSpecs 目录中。

    如果你作为容器中的辅助 gMSA 运行服务或进程, 你可以创建包含其他 gMSA 帐户的凭据规范。 若要执行此操作, `-AdditionalAccounts`请使用参数:

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -AdditionalAccounts LogAgentSvc, OtherSvc
    ```

    有关支持的参数的完整列表, 请`Get-Help New-CredentialSpec`运行。

4. 你可以使用以下 cmdlet 显示所有凭据规范及其完整路径的列表:

    ```powershell
    Get-CredentialSpec
    ```

## <a name="configure-your-application-to-use-the-gmsa"></a>将应用程序配置为使用 gMSA

在典型配置中, 只有当容器计算机帐户尝试对网络资源进行身份验证时, 才会为容器提供一个 gMSA 帐户。 这意味着, 如果应用需要使用 gMSA 标识, 你的应用将需要作为**本地系统**或**网络服务**运行。

### <a name="run-an-iis-app-pool-as-network-service"></a>将 IIS 应用池作为网络服务运行

如果你在容器中托管 IIS 网站, 你需要执行的所有操作都将你的应用池标识设置为**网络服务**。 你可以通过添加以下命令在 Dockerfile 中执行此操作:

```dockerfile
RUN %windir%\system32\inetsrv\appcmd.exe set AppPool DefaultAppPool -processModel.identityType:NetworkService
```

如果以前为 IIS 应用池使用了静态用户凭据, 请考虑 gMSA 作为这些凭据的替换项。 你可以更改开发环境、测试和生产环境之间的 gMSA, IIS 将自动获取当前标识, 而无需更改容器映像。

### <a name="run-a-windows-service-as-network-service"></a>作为网络服务运行 Windows 服务

如果你的容器化应用作为 Windows 服务运行, 你可以将该服务设置为在 Dockerfile 中作为**网络服务**运行:

```dockerfile
RUN sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""
```

### <a name="run-arbitrary-console-apps-as-network-service"></a>将任意控制台应用作为网络服务运行

对于未在 IIS 或服务管理器中托管的泛型控制台应用, 通常最简单的做法是将容器作为**网络服务**运行, 以便应用自动继承 gMSA 上下文。 此功能可从 Windows Server 版本1709获得。

将以下行添加到你的 Dockerfile, 以使其在默认情况下作为网络服务运行:

```dockerfile
USER "NT AUTHORITY\NETWORK SERVICE"
```

你还可以通过`docker exec`一次性地将容器作为网络服务连接到。 如果当容器通常不作为网络服务运行时, 如果正在运行的容器中出现连接问题, 此功能将非常有用。

```powershell
# Opens an interactive PowerShell console in the container (id = 85d) as the Network Service account
docker exec -it --user "NT AUTHORITY\NETWORK SERVICE" 85d powershell
```

## <a name="run-a-container-with-a-gmsa"></a>使用 gMSA 运行容器

若要使用 gMSA 运行容器, 请将凭据规范文件提供给 docker `--security-opt`的参数[运行](https://docs.docker.com/engine/reference/run):

```powershell
# For Windows Server 2016, change the image name to mcr.microsoft.com/windows/servercore:ltsc2016
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

>[!IMPORTANT]
>在 Windows Server 2016 版本1709和1803上, 容器的主机名必须匹配 gMSA 短名称。

在前面的示例中, gMSA SAM 帐户名为 "webapp01", 因此容器主机名也被命名为 "webapp01"。

在 Windows Server 2019 及更高版本中, 不需要主机名字段, 但容器仍将通过 gMSA 名称而不是主机名来标识自己, 即使你显式提供了不同的名称也是如此。

若要检查 gMSA 是否正常工作, 请在容器中运行以下 cmdlet:

```powershell
# Replace contoso.com with your own domain
PS C:\> nltest /sc_verify:contoso.com

Flags: b0 HAS_IP  HAS_TIMESERV
Trusted DC Name \\dc01.contoso.com
Trusted DC Connection Status Status = 0 0x0 NERR_Success
Trust Verification Status = 0 0x0 NERR_Success
The command completed successfully
```

如果受信任的 DC 连接状态和信任验证状态不`NERR_Success`是, 请查看[疑难解答](#troubleshooting)部分, 了解有关如何调试问题的提示。

你可以通过运行以下命令并检查客户端名称来验证容器内的 gMSA 标识:

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

若要将 PowerShell 或其他控制台应用打开为 gMSA 帐户, 可以让容器在网络服务帐户下运行, 而不是在普通 ContainerAdministrator (或 ContainerUser NanoServer) 帐户下运行:

```powershell
# NOTE: you can only run as Network Service or SYSTEM on Windows Server 1709 and later
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 --user "NT AUTHORITY\NETWORK SERVICE" -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

当你作为网络服务运行时, 你可以通过尝试连接到域控制器上的 SYSVOL 来测试网络身份验证, gMSA:

```powershell
# This command should succeed if you're successfully running as the gMSA
PS C:\> dir \\contoso.com\SYSVOL


    Directory: \\contoso.com\sysvol


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d----l        2/27/2019   8:09 PM                contoso.com
```

## <a name="orchestrate-containers-with-gmsa"></a>具有 gMSA 的协调容器

在生产环境中, 你将经常使用容器 orchestrator 部署和管理你的应用和服务。 每个 orchestrator 都有其自己的管理范例, 并负责接受凭据规范以提供给 Windows 容器平台。

当您与 gMSAs 协调容器时, 请确保:

> [!div class="checklist"]
> * 可以计划运行具有 gMSAs 的容器的所有容器主机都已加入域
> * 容器主机具有检索容器使用的所有 gMSAs 的密码的访问权限
> * 将创建凭据规范文件并将其上载到 orchestrator, 或复制到每个容器主机, 具体取决于 orchestrator 如何首选处理它们。
> * 容器网络允许容器与 Active Directory 域控制器通信以检索 gMSA 票证

### <a name="how-to-use-gmsa-with-service-fabric"></a>如何将 gMSA 与 Service Fabric 配合使用

当你在应用程序清单中指定凭据规范位置时, Service Fabric 支持使用 gMSA 运行 Windows 容器。 你需要创建凭据规范文件并将其放在每个主机上 Docker 数据目录的**CredentialSpecs**子目录中, 以便服务结构可以找到它。 你可以运行**CredentialSpec** Cmdlet ( [CredentialSpec PowerShell 模块](https://aka.ms/credspec)的一部分) 验证凭据规范是否位于正确的位置。

有关如何配置你的应用程序的详细信息, 请参阅[快速入门: 将 windows 容器部署到 Service fabric](https://docs.microsoft.com/azure/service-fabric/service-fabric-quickstart-containers)和[设置适用于 Service fabric 的 Windows 容器的 gMSA](https://docs.microsoft.com/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers) 。

### <a name="how-to-use-gmsa-with-docker-swarm"></a>如何将 gMSA 与 Docker 配合使用 Swarm

若要将 gMSA 与由 Docker Swarm 托管的容器结合使用, 请使用该`--credential-spec`参数运行[Docker 服务 create](https://docs.docker.com/engine/reference/commandline/service_create/)命令:

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

有关如何将凭据规范与 Docker 服务配合使用的详细信息, 请参阅[Docker Swarm 示例](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only)。

### <a name="how-to-use-gmsa-with-kubernetes"></a>如何将 gMSA 与 Kubernetes 结合使用

对使用 Kubernetes 中的 gMSAs 的 Windows 容器进行计划的支持在 Kubernetes 1.14 中以 alpha 功能的形式提供。 请参阅为[Windows 箱和容器配置 gMSA](https://kubernetes.io/docs/tasks/configure-pod-container/configure-gmsa) , 了解有关此功能的最新信息以及如何在 Kubernetes 分发中进行测试。

## <a name="example-uses"></a>示例使用

### <a name="sql-connection-strings"></a>SQL 连接字符串

当服务作为容器中的本地系统或网络服务运行时，它可以使用 Windows 集成身份验证连接到 Microsoft SQL Server。

下面是使用容器标识对 SQL Server 进行身份验证的连接字符串的示例:

```sql
Server=sql.contoso.com;Database=MusicStore;Integrated Security=True;MultipleActiveResultSets=True;Connect Timeout=30
```

在 Microsoft SQL Server上，使用域和 gMSA 名称（以 $ 结尾）创建一个登录名。 创建登录后, 您可以将其添加到数据库中的用户并授予其相应的访问权限。

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

若要查看它是否在操作中, 请查看会话中的 Microsoft Ignite 2016 中提供的[录制演示](https://youtu.be/cZHPz80I-3s?t=2672), "遍历 Containerization 的路径-将工作负荷转换为容器。"

## <a name="troubleshooting"></a>故障排除

### <a name="known-issues"></a>已知问题

#### <a name="container-hostname-must-match-the-gmsa-name-for-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>容器主机名必须与 Windows Server 2016 和 Windows 10 版本1709和1803的 gMSA 名称相匹配

如果你运行的是 Windows Server 2016、版本1709或 1803, 则你的容器的主机名必须与你的 gMSA SAM 帐户名相匹配。

当主机名与 gMSA 名称不匹配时, 入站 NTLM 身份验证请求和名称/SID 转换 (由许多库 (如 ASP.NET 成员身份角色提供程序) 使用将失败。 即使主机名和 gMSA 名称不匹配, Kerberos 仍将继续正常工作。

此限制在 Windows Server 2019 中已修复, 在此情况下, 容器现在始终在网络上使用其 gMSA 名称, 而不考虑分配的主机名。

#### <a name="using-a-gmsa-with-more-than-one-container-simultaneously-leads-to-intermittent-failures-on-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>将 gMSA 与多个容器同时使用时, Windows Server 2016 和 Windows 10、版本1709和1803中将导致间歇性故障

由于所有容器都需要使用同一主机名, 因此第二个问题会影响 Windows Server 2019 和 Windows 10 版本1809之前的 Windows 版本。 当为多个容器分配相同的标识和主机名时, 当两个容器同时与同一域控制器对话时, 可能会出现争用条件。 当另一个容器与同一域控制器通信时, 它将取消与使用相同标识的任何以前的容器的通信。 这可能导致间歇性身份验证失败, 有时在容器内运行`nltest /sc_verify:contoso.com`时可能会被视为信任失败。

我们更改了 Windows Server 2019 中的行为以将容器标识与计算机名称分开, 从而允许多个容器同时使用相同的 gMSA。

#### <a name="you-cant-use-gmsas-with-hyper-v-isolated-containers-on-windows-10-versions-1703-1709-and-1803"></a>在 Windows 10 版本1703、1709和1803上, 不能将 gMSAs 与 Hyper-v 隔离容器配合使用

当你尝试在 Windows 10 和 Windows Server 版本1703、1709和1803上使用具有 Hyper-v 隔离容器的 gMSA 时, 容器初始化将挂起或失败。

此错误已在 Windows Server 2019 和 Windows 10 版本1809中修复。 你还可以在 Windows Server 2016 和 Windows 10 版本1607的 gMSAs 上运行 Hyper-v 隔离容器。

### <a name="general-troubleshooting-guidance"></a>常规疑难解答指南

如果在使用 gMSA 运行容器时遇到错误, 以下说明可帮助你识别根本原因。

#### <a name="make-sure-the-host-can-use-the-gmsa"></a>请确保主机可以使用 gMSA

1. 验证主机是否已加入域以及是否可以访问域控制器。
2. 从 RSAT 安装广告 PowerShell 工具并运行[Test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount)以查看计算机是否有权检索 gMSA。 如果 cmdlet 返回**False**, 则计算机无法访问 gMSA 密码。

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

3. 如果**ADServiceAccount**返回**False**, 请验证主机属于可以访问 gMSA 密码的安全组。

    ```powershell
    # Get the current computer's group membership
    Get-ADComputer $env:computername | Get-ADPrincipalGroupMembership | Select-Object DistinguishedName

    # Get the groups allowed to retrieve the gMSA password
    # Change "WebApp01" for your own gMSA name
    (Get-ADServiceAccount WebApp01 -Properties PrincipalsAllowedToRetrieveManagedPassword).PrincipalsAllowedToRetrieveManagedPassword
    ```

4. 如果你的主机属于授权检索 gMSA 密码的安全组, 但仍未通过**ADServiceAccount 的测试**, 则你可能需要重新启动计算机才能获取反映其当前组成员身份的新票证。

#### <a name="check-the-credential-spec-file"></a>检查凭据规范文件

1. 从[CredentialSpec PowerShell 模块](https://aka.ms/credspec)中运行**CredentialSpec** , 以查找计算机上的所有凭据规范。 凭据规范必须存储在 Docker 根目录下的 "CredentialSpecs" 目录中。 你可以通过运行**docker 信息-f "{{} 找到 docker root 目录。DockerRootDir}} "**。
2. 打开 CredentialSpec 文件, 确保以下字段正确填写:
    - **Sid**: gMSA 帐户的 sid
    - **MachineAccountName**: GMSA SAM 帐户名 (不要包含完整的域名或美元符号)
    - **DnsTreeName**: Active Directory 林的 FQDN
    - **DnsName**: gMSA 所属的域的 FQDN
    - **NetBiosName**: gMSA 所属的域的 NETBIOS 名称
    - **GroupManagedServiceAccounts/Name**: gMSA SAM 帐户名 (不包括完整的域名称或美元符号)
    - **GroupManagedServiceAccounts/作用域**: 一个用于域 FQDN 的条目, 一个用于 NETBIOS

    你的输入应类似于以下示例: 完整的凭据规范:

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

3. 验证针对你的业务流程解决方案的凭据规范文件的路径是否正确。 如果您使用的是 Docker, 请确保容器运行命令包括`--security-opt="credentialspec=file://NAME.json"`"CredentialSpec", 其中 "name. json" 使用名称输出替换为**Get**。 该名称是平面文件名, 相对于 Docker 根目录下的 CredentialSpecs 文件夹。

#### <a name="check-the-firewall-configuration"></a>检查防火墙配置

如果你在容器或主机网络上使用严格的防火墙策略, 则可能会阻止与 Active Directory 域控制器或 DNS 服务器的连接。

| 协议和端口 | 用途 |
|-------------------|---------|
| TCP 和 UDP 53 | DNS |
| TCP 和 UDP 88 | Kerberos |
| TCP 139 | NetLogon |
| TCP 和 UDP 389 | 适用 |
| TCP 636 | LDAP SSL |

你可能需要允许访问其他端口, 具体取决于你的容器发送到域控制器的流量类型。
有关 Active directory 使用的所有端口的完整列表, 请参阅[Active directory 和 Active Directory 域服务端口要求](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd772723(v=ws.10)#communication-to-domain-controllers)。

#### <a name="check-the-container"></a>检查容器

1. 如果你运行的是 windows Server 2019 或 Windows 10 版本1809之前的 Windows 版本, 你的容器主机名必须匹配 gMSA 名称。 确保`--hostname`参数与 gMSA 短名称 (不是域组件, 例如 "webapp01", 而不是 "webapp01.contoso.com") 相匹配。

2. 检查容器网络配置以验证容器能否解析和访问 gMSA 的域的域控制器。 错误配置容器中的 DNS 服务器是身份问题的常见原因。

3. 通过在容器中运行以下 cmdlet 来检查容器是否有有效的连接 (使用`docker exec`或等效):

    ```powershell
    nltest /sc_verify:contoso.com
    ```

    如果 gMSA 可用且网络`NERR_SUCCESS`连接允许容器与域通信, 则应返回信任验证。 如果失败, 请验证主机和容器的网络配置。 两者都需要能够与域控制器进行通信。

4. 确保你的应用[配置为使用 gMSA](#configure-your-application-to-use-the-gmsa)。 使用 gMSA 时, 容器内的用户帐户不会更改。 相反, 系统帐户在与其他网络资源交谈时使用 gMSA。 这意味着你的应用需要作为网络服务或本地系统运行, 才能利用 gMSA 标识。

    > [!TIP]
    > 如果你运行`whoami`或使用其他工具来标识容器中的当前用户上下文, 则不会看到 gMSA 名称本身。 这是因为你始终以本地用户 (而不是域标识) 的身份登录到容器。 GMSA 在与网络资源进行交谈时由计算机帐户使用, 这就是你的应用需要作为网络服务或本地系统运行的原因。

5. 最后, 如果你的容器似乎配置正确, 但用户或其他服务无法自动对你的容器化应用进行身份验证, 请检查你的 gMSA 帐户上的 Spn。 客户端将通过其访问你的应用程序的名称找到 gMSA 帐户。 这可能意味着, 如果客户通过负载`host`平衡器或其他 DNS 名称连接到你的应用, 你将需要 gMSA 的其他 spn。

## <a name="additional-resources"></a>其他资源

- [组托管服务帐户概述](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)
