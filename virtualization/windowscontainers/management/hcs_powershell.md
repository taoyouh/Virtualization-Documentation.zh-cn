# 管理互操作性

**这是初步内容，可能还会更改。**

大多数情况下，使用 PowerShell 创建的 Windows 容器需要使用 PowerShell 进行管理，而使用 Docker 创建的容器需要使用 Docker 进行管理。 即便如此，主机计算 PowerShell 模块仍可发现并停止**正在运行**的容器，而无需考虑它们的创建方式。 此模块的执行方式类似于在容器主机上运行的容器的“任务管理器”。

## 显示所有容器

若要返回容器列表，请使用 `Get-ComputeProcess` 命令。

```powershell
PS C:\> Get-ComputeProcess

Id                                                Name                                      Owner       Type
--                                                ----                                      -----       ----
2088E0FA-1F7C-44DE-A4BC-1E29445D082B              DEMO1                                     VMMS   Container
373959AC-1BFA-46E3-A472-D330F5B0446C              DEMO2                                     VMMS   Container
d273c80b6e..                                      d273c80b6e..                              docker Container
e49cd35542..                                      e49cd35542..                              docker Container
```

## 停止容器

若要停止容器（无需考虑是使用 PowerShell 还是 Docker 创建的容器），请使用 `Stop-ComputeProcess` 命令。

> 撰写本文时，需要重新启动 VMMS 服务，以便在使用 `Get-Container` 命令时，容器显示为停止状态。

```powershell
PS C:\> Stop-ComputeProcess -Id 2088E0FA-1F7C-44DE-A4BC-1E29445D082B -Force
```




