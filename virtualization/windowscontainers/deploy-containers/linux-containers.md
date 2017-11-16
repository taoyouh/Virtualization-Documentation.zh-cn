# <a name="linux-containers"></a>Linux 容器

此功能使用 [Hyper-V 隔离](../manage-containers/hyperv-container.md)运行采用正好的操作系统版本的 Linux 内核以支持容器。 用于构建此容器的 Windows 和 Hyper-V 的转变始于 _Windows 10 Fall Creators Update_ 和 _Windows Server 版本 1709_，但这一成果也离不开开源 [Moby 项目](https://www.github.com/moby/moby)的开展，Docker 技术正是依托 Moby 项目构建的，Linux 内核也是如此。 

![Linux 容器预览视频](https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4)

如果要试用此功能，你将需要：

- Windows 10 或者 Windows Server Insider Preview（内部版本 16267 或更高版本）
- 基于 Moby 主分支的、带有 `--experimental` 标志运行的 Docker 守护程序的某个版本
- 选择兼容的 Linux 映像

可获取有关此预览的入门指南：

- [Docker Enterprise Edition Preview](https://blog.docker.com/2017/09/docker-windows-server-1709/) 包含 LinuxKit 系统和可运行 Linux 容器的 Docker EE 预览版。 如果要了解更多背景知识，还可以查阅[使用 LinuxKit 在 Windows 上预览 Linux 容器](https://go.microsoft.com/fwlink/?linkid=857061)
- [使用 Hyper-V 隔离在 Windows 10 和 Windows Server 上运行 Ubuntu 容器](https://go.microsoft.com/fwlink/?linkid=857067)


## <a name="work-in-progress"></a>正在进行的工作

可在 [GitHub](https://github.com/moby/moby/issues/33850) 上跟踪 Moby 项目的当前进展


### <a name="known-app-issues"></a>已知的应用问题

这些应用程序均要求卷映射，其中一些限制条件在[绑定挂载](#Bind-mounts)下有介绍。 它们将无法正常启动或运行。

- Mysql
- Postgress
- Wordpress
- Jenkins
- Mariadb
- Rabbitmq


### <a name="bind-mounts"></a>绑定挂载

带有 `docker run -v ...` 的绑定挂载卷将文件存储于 Windows NTFS 文件系统，因此对于 POSIX 操作需要进行一些转换。 有些文件系统操作当前只得到部分实现或者未实现，这可能导致一些应用无法兼容。

以下操作目前对绑定挂载卷无效：

- MkNod
- XAttrWalk
- XAttrCreate
- 锁定
- Getlock
- Auth
- 刷新
- INotify

还有一些操作未能完全实现：

- GetAttr - Nlink 计数一直报告为 2
- 打开 - 仅 ReadWrite、WriteOnly 和 ReadOnly 标志得到实现