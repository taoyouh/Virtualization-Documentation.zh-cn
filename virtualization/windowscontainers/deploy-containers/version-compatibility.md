---
title: Windows 容器版本兼容性
description: Windows 如何跨多个版本运行内部版本和容器
keywords: 元数据, 容器, 版本
author: taylorb-microsoft
ms.openlocfilehash: 32d40997ffef47e4eae2d06303f45522623a5e54
ms.sourcegitcommit: 530712469552a1ef458883001ee748bab2c65ef7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/26/2020
ms.locfileid: "77628942"
---
# <a name="windows-container-version-compatibility"></a>Windows 容器版本兼容性

Windows Server 2016 和 Windows 10 周年更新（版本14393）是可以生成并运行 Windows Server 容器的第一个 Windows 版本。 使用这些版本生成的容器可以在更新的版本上运行，但在开始之前，需要了解几个事项。

由于我们一直在改进 Windows 容器功能，我们不得不进行一些可能影响兼容性的变更。 旧的容器将在具有[hyper-v 隔离](../manage-containers/hyperv-container.md)的较新主机上运行相同的版本，并将使用相同（较旧）的内核版本。 但是，如果要基于较新的 Windows 版本运行容器，则它只能在较新的主机上运行。

## <a name="windows-server-host-os-compatibility"></a>Windows Server 主机操作系统兼容性

<!-- start tab view -->
# <a name="windows-server-version-1909"></a>[Windows Server，版本1909](#tab/windows-server-1909)

|容器基本映像操作系统版本|支持 Hyper-v 隔离|支持进程隔离|
|---|:---:|:---:|
|Windows Server，版本1909|&#10004;|&#10004;|
|Windows Server 版本 1903|&#10004;|&#10060;|
|Windows Server 2019|&#10004;|&#10060;|
|Windows Server 2016|&#10004;|&#10060;|

# <a name="windows-server-version-1903"></a>[Windows Server，版本1903](#tab/windows-server-1903)

|容器基本映像操作系统版本|支持 Hyper-v 隔离|支持进程隔离|
|---|:---:|:---:|
|Windows Server，版本1909|&#10060;|&#10060;|
|Windows Server 版本 1903|&#10004;|&#10004;|
|Windows Server 2019|&#10004;|&#10060;|
|Windows Server 2016|&#10004;|&#10060;|

# <a name="windows-server-2019"></a>[Windows Server 2019](#tab/windows-server-2019)

|容器基本映像操作系统版本|支持 Hyper-v 隔离|支持进程隔离|
|---|:---:|:---:|
|Windows Server，版本1909|&#10060;|&#10060;|
|Windows Server 版本 1903|&#10060;|&#10060;|
|Windows Server 2019|&#10004;|&#10004;|
|Windows Server 2016|&#10004;|&#10060;|

# <a name="windows-server-2016"></a>[Windows Server 2016](#tab/windows-server-2016)

|容器基本映像操作系统版本|支持 Hyper-v 隔离|支持进程隔离|
|---|:---:|:---:|
|Windows Server，版本1909|&#10060;|&#10060;|
|Windows Server 版本 1903|&#10060;|&#10060;|
|Windows Server 2019|&#10060;|&#10060;|
|Windows Server 2016|&#10004;|&#10004;|

---
<!-- stop tab view -->

## <a name="windows-10-host-os-compatibility"></a>Windows 10 主机操作系统兼容性

<!-- start tab view -->

# <a name="windows-10-version-1909"></a>[Windows 10 版本1909](#tab/windows-10-1909)

|容器基本映像操作系统版本|支持 Hyper-v 隔离|支持进程隔离|
|---|:---:|:---:|
|Windows Server，版本1909|&#10004;|&#10060;|
|Windows Server 版本 1903|&#10004;|&#10060;|
|Windows Server 2019|&#10004;|&#10060;|
|Windows Server 2016|&#10004;|&#10060;|

# <a name="windows-10-version-1903"></a>[Windows 10 版本1903](#tab/windows-10-1903)

|容器基本映像操作系统版本|支持 Hyper-v 隔离|支持进程隔离|
|---|:---:|:---:|
|Windows Server，版本1909|&#10060;|&#10060;|
|Windows Server 版本 1903|&#10004;|&#10060;|
|Windows Server 2019|&#10004;|&#10060;|
|Windows Server 2016|&#10004;|&#10060;|

# <a name="windows-10-version-1809"></a>[Windows 10 版本1809](#tab/windows-10-1809)

|容器基本映像操作系统版本|支持 Hyper-v 隔离|支持进程隔离|
|---|:---:|:---:|
|Windows Server，版本1909|&#10060;|&#10060;|
|Windows Server 版本 1903|&#10060;|&#10060;|
|Windows Server 2019|&#10004;|&#10060;|
|Windows Server 2016|&#10004;|&#10060;|

---
<!-- stop tab view -->

## <a name="matching-container-host-version-with-container-image-versions"></a>将容器主机版本与容器映像版本匹配

### <a name="windows-server-containers"></a>Windows Server 容器

由于 Windows Server 容器和基础主机共享单个内核，因此容器的基本映像操作系统版本必须与主机的版本匹配。 如果版本不同，则容器可能会启动，但不能保证完整功能。 Windows 操作系统具有四个级别的版本控制：主版本、次版本、内部版本和修订版本。 例如，版本10.0.14393.103 的主版本为10，次版本号为0，版本号为14393，版本号为103。 仅当发布新版本的操作系统时（如版本1709、1903等），内部版本号才会更改。 应用 Windows 更新后，会相应更新修订版本号。

#### <a name="build-number-new-release-of-windows"></a>内部版本号（新版本的 Windows）

当容器主机和容器映像之间的内部版本号不同时，将阻止 Windows Server 容器启动。 例如，当容器主机版本为 10.0.14393. * （Windows Server 2016），而容器映像为版本 10.0.16299. * （Windows Server 版本1709）时，容器将无法启动。  

#### <a name="revision-number-patching"></a>修订号（修补）

当容器主机和容器映像的修订号不同时，将阻止基于 Windows Server 2016 的容器启动。 例如，如果容器主机版本为10.0.14393。**1914** （应用了 KB4051033 的 Windows Server 2016）和容器映像的版本为10.0.14393。**1944** （应用了 KB4053579 的 Windows Server 2016），则映像将无法启动。

但是，对于使用 Windows Server 版本1809及更高版本的主机或映像，此规则不适用，并且主机和容器映像不需要匹配的修订版本。 

建议你通过最新的修补程序和更新使你的系统（主机和容器）保持最新状态，以确保安全。

#### <a name="practical-application"></a>实用应用程序

示例1：容器主机运行的 Windows Server 2016 应用了 KB4041691。 部署到此主机的任何 Windows Server 容器都必须基于版本10.0.14393.1770 容器基本映像。 如果将 KB4053579 应用到主机容器，则还必须更新这些图像以确保主机容器支持它们。

示例2：容器主机运行的 Windows Server 版本1809应用了 KB4534273。 部署到此主机的任何 Windows Server 容器都必须基于 Windows Server 版本1809（10.0.17763）容器基本映像，但不需要与主机 KB 匹配。 如果将 KB4534273 应用于主机，则仍将支持容器映像，但建议对其进行更新，以解决任何潜在的安全问题。

#### <a name="querying-version"></a>查询版本

方法1：在版本1709中引入，cmd 提示符和**ver**命令现在返回修订详细信息。

```batch
Microsoft Windows [Version 10.0.16299.125]
(c) 2017 Microsoft Corporation. All rights reserved.

C:\>ver

Microsoft Windows [Version 10.0.16299.125]
```

方法2：查询以下注册表项： HKEY_LOCAL_MACHINE \Software\Microsoft\Windows NT\CurrentVersion

例如：

```batch
C:\>reg query "HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion" /v BuildLabEx
```

```batch
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Users\Administrator> (Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\').BuildLabEx
14393.321.amd64fre.rs1_release_inmarket.161004-2338
```

若要检查基本映像使用的版本，请查看 Docker 中心上的标记或映像说明中提供的映像哈希表。 " [Windows 10 更新历史记录](https://support.microsoft.com/help/12387/windows-10-update-history)" 页将列出每个版本发布的时间。

### <a name="hyper-v-isolation-for-containers"></a>容器的 hyper-v 隔离

您可以运行具有或不带 Hyper-v 隔离的 Windows 容器。 Hyper-V 隔离使用优化的 VM 在容器周围创造安全边界。 与在容器和主机之间共享内核的标准 Windows 容器不同，每个 Hyper-v 独立容器都有自己的 Windows 内核实例。 这意味着容器主机和映像中可以有不同的操作系统版本（有关详细信息，请参阅以下兼容性矩阵）。  

若要运行带有 Hyper-V 隔离的容器，只需在你的 Docker 运行命令中添加标记 `--isolation=hyperv`。

## <a name="errors-from-mismatched-versions"></a>不匹配版本引发的错误

如果尝试运行不受支持的组合，会收到以下错误：

```dockerfile
docker: Error response from daemon: container b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Owner":"docker","IsDummy":false,"VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Layers":[{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"b81ed896222e","MappedDirectories":[],"HvPartition":false,"EndpointList":["002a0d9e-13b7-42c0-89b2-c1e80d9af243"],"Servicing":false,"AllowUnqualifiedDNSQuery":true}.
```

可以通过三种方式解决此错误：

- 基于 `mcr.microsoft.com/windows/nanoserver` 或 `mcr.microsoft.com/windows/servercore` 的正确版本重新生成容器
- 如果主机较新，则运行**docker run--隔离 = hyperv ...**
- 尝试在具有相同 Windows 版本的其他主机上运行容器

## <a name="choose-which-container-os-version-to-use"></a>选择要使用的容器操作系统版本

>[!NOTE]
>从2019年4月16日起，不再为[Windows 基本操作系统容器映像](https://hub.docker.com/_/microsoft-windows-base-os-images)发布或维护 "最新" 标记。 从这些存储库拉取或引用图像时，请声明特定标记。

您必须知道您需要为容器使用哪个版本。 例如，如果想要将 Windows Server 版本1809用作容器操作系统，并想要为其提供最新的修补程序，则应在指定所需的基本 OS 容器映像版本时使用标记 `1809`，如下所示：

``` dockerfile
FROM mcr.microsoft.com/windows/nanoserver:1809
...
```

但是，如果需要 Windows Server 版本1809的特定修补程序，则可以在标记中指定 KB 号。 例如，若要获取 Windows Server 版本1809中的 Nano Server 基本操作系统容器映像，并将 KB4493509 应用到该映像，请按如下所示进行指定：

``` dockerfile
FROM mcr.microsoft.com/windows/nanoserver:1809-KB4493509
...
```

你还可以通过在标记中指定 OS 版本来指定你以前使用的架构所需的准确修补程序：

``` dockerfile
FROM mcr.microsoft.com/windows/nanoserver:10.0.17763.437
...
```

基于 Windows Server 2019 和 Windows Server 2016 的 Server Core 基础映像是长期[服务通道（LTSC）](https://docs.microsoft.com/windows-server/get-started-19/servicing-channels-19#long-term-servicing-channel-ltsc)版本。 例如，如果想要将 Windows Server 2019 作为服务器核心映像的容器操作系统，并希望获得最新的修补程序，可以指定 LTSC 版本，如下所示：

``` dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019
...
```

## <a name="matching-versions-using-docker-swarm"></a>使用 Docker Swarm 对版本进行匹配

Docker Swarm 当前没有用于将容器使用的 Windows 版本与相同版本的主机匹配的内置方式。 如果将服务更新为使用较新的容器，则它将成功运行。

如果需要长时间运行多个版本的 Windows，可采用以下两种方法：将 Windows 主机配置为始终使用 Hyper-v 隔离或使用标签约束。

### <a name="finding-a-service-that-wont-start"></a>查找无法启动的服务

如果服务无法启动，则会看到 `MODE` `replicated`，但 `REPLICAS` 会停滞为0。 若要查看操作系统版本是否有问题，请运行以下命令：

运行**docker service ls**以查找服务名称：

```dockerfile
ID                  NAME                MODE                REPLICAS            IMAGE                                             PORTS
xh6mwbdq2uil        angry_liskov        replicated          0/1                 microsoft/iis:windowsservercore-10.0.14393.1715
```

运行**docker service ps （服务名称）** 以获取状态和最新尝试：

```dockerfile
C:\Program Files\Docker>docker service ps angry_liskov
ID                  NAME                 IMAGE                                             NODE                DESIRED STATE       CURRENT STATE               ERROR                              PORTS
klkbhn742lv0        angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Ready               Ready 3 seconds ago
y5blbdum70zo         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 24 seconds ago       "starting container failed: co…"
yjq6zwzqj8kt         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 31 seconds ago       "starting container failed: co…"

ytnnv80p03xx         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
xeqkxbsao57w         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
```

如果看到 `starting container failed: ...`，则可以看到**docker service ps 的完整错误--trunc （容器名称）** ：

```dockerfile
C:\Program Files\Docker>docker service ps --no-trunc angry_liskov
ID                          NAME                 IMAGE                                                                                                                     NODE                DESIRED STATE       CURRENT STATE                     ERROR                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          PORTS
dwsd6sjlwsgic5vrglhtxu178   angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Running             Starting less than a second ago                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              
y5blbdum70zoh1f6uhx5nxsfv    \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Shutdown            Failed 39 seconds ago             "starting container failed: container e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Owner":"docker","VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Layers":[{"ID":"bcf2630f-ea95-529b-b33c-e5cdab0afdb4","Path":"C:\\ProgramData\\docker\\windowsfilter\\200235127f92416724ae1d53ed3fdc86d78767132d019bdda1e1192ee4cf3ae4"},{"ID":"e3ea10a8-4c2f-5b93-b2aa-720982f116f6","Path":"C:\\ProgramData\\docker\\windowsfilter\\0ccc9fa71a9f4c5f6f3bc8134fe3533e454e09f453de662cf99ab5d2106abbdc"},{"ID":"cff5391f-e481-593c-aff7-12e080c653ab","Path":"C:\\ProgramData\\docker\\windowsfilter\\a49576b24cd6ec4a26202871c36c0a2083d507394a3072186133131a72601a31"},{"ID":"499cb51e-b891-549a-b1f4-8a25a4665fbd","Path":"C:\\ProgramData\\docker\\windowsfilter\\fdf2f52c4323c62f7ff9b031c0bc3af42cf5fba91098d51089d039fb3e834c08"},{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"e7b5d3adba7e","HvPartition":false,"EndpointList":["298bb656-8800-4948-a41c-1b0500f3d94c"],"AllowUnqualifiedDNSQuery":true}"
```

这与 `CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101)`的错误相同。

### <a name="fix---update-the-service-to-use-a-matching-version"></a>修复 - 将服务更新为使用匹配版本

对于 Docker Swarm，有两点注意事项。 如果有一个包含文件的撰写文件使用的是未创建的映像，则需要相应地更新该引用。 例如：

``` yaml
version: '3'

services:
  YourServiceName:
    image: microsoft/windowsservercore:1709
...
```

另一个考虑因素是，如果你所指向的映像是你自己创建的映像（例如 contoso/myimage）：

```yaml
version: '3'

services:
  YourServiceName:
    image: contoso/myimage
...
```

在这种情况下，应使用不[匹配版本](#errors-from-mismatched-versions)中所述的方法来修改该 dockerfile，而不是使用 docker 编写行。

### <a name="mitigation---use-hyper-v-isolation-with-docker-swarm"></a>缓解 - 结合使用 Hyper-V 隔离与 Docker Swarm

建议在每个容器的基础上使用 Hyper-v 隔离，但代码尚未完成。 你可以在 [GitHub](https://github.com/moby/moby/issues/31616) 上关注进展。 在实现之前，我们需要将主机配置为始终使用 Hyper-V 隔离运行。

这需要更改 Docker 服务配置，然后重启 Docker 引擎。

1. 编辑 `C:\ProgramData\docker\config\daemon.json`
2. 使用 `"exec-opts":["isolation=hyperv"]` 添加行

    >[!NOTE]
    >默认情况下，daemon 文件不存在。 如果在查看目录时发现该文件不存在，你必须创建该文件。 然后，您将需要复制以下内容：

    ```JSON
    {
        "exec-opts":["isolation=hyperv"]
    }
    ```

3. 关闭并保存该文件，然后通过在 PowerShell 中运行以下 cmdlet 来重新启动 docker 引擎：

    ```powershell
    Stop-Service docker
    Start-Service docker
    ```

4. 重新启动服务后，启动容器。 运行后，可以通过使用以下 cmdlet 检查容器来验证容器的隔离级别：

    ```powershell
    docker inspect --format='{{json .HostConfig.Isolation}}' $instanceNameOrId
    ```

将返回“process”或“hyperv”结果。 如果已按照上述内容对 daemon.json 文件进行修改和设置，显示的结果将是后者。

### <a name="mitigation---use-labels-and-constraints"></a>缓解 - 使用标签和限制

下面介绍如何使用标签和约束来匹配版本：

1. 向每个节点添加标签。

    在每个节点上，添加两个标签： `OS` 和 `OsVersion`。 这样做的假设前提是你在本地运行但可以修改为在远程主机上对标签进行设置。

    ```powershell
    docker node update --label-add OS="windows" $ENV:COMPUTERNAME
    docker node update --label-add OsVersion="$((Get-ComputerInfo).OsVersion)" $ENV:COMPUTERNAME
    ```

    之后，你可以通过运行**docker 节点检查**命令来检查这些内容，该命令应显示新添加的标签：

    ```yaml
           "Spec": {
                "Labels": {
                   "OS": "windows",
                   "OsVersion": "10.0.16296"
               },
                "Role": "manager",
                "Availability": "active"
            }
    ```

2. 添加一个服务约束。

    现在您已标记每个节点，您可以更新确定服务位置的约束。 在下面的示例中，将 "contoso_service" 替换为实际服务的名称：

    ```powershell
    docker service update \
        --constraint-add "node.labels.OS == windows" \
        --constraint-add "node.labels.OsVersion == $((Get-ComputerInfo).OsVersion)" \
        contoso_service
    ```

    这将实施并限制节点运行的位置。

若要了解有关如何使用服务约束的详细信息，请查看[服务创建引用](https://docs.docker.com/engine/reference/commandline/service_create/#specify-service-constraints-constraint)。

## <a name="matching-versions-using-kubernetes"></a>使用 Kubernetes 对版本进行匹配

当在 Kubernetes 中计划 pod 时，可能会出现[使用 Docker Swarm 的匹配版本](#matching-versions-using-docker-swarm)中所述的相同问题。 可以通过类似的策略避免此问题：

- 基于开发和生产中的相同操作系统版本重新生成容器。 若要了解如何操作，请参阅[选择要使用的容器操作系统版本](#choose-which-container-os-version-to-use)。
- 如果 Windows Server 2016 和 Windows Server 1709 版节点都在同一群集中，则使用节点标签和 nodeSelectors 确保在兼容节点上安排盒
- 基于操作系统版本使用单独的群集

### <a name="finding-pods-failed-on-os-mismatch"></a>查找 Pod 失败，因为操作系统不匹配

在这种情况下，部署中包括一个在具有不匹配的 OS 版本且未启用 Hyper-v 隔离的节点上计划的 pod。

相同的错误也出现在标有 `kubectl describe pod <podname>` 的事件中。 多次尝试后，pod 状态可能 `CrashLoopBackOff`。

```
$ kubectl -n plang describe pod fabrikamfiber.web-789699744-rqv6p

Name:           fabrikamfiber.web-789699744-rqv6p
Namespace:      plang
Node:           38519acs9011/10.240.0.6
Start Time:     Mon, 09 Oct 2017 19:40:30 +0000
Labels:         io.kompose.service=fabrikamfiber.web
                pod-template-hash=789699744
Annotations:    kubernetes.io/created-by={"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicaSet","namespace":"plang","name":"fabrikamfiber.web-789699744","uid":"b5062a08-ad29-11e7-b16e-000d3a...
Status:         Running
IP:             10.244.3.169
Created By:     ReplicaSet/fabrikamfiber.web-789699744
Controlled By:  ReplicaSet/fabrikamfiber.web-789699744
Containers:
  fabrikamfiberweb:
    Container ID:       docker://eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a
    Image:              patricklang/fabrikamfiber.web:latest
    Image ID:           docker-pullable://patricklang/fabrikamfiber.web@sha256:562741016ce7d9a232a389449a4fd0a0a55aab178cf324144404812887250ead
    Port:               80/TCP
    State:              Waiting
      Reason:           CrashLoopBackOff
    Last State:         Terminated
      Reason:           ContainerCannotRun
      Message:          container eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a","Owner":"docker","IsDummy":false,"VolumePath":"\\\\?\\Volume{037b6606-bc9c-461f-ae02-829c28410798}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a","Layers":[{"ID":"f8bc427f-7aa3-59c6-b271-7331713e9451","Path":"C:\\ProgramData\\docker\\windowsfilter\\e206d2514a6265a76645b9d6b3dc6a78777c34dbf5da9fa2d564651645685881"},{"ID":"a6f35e41-a86c-5e4d-a19a-a6c2464bfb47","Path":"C:\\ProgramData\\docker\\windowsfilter\\0f21f1e28ef13030bbf0d87cbc97d1bc75f82ea53c842e9a3250a2156ced12d5"},{"ID":"4f624ca7-2c6d-5c42-b73f-be6e6baf2530","Path":"C:\\ProgramData\\docker\\windowsfilter\\4d9e2ad969fbd74fd58c98b5ab61e55a523087910da5200920e2b6f641d00c67"},{"ID":"88e360ff-32af-521d-9a3f-3760c12b35e2","Path":"C:\\ProgramData\\docker\\windowsfilter\\9e16a3d53d3e9b33344a6f0d4ed34c8a46448ee7636b672b61718225b8165e6e"},{"ID":"20f1a4e0-a6f3-5db3-9bf2-01fd3e9e855a","Path":"C:\\ProgramData\\docker\\windowsfilter\\7eec7f59f9adce38cc0a6755da58a3589287d920d37414b5b21b5b543d910461"},{"ID":"c2b3d728-4879-5343-a92a-b735752a4724","Path":"C:\\ProgramData\\docker\\windowsfilter\\8ed04b60acc0f65f541292a9e598d5f73019c8db425f8d49ea5819eab16a42f3"},{"ID":"2973e760-dc59-5800-a3de-ab9d93be81e5","Path":"C:\\ProgramData\\docker\\windowsfilter\\cc71305d45f09ce377ef497f28c3a74ee027c27f20657d2c4a5f157d2457cc75"},{"ID":"454a7d36-038c-5364-8a25-fa84091869d6","Path":"C:\\ProgramData\\docker\\windowsfilter\\9e8cde1ce8f5de861a5f22841f1ab9bc53d5f606d06efeacf5177f340e8d54d0"},{"ID":"9b748c8c-69eb-55fb-a1c1-5688cac4efd8","Path":"C:\\ProgramData\\docker\\windowsfilter\\8e02bf5404057055a71d542780a2bb2731be4b3707c01918ba969fb4d83b98ec"},{"ID":"bfde5c26-b33f-5424-9405-9d69c2fea4d0","Path":"C:\\ProgramData\\docker\\windowsfilter\\77483cedfb0964008c33d92d306734e1fab3b5e1ebb27e898f58ccfd108108f2"},{"ID":"bdabfbf5-80d1-57f1-86f3-448ce97e2d05","Path":"C:\\ProgramData\\docker\\windowsfilter\\aed2ebbb31ad24b38ee8521dd17744319f83d267bf7f360bc177e27ae9a006cf"},{"ID":"ad9b34f2-dcee-59ea-8962-b30704ae6331","Path":"C:\\ProgramData\\docker\\windowsfilter\\d44d3a675fec1070b61d6ea9bacef4ac12513caf16bd6453f043eed2792f75d8"}],"HostName":"fabrikamfiber.web-789699744-rqv6p","MappedDirectories":[{"HostPath":"c:\\var\\lib\\kubelet\\pods\\b50f0027-ad29-11e7-b16e-000d3afd2878\\volumes\\kubernetes.io~secret\\default-token-rw9dn","ContainerPath":"c:\\var\\run\\secrets\\kubernetes.io\\serviceaccount","ReadOnly":true,"BandwidthMaximum":0,"IOPSMaximum":0}],"HvPartition":false,"EndpointList":null,"NetworkSharedContainerName":"586870f5833279678773cb700db3c175afc81d557a75867bf39b43f985133d13","Servicing":false,"AllowUnqualifiedDNSQuery":false}
      Exit Code:        128
      Started:          Mon, 09 Oct 2017 20:27:08 +0000
      Finished:         Mon, 09 Oct 2017 20:27:08 +0000
    Ready:              False
    Restart Count:      10
    Environment:        <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-rw9dn (ro)
Conditions:
  Type          Status
  Initialized   True
  Ready         False
  PodScheduled  True
Volumes:
  default-token-rw9dn:
    Type:       Secret (a volume populated by a Secret)
    SecretName: default-token-rw9dn
    Optional:   false
QoS Class:      BestEffort
Node-Selectors: beta.kubernetes.io/os=windows
Tolerations:    <none>
Events:
  FirstSeen     LastSeen        Count   From                    SubObjectPath                           Type            Reason                  Message
  ---------     --------        -----   ----                    -------------                           --------        ------                  -------
  49m           49m             1       default-scheduler                                               Normal          Scheduled               Successfully assigned fabrikamfiber.web-789699744-rqv6p to 38519acs9011
  49m           49m             1       kubelet, 38519acs9011                                           Normal          SuccessfulMountVolume   MountVolume.SetUp succeeded for volume "default-token-rw9dn"
  29m           29m             1       kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Warning         Failed                  Failed to pull image "patricklang/fabrikamfiber.web:latest": rpc error: code = 2 desc = Error response from daemon: {"message":"Get https://registry-1.docker.io/v2/: dial tcp: lookup registry-1.docker.io: no such host"}
  49m           3m              12      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Normal          Pulling                 pulling image "patricklang/fabrikamfiber.web:latest"
  33m           3m              11      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Normal          Pulled                  Successfully pulled image "patricklang/fabrikamfiber.web:latest"
  33m           3m              11      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Normal          Created                 Created container
  33m           2m              11      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Warning         Failed                  Error: failed to start container "fabrikamfiberweb": Error response from daemon: {"message":"container fabrikamfiberweb encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {\"SystemType\":\"Container\",\"Name\":\"fabrikamfiberweb\",\"Owner\":\"docker\",\"IsDummy\":false,\"VolumePath\":\"\\\\\\\\?\\\\Volume{037b6606-bc9c-461f-ae02-829c28410798}\",\"IgnoreFlushesDuringBoot\":true,\"LayerFolderPath\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\fabrikamfiberweb\",\"Layers\":[{\"ID\":\"f8bc427f-7aa3-59c6-b271-7331713e9451\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\e206d2514a6265a76645b9d6b3dc6a78777c34dbf5da9fa2d564651645685881\"},{\"ID\":\"a6f35e41-a86c-5e4d-a19a-a6c2464bfb47\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\0f21f1e28ef13030bbf0d87cbc97d1bc75f82ea53c842e9a3250a2156ced12d5\"},{\"ID\":\"4f624ca7-2c6d-5c42-b73f-be6e6baf2530\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\4d9e2ad969fbd74fd58c98b5ab61e55a523087910da5200920e2b6f641d00c67\"},{\"ID\":\"88e360ff-32af-521d-9a3f-3760c12b35e2\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\9e16a3d53d3e9b33344a6f0d4ed34c8a46448ee7636b672b61718225b8165e6e\"},{\"ID\":\"20f1a4e0-a6f3-5db3-9bf2-01fd3e9e855a\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\7eec7f59f9adce38cc0a6755da58a3589287d920d37414b5b21b5b543d910461\"},{\"ID\":\"c2b3d728-4879-5343-a92a-b735752a4724\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\8ed04b60acc0f65f541292a9e598d5f73019c8db425f8d49ea5819eab16a42f3\"},{\"ID\":\"2973e760-dc59-5800-a3de-ab9d93be81e5\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\cc71305d45f09ce377ef497f28c3a74ee027c27f20657d2c4a5f157d2457cc75\"},{\"ID\":\"454a7d36-038c-5364-8a25-fa84091869d6\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\9e8cde1ce8f5de861a5f22841f1ab9bc53d5f606d06efeacf5177f340e8d54d0\"},{\"ID\":\"9b748c8c-69eb-55fb-a1c1-5688cac4efd8\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\8e02bf5404057055a71d542780a2bb2731be4b3707c01918ba969fb4d83b98ec\"},{\"ID\":\"bfde5c26-b33f-5424-9405-9d69c2fea4d0\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\77483cedfb0964008c33d92d306734e1fab3b5e1ebb27e898f58ccfd108108f2\"},{\"ID\":\"bdabfbf5-80d1-57f1-86f3-448ce97e2d05\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\aed2ebbb31ad24b38ee8521dd17744319f83d267bf7f360bc177e27ae9a006cf\"},{\"ID\":\"ad9b34f2-dcee-59ea-8962-b30704ae6331\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\d44d3a675fec1070b61d6ea9bacef4ac12513caf16bd6453f043eed2792f75d8\"}],\"HostName\":\"fabrikamfiber.web-789699744-rqv6p\",\"MappedDirectories\":[{\"HostPath\":\"c:\\\\var\\\\lib\\\\kubelet\\\\pods\\\\b50f0027-ad29-11e7-b16e-000d3afd2878\\\\volumes\\\\kubernetes.io~secret\\\\default-token-rw9dn\",\"ContainerPath\":\"c:\\\\var\\\\run\\\\secrets\\\\kubernetes.io\\\\serviceaccount\",\"ReadOnly\":true,\"BandwidthMaximum\":0,\"IOPSMaximum\":0}],\"HvPartition\":false,\"EndpointList\":null,\"NetworkSharedContainerName\":\"586870f5833279678773cb700db3c175afc81d557a75867bf39b43f985133d13\",\"Servicing\":false,\"AllowUnqualifiedDNSQuery\":false}"}
  33m           11s             151     kubelet, 38519acs9011                                           Warning         FailedSync              Error syncing pod
  32m           11s             139     kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Warning         BackOff                 Back-off restarting failed container
```

### <a name="mitigation---using-node-labels-and-nodeselector"></a>缓解-使用节点标签和 nodeSelector

运行**kubectl 获取节点**以获取所有节点的列表。 然后，可以运行**kubectl 说明节点（节点名称）** 来获取更多详细信息。

在下面的示例中，两个 Windows 节点运行不同的版本：

```
$ kubectl get node

NAME                        STATUS    AGE       VERSION
38519acs9010                Ready     21h       v1.7.7-7+e79c96c8ff2d8e
38519acs9011                Ready     4h        v1.7.7-25+bc3094f1d650a2
k8s-linuxpool1-38519084-0   Ready     21h       v1.7.7
k8s-master-38519084-0       Ready     21h       v1.7.7

$ kubectl describe node 38519acs9010

Name:                   38519acs9010
Role:
Labels:                 beta.kubernetes.io/arch=amd64
                        beta.kubernetes.io/instance-type=Standard_D2_v2
                        beta.kubernetes.io/os=windows
                        failure-domain.beta.kubernetes.io/region=westus2
                        failure-domain.beta.kubernetes.io/zone=0
                        kubernetes.io/hostname=38519acs9010
Annotations:            node.alpha.kubernetes.io/ttl=0
                        volumes.kubernetes.io/controller-managed-attach-detach=true
Taints:                 <none>
CreationTimestamp:      Fri, 06 Oct 2017 01:41:02 +0000

...
  
System Info:
 Machine ID:                    38519acs9010
 System UUID:
 Boot ID:
 Kernel Version:                10.0 14393 (14393.1715.amd64fre.rs1_release_inmarket.170906-1810)
 OS Image:
 Operating System:              windows
 Architecture:                  amd64
 ...
 
$ kubectl describe node 38519acs9011
Name:                   38519acs9011
Role:
Labels:                 beta.kubernetes.io/arch=amd64
                        beta.kubernetes.io/instance-type=Standard_DS1_v2
                        beta.kubernetes.io/os=windows
                        failure-domain.beta.kubernetes.io/region=westus2
                        failure-domain.beta.kubernetes.io/zone=0
                        kubernetes.io/hostname=38519acs9011
Annotations:            node.alpha.kubernetes.io/ttl=0
                        volumes.kubernetes.io/controller-managed-attach-detach=true
Taints:                 <none>
CreationTimestamp:      Fri, 06 Oct 2017 18:13:25 +0000
Conditions:
...

System Info:
 Machine ID:                    38519acs9011
 System UUID:
 Boot ID:
 Kernel Version:                10.0 16299 (16299.0.amd64fre.rs3_release.170922-1354)
 OS Image:
 Operating System:              windows
 Architecture:                  amd64
...

```

下面的示例演示如何匹配这些版本：

1. 记下每个节点名称，并从系统信息中 `Kernel Version`。

    在我们的示例中，信息如下所示：

    名称         | 版本
    -------------|--------------------------------------------------------
    38519acs9010 | 14393.1715.amd64fre.rs1_release_inmarket.170906-1810
    38519acs9011 | 16299.0.amd64fre.rs3_release.170922-1354

2. 向每个节点添加 `beta.kubernetes.io/osbuild` 标签。 Windows Server 2016 需要主要版本和次要版本（在本示例中为14393.1715），才能支持而无需 Hyper-v 隔离。 Windows Server 版本1709只需要匹配的主要版本（在本示例中为16299）。

    在此示例中，用于添加标签的命令如下所示：

    ```
    $ kubectl label node 38519acs9010 beta.kubernetes.io/osbuild=14393.1715


    node "38519acs9010" labeled
    $ kubectl label node 38519acs9011 beta.kubernetes.io/osbuild=16299

    node "38519acs9011" labeled

    ```

3. 通过运行**kubectl get 节点--show 标签**来检查标签是否存在。

    在此示例中，输出如下所示：

    ```
    $ kubectl get nodes --show-labels

    NAME                        STATUS                     AGE       VERSION                    LABELS
    38519acs9010                Ready,SchedulingDisabled   3d        v1.7.7-7+e79c96c8ff2d8e    beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=14393.1715,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9010
    38519acs9011                Ready                      3d        v1.7.7-25+bc3094f1d650a2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_DS1_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=16299,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9011
    k8s-linuxpool1-38519084-0   Ready                      3d        v1.7.7                     agentpool=linuxpool1,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-linuxpool1-38519084-0,kubernetes.io/role=agent
    k8s-master-38519084-0       Ready                      3d        v1.7.7                     beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-master-38519084-0,kubernetes.io/role=master
    ```

4. 将节点选择器添加到部署。 在此示例中，我们会将 `nodeSelector` 添加到容器规范中，`beta.kubernetes.io/os` = windows，`beta.kubernetes.io/osbuild` = 14393. * 或16299来匹配容器使用的基本操作系统。

    以下是运行适用于 Windows Server 2016 的容器的完整示例：

    ```yaml
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      annotations:
        kompose.cmd: kompose convert -f docker-compose-combined.yml
        kompose.version: 1.2.0 (99f88ef)
      creationTimestamp: null
      labels:
        io.kompose.service: fabrikamfiber.web
      name: fabrikamfiber.web
    spec:
      replicas: 1
      strategy: {}
      template:
        metadata:
          creationTimestamp: null
          labels:
            io.kompose.service: fabrikamfiber.web
        spec:
          containers:
          - image: patricklang/fabrikamfiber.web:latest
            name: fabrikamfiberweb
            ports:
            - containerPort: 80
            resources: {}
          restartPolicy: Always
          nodeSelector:
            "beta.kubernetes.io/os": windows
            "beta.kubernetes.io/osbuild": "14393.1715"
    status: {}
    ```

    现在可以使用更新后的部署来启动 Pod。 节点选择器也会显示在 `kubectl describe pod <podname>`中，因此你可以运行该命令来验证它们是否已添加。

    示例的输出如下所示：

    ```
    $ kubectl -n plang describe po fa

    Name:           fabrikamfiber.web-1780117715-5c8vw
    Namespace:      plang
    Node:           38519acs9010/10.240.0.4
    Start Time:     Tue, 10 Oct 2017 01:43:28 +0000
    Labels:         io.kompose.service=fabrikamfiber.web
                    pod-template-hash=1780117715
    Annotations:    kubernetes.io/created-by={"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicaSet","namespace":"plang","name":"fabrikamfiber.web-1780117715","uid":"6a07aaf3-ad5c-11e7-b16e-000d3...
    Status:         Running
    IP:             10.244.1.84
    Created By:     ReplicaSet/fabrikamfiber.web-1780117715
    Controlled By:  ReplicaSet/fabrikamfiber.web-1780117715
    Containers:
      fabrikamfiberweb:
        Container ID:       docker://c94594fb53161f3821cf050d9af7546991aaafbeab41d333d9f64291327fae13
        Image:              patricklang/fabrikamfiber.web:latest
        Image ID:           docker-pullable://patricklang/fabrikamfiber.web@sha256:562741016ce7d9a232a389449a4fd0a0a55aab178cf324144404812887250ead
        Port:               80/TCP
        State:              Running
          Started:          Tue, 10 Oct 2017 01:43:42 +0000
        Ready:              True
        Restart Count:      0
        Environment:        <none>
        Mounts:
          /var/run/secrets/kubernetes.io/serviceaccount from default-token-rw9dn (ro)
    Conditions:
      Type          Status
      Initialized   True
      Ready         True
      PodScheduled  True
    Volumes:
      default-token-rw9dn:
        Type:       Secret (a volume populated by a Secret)
        SecretName: default-token-rw9dn
        Optional:   false
    QoS Class:      BestEffort
    Node-Selectors: beta.kubernetes.io/os=windows
                    beta.kubernetes.io/osbuild=14393.1715
    Tolerations:    <none>
    Events:
      FirstSeen     LastSeen        Count   From                    SubObjectPath                           Type            Reason                  Message
      ---------     --------        -----   ----                    -------------                           --------        ------                  -------
      5m            5m              1       default-scheduler                                               Normal          Scheduled               Successfully assigned fabrikamfiber.web-1780117715-5c8vw to 38519acs9010
      5m            5m              1       kubelet, 38519acs9010                                           Normal          SuccessfulMountVolume   MountVolume.SetUp succeeded for volume "default-token-rw9dn"
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Pulling                 pulling image "patricklang/fabrikamfiber.web:latest"
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Pulled                  Successfully pulled image "patricklang/fabrikamfiber.web:latest"
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Created                 Created container
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Started                 Started container
    ```
