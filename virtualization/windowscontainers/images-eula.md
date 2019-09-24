
# <a name="microsoft-software-supplemental-license"></a>MICROSOFT 软件补充许可证
# <a name="for-windows-container-base-image"></a>对于 WINDOWS 容器基本映像 

此附加许可证适用于 Windows 容器基本映像（"容器映像"）。  如果您遵守本补充许可证的条款，您可以使用如下所述的容器图像。

容器图像只能与的有效许可副本配合使用：
* Windows Server Standard 或 Windows Server Datacenter 软件（统称为 "服务器主机软件"）或
* Microsoft Windows 操作系统（版本10）软件（"客户端主机软件"）或
* Windows 10 IoT 企业版和 Windows 10 IoT 核心版（统称为 "IoT 主机软件"）。

服务器主机软件、客户端主机软件和 IoT 主机软件统称为 "主机软件"，而主机软件许可证是 "主机许可证"。

如果您没有相应版本的主机许可证，则不能使用容器图像。  可能会应用某些限制和附加条款，此处对此进行了介绍。  如果此处的许可条款与主机许可证冲突，则此附加许可证将针对容器映像进行管理。 接受此附加许可证或使用容器图像即表示你同意所有这些条款。  如果您不接受并遵守这些条款，则不能使用容器图像。  

## <a name="definitions"></a>码 

**Windows Server 容器**（没有 Hyper-v 隔离）是 Microsoft Windows Server 软件的一项功能。 

**具有 Hyper-v 隔离的 Windows Server 容器。**  Microsoft Windows Server （版本10）许可证条款的第2部分（即 k）已被特此删除并替换为下面 "更新" 中所示的修订条款。  

已更新：具有 Hyper-v 隔离的 Windows Server 容器（以前称为 Hyper-v 容器）是 Windows Server 中的一种容器技术，它利用虚拟操作系统环境托管一个或多个 Windows 服务器容器。  用于托管一个或多个 Windows Server 容器的每个 Hyper-v 隔离实例均视为一个虚拟操作系统环境。  

## <a name="license-terms"></a>许可条款

**主机许可证。** 主机许可条款适用于你对容器映像的使用，以及使用容器映像创建的、与虚拟机不同的所有 Windows 容器。

**使用权利。**  容器映像可用于创建独立的虚拟化 Windows 操作系统环境，该环境至少包含一个添加主要功能和重要功能的应用程序。 你可以仅使用容器映像在主机软件上创建、构建和运行 Windows 容器。  对主机软件的更新可能不会更新容器映像，因此你可以基于更新的容器映像重新创建任何 Windows 容器。   
 
**条款.**  您不得从容器图像中删除此附加许可证文档文件。  您不能对您在容器内运行的应用程序启用远程访问，以避免许可证收费。  您不得对容器映像进行反向工程、反编译或反汇编，也不能尝试执行此操作，但仅限于第三方许可条款所需的范围，以控制软件中可能包含的某些开放源代码组件的使用。  可能适用于主机许可证的其他限制。

## <a name="additional-terms"></a>附加条款

**客户端主机软件。**  在客户端主机软件上运行容器图像时，你可能会运行实例化为 Windows 容器的任意数量的容器映像，仅用于测试或开发目的。  在客户端主机软件的生产环境中，不能使用这些 Windows 容器。

**IoT 主机软件。**  在 IoT 主机软件上运行容器图像时，你可以运行实例化为 Windows 容器的任意数量的容器映像，仅用于测试或开发目的。 如果已同意适用于 Windows 10 Core 运行时映像的 Microsoft 商业术语或 Windows 10 IoT Enterprise Device License （"Windows IoT 商业版协议"），则只能使用生产环境中的容器图像。 Windows IoT 商业版协议中的其他条款和限制适用于您在生产环境中使用容器映像的情况。

**第三方软件。** 容器映像可能包括在此附加许可证下授权的第三方应用程序，或者在其自己的条款下提供。 第三方应用程序的许可条款、声明和确认（如果有）可在http://aka.ms/thirdpartynotices随附的声明文件中联机或在随附的声明文件中联机访问。 即使此类应用受其他协议的制约，对主机许可证中的损害的赔偿和排除的限制也适用于适用法律允许的范围。  
  
**开放源组件。** 容器图像可能包含具有源代码可用性义务的开放源代码许可证下许可的第三方软件。 这些许可证的副本包含在 ThirdPartyNotices 文件或其他随附的声明文件中。 你可以通过发送 money 订单或检查 $5.00 来从 Microsoft 获取完整的相应源代码，前提是通过发送 money 订单或检查以：源代码合规性团队、Microsoft Corporation、1 Microsoft 方法、Redmond、WA 98052、美国的要求。 请在你的付款的备注行中包含名称 "Microsoft 软件的 Windows 容器基本版许可证的附加许可证" （开放源组件名称和版本号）。 您还可以在上http://aka.ms/getsource找到源的副本。
