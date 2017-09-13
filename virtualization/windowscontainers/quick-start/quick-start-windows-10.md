---
title: Windows Container on Windows 10
description: Container deployment quick start
keywords: docker, containers
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 2c5742ba11f830762a337f31ebcd1ae700cb3905
ms.sourcegitcommit: 015f8c438cd1e1331e5388280facce4b9ec939ac
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/26/2017
---
# Windows Containers on Windows 10

The exercise will walk through basic deployment and use of the Windows container feature on Windows 10 Professional or Enterprise (Anniversary Edition). After completion, you will have installed Docker for Windows and run a simple container. 如果你需要熟悉容器，可在[关于容器](../about/index.md)中找到此信息。

本快速入门特定于 Windows 10。 Additional quick start documentation can be found in the table of contents on the left hand side of this page.

***Hyper-V isolation:*** Windows Server Containers require Hyper-V isolation on Windows 10 in order to provide developers with the same kernel version and configuration that will be used in production, more about this can be found on the [About Windows container](../about/index.md) page.

**Prerequisites:**

- One physical computer system running Windows 10 Anniversary Edition or Creators Update (Professional or Enterprise).   
- This quick start can be run on a Windows 10 virtual machine but nested virtualization will need to be enabled. More information can be found in the [Nested Virtualization Guide](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).

> You must install critical updates for Windows Containers to work.
> To check your OS version, run `winver.exe`, and compare the version shown to [Windows 10 update history](https://support.microsoft.com/en-us/help/12387/windows-10-update-history).
> Make sure you have 14393.222 or later before continuing.

## 1. Install Docker for Windows

[Download Docker for Windows](https://download.docker.com/win/stable/InstallDocker.msi) and run the installer. [Detailed installation instructions](https://docs.docker.com/docker-for-windows/install) are available in the Docker documentation.

## 2. Switch to Windows containers

After installation Docker for Windows defaults to running Linux containers. Switch to Windows containers using either the Docker tray-menu or by running the following command in a PowerShell prompt `& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon`.

![](./media/docker-for-win-switch.png)

## 3. Install Base Container Images

Windows containers are built from base images. The following command will pull the Nano Server base image.

```none
docker pull microsoft/nanoserver
```

Once the image is pulled, running `docker images` will return a list of installed images, in this case the Nano Server image.

```none
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> Please read the Windows Containers OS Image EULA which can be found here – [EULA](../images-eula.md).

## 4. Run Your First Container

For this simple example a ‘Hello World’ container image will be created and deployed. For the best experience run these commands in an elevated Windows CMD shell or PowerShell.

> Windows PowerShell ISE does not work for interactive sessions with containers. Even though the container is running, it will appear to hang.

First, start a container with an interactive session from the `nanoserver` image. Once the container has started, you will be presented with a command shell from within the container.  

```none
docker run -it microsoft/nanoserver cmd
```

Inside the container we will create a simple ‘Hello World’ script.

```none
powershell.exe Add-Content C:\helloworld.ps1 'Write-Host "Hello World"'
```   

When completed, exit the container.

```none
exit
```

You will now create a new container image from the modified container. To see a list of containers run the following and take note of the container id.

```none
docker ps -a
```

Run the following command to create the new ‘HelloWorld’ image. Replace <containerid> with the id of your container.

```none
docker commit <containerid> helloworld
```

When completed, you now have a custom image that contains the hello world script. This can be seen with the following command.

```none
docker images
```

Finally, to run the container, use the `docker run` command.

```none
docker run --rm helloworld powershell c:\helloworld.ps1
```

The outcome of the `docker run` command is that a Hyper-V container was created from the 'HelloWorld' image, a sample 'Hello World' script was then executed (output echoed to the shell), and then the container stopped and removed.
Subsequent Windows 10 and container quick starts will dig into creating and deploying applications in containers on Windows 10.

## 后续步骤

继续学习下一个教程以查看[生成示例应用](./building-sample-app.md)的示例
