# <a name="known-issues-for-insider-builds"></a>预览体验计划内部版本的已知问题

## <a name="build-16237"></a>版本 16237

- Hyper-v 隔离工作不正常。 若要在内部版本16237中使用 Hyper-v 隔离, 则需要此解决方法。 在 PowerShell 中运行以下命令：

```PowerShell
Get-ComputeProcess | ? IsTemplate -eq $true | Stop-ComputeProcess -Force
Set-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers\' -Name TemplateVmCount -Type dword -Value 0 -Force
```

- Nano Server 现在作为用户运行, 因此需要管理员权限的命令将失败。 包括“RUN setx /M PATH”等行会导致版本失败。 对于这种情况，你可以使用此替代方法：

```dockerfile
RUN setx PATH <path>
```
