---
title: 隔离模式
description: 说明 Hyper-v 隔离如何与进程隔离容器不同。
keywords: docker, 容器
author: crwilhit
ms.date: 09/26/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: fa95ffe1c699a2c837076fcc1b662f6b792b7dfb
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909747"
---
# <a name="isolation-modes"></a>隔离模式

Windows 容器提供了两种不同的运行时隔离模式： `process` 和 `Hyper-V` 隔离。 在这两种隔离模式下运行的容器的创建、管理和功能都是相同的。 它们也生成和使用相同的容器映像。 隔离模式的不同之处在于在容器、主机操作系统以及在该主机上运行的所有其他容器之间创建的隔离程度。

## <a name="process-isolation"></a>进程隔离

这是 "传统" 容器隔离模式， [Windows 容器概述](../about/index.md)中对其进行了介绍。 通过进程隔离，多个容器实例可在给定主机上并发运行，并通过命名空间、资源控制和进程隔离技术提供隔离。 在此模式下运行时，容器与主机以及彼此共享同一个内核。  这大致与 Linux 容器的运行方式相同。

![](media/container-arch-process.png)

## <a name="hyper-v-isolation"></a>Hyper-V 隔离
此隔离模式提供主机和容器版本之间的增强安全性和更广泛的兼容性。 使用 Hyper-v 隔离时，在主机上并发运行多个容器实例;但是，每个容器都在高度优化的虚拟机中运行，并有效地获取其自己的内核。 虚拟机的状态在每个容器和容器主机之间提供硬件级隔离。

![](media/container-arch-hyperv.png)

## <a name="isolation-examples"></a>隔离示例

### <a name="create-container"></a>创建容器

通过 Docker 管理 Hyper-v 隔离容器与管理进程隔离容器几乎完全相同。 若要使用 Hyper-v 隔离完全 Docker 创建容器，请使用 `--isolation` 参数设置 `--isolation=hyperv`。

```cmd
docker run -it --isolation=hyperv mcr.microsoft.com/windows/servercore:ltsc2019 cmd
```

若要使用进程隔离完全 Docker 创建容器，请使用 `--isolation` 参数设置 `--isolation=process`。

```cmd
docker run -it --isolation=process mcr.microsoft.com/windows/servercore:ltsc2019 cmd
```

Windows Server 上运行的 windows 容器默认为在进程隔离下运行。 Windows 10 专业版和企业版上运行的 windows 容器默认为通过 Hyper-v 隔离运行。 从 Windows 10 年10月2018版更新开始，运行 Windows 10 专业版或企业版主机的用户可以运行包含进程隔离的 Windows 容器。 用户必须使用 `--isolation=process` 标志直接请求进程隔离。

> [!WARNING]
> 在 Windows 10 专业版和企业版上通过进程隔离运行用于开发/测试。 你的主机必须运行 Windows 10 build 17763 +，并且你必须有包含引擎18.09 或更高版本的 Docker 版本。
> 
> 应该继续使用 Windows Server 作为生产部署的主机。 通过在 Windows 10 专业版和企业版上使用此功能，还必须确保主机和容器版本标记相匹配，否则容器可能无法启动或出现未定义的行为。

### <a name="isolation-explanation"></a>隔离说明

此示例演示了进程与 Hyper-v 隔离之间的隔离功能之间的差异。

此处，会部署一个与进程隔离的容器，并将托管一个长时间运行的 ping 进程。

``` cmd
docker run -d mcr.microsoft.com/windows/servercore:ltsc2019 ping localhost -t
```

使用 `docker top` 命令，会返回 ping 进程，如容器中所见。 此实例中进程的 ID 为 3964。

``` cmd
docker top 1f8bf89026c8f66921a55e773bac1c60174bb6bab52ef427c6c8dbc8698f9d7a

3964 ping
```

在容器主机上，`get-process` 命令可用于从主机中返回任何运行的 ping 进程。 在此实例中有一个进程，并且该进程 ID 与容器中的进程 ID 相匹配。 它是通过容器和主机所看到的同一进程。

```
get-process -Name ping

Handles  NPM(K)    PM(K)      WS(K) VM(M)   CPU(s)     Id  SI ProcessName
-------  ------    -----      ----- -----   ------     --  -- -----------
     67       5      820       3836 ...71     0.03   3964   3 PING
```

相反，此示例还会启动带有 ping 进程的 Hyper-v solated 容器。

```
docker run -d --isolation=hyperv mcr.microsoft.com/windows/servercore:ltsc2019 ping localhost -t
```

同样，`docker top` 可用于从容器中返回正在运行的进程。

```
docker top 5d5611e38b31a41879d37a94468a1e11dc1086dcd009e2640d36023aa1663e62

1732 ping
```

但是，在容器主机上搜索进程时，找不到 ping 进程，并引发错误。

```
get-process -Name ping

get-process : Cannot find a process with the name "ping". Verify the process name and call the cmdlet again.
At line:1 char:1
+ get-process -Name ping
+ ~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (ping:String) [Get-Process], ProcessCommandException
    + FullyQualifiedErrorId : NoProcessFoundForGivenName,Microsoft.PowerShell.Commands.GetProcessCommand
```

最后，在主机上，`vmwp` 进程可见，该进程即为运行中的虚拟机，其正在封装运行中的容器并从主机操作系统保护运行中的进程。

```
get-process -Name vmwp

Handles  NPM(K)    PM(K)      WS(K) VM(M)   CPU(s)     Id  SI ProcessName
-------  ------    -----      ----- -----   ------     --  -- -----------
   1737      15    39452      19620 ...61     5.55   2376   0 vmwp
```
