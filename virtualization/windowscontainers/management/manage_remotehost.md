---
title: "Windows Docker 主机远程管理"
description: "如何安全管理运行 Windows Server 的远程 Docker 主机。"
keywords: "docker, 容器"
author: taylorb-microsoft
ms.date: 02/14/2017
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 0cc1b621-1a92-4512-8716-956d7a8fe495
ms.openlocfilehash: 1ab2a9b823c5c903bd08b476f5caef65ec6e3207
ms.sourcegitcommit: 65de5708bec89f01ef7b7d2df2a87656b53c3145
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/21/2017
---
# Windows Docker 主机远程管理

即使没有 `docker-machine`，也仍然可以在 Windows Server 2016 VM 上创建可远程访问的 Docker 主机。

步骤非常简单：

* 使用 [dockertls](https://hub.docker.com/r/stefanscherer/dockertls-windows/) 在服务器上创建证书。 如果要使用 IP 地址创建证书，可能需要考虑使用静态 IP，以免在 IP 地址更改时需要重新创建证书。

* 重启 docker 服务 `Restart-Service Docker`
* 创建允许入站流量的 NSG 规则，使端口 docker 的 TLS 端口 2375 和 2376 可用。 请注意，要建立安全连接，你仅需允许 2376。  
  门户应该会显示 NSG 配置，如：  
  ![NGS](media/nsg.png)  
  
* 允许通过 Windows 防火墙入站连接。 
```
New-NetFirewallRule -DisplayName 'Docker SSL Inbound' -Profile @('Domain', 'Public', 'Private') -Direction Inbound -Action Allow -Protocol TCP -LocalPort 2376
```
* 将文件 `ca.pem`、“cert.pem”和“key.pem”从计算机上的“用户”下的 docker 文件夹（例如 `c:\users\chris\.docker`）复制到本地计算机。 例如，你可以从 RDP 会话复制 (ctrl-c) 粘贴 (ctrl-v) 文件。 
* 确认你可以连接到远程 Docker 主机。 运行
```
docker -D -H tcp://wsdockerhost.southcentralus.cloudapp.azure.com:2376 --tlsverify --tlscacert=c:\
users\foo\.docker\client\ca.pem --tlscert=c:\users\foo\.docker\client\cert.pem --tlskey=c:\users\foo\.doc
ker\client\key.pem ps
```


## 疑难解答
### 请尝试不使用 TLS 进行连接，以确定你的 NSG 防火墙设置是否正确
连接错误通常表明其自身出现错误，如：
```
error during connect: Get https://wsdockerhost.southcentralus.cloudapp.azure.com:2376/v1.25/version: dial tcp 13.85.27.177:2376: connectex: A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond.
```

允许不加密连接，方法是添加至 
```
{
    "tlsverify":  false,
}
```
`c"\programdata\docker\config\daemon.json`，然后重启服务。

使用命令行连接到远程主机，如：
```
docker -H tcp://wsdockerhost.southcentralus.cloudapp.azure.com:2376 --tlsverify=0 version
```

### 证书问题
如果使用非针对 IP 地址或 DNS 名称创建的证书访问 Docker 主机，将出现错误：
```
error during connect: Get https://w.x.y.c.z:2376/v1.25/containers/json: x509: certificate is valid for 127.0.0.1, a.b.c.d, not w.x.y.z
```
确保 w.x.y.z 是主机公共 IP 的 DNS 名称，且 DNS 名称匹配证书的[公用名](https://www.ssl.com/faqs/common-name/)（`SERVER_NAME` 环境变量）或提供给 dockertls 的 `IP_ADDRESSES` 变量中的其中一个 IP 地址

### 加密/x509 警告
你可能会收到一条警告 
```
level=warning msg="Unable to use system certificate pool: crypto/x509: system root pool is not available on Windows"
```
警告是良性的。
