



# 容器共享文件夹

**这是初步内容，可能还会更改。**

共享文件夹允许在容器主机和容器之间共享数据。 创建共享文件夹后，共享文件夹将在容器内部可用。 在主机的共享文件夹中放置的任何数据将在容器内部可用。 在容器内部的共享文件夹中放置的任何数据将在主机上可用。 主机上的单个文件夹可以与多个容器共享，在此配置中，数据可以在运行的容器之间共享。

## 管理数据 - PowerShell

### 创建共享文件夹

若要创建共享文件夹，请使用 `Add-ContainerSharedFolder` 命令。 下面的示例在容器中创建目录 `c:\shared_data`，该目录会映射到主机上的目录 `c:\data_source`。

> 添加共享文件夹时，容器必须处于停止状态。

```powershell
PS C:\> Add-ContainerSharedFolder -ContainerName DEMO -SourcePath c:\data_source -DestinationPath c:\shared_data

ContainerName SourcePath       DestinationPath AccessMode
------------- ----------       --------------- ----------
DEMO          c:\data_source   c:\shared_data  ReadWrite
```

### 仅读取共享文件夹

```powershell
PS C:\> Add-ContainerSharedFolder -ContainerName DEMO -SourcePath c:\sf1 -DestinationPath c:\sf2 -AccessMode ReadOnly

ContainerName SourcePath DestinationPath AccessMode
------------- ---------- --------------- ----------
DEMO         c:\sf1     c:\sf2          ReadOnly
```

### 列出共享文件夹

若要查看特定容器的共享文件夹列表，请使用 `Get-ContainerSharedFolder` 命令。

```powershell
PS C:\> Get-ContainerSharedFolder -ContainerName DEMO2

ContainerName SourcePath DestinationPath AccessMode
------------- ---------- --------------- ----------
DEMO         c:\source  c:\source       ReadWrite
```

### 修改共享文件夹

若要修改现有的共享文件夹配置，请使用 `Set-ContainerSharedFolder` 命令。

```powershell
PS C:\> Set-ContainerSharedFolder -ContainerName SFRO -SourcePath c:\sf1 -DestinationPath c:\sf1
```

### 删除共享文件夹

若要删除共享文件夹，请使用 `Remove-ContainerSharedFolder` 命令。

> 删除共享文件夹时，容器必须处于停止状态

```powershell
PS C:\> Remove-ContainerSharedFolder -ContainerName DEMO2 -SourcePath c:\source -DestinationPath c:\source
```
## 管理数据 - Docker

### 装载卷

在使用 Docker 管理 Windows 容器时，可以使用 `-v` 选项装载卷。

在下面的示例中，源文件夹是 c:\source，而目标文件夹是 c:\destination。

```powershell
PS C:\> docker run -it -v c:\source:c:\destination 1f62aaf73140 cmd
```

有关使用 Docker 管理容器中的数据的详细信息，请参阅 [Docker.com 上的 Docker 卷](https://docs.docker.com/userguide/dockervolumes/)。

## 视频演练

<iframe src="https://channel9.msdn.com/Blogs/containers/Container-Fundamentals--Part-3-Shared-Folders/player#ccLang=zh-cn" width="800" height="450"  allowFullScreen="true" frameBorder="0" scrolling="no"></iframe>





<!--HONumber=Feb16_HO3-->


