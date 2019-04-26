---
title: Windows 容器基本映像历史记录
description: 与 SHA256 层哈希一起发布的 Windows 容器映像列表
keywords: docker, 容器, 哈希
author: patricklang
ms.date: 01/12/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 88e6e080-cf8f-41d8-a301-035959dc5ce0
ms.openlocfilehash: eb2df861021f889e335d49a337b379ecbf17a47d
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/26/2019
ms.locfileid: "9576918"
---
# <a name="windows-container-base-image-history"></a>Windows 容器基本映像历史记录

每个 Windows 容器均是在 Microsoft 所提供的基本操作系统的基础上生成。 如果你不确定是为哪个版本的 Windows 生成容器，你可以运行 `docker inspect <tag>`，将前 1 行或前两行与下表进行匹配。

例如，将显示 `docker inspect microsoft/windowsservercore:10.0.14393.447`

```
...
"RootFS": {
    "Type": "layers",
    "Layers": [
        "sha256:3fd27ecef6a323f5ea7f3fde1f7b87a2dbfb1afa797f88fd7d20e8dbdc856f67",
        "sha256:b9454c3094c68005f07ae8424021ff0e7906dac77a172a874cd5ec372528fc15"
    ]
}
```

Microsoft 提供的映像中有哪两个层。 第一层是恒定的，表示初始的 Windows Server 版本，第二层则基于所包含的最新累积更新而变化。

如果想要找出每个版本的变化，请在 [Windows 10 和 Windows Server 2016 更新历史记录](https://support.microsoft.com/en-us/help/12387/windows-10-update-history)中查找相应版本的知识库


## <a name="tools-to-simplify-this-process"></a>可简化此过程的工具

Stefan Scherer 已经创建了一个工具，该工具可以读取映像清单文件并且无需下载整个容器即可确定版本。 请查看他的[博客](https://stefanscherer.github.io/winspector/)和 [GitHub](https://github.com/StefanScherer/winspector) 存储库了解详细信息。


## <a name="image-versions"></a>映像版本

<table>
    <tr>
        <th>Windows 版本</th>
        <th>microsoft/windowsservercore</th>
        <th>microsoft/nanoserver</th>
    </tr>
    <tr>
        <td>10.0.14393.206</td>
        <td>sha256:3fd27ecef6a323f5ea7f3fde1f7b87a2dbfb1afa797f88fd7d20e8dbdc856f67</td>
        <td>sha256:342d4e407550c52261edd20cd901b5ce438f0b1e940336de3978210612365063</td>
    </tr>
    <tr>
        <td>10.0.14393.321</td>
        <td>sha256:3fd27ecef6a323f5ea7f3fde1f7b87a2dbfb1afa797f88fd7d20e8dbdc856f67<br/>
        sha256:cc6b0a07c696c3679af48ab4968de1b42d35e568f3d1d72df21f0acb52592e0b</td>
        <td>sha256:342d4e407550c52261edd20cd901b5ce438f0b1e940336de3978210612365063<br/>
        sha256:2c195a33d84d936c7b8542a8d9890a2a550e7558e6ac73131b130e5730b9a3a5</td>
    </tr>
    <tr>
        <td>10.0.14393.447</td>
        <td>sha256:3fd27ecef6a323f5ea7f3fde1f7b87a2dbfb1afa797f88fd7d20e8dbdc856f67<br/>
        sha256:b9454c3094c68005f07ae8424021ff0e7906dac77a172a874cd5ec372528fc15</td>
        <td>sha256:342d4e407550c52261edd20cd901b5ce438f0b1e940336de3978210612365063<br/>
        sha256:c8606bedb07a714a6724b8f88ce85b71eaf5a1c80b4c226e069aa3ccbbe69154</td>
    </tr>
    <tr>
        <td>10.0.14393.576</td>
        <td>sha256:f358be10862ccbc329638b9e10b3d497dd7cd28b0e8c7931b4a545c88d7f7cd6<br/>
        sha256:de57d9086f9a337bb084b78f5f37be4c8f1796f56a1cd3ec8d8d1c9c77eb693c</td>
        <td>sha256:6c357baed9f5177e8c8fd1fa35b39266f329535ec8801385134790eb08d8787d<br/>
        sha256:0d812bf7a7032db75770c3d5b92c0ac9390ca4a9efa0d90ba2f55ccb16515381</td>
    </tr>
    <tr>
        <td>10.0.14393.693</td>
        <td>sha256:f358be10862ccbc329638b9e10b3d497dd7cd28b0e8c7931b4a545c88d7f7cd6<br/>
        sha256:c28d44287ce521eac86e0296c7677f5d8ca1e86d1e45e7618ec900da08c95df3</td>
        <td>sha256:6c357baed9f5177e8c8fd1fa35b39266f329535ec8801385134790eb08d8787d<br/>
        sha256:dd33c5d8d8b3c230886132c328a7801547f13de1dac9a629e2739164a285b3ab</td>
    </tr>
</table>

