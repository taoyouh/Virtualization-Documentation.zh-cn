# Docker 和 Windows

**这是初步内容，可能还会更改。**

Docker 是一个容器部署和管理平台，适用于 Linux 和 Windows 容器。 Docker 用于创建、管理和删除容器和容器映像。 Docker 支持在公共注册表 (Docker Hub) 和私有注册表 (Docker Trusted Registries) 中存储容器映像。 此外，Docker 还通过 Docker Swarm 提供主机群集功能，并通过 Docker Compose 提供部署自动化。 有关 Docker 和 Docker 工具集的详细信息，请访问 [Docker.com](https://www.docker.com/)。

> 必须启用 Windows 容器功能，然后才能使用 Docker 创建和管理 Windows Server 和 Hyper-V 容器。 有关启用此功能的说明，请参阅[容器主机部署指南](./docker_windows.md)。

## Windows Server

### 安装 Docker

Windows Server 或 Windows Server Core 不附带 Docker 守护程序和 CLI，并且这些功能不与 Windows 容器功能一起安装。 将需要单独安装 Docker。 本文档将演练手动安装 Docker 守护程序和 Docker 客户端的过程。 还将提供完成这些任务的自动化方法。

Docker 守护程序和 Docker 命令行接口已使用 Go 语言进行开发。 此时，docker.exe 不安装为 Windows 服务。 有多种方法可用于创建 Windows 服务，此处所示的一个示例使用 `nssm.exe`。

从 `https://aka.ms/tp4/docker` 下载 docker.exe，并将其放在容器主机上的 System32 目录中。

```powershell
PS C:\> wget https://aka.ms/tp4/docker -OutFile $env:SystemRoot\system32\docker.exe
```

创建名为 `c:\programdata\docker` 的目录。 在此目录中，创建名为 `runDockerDaemon.cmd` 的文件。

```powershell
PS C:\> New-Item -ItemType File -Path C:\ProgramData\Docker\runDockerDaemon.cmd -Force
```

将以下文本复制到 `runDockerDaemon.cmd` 文件中。 此批处理文件使用命令 `docker daemon –D –b “Virtual Switch”` 启动 Docker 守护程序。 注意：此文件中的虚拟交换机名称需要与容器将用于网络连接的虚拟交换机名称匹配。

```powershell
@echo off
set certs=%ProgramData%\docker\certs.d

if exist %ProgramData%\docker (goto :run)
mkdir %ProgramData%\docker

:run
if exist %certs%\server-cert.pem (goto :secure)

docker daemon -D -b "Virtual Switch"
goto :eof

:secure
docker daemon -D -b "Virtual Switch" -H 0.0.0.0:2376 --tlsverify --tlscacert=%certs%\ca.pem --tlscert=%certs%\server-cert.pem --tlskey=%certs%\server-key.pem
```
从 [https://nssm.cc/release/nssm-2.24.zip](https://nssm.cc/release/nssm-2.24.zip) 下载 nssm.exe。

```powershell
PS C:\> wget https://nssm.cc/release/nssm-2.24.zip -OutFile $env:ALLUSERSPROFILE\nssm.zip
```

提取文件，并将 `nssm-2.24\win64\nssm.exe` 复制到 `c:\windows\system32` 目录中。

```powershell
PS C:\> Expand-Archive -Path $env:ALLUSERSPROFILE\nssm.zip $env:ALLUSERSPROFILE
PS C:\> Copy-Item $env:ALLUSERSPROFILE\nssm-2.24\win64\nssm.exe $env:SystemRoot\system32
```
运行 `nssm install` 来配置 Docker 服务。

```powershell
PS C:\> start-process nssm install
```

在 NSSM 服务安装程序中将以下数据输入到相应的字段中。

应用程序选项卡：

- **路径：**C:\Windows\System32\cmd.exe

- **启动目录：**C:\Windows\System32

- **参数：**/s /c C:\ProgramData\docker\runDockerDaemon.cmd

- **服务名称** - Docker

![](media/nssm1.png)

详细信息选项卡：

- **显示名称：**Docker

- **说明：**Docker 守护程序为 Docker 客户端提供容器的管理功能。


![](media/nssm2.png)

IO 选项卡：

- **输出 (stdout)：**C:\ProgramData\docker\daemon.log

- **错误 (stderr)：**C:\ProgramData\docker\daemon.log


![](media/nssm3.png)

操作完成后，单击“安装服务”按钮。

完成此操作后，当 Windows 启动时，Docker 守护程序（服务）也将启动。

### 删除 Docker

如果按照此指南从 docker.exe 创建 Windows 服务，以下命令将删除该服务。

```powershell
PS C:\> sc.exe delete Docker

[SC] DeleteService SUCESS
```

## Nano Server

### 安装 Docker

从 `https://aka.ms/tp4/docker` 下载 docker.exe，并将其复制到 Nano Server 容器主机的 `windows\system32` 文件夹。

运行以下命令来启动 Docker 守护程序。 每次启动容器主机时都需要运行此程序。 此命令启动 Docker 守护程序、为容器连接性指定虚拟交换机并将该守护程序设置为在端口 2375 上侦听传入的 Docker 请求。 在此配置中，可以从远程计算机管理 Docker。

```powershell
PS C:\> start-process cmd "/k docker daemon -D -b <Switch Name> -H 0.0.0.0:2375”
```

### 删除 Docker

若要从 Nano Server 删除 Docker 守护程序和 CLI，请从 Windows\system32 目录中删除 `docker.exe`。

```powershell
PS C:\> Remove-Item $env:SystemRoot\system32\docker.exe
```

### 交互式 Nano 会话

> 有关远程管理 Nano Server 的信息，请参阅 [Nano Server 入门](https://technet.microsoft.com/en-us/library/mt126167.aspx#bkmk_ManageRemote)。

以交互方式管理 Nano Server 主机上的容器时，可能会收到此错误。

```powershell
docker : cannot enable tty mode on non tty input
+ CategoryInfo          : NotSpecified: (cannot enable tty mode on non tty input:String) [], RemoteException
+ FullyQualifiedErrorId : NativeCommandError 
```

当尝试使用 -it 通过交互式会话运行容器时，可能会出现这种情况：

```powershell
Docker run -it <image> <command>
```
或尝试附加到正在运行的容器：

```powershell
Docker attach <container name>
```

若要使用 Docker 在 Nano Server 主机上创建的容器来创建交互式会话，则必须远程管理 Docker 守护程序。 若要执行此操作，请从[此位置](https://aka.ms/ContainerTools)下载 docker.exe 并将其复制到远程系统。

首先，需要设置 Nano Server 中的 Docker 守护程序以侦听远程命令。 可以通过在 Nano Server 中运行此命令来执行此操作：

```powershell
docker daemon -D -H <ip address of Nano Server>:2375
```

现在，在计算机上，打开 PowerShell 或 CMD 会话，并运行通过 `-H` 指定远程主机的 Docker 命令。

```powershell
.\docker.exe -H tcp://<ip address of Nano Server>:2375
```

例如，如果想要查看可用的映像：

```powershell
.\docker.exe -H tcp://<ip address of Nano Server>:2375 images
```




<!--HONumber=Jan16_HO3-->
