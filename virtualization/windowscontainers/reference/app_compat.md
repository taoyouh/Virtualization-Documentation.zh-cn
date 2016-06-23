---
title: Windows 容器中的应用程序兼容性
description: Windows 容器中的应用程序兼容性。
keywords: docker, containers
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 3e524458-bd03-400e-913f-210335add8dc
---

# Windows 容器中的应用程序兼容性

**这是初步内容，可能还会更改。** 

这是预览版。  尽管在 Windows 上运行的应用程序最终也应在容器中运行，但这是查看我们的当前应用程序兼容性状态的好时机。

本文档的唯一目的是分享我们的经验。

此列表是否缺少某些内容？  通过[论坛](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)告诉我们你环境中的失败和成功之处。

## Windows Server 容器

我们已尝试在 Windows Server 容器中运行以下应用程序。  这些结果不会保证应用程序正常运行。

| **Name** | **版本** | **Windows Server Core 基础映像** | **Nano Server 基础映像** | **备注** |
|:-----|:-----|:-----|:-----|:-----|
| .NET | 3.5 | “是” | Unknown |  | 
| .NET | 4.6 | “是” | Unknown |  | 
| .NET CLR | 5 beta 6 | “是” | “是”| X64 和 x86 两者 | 
| Active Python | 3.4.1 | “是” | “是” | |
| Apache Cassandra || “是” | 未知 | 
| Apache CouchDB | 1.6.1 | 否 | 否 | |
| Apache Hadoop | | “是” | 否 | |
| Apache HTTPD | 2.4 | “是” | “是” | 如果已加载重复数据删除筛选器，则不会安装 VC ++ 运行时。 卸载重复数据删除，使用 `fltmc unload dedup` |
| Apache Tomcat | 8.0.24 x64 | “是” | Unknown | |
| ASP.NET | 4.6 | “是” | 未知 | |
| ASP.NET | 5 beta 6 | “是” | “是” | X64 和 x86 两者 |
| Django | |“是”|“是”| |
| Go | 1.4.2 | “是” | “是” | |
| Internet 信息服务 | 10.0 | “是” | 是 | HTTPS/TLS 无法工作。  如果已加载重复数据删除筛选器，则不会安装 VC ++ 运行时。 卸载重复数据删除，使用 `fltmc unload dedup` |
| Java | 1.8.0_51 | “是” | “是” | 使用服务器版本。 客户端版本未正确安装 |
| MongoDB | 3.0.4 | “是” | 未知 | |
| MySQL | 5.6.26 | “是” | “是” | |
| NGinx | 1.9.3 | “是” | “是” | |
| Node.js | 0.12.6 | 部分 | 部分 | NPM 无法下载程序包。 |
| Perl | | “是” | 未知 | |
| PHP | 5.6.11 | “是” | “是” | 如果已加载重复数据删除筛选器，则不会安装 VC ++ 运行时。 卸载重复数据删除，使用 `fltmc unload dedup` |
| PostgreSQL | 9.4.4 | “是” | Unknown | 如果已加载重复数据删除筛选器，则不会安装 VC ++ 运行时。 卸载重复数据删除，使用 `fltmc unload dedup` |
| Python | 3.4.3 | “是” | “是” | |
| R | 3.2.1 | 否 | 否 | |
| RabbitMQ | 3.5.x | “是” | Unknown | |
| Redis | 2.8.2101 | “是” | “是” | |
| Ruby | 2.2.2 | “是” | “是” | X64 和 x86 两者 | 
| Ruby on Rails | 4.2.3 | “是” | “是” | |
| SQLite | 3.8.11.1 | “是” | 否 | |
| SQL Server Express | 2014 | “是” | 未知 | 你可以通过生成此[社区提供的 Dockerfile](https://github.com/brogersyh/Dockerfiles-for-windows/tree/master/sqlexpress)（用于安装 SQL Express 2014）快速启动。 |
| Sysinternals 工具 | * | 是 | “是” | 仅尝试了不需要 GUI 的工具。 PsExec 在当前设计下不起作用 | 

## Hyper-V 容器

我们已尝试在 Hyper-V 容器中运行以下应用程序。  这些结果不会保证应用程序正常工作。

| **Name** | **版本** | **Nano Server 基础映像** | **备注** |
|:-----|:-----|:-----|:-----|
| Apache Hadoop | | 否 | |
| Apache HTTPD | 2.4 | “是” | 如果已加载重复数据删除筛选器，则不会安装 VC ++ 运行时。 卸载重复数据删除，使用 `fltmc unload dedup` |
| ASP.NET | 5 beta 6 | “是” | X64 和 x86 两者 |
| Django |  | “是” | 如果映像使用 DockerFile 创建，并且复制 Python 二进制文件作为它的一部分，则 Python 不起作用。 启动容器，然后复制 Python 二进制文件。 |
| Go | 1.4.2 | “是” | |
| Internet 信息服务 | 10.0 | 是 | HTTPS/TLS 无法工作。  IIS 无法直接使用 dism 安装。  使用 dism 命令执行 IIS 的无人参与安装。 |
| Java | 1.8.0_51 | “是” | 使用服务器版本。 客户端版本未正确安装 |
| MySQL | 5.6.26 | “是” | |
| NGinx | 1.9.3 | “是” | |
| Node.js | 0.12.6 | 部分 | NPM 无法下载程序包。 |
| Python | 3.4.3 | “是” |  如果映像使用 DockerFile 创建，并且复制 Python 二进制文件作为它的一部分，则 Python 不起作用。 启动容器，然后复制 Python 二进制文件。 |
| Redis | 2.8.2101 | “是” | |
| Ruby | 2.2.2 | “是” | X64 和 x86 两者 | 
| Ruby on Rails | 4.2.3 | “是” | |
| Sysinternals 工具 | | “是” | 仅尝试了不需要 GUI 的工具。 PsExec 在当前设计下不起作用。 |

## 告诉我们你的体验
此列表是否缺少某些内容？  通过[论坛](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)告诉我们你环境中的失败和成功之处。


<!--HONumber=May16_HO3-->


