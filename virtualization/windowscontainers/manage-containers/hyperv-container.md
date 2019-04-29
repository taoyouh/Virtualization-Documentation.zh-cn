---
title: Hyper-V 隔离
description: 解释的 HYPER-V 隔离有何不同进程隔离容器中。
keywords: docker, 容器
author: scooley
ms.date: 09/13/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: 2ff2d1204e1f973d49af5e1d4441e4eacd946101
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/26/2019
ms.locfileid: "9576898"
---
# <a name="hyper-v-isolation"></a>Hyper-V 隔离

Windows 容器技术包括两个不同级别的隔离容器、 进程和 HYPER-V 隔离。 这两种类型的创建、 管理和功能完全相同。 它们也生成和使用相同的容器映像。 它们之间的不同之处是在容器、主机操作系统以及在该主机上运行的所有其他容器之间创建的隔离级别。

**进程隔离**– 多个容器实例可同时运行主机，隔离通过提供的命名空间、 资源控制，以及进程隔离技术。  容器与主机中，以及彼此共享同一个内核。  这是大约相同如何在 Linux 上运行容器。

**HYPER-V 隔离**– 多个容器实例可同时运行在主机上，但是，每个容器都在特定虚拟机的内部运行。 这提供了每个容器，以及在容器主机之间的内核级别隔离。

## <a name="hyper-v-isolation-examples"></a>HYPER-V 隔离示例

### <a name="create-container"></a>创建容器

管理 HYPER-V 隔离容器与 Docker 是管理 Windows Server 容器几乎相同。 若要创建具有 HYPER-V 隔离容器彻底 Docker，使用`--isolation`参数，以设置`--isolation=hyperv`。

``` cmd
docker run -it --isolation=hyperv mcr.microsoft.com/windows/nanoserver:1809 cmd
```

### <a name="isolation-explanation"></a>隔离说明

此示例演示了隔离功能在 Windows Server 和 HYPER-V 隔离之间的差异。

此处，进程隔离的容器正在部署，并将承载一个长时间运行 ping 进程。

``` cmd
docker run -d mcr.microsoft.com/windows/servercore:1809 ping localhost -t
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

为了便于对比，此示例使用 ping 进程启动 HYPER-V 隔离的容器。

```
docker run -d --isolation=hyperv mcr.microsoft.com/windows/nanoserver:1809 ping -t localhost
```

同样，`docker top` 可用于从容器中返回正在运行的进程。

```
docker top 5d5611e38b31a41879d37a94468a1e11dc1086dcd009e2640d36023aa1663e62

1732 ping
```

但是，在搜索容器主机上的进程时，找不到 ping 进程，并且会引发错误。

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
