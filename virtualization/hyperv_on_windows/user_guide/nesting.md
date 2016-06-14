---
title: 嵌套虚拟化
description: 嵌套虚拟化
keywords: windows 10, hyper-v
author: theodthompson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 68c65445-ce13-40c9-b516-57ded76c1b15
---

# 嵌套虚拟化

嵌套虚拟化提供在虚拟化环境中运行 Hyper-V 主机的能力。 换而言之，借助嵌套虚拟化，Hyper-V 主机本身可进行虚拟化。 嵌套虚拟化的一些用例为：在虚拟化环境中运行 Hyper-V 实验室，向其他用户提供虚拟化服务而无需单个硬件，并且当在虚拟化的容器主机上运行 Hyper-V 容器时，Windows 容器技术依赖于嵌套虚拟化。 本文档将详细介绍软件和硬件先决条件、配置步骤，并提供故障排除的详细信息。

> 嵌套虚拟化目前以预览版提供，且不应用于生产。

## 先决条件

- 运行版本 10565 或更高版本的 Windows 预览体验成员版本（Windows Server 2016、Nano Server 或 Windows 10）。
- 这两个虚拟机监控程序（父级与子级）都必须运行相同的 Windows 版本（10565 或更高版本）。
- 最少 4 GB 的可用 RAM。
- 具有 Intel VT-x 技术的 Intel 处理器。

## 配置嵌套虚拟化

首先，创建与主机运行相同版本的虚拟机，**不要打开虚拟机**。 有关详细信息，请参阅[创建虚拟机](../quick_start/walkthrough_create_vm.md)。

创建虚拟机后，在父虚拟机监控程序上运行以下命令，这一操作将在虚拟机上启用嵌套虚拟化。

```none
Set-VMProcessor -VMName <virtual machine> -ExposeVirtualizationExtensions $true -Count 2
```

运行嵌套 Hyper-V 主机时，必须在虚拟机上禁用动态内存。 这可以通过虚拟机的属性，或者通过使用以下 PowerShell 命令进行配置。

```none
Set-VMMemory <virtual machine> -DynamicMemoryEnabled $false
```

为了使嵌套虚拟机接收 IP 地址，必须启用 MAC 地址欺骗。 使用以下 PowerShell 命令完成此操作。

```none
Get-VMNetworkAdapter -VMName <virtual machine> | Set-VMNetworkAdapter -MacAddressSpoofing On
```

完成这些步骤后，可以启动虚拟机并安装 Hyper-V。 有关安装 Hyper-V 的详细信息，请参阅[安装 Hyper-V]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_install)。

## 配置脚本

或者，以下脚本可用于启用并配置嵌套虚拟化。 以下命令将下载并运行脚本。
  
```none
# download script
Invoke-WebRequest https://raw.githubusercontent.com/Microsoft/Virtualization-Documentation/master/hyperv-tools/Nested/Enable-NestedVm.ps1 -OutFile .\Enable-NestedVm.ps1 

# run script
.\Enable-NestedVm.ps1 -VmName "DemoVM"
```

## 已知问题

- 启用了 Device Guard 的主机无法向来宾公开虚拟化扩展。
- 启用了基于虚拟化的安全 (VBS) 的主机无法向来宾公开虚拟化扩展。 若要使用嵌套虚拟化，必须先禁用 VBS。
- 如果使用的密码为空，虚拟机连接可能会断开。 更改密码，该问题就应该会得到解决。
- 在虚拟机中启用嵌套虚拟化后，以下功能将不再与该 VM 兼容。  
  * 调整运行时内存大小。
  * 将检查点应用于正在运行的虚拟机。
  * 不能实时迁移正托管其他虚拟机的虚拟机。
  * 保存和还原不起作用。

## 常见问题和疑难解答

我的虚拟机无法启动，我应该怎么做？

1. 请确保动态内存已关闭。
2. 在主机上从提升的提示符中运行[此 PowerShell 脚本](https://raw.githubusercontent.com/Microsoft/Virtualization-Documentation/master/hyperv-tools/Nested/Get-NestedVirtStatus.ps1)。
  
## 反馈

通过 Windows 反馈应用、[虚拟化论坛](https://social.technet.microsoft.com/Forums/windowsserver/En-us/home?forum=winserverhyperv)，或通过 [GitHub](https://github.com/Microsoft/Virtualization-Documentation) 报告其他问题。



<!--HONumber=Jun16_HO2-->


