---
author: neilpeterson
redirect_url: ../quick_start/manage_docker
---

# Windows Server 容器管理

**这是初步内容，可能还会更改。**

容器生命周期包括启动、停止和删除容器等操作。 在执行这些操作时，你可能还需要检索容器映像的列表、管理容器网络并限制容器资源。 本文档将详细说明使用 Docker 的基本容器管理任务，也将链接到进行深入说明的文章。

## 管理容器

### 创建容器

使用 `docker run` 通过 Docker 创建容器。

```none
PS C:\> docker run -p 80:80 windowsservercoreiis
```

有关 Docker `run` 命令的详细信息，请参阅 [Docker run 参考](https://docs.docker.com/engine/reference/run/)。

### 停止容器

使用 `docker stop` 命令通过 Docker 停止容器。

```none
PS C:\> docker stop tender_panini

tender_panini
```

此示例使用 Docker 停止所有运行的容器。

```none
PS C:\> docker stop $(docker ps -q)

fd9a978faac8
b51e4be8132e
```

### 删除容器

若要使用 Docker 删除容器，请使用 `docker rm` 命令。

```none
PS C:\> docker rm prickly_pike

prickly_pike
```

使用 Docker 删除所有容器。

```none
PS C:\> docker rm $(docker ps -aq)

dc3e282c064d
2230b0433370
```

有关 Docker rm 命令的详细信息，请参阅 [Docker rm 参考](https://docs.docker.com/engine/reference/commandline/rm/)。






<!--HONumber=Apr16_HO5-->


