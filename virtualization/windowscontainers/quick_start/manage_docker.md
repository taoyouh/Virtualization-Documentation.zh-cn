# Windows 容器快速入门 - Docker

Windows 容器可用于在单个计算机系统上快速部署多个独立应用程序。 本练习将使用 Docker 演示 Windows 容器的创建与管理。 完成本练习后，你应基本了解 Docker 如何与 Windows 容器集成，并且将获得实践经验和技术。

本演练将详细介绍 Windows Server 容器和 Hyper-V 容器。 每种类型的容器都有其自己的基本要求。 Windows 容器文档中包含了快速部署容器主机的过程。 这是 Windows 容器快速入门的最简单方法。 如果你还没有容器主机，请参阅[容器主机部署快速入门](./container_setup.md)。

每个练习都将需要以下项目。

**Windows Server 容器：**

- 在本地或 Azure 中运行 Windows Server 2016（完整版或核心版）的 Windows 容器主机。

**Hyper-V 容器：**

- 通过嵌套虚拟化启用的 Windows 容器主机。
- Windows Server 2016 媒体 - [下载](https://aka.ms/tp4/serveriso)。

> Microsoft Azure 不支持 Hyper-V 容器。 若要完成 Hyper-V 容器练习，你需要本地容器主机。

## Windows Server 容器

Windows Server 容器提供一个独立的可移植并且受资源控制的操作环境，用于运行应用程序和托管进程。 Windows Server 容器通过进程和命名空间的隔离，提供了容器和主机之间的隔离。

### 创建容器

在创建容器之前，请使用 `docker images` 命令列出安装在主机上的容器映像。

```powershell
PS C:\> docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
windowsservercore   10.0.10586.0        6801d964fda5        2 weeks ago         0 B
windowsservercore   latest              6801d964fda5        2 weeks ago         0 B
nanoserver          10.0.10586.0        8572198a60f1        2 weeks ago         0 B
nanoserver          latest              8572198a60f1        2 weeks ago         0 B
```

在本示例中，使用 Windows Server Core 映像创建容器。 使用 `docker run` 命令完成此操作。 有关 `docker run` 的详细信息，请参阅 [docker.com 上的 Docker Run 参考](https://docs.docker.com/engine/reference/run/)。

本示例将创建一个名为 `iisbase` 的容器，并启动与该容器的交互式会话。

```powershell
C:\> docker run --name iisbase -it windowsservercore cmd
```

创建容器后，你将从该容器内部进入 shell 会话。


### 创建 IIS 映像

IIS 将安装，然后将从该容器中创建映像。 若要安装 IIS，请运行以下内容。

```powershell
C:\> powershell.exe Install-WindowsFeature web-server
```

完成后，退出交互式 shell 会话。

```powershell
C:\> exit
```

最后，使用 `docker commit` 将该容器提交到新的容器映像。 本示例将创建一个名为 `windowsservercoreiis` 的新容器映像。

```powershell
C:\> docker commit iisbase windowsservercoreiis
4193c9f34e320c4e2c52ec52550df225b2243927ed21f014fbfff3f29474b090
```

可以使用 `docker images` 命令查看新 IIS 映像。

```powershell
C:\> docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
windowsservercoreiis   latest              4193c9f34e32        4 minutes ago       170.8 MB
windowsservercore      10.0.10586.0        6801d964fda5        2 weeks ago         0 B
windowsservercore      latest              6801d964fda5        2 weeks ago         0 B
nanoserver             10.0.10586.0        8572198a60f1        2 weeks ago         0 B
nanoserver             latest              8572198a60f1        2 weeks ago         0 B
```

### 配置网络

在使用 Docker 创建容器之前，需要为 Windows 防火墙创建一个规则，以允许到容器的网络连接。 运行以下 PowerShell 脚本，以为端口 80 创建一个规则。 注意 - 这需要在 PowerShell 会话中运行。

```powershell
if (!(Get-NetFirewallRule | where {$_.Name -eq "TCP80"})) {
    New-NetFirewallRule -Name "TCP80" -DisplayName "HTTP on TCP/80" -Protocol tcp -LocalPort 80 -Action Allow -Enabled True
}
```

你可能还需要记下容器主机 IP 地址。 将在整个练习中使用该地址。

### 创建 IIS 容器

你现在拥有一个包含 IIS 的容器映像，可用于部署为操作环境准备的 IIS。

若要从新映像创建容器，请使用 `docker run` 命令，此时要指定 IIS 映像的名称。 请注意，本示例已指定参数 `-p 80:80`。 由于容器已连接到通过网络地址转换提供 IP 地址的虚拟交换机，因此需要将某个端口从容器主机映射到容器 NAT IP 地址上的端口。 有关 `-p` 的详细信息，请参阅 [docker.com 上的 Docker Run 参考](https://docs.docker.com/engine/reference/run/)

```powershell
C:\> docker run --name iisdemo -it -p 80:80 windowsservercoreiis cmd
```

创建容器后，打开浏览器，并浏览到容器主机的 IP 地址。 由于主机的端口 80 已映射到容器的端口 80，因此应显示 IIS 初始屏幕。

![](media/iis1.png)

### 创建应用程序

运行以下命令来删除 IIS 初始屏幕。

```powershell
C:\> del C:\inetpub\wwwroot\iisstart.htm
```

运行以下命令来将默认 IIS 站点替换为新的静态站点。

```powershell
C:\> echo "Hello World From a Windows Server Container" > C:\inetpub\wwwroot\index.html
```

再次浏览到容器主机的 IP 地址，你现在应该可以看到“Hello World”应用程序。 请注意，为了查看更新的应用程序，你可能需要关闭任何现有浏览器连接或清除浏览器缓存。

![](media/HWWINServer.png)

退出与容器的交互式对话。

```powershell
C:\> exit
```

删除容器

```powershell
C:\> docker rm iisdemo
```
删除 IIS 映像。

```powershell
C:\> docker rmi windowsservercoreiis
```

## Dockerfile

通过上一练习，已手动创建和修改容器，并已将其捕获到新容器映像中。 Docker 包含用于自动执行此过程的方法，即使用所谓的 dockerfile。 本练习将产生与上一练习相同的结果，但是这次该过程将完全自动执行。

### 创建 IIS 映像

在容器主机上，创建目录 `c:\build`，并在此目录中创建一个名为 `dockerfile` 的文件。

```powershell
C:\> powershell new-item c:\build\dockerfile -Force
```

使用记事本打开 dockerfile。

```powershell
C:\> notepad c:\build\dockerfile
```

将以下文本复制到 dockerfile 并保存该文件。 这些命令指示 Docker 使用 `windowsservercore` 作为基础创建新映像，并包含使用 `RUN` 指定的修改。 有关 Dockerfile 的详细信息，请参阅 [docker.com 上的 Dockerfile 参考](http://docs.docker.com/engine/reference/builder/)。

```powershell
FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
```

此命令将启动自动映像生成过程。 `-t` 参数指示此过程来将新映像命名为 `iis`。

```powershell
C:\> docker build -t iis c:\Build
```

完成后，你可以验证是否已使用 `docker images` 命令创建映像。

```powershell
C:\> docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
iis                 latest              abb93867b6f4        26 seconds ago      209 MB
windowsservercore   10.0.10586.0        6801d964fda5        2 weeks ago         0 B
windowsservercore   latest              6801d964fda5        2 weeks ago         0 B
nanoserver          10.0.10586.0        8572198a60f1        2 weeks ago         0 B
nanoserver          latest              8572198a60f1        2 weeks ago         0 B
```

### 部署 IIS 容器

现在，就像执行上一练习一样部署容器，从而将主机的端口 80 映射到容器的端口 80。

```powershell
C:\> docker run --name iisdemo -it -p 80:80 iis cmd
```

创建容器后，浏览到容器主机的 IP 地址。 你应该可以看到 Hello World 应用程序。

![](media/dockerfile2.png)

退出与容器的交互式对话。

```powershell
C:\> exit
```

删除容器

```powershell
C:\> docker rm iisdemo
```
删除 IIS 映像。

```powershell
C:\> docker rmi iis
```

## Hyper-V 容器

Hyper-V 容器通过 Windows Server 容器提供额外的隔离层。 每个 Hyper-V 容器都在高度优化的虚拟机中创建。 如果 Windows Server 容器与容器主机共享内核，Hyper-V 容器将会完全隔离。 Hyper-V 容器的创建和管理方式与 Windows Server 容器相同。 有关 Hyper-V 容器的详细信息，请参阅[管理 Hyper-V 容器](../management/hyperv_container.md)。

> Microsoft Azure 不支持 Hyper-V 容器。 若要完成 Hyper-V 练习，你需要本地容器主机。

### 创建容器

由于容器将要运行 Nano Server 操作系统映像，因此需要 Nano Server IIS 程序包才能安装 IIS。 这些程序包可以在 `NanoServer\Packages` 目录下的 Windows Server 2016 TP4 安装媒体中找到。

在本示例中，使用 `docker run` 的 `-v` 参数，可以将容器主机中的目录提供给正在运行的容器。 在执行此操作之前，需要对源目录进行配置。

在容器主机上创建一个将与容器共享的目录。 如果你已经完成 PowerShell 演练，此目录和所需的文件可能已经存在。

```powershell
C:\> powershell New-Item -Type Directory c:\share\en-us
```

将 `Microsoft-NanoServer-IIS-Package.cab` 从 `NanoServer\Packages` 复制到容器主机上的 `c:\share`。

将 `NanoServer\Packages\en-us\Microsoft-NanoServer-IIS-Package.cab` 复制到容器主机上的 `c:\share\en-us`。

在 c:\share 文件夹中，创建名为 unattend.xml 的文件，并将此文本复制到 unattend.xml 文件中。

```powershell
<?xml version="1.0" encoding="utf-8"?>
<unattend xmlns="urn:schemas-microsoft-com:unattend">
    <servicing>
        <package action="install">
            <assemblyIdentity name="Microsoft-NanoServer-IIS-Package" version="10.0.10586.0" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" />
            <source location="c:\iisinstall\Microsoft-NanoServer-IIS-Package.cab" />
        </package>
        <package action="install">
            <assemblyIdentity name="Microsoft-NanoServer-IIS-Package" version="10.0.10586.0" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="en-US" />
            <source location="c:\iisinstall\en-us\Microsoft-NanoServer-IIS-Package.cab" />
        </package>
    </servicing>
</unattend>
```

完成后，应对容器主机上的 `c:\share` 目录进行上述配置。

```
c:\share
|-- en-us
|    |-- Microsoft-NanoServer-IIS-Package.cab
|
|-- Microsoft-NanoServer-IIS-Package.cab
|-- unattend.xml
```

若要使用 Docker 创建 Hyper-V 容器，请指定 `--isolation=hyperv` 参数。 本示例会将 `c:\share` 目录从主机装载到容器的 `c:\iisinstall` 目录，然后创建与该容器的交互式 shell 会话。

```powershell
C:\> docker run --name iisnanobase -it -v c:\share:c:\iisinstall --isolation=hyperv nanoserver cmd
```

### 创建 IIS 映像

在容器 shell 会话内部，可以使用 `dism` 安装 IIS。 在容器中运行以下命令来安装 IIS。

```powershell
C:\> dism /online /apply-unattend:c:\iisinstall\unattend.xml
```

完成 IIS 安装后，使用以下命令手动启动 IIS。

```powershell
C:\> Net start w3svc
```

退出容器会话。

```powershell
C:\> exit
```

### 创建 IIS 容器

修改后的 Nano Server 容器现在可以提交到新容器映像。 若要执行此操作，请使用 `docker commit` 命令。

```powershell
C:\> docker commit iisnanobase nanoserveriis
```

在返回容器映像的列表时可以看到结果。

```powershell
C:\> docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
nanoserveriis       latest              444435a4e30f        About a minute ago   69.14 MB
windowsservercore   10.0.10586.0        6801d964fda5        2 weeks ago          0 B
windowsservercore   latest              6801d964fda5        2 weeks ago          0 B
nanoserver          10.0.10586.0        8572198a60f1        2 weeks ago          0 B
nanoserver          latest              8572198a60f1        2 weeks ago          0 B
```

### 创建应用程序

Nano Server IIS 映像现在可部署到新容器。

```powershell
C:\> docker run -it -p 80:80 --isolation=hyperv nanoserveriis cmd
```

运行以下命令来删除 IIS 初始屏幕。

```powershell
C:\> del C:\inetpub\wwwroot\iisstart.htm
```

运行以下命令来将默认 IIS 站点替换为新的静态站点。

```powershell
C:\> echo "Hello World From a Hyper-V Container" > C:\inetpub\wwwroot\index.html
```

浏览到容器主机的 IP 地址，你现在应该可以看到“Hello World”应用程序。 请注意，为了查看更新的应用程序，你可能需要关闭任何现有浏览器连接或清除浏览器缓存。

![](media/HWWINServer.png)

退出与容器的交互式对话。

```powershell
C:\> exit
```






<!--HONumber=Feb16_HO2-->
