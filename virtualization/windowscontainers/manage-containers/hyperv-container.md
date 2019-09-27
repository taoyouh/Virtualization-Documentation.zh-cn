---
title: 隔离模式
description: 有关 Hyper-v 隔离与进程隔离的容器有何区别的说明。
keywords: docker, 容器
author: crwilhit
ms.date: 09/26/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: fa95ffe1c699a2c837076fcc1b662f6b792b7dfb
ms.sourcegitcommit: e9dda81f1f68359ece9ef132a184a30880bcdb1b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/27/2019
ms.locfileid: "10161724"
---
# <a name="isolation-modes"></a>隔离模式

Windows 容器提供两种不同的运行时隔离`process`模式`Hyper-V` ：和隔离模式。 在两个隔离模式下运行的容器都创建、托管且功能相同。 它们也生成和使用相同的容器映像。 隔离模式的区别在于在容器、主机操作系统和该主机上运行的所有其他容器之间创建的隔离级别。

## <a name="process-isolation"></a>进程隔离

这是容器的 "传统" 隔离模式，它是[Windows 容器概述](../about/index.md)中介绍的内容。 使用进程隔离，通过命名空间、资源控制和进程隔离技术提供隔离的给定主机上并发运行多个容器实例。 在此模式下运行时，容器与主机以及彼此共享同一内核。  这与 Linux 容器的运行方式大致相同。

![](media/container-arch-process.png)

## <a name="hyper-v-isolation"></a>Hyper-V 隔离
此隔离模式提供了主机和容器版本之间的增强安全性和更广泛的兼容性。 通过 Hyper-v 隔离，在主机上并发运行多个容器实例;但是，每个容器都在高度优化的虚拟机内运行，并有效地获取其自己的内核。 虚拟机的存在在每个容器和容器主机之间提供硬件级别隔离。

![](media/container-arch-hyperv.png)

## <a name="isolation-examples"></a>隔离示例

### <a name="create-container"></a>创建容器

管理具有 Docker 的 Hyper-v 隔离容器几乎等同于管理进程隔离的容器。 若要使用 Hyper-v 隔离完全 Docker 创建容器，请使用`--isolation`参数进行设置。 `--isolation=hyperv`

```cmd
docker run -it --isolation=hyperv mcr.microsoft.com/windows/servercore:ltsc2019 cmd
```

若要使用进程隔离完全 Docker 创建容器，请使用`--isolation`参数进行设置`--isolation=process`。

```cmd
docker run -it --isolation=process mcr.microsoft.com/windows/servercore:ltsc2019 cmd
```

Windows Server 上运行的 windows 容器默认为通过进程隔离运行。 Windows 10 专业版和企业版上运行的 windows 容器默认为通过 Hyper-v 隔离运行。 从 Windows 10 年 2018 10 月的更新开始，运行 Windows 10 专业版或企业版主机的用户可以运行具有进程隔离的 Windows 容器。 用户必须通过使用`--isolation=process`标志直接请求进程隔离。

> [!WARNING]
> 在 Windows 10 专业版和企业版上通过进程隔离运行的目的是用于开发/测试。 你的主机必须运行 Windows 10 内部版本 17763 +，并且你必须具有带有 Engine 18.09 或更高版本的 Docker 版本。
> 
> 你应继续使用 Windows Server 作为生产部署的主机。 通过在 Windows 10 专业版和企业版上使用此功能，你还必须确保你的主机和容器版本标记匹配，否则容器可能无法启动或表现出未定义的行为。

### <a name="isolation-explanation"></a>隔离说明

此示例演示了进程和 Hyper-v 隔离之间的隔离功能之间的差异。

此处，将部署进程隔离的容器，并将托管运行时间较长的 ping 进程。

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

为对比度，此示例还启动带有 ping 进程的 Hyper-v solated 容器。

```
docker run -d --isolation=hyperv mcr.microsoft.com/windows/servercore:ltsc2019 ping localhost -t
```

同样，`docker top` 可用于从容器中返回正在运行的进程。

```
docker top 5d5611e38b31a41879d37a94468a1e11dc1086dcd009e2640d36023aa1663e62

1732 ping
```

但是，当在容器主机上搜索进程时，找不到 ping 进程，并引发错误。

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
