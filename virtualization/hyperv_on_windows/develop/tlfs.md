# 虚拟机监控程序规范

## 虚拟机监控程序顶层功能规范

Hyper-V 虚拟机监控程序顶层功能规范 (TLFS) 描述了虚拟机监控程序对其他操作系统组件的外部可见的行为。 此规范对来宾操作系统开发人员很有用。

> 此规范根据 Microsoft 开放规范承诺书而提供。 阅读以下内容，进一步了解有关 [Microsoft 开放规范承诺书](https://msdn.microsoft.com/en-us/openspecifications)的详细信息。

#### 下载

 版本| 文档
--- | ---
 Windows Server 2012 R2（修订版 B）| [Hypervisor Top Level Functional Specification v4.0b.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v4.0b.pdf)
 Windows Server 2012 R2| [Hypervisor Top Level Functional Specification v4.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v4.0.pdf)
 Windows Server 2012| [Hypervisor Top Level Functional Specification v3.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v3.0.pdf)
 Windows Server 2008 R2| [Hypervisor Top Level Functional Specification v2.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v2.0.pdf)

## 实现 Microsoft 虚拟机管理程序接口的要求

Windows 操作系统需要一组受限制的虚拟机管理程序接口在来宾虚拟机（也称为“HV #1”接口）中运行。 此外，可以通过 Microsoft 兼容的虚拟机监控程序来实现一些可选功能。 这些选项将更改虚拟机中的 Windows 行为。 “实现 Microsoft 虚拟机管理程序接口的要求”描述了由 Microsoft 兼容的虚拟机监控程序实现的必需和可选功能。

#### 下载

[Requirements for Implementing the Microsoft Hypervisor Interface.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Requirements%20for%20Implementing%20the%20Microsoft%20Hypervisor%20Interface.pdf)





<!--HONumber=Mar16_HO2-->


