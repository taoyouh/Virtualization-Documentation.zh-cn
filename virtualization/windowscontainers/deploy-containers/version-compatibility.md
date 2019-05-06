---
title: Windows 容器版本兼容性
description: Windows 如何跨多个版本运行内部版本和容器
keywords: 元数据, 容器, 版本
author: taylorb-microsoft
ms.openlocfilehash: 76549bbfbaf374acb79f1be4280949aecf4e87f0
ms.sourcegitcommit: c48dcfe43f73b96e0ebd661164b6dd164c775bfa
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "9610277"
---
# <a name="windows-container-version-compatibility"></a>Windows 容器版本兼容性

Windows Server 2016 和 Windows 10 周年更新 （这两个版本 14393） 是第一批 Windows 版本，可以生成并运行 Windows Server 容器。 使用这些版本生成的容器可以在 Windows Server 版本 1709 等更高版本上运行，但在开始前你需要了解一些事项。

由于我们一直在改进 Windows 容器功能，我们不得不进行一些可能影响兼容性的变更。 早期的容器相同更高版本的主机上运行带有[HYPER-V 隔离](../manage-containers/hyperv-container.md)，并将使用相同的 （早期） 内核版本。 但是，如果你想要运行基于更高版本的 Windows 版本的容器，它只能运行在更高版本的主机版本上。

|容器操作系统版本|主机操作系统版本|兼容性|
|---|---|---|
|Windows Server 2016<br>内部版本：14393.* |Windows Server 2016<br>内部版本：14393.* |支持`process`或`hyperv`隔离|
|Windows Server 2016<br>内部版本：14393.* |Windows Server 版本 1709<br>内部版本：16299.* |仅支持`hyperv`隔离|
|Windows Server 2016<br>内部版本：14393.* |Windows 10 Fall Creators Update<br>内部版本：16299.* |仅支持`hyperv`隔离|
|Windows Server 2016<br>内部版本：14393.* |Windows Server 版本 1803<br>生成 17134.* |仅支持`hyperv`隔离|
|Windows Server 2016<br>内部版本：14393.* |Windows 10 版本 1803<br>生成 17134.* |仅支持`hyperv`隔离|
|Windows Server 2016<br>内部版本：14393.* |Windows Server 2019<br>生成 17763.* |仅支持`hyperv`隔离|
|Windows Server 2016<br>内部版本：14393.* |Windows 10 版本 1809<br>生成 17763.* |仅支持`hyperv`隔离|
|Windows Server 版本 1709<br>内部版本：16299.* |Windows Server 2016<br>内部版本：14393.* |不支持|
|Windows Server 版本 1709<br>内部版本：16299.* |Windows Server 版本 1709<br>内部版本：16299.* |支持`process`或`hyperv`隔离|
|Windows Server 版本 1709<br>内部版本：16299.* |Windows 10 Fall Creators Update<br>内部版本：16299.* |仅支持`hyperv`隔离|
|Windows Server 版本 1709<br>内部版本：16299.* |Windows Server 版本 1803<br>生成 17134.* |仅支持`hyperv`隔离|
|Windows Server 版本 1709<br>内部版本：16299.* |Windows 10 版本 1803<br>生成 17134.* |仅支持`hyperv`隔离|
|Windows Server 版本 1709<br>内部版本：16299.* |Windows Server 2019<br>生成 17763.* |仅支持`hyperv`隔离|
|Windows Server 版本 1709<br>内部版本：16299.* |Windows 10 版本 1809<br>生成 17763.* |仅支持`hyperv`隔离|
|Windows Server 版本 1803<br>生成 17134.* |Windows Server 2016<br>内部版本：14393.* |不支持|
|Windows Server 版本 1803<br>生成 17134.* |Windows Server 版本 1709<br>内部版本：16299.* |不支持|
|Windows Server 版本 1803<br>生成 17134.* |Windows 10 Fall Creators Update<br>内部版本：16299.* |不支持|
|Windows Server 版本 1803<br>生成 17134.* |Windows Server 版本 1803<br>生成 17134.* |支持`process`或`hyperv`隔离|
|Windows Server 版本 1803<br>生成 17134.* |Windows 10 版本 1803<br>生成 17134.* |仅支持`hyperv`隔离|
|Windows Server 版本 1803<br>生成 17134.* |Windows Server 2019<br>生成 17763.* |仅支持`hyperv`隔离|
|Windows Server 版本 1803<br>生成 17134.* |Windows 10 版本 1809<br>生成 17763.* |仅支持`hyperv`隔离|
|Windows Server 2019<br>生成 17763.* |Windows Server 2016<br>内部版本：14393.* |不支持|
|Windows Server 2019<br>生成 17763.* |Windows Server 版本 1709<br>内部版本：16299.* |不支持
|Windows Server 2019<br>生成 17763.* |Windows 10 Fall Creators Update<br>内部版本：16299.* |不支持|
|Windows Server 2019<br>生成 17763.* |Windows Server 版本 1803<br>生成 17134.* |不支持|
|Windows Server 2019<br>生成 17763.* |Windows 10 版本 1803<br>生成 17134.* |不支持|
|Windows Server 2019<br>生成 17763.* |Windows Server 2019<br>生成 17763.* |支持`process`或`hyperv`隔离|
|Windows Server 2019<br>生成 17763.* |Windows 10 版本 1809<br>生成 17763.* |仅支持`hyperv`隔离|

## <a name="matching-container-host-version-with-container-image-versions"></a>容器主机版本与容器映像版本相匹配

### <a name="windows-server-containers"></a>Windows Server 容器

由于 Windows Server 容器和基础主机共享一个内核，容器的基本映像必须匹配的主机。 如果版本不同，容器可以启动，但得不保证。 Windows 操作系统具有四个级别的版本： 主要、 次要版本和修订版本。 例如，版本 10.0.14393.103 必须 10 的主要版本、 次要版本为 0，14393，内部版本号和 103 修订号。 内部版本号才会改变时操作系统的新版本发布，诸如版本 1709、 1803、 Fall Creators Update 中，等等。 应用 Windows 更新后，会相应更新修订版本号。

#### <a name="build-number-new-release-of-windows"></a>内部版本号 （新版本的 Windows）

Windows Server 容器会阻止启动时不同，容器主机和容器映像的内部版本号。 例如，当容器主机是版本 10.0.14393.* (Windows Server 2016) 和容器映像是版本 10.0.16299.* (Windows Server 版本 1709年)，该容器将不会启动。  

#### <a name="revision-number-patching"></a>修订号 （修补）

Windows Server 容器不会阻止启动不同的容器主机和容器映像的修订号时。 例如，如果容器主机是版本 10.0.14393.1914 (与应用的 KB4051033 的 Windows Server 2016)，容器映像是版本 10.0.14393.1944 (与应用的 KB4053579 的 Windows Server 2016)，然后该图像将仍然开始菜单即使他们的修订版本数量是不同。

对于基于 Windows Server 2016 的主机或图像，容器映像的修订版必须与要处于支持配置中的主机匹配。 但是，对于主机或使用 Windows Server 版本 1709年和更高版本的图像，不适用于此规则，并且主机和容器映像不需要具有匹配的修订版。 我们建议你保留在系统使用最新修补程序和更新保持最新状态。

#### <a name="practical-application"></a>实际应用程序

示例 1： 容器主机正在运行应用了 KB4041691 Windows Server 2016。 部署到此主机的任何 Windows Server 容器都必须基于版本 10.0.14393.1770 容器基本映像。 如果主机容器中应用了 KB4053579，还必须更新以确保主机容器支持它们的图像。

示例 2： 容器主机正在运行应用了 KB4043961 Windows Server 版本 1709年。 部署到此主机的任何 Windows Server 容器必须基于 Windows Server 版本 1709 (10.0.16299) 容器基本映像，但不需要与主机 KB 匹配。 如果对主机应用 KB4054517，则仍会支持容器映像，但我们建议你更新它们以处理任何潜在的安全问题。

#### <a name="querying-version"></a>查询版本

方法 1： 在版本 1709年中引入，cmd 提示符和**ver**命令现在会返回修订版详细信息。

```batch
Microsoft Windows [Version 10.0.16299.125]
(c) 2017 Microsoft Corporation. All rights reserved.

C:\>ver

Microsoft Windows [Version 10.0.16299.125]
```

方法 2： 查询以下注册表项： HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion

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

若要检查的是哪个版本你基本映像的使用，查看 Docker 中心或映像说明提供的图像哈希表上的标记。 每个版本和修订版本发布时，列出了[Windows 10 更新历史记录](https://support.microsoft.com/en-us/help/12387/windows-10-update-history)页面。

### <a name="hyper-v-isolation-for-containers"></a>容器 HYPER-V 隔离

你可以使用或不具备 HYPER-V 隔离运行 Windows 容器。 Hyper-V 隔离使用优化的 VM 在容器周围创造安全边界。 与标准 Windows 容器容器和主机之间共享内核，每个 HYPER-V 隔离的容器都有其自己的 Windows 内核实例。 这意味着你可以在容器主机和映像中的不同操作系统版本 （有关详细信息，请参阅下面的兼容性矩阵）。  

若要运行带有 Hyper-V 隔离的容器，只需在你的 Docker 运行命令中添加标记 `--isolation=hyperv`。

## <a name="errors-from-mismatched-versions"></a>不匹配版本引发的错误

如果你尝试运行不受支持的组合，你将收到以下错误：

```dockerfile
docker: Error response from daemon: container b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Owner":"docker","IsDummy":false,"VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Layers":[{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"b81ed896222e","MappedDirectories":[],"HvPartition":false,"EndpointList":["002a0d9e-13b7-42c0-89b2-c1e80d9af243"],"Servicing":false,"AllowUnqualifiedDNSQuery":true}.
```

有三种方法可以解决此错误：

- 重新生成基于的正确版本的容器`microsoft/nanoserver`或 `microsoft/windowsservercore`
- 如果主机是更高版本，请运行**docker run-隔离 = hyperv...**
- 请尝试使用相同的 Windows 版本的不同主机上运行容器

## <a name="choose-which-container-os-version-to-use"></a>选择要使用的容器操作系统版本

>[!NOTE]
>在"最新"标记将随 Windows Server 2016，当前[Long-Term Servicing Channel 产品](https://docs.microsoft.com/en-us/windows-server/get-started/semi-annual-channel-overview)更新。 以下是有关说明匹配的 Windows Server 版本 1709年版本的容器映像。

你必须知道你需要为你的容器使用哪个版本。 例如，如果你使用的 Windows Server 版本 1709年并想要为其具有最新修补程序，则应使用标记`1709`时指定你希望哪个版本的基础操作系统容器映像，如下所示：

``` dockerfile
FROM microsoft/windowsservercore:1709
...
```

但是，如果你希望 Windows Server 版本 1709年的特定的修补程序，你可以在标记中指定 KB 编号。 例如，若要获取从 Windows Server 版本 1709 kb4043961 到它的 Nano Server 基础操作系统容器映像，你应指定它如下所示：

``` dockerfile
FROM microsoft/nanoserver:1709_KB4043961
...
```

如果你需要 Nano Server 基础操作系统容器映像从 Windows Server 2016，你仍然可以通过使用"最新"标记获取这些基础操作系统容器映像的最新版本：

``` dockerfile
FROM microsoft/nanoserver
...
```

你还可以与我们使用了之前，通过在标记中指定的操作系统版本的架构来指定所需的确切修补程序：

``` dockerfile
FROM microsoft/nanoserver:10.0.14393.1770
...
```

## <a name="matching-versions-using-docker-swarm"></a>使用 Docker Swarm 对版本进行匹配

Docker 群当前没有匹配的容器使用到具有相同版本的主机的 Windows 版本的内置方法。 如果你更新使用较新容器的服务，它将成功运行。

如果你需要较长时间内运行多个 Windows 版本，有两种方法可以采取： 通过 Windows 主机配置为始终使用 HYPER-V 隔离，或使用标签限制。

### <a name="finding-a-service-that-wont-start"></a>查找无法启动的服务

如果服务无法启动，你将看到`MODE`是`replicated`，但`REPLICAS`将停留在 0。 若要查看的操作系统版本是否有问题，请运行以下命令：

运行**docker 服务 ls**若要查找服务名称：

```dockerfile
ID                  NAME                MODE                REPLICAS            IMAGE                                             PORTS
xh6mwbdq2uil        angry_liskov        replicated          0/1                 microsoft/iis:windowsservercore-10.0.14393.1715
```

运行**docker 服务 ps （服务名称）** 若要获取状态和最近尝试：

```dockerfile
C:\Program Files\Docker>docker service ps angry_liskov
ID                  NAME                 IMAGE                                             NODE                DESIRED STATE       CURRENT STATE               ERROR                              PORTS
klkbhn742lv0        angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Ready               Ready 3 seconds ago
y5blbdum70zo         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 24 seconds ago       "starting container failed: co…"
yjq6zwzqj8kt         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 31 seconds ago       "starting container failed: co…"

ytnnv80p03xx         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
xeqkxbsao57w         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
```

如果你看到`starting container failed: ...`，你可以看到带有**docker 服务 ps-无 trunc （容器名称）** 的完整错误：

```dockerfile
C:\Program Files\Docker>docker service ps --no-trunc angry_liskov
ID                          NAME                 IMAGE                                                                                                                     NODE                DESIRED STATE       CURRENT STATE                     ERROR                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          PORTS
dwsd6sjlwsgic5vrglhtxu178   angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Running             Starting less than a second ago                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              
y5blbdum70zoh1f6uhx5nxsfv    \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Shutdown            Failed 39 seconds ago             "starting container failed: container e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Owner":"docker","VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Layers":[{"ID":"bcf2630f-ea95-529b-b33c-e5cdab0afdb4","Path":"C:\\ProgramData\\docker\\windowsfilter\\200235127f92416724ae1d53ed3fdc86d78767132d019bdda1e1192ee4cf3ae4"},{"ID":"e3ea10a8-4c2f-5b93-b2aa-720982f116f6","Path":"C:\\ProgramData\\docker\\windowsfilter\\0ccc9fa71a9f4c5f6f3bc8134fe3533e454e09f453de662cf99ab5d2106abbdc"},{"ID":"cff5391f-e481-593c-aff7-12e080c653ab","Path":"C:\\ProgramData\\docker\\windowsfilter\\a49576b24cd6ec4a26202871c36c0a2083d507394a3072186133131a72601a31"},{"ID":"499cb51e-b891-549a-b1f4-8a25a4665fbd","Path":"C:\\ProgramData\\docker\\windowsfilter\\fdf2f52c4323c62f7ff9b031c0bc3af42cf5fba91098d51089d039fb3e834c08"},{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"e7b5d3adba7e","HvPartition":false,"EndpointList":["298bb656-8800-4948-a41c-1b0500f3d94c"],"AllowUnqualifiedDNSQuery":true}"
```

这是为相同的错误`CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101)`。

### <a name="fix---update-the-service-to-use-a-matching-version"></a>修复 - 将服务更新为使用匹配版本

对于 Docker Swarm，有两点注意事项。 必须具有使用的图像未创建的服务的 compose 文件的情况下，你会想要引用进行相应更新。 例如：

``` yaml
version: '3'

services:
  YourServiceName:
    image: microsoft/windowsservercore:1709
...
```

另一点注意事项是指向的映像是否是你自己创建 (例如，contoso/myimage):

```yaml
version: '3'

services:
  YourServiceName:
    image: contoso/myimage
...
```

在此情况下，应使用[匹配版本引发的错误](#errors-from-mismatched-versions)中所述的方法修改该 dockerfile 而不是修改 docker-compose 行。

### <a name="mitigation---use-hyper-v-isolation-with-docker-swarm"></a>缓解 - 结合使用 Hyper-V 隔离与 Docker Swarm

一种方案支持按容器逐个使用 HYPER-V 隔离，但代码尚未完成。 你可以在 [GitHub](https://github.com/moby/moby/issues/31616) 上关注进展。 在实现之前，我们需要将主机配置为始终使用 Hyper-V 隔离运行。

这需要更改 Docker 服务配置，然后重启 Docker 引擎。

1. 编辑 `C:\ProgramData\docker\config\daemon.json`
2. 针对以下内容添加一行 `"exec-opts":["isolation=hyperv"]`

    >[!NOTE]
    >默认情况下，daemon.json 文件不存在。 如果在查看目录时发现该文件不存在，你必须创建该文件。 然后你会想要复制以下：

    ```JSON
    {
        "exec-opts":["isolation=hyperv"]
    }
    ```

3. 关闭并保存该文件，然后重启 docker 引擎在 PowerShell 中运行以下 cmdlet:

    ```powershell
    Stop-Service docker
    Start-Service docker
    ```

4. 已重新启动该服务后，启动容器。 它们运行后，可以通过检查以下 cmdlet 的容器来验证容器的隔离级别：

    ```powershell
    docker inspect --format='{{json .HostConfig.Isolation}}' $instanceNameOrId
    ```

将返回“process”或“hyperv”结果。 如果已按照上述内容对 daemon.json 文件进行修改和设置，显示的结果将是后者。

### <a name="mitigation---use-labels-and-constraints"></a>缓解 - 使用标签和限制

下面介绍了如何使用标签和约束以匹配版本：

1. 将标签添加到每个节点。

    在每个节点上，添加两个标签：`OS`和`OsVersion`。 这样做的假设前提是你在本地运行但可以修改为在远程主机上对标签进行设置。

    ```powershell
    docker node update --label-add OS="windows" $ENV:COMPUTERNAME
    docker node update --label-add OsVersion="$((Get-ComputerInfo).OsVersion)" $ENV:COMPUTERNAME
    ```

    之后，你可以查看这些通过运行**docker 节点检查**命令，应显示新添加的标签：

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

2. 添加服务限制。

    既然你已标记为每个节点，你可以更新确定服务放置的约束。 在以下示例中，将"contoso_service 为"替换你实际服务的名称：

    ```powershell
    docker service update \
        --constraint-add "node.labels.OS == windows" \
        --constraint-add "node.labels.OsVersion == $((Get-ComputerInfo).OsVersion)" \
        contoso_service
    ```

    这将实施并限制节点运行的位置。

若要了解有关如何使用服务限制的详细信息，请查看[服务创建引用](https://docs.docker.com/engine/reference/commandline/service_create/#specify-service-constraints-constraint)。

## <a name="matching-versions-using-kubernetes"></a>使用 Kubernetes 对版本进行匹配

在 Kubernetes 中计划 pod 时，也会出现相同问题，[使用 Docker 群的匹配版本](#matching-versions-using-docker-swarm)中所述。 使用类似的策略，可以避免此问题：

- 重新生成容器，具体取决于在开发和生产中的相同操作系统版本。 若要了解如何操作，请参阅[选择要使用的操作系统版本的容器](#choose-which-container-os-version-to-use)。
- 使用节点标签和 Nodeselector 来确保如果 Windows Server 2016 和 Windows Server 版本 1709年的节点均在同一群集中 pod 兼容节点上计划
- 基于操作系统版本使用单独的群集

### <a name="finding-pods-failed-on-os-mismatch"></a>查找 Pod 失败，因为操作系统不匹配

在此情况下，部署所包含的 pod 计划位于采用不匹配的操作系统版本，且未启用 HYPER-V 隔离的节点。

相同的错误也出现在标有 `kubectl describe pod <podname>` 的事件中。 在多次尝试，pod 状态将可能是`CrashLoopBackOff`。

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

运行**kubectl 获取节点**来获取所有节点的列表。 在此之后，你可以运行**kubectl 描述节点 （节点名称）** 以获取更多详细信息。

在以下示例中，两个 Windows 节点运行不同版本：

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

让我们使用此示例显示了如何匹配版本：

1. 请记下每个节点的名称和`Kernel Version`从系统信息。

    在我们的示例，信息将如下所示：

    名称         | 版本
    -------------|--------------------------------------------------------
    38519acs9010 | 14393.1715.amd64fre.rs1_release_inmarket.170906-1810
    38519acs9011 | 16299.0.amd64fre.rs3_release.170922-1354

2. 向每个节点添加 `beta.kubernetes.io/osbuild` 标签。 Windows Server 2016 需要主要和次要版本 (在此示例中的版本 14393.1715) 必须支持无需 HYPER-V 隔离。 Windows Server 版本 1709年仅需要的主要版本 (16299 在此示例中) 以匹配。

    在此示例中，添加标签的命令如下所示：

    ```
    $ kubectl label node 38519acs9010 beta.kubernetes.io/osbuild=14393.1715


    node "38519acs9010" labeled
    $ kubectl label node 38519acs9011 beta.kubernetes.io/osbuild=16299

    node "38519acs9011" labeled

    ```

3. 检查存在标签通过运行**kubectl 获取节点-显示标签**。

    在此示例中，输出将如下所示：

    ```
    $ kubectl get nodes --show-labels

    NAME                        STATUS                     AGE       VERSION                    LABELS
    38519acs9010                Ready,SchedulingDisabled   3d        v1.7.7-7+e79c96c8ff2d8e    beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=14393.1715,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9010
    38519acs9011                Ready                      3d        v1.7.7-25+bc3094f1d650a2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_DS1_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=16299,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9011
    k8s-linuxpool1-38519084-0   Ready                      3d        v1.7.7                     agentpool=linuxpool1,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-linuxpool1-38519084-0,kubernetes.io/role=agent
    k8s-master-38519084-0       Ready                      3d        v1.7.7                     beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-master-38519084-0,kubernetes.io/role=master
    ```

4. 将节点选择器添加到部署。 在此示例的情况，我们将添加`nodeSelector`为使用的容器规范`beta.kubernetes.io/os`= windows 和`beta.kubernetes.io/osbuild`= 14393 或 16299 才可由容器基本操作系统匹配。

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

    现在可以使用更新后的部署来启动 Pod。 节点选择器还会显示在`kubectl describe pod <podname>`，因此你可以运行该命令以确认它们已经添加。

    对于我们的示例输出如下所示：

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
