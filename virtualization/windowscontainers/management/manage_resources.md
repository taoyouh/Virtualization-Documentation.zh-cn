



# 容器资源管理

**这是初步内容，可能还会更改。**

Windows 容器包括管理容器可以使用多少 CPU、磁盘 IO、网络和内存资源的功能。 通过限制容器资源使用，允许有效使用主机资源，并阻止过度使用。 本文档将详细介绍如何使用 PowerShell 和 Docker 管理容器资源。

## 使用 PowerShell 管理资源

### 内存

在使用 `New-Container` 命令的 `-MaximumMemoryBytes` 参数创建容器时，可以设置容器内存限制。 此示例将最大内存设置为 256mb。

```powershell
PS C:\> New-Container -Name TestContainer -MaximumMemoryBytes 256MB -ContainerimageName WindowsServerCore
```
你还可以使用 `Set-ContainerMemory` cmdlet 设置现有容器的内存限制。

```powershell
PS C:\> Set-ContainerMemory -ContainerName TestContainer -MaximumBytes 256mb
```

### 网络带宽

可以在现有容器上设置网络带宽限制。 若要执行此操作，请使用 `Get-ContainerNetworkAdapter` 命令确保容器具有网络适配器。 如果不存在网络适配器，请使用 `Add-ContainerNetworkAdapter` 命令创建一个。 最后，使用 `Set-ContainerNetworkAdapter` 命令限制容器的最大出口网络带宽。

以下示例将最大带宽设置为 100Mbps。

```powershell
PS C:\> Set-ContainerNetworkAdapter -ContainerName TestContainer -MaximumBandwidth 100000000
```

### CPU

通过设置 CPU 的最大百分比或通过设置容器的相对权重，你可以限制容器可以使用的计算量。 虽然混合这两个 CPU 管理方案不会受阻止，但不推荐这样做。 默认情况下，所有容器都可以充分利用处理器（最多为 100%）和 100 的相对权重。

下面将容器的相对权重设置为 1000。 容器的默认权重是 100，所以此容器在具有某个容器的 10 倍优先级时，可设置为默认容器。 最大值为 10000。

```powershell
PS C:\> Set-ContainerProcessor -ContainerName Container1 -RelativeWeight 10000
```

对于 CPU 时间的百分比，你也可以对容器可使用的 CPU 量设置硬性限制。 默认情况下，容器可以使用 100% 的 CPU。 下面将容器可以使用的最大 CPU 百分比设置为 30%。 使用 -Maximum 标志可将 RelativeWeight 自动设置为 100。

```powershell
PS C:\> Set-ContainerProcessor -ContainerName Container1 -Maximum 30
```

### 存储 IO

对于带宽（每秒字节数）或 8k 已标准化 IOPS，你可以限制容器可以使用多少 IO。 这两个参数可以一起设置。 达到第一个限制时会发生限流。

```powershell
PS C:\> Set-ContainerStorage -ContainerName Container1 -MaximumBandwidth 1000000
```
```powershell
PS C:\> Set-ContainerStorage -ContainerName Container1 -MaximumIOPS 32
```

## 使用 Docker 管理资源

我们提供通过 Docker 管理容器资源子集的功能。 具体而言，我们允许用户指定如何在容器之间共享 CPU。

### CPU

可以在运行时通过 --cpu-shares 标志管理容器之间的 CPU 共享。 默认情况下，所有容器均享有相等比例的 CPU 时间。 若要更改容器使用的 CPU 的相对共享，请运行值范围从 1 到 10000 的 --cpu-shares 标志。 默认情况下，所有容器接收的权重为 5000。 有关 CPU 共享约束的详细信息，请参阅 [Docker Run 参考](https://docs.docker.com/engine/reference/run/#cpu-share-constraint)。

```powershell 
C:\> docker run -it --cpu-shares 2 --name dockerdemo windowsservercore cmd
```

## 已知问题

- Hyper-V 容器当前不支持 CPU 和 IO 资源控制。
- 容器共享文件夹当前不支持 IO 资源控制。

## 视频演练

<iframe src="https://channel9.msdn.com/Blogs/containers/Container-Fundamentals--Part-4-Resource-Management/player#ccLang=zh-cn" width="800" height="450"  allowFullScreen="true" frameBorder="0" scrolling="no"></iframe>







<!--HONumber=Feb16_HO4-->


