# [Windows 文档中的容器](index.md) 

# 概述
## [关于 Windows 容器](about/index.md)
## [系统要求](deploy-containers/system-requirements.md)
## [常见问题解答](about/faq.md)

# 入门
## [设置你的环境](quick-start/set-up-environment.md)
## [运行第一个容器](quick-start/run-your-first-container.md)
## [Containerize 示例应用](quick-start/building-sample-app.md)

# 教程
## 构建 Windows 容器
### [编写 Dockerfile](manage-docker/manage-windows-dockerfile.md)
### [优化 Dockerfile](manage-docker/optimize-windows-dockerfile.md)
## Windows 上的 Kubernetes
### [Windows 上的 Kubernetes](kubernetes/getting-started-kubernetes-windows.md)
### [创建 Kubernetes 母版](kubernetes/creating-a-linux-master.md)
### [选择网络解决方案](kubernetes/network-topologies.md)
### [加入 Windows 工作人员](kubernetes/joining-windows-workers.md)
### [加入 Linux 工作人员](kubernetes/joining-linux-workers.md)
### [部署 Kubernetes 资源](kubernetes/deploying-resources.md)
### [故障排除](kubernetes/common-problems.md)
### [Kubernetes 上的 Windows 服务](kubernetes/kube-windows-services.md)
### [编译 Kubernetes 二进制文件](kubernetes/compiling-kubernetes-binaries.md)
## Windows 上的服务结构
### [部署首个容器](/azure/service-fabric/service-fabric-quickstart-containers)
### [在 Windows 容器中部署 .NET 应用程序](/azure/service-fabric/service-fabric-host-app-in-a-container)
## Windows 上的 Linux 容器
### [概述](deploy-containers/linux-containers.md)
### [运行第一个 LCOW 容器](quick-start/quick-start-windows-10-linux.md)

# 概念
## Windows 容器概要
### [资源控制](manage-containers/resource-controls.md)
### [Hyper-V 隔离](manage-containers/hyperv-container.md)
### [版本兼容性](deploy-containers/version-compatibility.md)
### [容器基映像](manage-containers/container-base-images.md)
## Docker
### [Windows 上的 Docker 引擎](manage-docker/configure-docker-daemon.md)
### [Docker swarm](manage-containers/swarm-mode.md)
### [Windows Docker 主机的远程管理](management/manage_remotehost.md)
## 工作
### 组托管服务帐户
#### [创建 gMSA](manage-containers/manage-serviceaccounts.md)
#### [将你的应用配置为使用 gMSA](manage-containers/gmsa-configure-app.md)
#### [使用 gMSA 运行容器](manage-containers/gmsa-run-container.md)
#### [具有 gMSA 的协调容器](manage-containers/gmsa-orchestrate-containers.md)
#### [GMSAs 疑难解答](manage-containers/gmsa-troubleshooting.md)
### [打印机服务](deploy-containers/print-spooler.md)
## 网络
### [概述](container-networking/architecture.md)
### [网络拓扑和驱动程序](container-networking/network-drivers-topologies.md)
### [网络隔离和安全](container-networking/network-isolation-security.md)
### [配置高级网络选项](container-networking/advanced.md)
## 存储
### [概述](manage-containers/container-storage.md)
## 设备
### [硬件设备](deploy-containers/hardware-devices-in-containers.md)
### [GPU 加速](deploy-containers/gpu-acceleration.md)

# 参考
## [基本图像服务生命周期](deploy-containers/base-image-lifecycle.md)
## [反病毒优化](https://docs.microsoft.com/windows-hardware/drivers/ifs/anti-virus-optimization-for-windows-containers)
## [容器平台工具](deploy-containers/containerd.md)
## [容器操作系统映像 EULA](Images_EULA.md)

# 资源
## [容器示例](samples.md)
## [故障排除](troubleshooting.md)
## [容器论坛](https://social.msdn.microsoft.com/Forums/home?forum=windowscontainers)
## [社区视频和博客](communitylinks.md)