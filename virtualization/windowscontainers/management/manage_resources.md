---
title: "容器资源管理"
description: "使用 Windows 容器管理容器资源。"
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b2192e64-9d74-474e-8af0-2d8b3ad1deee
translationtype: Human Translation
ms.sourcegitcommit: cfa3c14e932f8b86edf6667200ac028ea0a16b67
ms.openlocfilehash: 82cc37e4bcf001e938dcff7308be16978fa955e2

---

# 容器资源管理

**这是初步内容，可能还会更改。** 

Windows 容器包括管理容器可以使用多少 CPU、磁盘 IO、网络和内存资源的功能。 通过限制容器资源使用，允许有效使用主机资源，并阻止过度使用。 本文档将详细介绍如何使用 Docker 管理容器资源。

## 使用 Docker 管理资源 

我们提供通过 Docker 管理容器资源子集的功能。 具体而言，我们允许用户指定如何在容器之间共享 CPU。 

### CPU

可以在运行时通过 --cpu-shares 标志管理容器之间的 CPU 共享。 默认情况下，所有容器均享有相等比例的 CPU 时间。 若要更改容器使用的 CPU 的相对共享，请运行值范围从 1 到 10000 的 --cpu-shares 标志。 默认情况下，所有容器接收的权重为 5000。 有关 CPU 共享约束的详细信息，请参阅 [Docker Run 参考]( https://docs.docker.com/engine/reference/run/#cpu-share-constraint)。 

```none 
docker run -it --cpu-shares 2 --name dockerdemo windowsservercore cmd
```

## 已知问题

- Hyper-V 容器当前不支持 CPU 和 IO 资源控制。
- 容器数据卷当前不支持 IO 资源控制。


<!--HONumber=Jun16_HO4-->


