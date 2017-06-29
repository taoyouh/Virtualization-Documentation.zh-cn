---
title: "用于 Windows 容器的 Active Directory 服务帐户"
description: "用于 Windows 容器的 Active Directory 服务帐户"
keywords: "docker, 容器, Active Directory"
author: PatrickLang
ms.date: 11/04/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: e864242e69a84d7636241ea2a772722add5b8b7c
ms.sourcegitcommit: bb171f4a858fefe33dd0748b500a018fd0382ea6
ms.translationtype: HT
ms.contentlocale: zh-CN
---
# <a name="active-directory-service-accounts-for-windows-containers"></a>用于 Windows 容器的 Active Directory 服务帐户

用户和其他服务可能需要进行身份验证才能连接到应用程序和服务，以确保数据安全并防止未经授权的使用。 Windows Active Directory (AD) 域本身支持密码和证书身份验证。 在 Windows 已加入域的主机上生成应用程序或服务时，如果作为本地系统或网络服务运行，则默认使用主机的身份。 否则，可能需要配置另一个 AD 帐户进行身份验证。

虽然 Windows 容器不能加入域，但它们也可以利用 Active Directory 域标识，类似于设备加入域时。 Windows Server 2012 R2 域控制器引入了一个新的域帐户，称为组托管服务帐户 (gMSA)，旨在供服务共享。 通过使用组托管服务帐户 (gMSA)，Windows 容器本身和它们托管的服务都可以配置为使用特定的 gMSA 作为其域标识。 作为本地系统或网络服务运行的任何服务将使用 Windows 容器的标识，就像现在使用已加入域的主机的标识一样。 容器映像中没有存储可能会无意暴露的密码或证书私钥，并且容器可以重新部署到开发、测试和生产环境，而无需重新构建以更改存储的密码或证书。 


# <a name="glossary--references"></a>术语和参考
- [Active Directory](http://social.technet.microsoft.com/wiki/contents/articles/1026.active-directory-services-overview.aspx) 是一项服务，用于在 Windows 上发现、搜索和复制用户、计算机和服务帐户信息。 
  - [Active Directory 域服务](https://technet.microsoft.com/en-us/library/dd448614.aspx)提供用于验证计算机和用户的 Windows Active Directory 域。 
  - 设备是 Active Directory 域的成员时，设备就属于_已加入域_。 “已加入域”是一种设备状态，它不仅为设备提供域计算机标识，而且还方便了各种加入域的服务。
  - [组托管服务帐户](https://technet.microsoft.com/en-us/library/jj128431(v=ws.11).aspx)，通常缩写为 gMSA，是一种 Active Directory 帐户，可轻松使用 Active Directory 保护服务，而无需共享密码。 多个计算机或容器可根据需要共享同一个 gMSA，以验证服务间的连接。
- _CredentialSpec_ PowerShell 模块 - 此模块用于配置要与容器一起使用的组托管服务帐户。 脚本模块和示例步骤位于 [windows-server-container-tools](https://github.com/Microsoft/Virtualization-Documentation/tree/live/windows-server-container-tools)，请参阅 ServiceAccount

# <a name="how-it-works"></a>工作原理

当前，组托管服务帐户通常用于保护一台计算机或服务与另一台计算机或服务之间的连接。 使用此帐户的一般步骤如下：

1. 创建 gMSA
2. 配置服务，使其作为 gMSA 运行
3. 在 Active Directory 中对运行服务的已加入域主机授予对 gMSA 秘钥的访问权限
4. 允许访问其他服务（如数据库或文件共享）上的 gMSA

启动服务时，已加入域的主机会自动从 Active Directory 获取 gMSA密钥，并使用该帐户运行服务。 由于该服务作为 gMSA 运行，它可以访问 gMSA 所允许的任何资源。

Windows 容器遵循类似的过程：

1. 创建 gMSA。 默认情况下，域管理员或帐户操作员必须执行此操作。 否则，他们可以将创建和管理 gMSA 的权限委派给使用它们管理服务的管理员。 请参阅 [gMSA 入门](https://technet.microsoft.com/en-us/library/jj128431(v=ws.11).aspx)
2. 授予加入域的容器主机对 gMSA 的访问权限
3. 允许访问其他服务（如数据库或文件共享）上的 gMSA
4. 使用 [windows-server-container-tools](https://github.com/Microsoft/Virtualization-Documentation/tree/live/windows-server-container-tools) 中的 CredentialSpec PowerShell 模块存储使用 gMSA 所需的设置
5. 使用其他选项启动容器 `--security-opt "credentialspec=..."`

启动容器时，作为本地系统或网络服务运行的已安装服务将显示为作为 gMSA 运行。 这类似于这些帐户在已加入域的主机上的工作方式，除了使用的是 gMSA 而不是计算机帐户。 

![关系图 - 服务账户](media/serviceaccount_diagram.png)


# <a name="example-uses"></a>示例用法


## <a name="sql-connection-strings"></a>SQL 连接字符串
当服务作为容器中的本地系统或网络服务运行时，它可以使用 Windows 集成身份验证连接到 Microsoft SQL Server。

示例：

```none
Server=sql.contoso.com;Database=MusicStore;Integrated Security=True;MultipleActiveResultSets=True;Connect Timeout=30
```

在 Microsoft SQL Server上，使用域和 gMSA 名称（以 $ 结尾）创建一个登录名。 创建登录名后，可以将其添加到数据库上的用户，并授予适当的访问权限。

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

若要查看它的实际操作，请查看 Microsoft Ignite 2016 会话“通往容器化—将工作负载转移到容器中”中提供的[录制演示](https://youtu.be/cZHPz80I-3s?t=2672)。
