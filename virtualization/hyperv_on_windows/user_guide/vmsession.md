# 使用 PowerShell Direct 管理 Windows

可以从 Windows 10 或 Windows Server Technical Preview Hyper-V 主机中使用 PowerShell Direct 远程管理 Windows 10 或 Windows Server Technical Preview 虚拟机。 不管 Hyper-V 主机或虚拟机上的网络配置或远程管理设置如何，PowerShell Direct 都允许在虚拟机中管理 PowerShell。 这使得 Hyper-V 管理员能够更简单地自动化虚拟机管理和配置，并为其编写脚本。

可通过两种方法运行 PowerShell Direct：
* 作为交互式会话 -- [转到此部分](vmsession.md#create-and-exit-an-interactive-powershell-session)以使用 PSSession cmdlet 创建和退出 PowerShell Direct 会话
* 若要执行一组命令或脚本 -- [转到此部分](vmsession.md#run-a-script-or-command-with-invoke-command)以通过 Invoke-Command cmdlet 运行脚本或命令


## 要求

**操作系统要求：**
* 主机操作系统必须运行 Windows 10、Windows Server Technical Preview 或更高版本。
* 虚拟机必须运行 Windows 10、Windows Server Technical Preview 或更高版本。

如果要管理较旧的虚拟机，请使用虚拟机连接 (VMConnect) 或[为虚拟机配置虚拟网络](http://technet.microsoft.com/library/cc816585.aspx)。

若要在虚拟机上创建 PowerShell Direct 会话，
* 虚拟机必须在主机上本地运行并已启动。
* 必须以 Hyper-V 管理员身份登录主机计算机。
* 必须为虚拟机提供有效用户凭据。

## 创建并退出交互式 PowerShell 会话

1. 在 Hyper-V 主机上，以管理员身份打开 Windows PowerShell。

3. 通过使用虚拟机名称或 GUID 运行以下命令之一来创建会话：
``` PowerShell
Enter-PSSession -VMName <VMName>
Enter-PSSession -VMGUID <VMGUID>
```

4. 运行所需的任意命令。 这些命令在与其创建了会话的虚拟机上运行。
5. 完成后，运行以下命令来关闭会话：
``` PowerShell
Exit-PSSession 
```

> 注意：如果会话无法连接，请确保将凭据用于正在连接的虚拟机，而非 Hyper-V 主机。

若要了解有关这些 cmdlet 的详细信息，请参阅 [Enter-PSSession](http://technet.microsoft.com/library/hh849707.aspx) 和 [Exit-PSSession](http://technet.microsoft.com/library/hh849743.aspx)。

## 使用 Invoke-Command 运行脚本或命令

你可以使用 [Invoke-Command](http://technet.microsoft.com/library/hh849719.aspx) cmdlet 在虚拟机上运行一组预先确定的命令。 下面是如何使用 Invoke-Command cmdlet 的示例，其中 PSTest 是虚拟机名称，而要运行的脚本 (foo.ps1) 位于 C:/ 驱动器上的脚本文件夹中：

 ``` PowerShell
 Invoke-Command -VMName PSTest -FilePath C:\script\foo.ps1 
 ```

若要运行单个命令，请使用 **-ScriptBlock** 参数：

 ``` PowerShell
 Invoke-Command -VMName PSTest -ScriptBlock { cmdlet } 
 ```

## 疑难解答

PowerShell Direct 显示了一小部分的常见错误消息。 以下是最常见的错误消息、一些原因和诊断问题的工具。

### 错误：远程会话可能已结束

**错误消息：**
```
Enter-PSSession : An error has occurred which Windows PowerShell cannot handle. A remote session might have ended.
```

**可能的原因：**
* 虚拟机未运行
* 来宾操作系统不支持 PowerShell Direct（请参阅[要求](#Requirements))）
* PowerShell 尚不可用于来宾
  * 操作系统没有完成启动
  * 操作系统无法正常启动
  * 某些启动时事件需要用户输入
* 无法验证来宾凭据
  * 提供的凭据不正确
  * 来宾操作系统中没有任何用户帐户（操作系统以前未启动）
  * 如果以管理员身份进行连接：管理员还未设置为活动用户。 在[此处](https://technet.microsoft.com/en-us/library/hh825104.aspx)了解详细信息。

你可以使用 [Get-VM](http://technet.microsoft.com/library/hh848479.aspx) cmdlet 检查使用中的凭据是否具有 Hyper-V 管理员角色并查看哪些 VM 在主机上本地运行并已启动。

### 错误：无法解析参数集

**错误消息：**
``` 
Enter-PSSession : Parameter set cannot be resolved using the specified named parameters.
```

**可能的原因：**  
在连接到虚拟机时，不支持 `-RunAsAdministrator`。

PowerShell Direct 在连接到虚拟机与 Windows 容器时具有不同的行为。 连接到 Windows 容器时，`-RunAsAdministrator` 标志将允许管理员连接，而无需显式凭据。 由于虚拟机未授予主机默示的管理员访问权限，因此你需要显式输入凭据。

使用 `-credential` 参数或通过在系统提示时手动输入，可将管理员凭据传递给虚拟机。


## 示例

查看 [GitHub](https://github.com/Microsoft/Virtualization-Documentation/search?l=powershell&q=-VMName+OR+-VMGuid&type=Code&utf8=%E2%9C%93) 上的示例。

有关如何在你的环境中使用 PowerShell Direct 的大量示例以及使用 PowerShell 编写 Hyper-V 脚本的提示和技巧，请参阅 [PowerShell Direct 代码段](../develop/powershell_snippets.md)。




<!--HONumber=Jan16_HO2-->
