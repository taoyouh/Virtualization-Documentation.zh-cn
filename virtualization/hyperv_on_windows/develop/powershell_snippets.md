# PowerShell 代码段

PowerShell 是适用于 Hyper-V 的出色脚本编写、自动化和管理工具。以下是浏览它可以执行的一些出色操作的工具箱。

所有的 Hyper-V 管理均要求以管理员身份运行，所以假设所有脚本和代码段必须从 Hyper-V Administrator 帐户以管理员身份运行。

如果不确定是否拥有正确权限，请键入 `Get-VM`；如果运行无误，即表示你已准备就绪。


## PowerShell Direct 工具

本部分中的所有脚本和代码段依赖于以下基础知识：

**要求**：
*  PowerShell Direct。 Windows 10 来宾和主机操作系统。

**常见变量**：  
`$VMName` -- 这是带有 VMName 的字符串。 通过 `Get-VM` 查看可用 VM 列表  
`$cred` -- 来宾操作系统的凭据。 可以使用 `$cred = Get-Credential` 填充

### 检查是否已启动来宾

Hyper-V 管理器不会让你看到来宾操作系统的内部情况，因而通常很难知道来宾操作系统是否已启动。

以下是同一功能的两个视图，第一个作为代码段，第二个作为 PowerShell 函数。

代码段：
``` PowerShell
if((Invoke-Command -VMName $VMName -Credential $cred {"Test"}) -ne "Test"){Write-Host "Not Booted"} else {Write-Host "Booted"}
```

Function:
``` PowerShell
function waitForPSDirect([string]$VMName, $cred){
   Write-Output "[$($VMName)]:: Waiting for PowerShell Direct (using $($cred.username))"
   while ((Invoke-Command -VMName $VMName -Credential $cred {"Test"} -ea SilentlyContinue) -ne "Test") {Sleep -Seconds 1}}
```

**结果**  
打印友好消息，并在成功连接 VM 前一直锁定。  
成功后不会发出提示。

### 在来宾联网前脚本一直处于锁定状态

通过 PowerShell Direct，可以在虚拟机接收某个 IP 地址前连接虚拟机内的 PowerShell 会话。

``` PowerShell
# Wait for DHCP
while ((Get-NetIPAddress | ? AddressFamily -eq IPv4 | ? IPAddress -ne 127.0.0.1).SuffixOrigin -ne "Dhcp") {sleep -Milliseconds 10}
```

** 结果 **
在接收 DHCP 租约前一直锁定。 因为此脚本不查找特定子网或 IP 地址，所以不论使用何种网络配置，它都可以运行。  
成功后不会发出提示。

## 使用 PowerShell 管理凭据

Hyper-V 脚本经常要求处理一台或多台虚拟机、Hyper-V 主机或两者的凭据。

在使用 PowerShell Direct 或标准 PowerShell 远程处理时，有很多方法可以做到这一点：

1. 第一种（且最简单的）方法是使相同的用户凭据在主机和来宾或本地和远程主机上有效。  
    如果你要使用 Microsoft 帐户登录或如果你位于域环境中，此操作会相当简单。  
    在此种情况下，你只需运行 `Invoke-Command -VMName "test" {get-process}` 即可。

2. 让 PowerShell 提示你输入凭据  
    如果凭据不匹配，你会自动收到允许你提供虚拟机的相应凭据的凭据提示。

3. 以变量形式存储凭据以供重复使用。
    运行如下所示的简单命令：
  ``` PowerShell
  $localCred = Get-Credential
  ```
  然后运行如下所示的内容：
  ``` PowerShell
  Invoke-Command -VMName "test" -Credential $localCred  {get-process} 
  ```
  这意味着针对每个脚本/PowerShell 会话，你都会收到一次输入凭据提示。

4. 将凭据以代码形式编写到脚本中。 **不要对任何实际工作负荷或系统执行此操作**
    > 警告：_不要在生产系统中执行此操作。 不要使用实际密码执行此操作。_

    可以使用如下所示的一些代码创建 PSCredential 对象：
  ``` PowerShell
  $localCred = New-Object -typename System.Management.Automation.PSCredential -argumentlist "Administrator", (ConvertTo-SecureString "P@ssw0rd" -AsPlainText -Force) 
  ```
  虽然很不安全，但对于测试却很有用。 现在，你在此会话中不会收到任何提示。





