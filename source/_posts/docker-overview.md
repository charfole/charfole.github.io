---
title: Docker镜像、容器、仓库基本概念介绍
date: 2023-09-21 17:05:45
tags:
- Docker
---

Docker是快速搭建开发环境的好帮手，之前对它的了解一直处于使用层面，最近阅读了[Docker的官方文档](https://docs.docker.com/)发现里面的概念还挺清晰易懂的，遂写下博客，记录Docker中镜像、容器、仓库这三个最重要的概念。

<!--more-->

## Docker基本概念

Docker是一个帮助应用进行开发、迁移和运行的平台，它将软件与系统架构进行隔离，从而使得软件可以被快速部署。

Docker通过容器来为应用提供一个轻量级的运行环境，在保证各个应用能通过容器来运行与协作的同时，降低了容器对环境的依赖。此外Docker还提供了一系列命令来帮助管理应用的生命周期。

如下图，Docker使用客户端/服务器架构，客户端发出指令（如docker run、docker build），服务端的守护进程负责侦听和接收指令，并对镜像、容器、网络和数据卷等内容进行管理。Docker客户端和服务端通过基于UNIX套接字的REST API进行通信。Docker Compose是另一个Docker客户端，允许用户使用一组容器来组成应用程序。

![image-20230921170151004](https://charfole-blog.oss-cn-shenzhen.aliyuncs.com/image/image-20230921170151004.png)

## Docker对象

### Docker镜像

Docker镜像是一个用于创建Docker容器的模板，通常一个镜像是基于其他镜像来构建的。通过创建Dockerfile可以来创建一个镜像，每条指令用于构建Docker中的不同层次，当改变某一层时将会重建这个镜像，此时只有受到影响的层次会改变。

一个镜像包含支持容器运行的所有东西，如：所有依赖、配置、脚本、二进制文件、环境变量、元数据等等。

### Docker容器

Docker容器是一个镜像的运行实例，它根据镜像来定义其初始化配置。容器可以连接多个网络，关联数据、甚至当前容器状态可以被用于复制来创建一个镜像。

Docker容器具有良好的隔离性，可以与其他容器进行隔离依赖，专注于运行自己的软件。隔离性的实现依赖于Linux的命名空间与cgroups。
与Linux中的[chroot](https://www.runoob.com/linux/linux-comm-chroot.html)命令类似，可看作是它的一个升级版。

Docker容器可用于创建一个安全的沙盒环境，其文件系统来源于镜像文件。容器在`chroot`的基础上添加了一些额外的功能，来保证环境更为安全。

### Docker仓库

Docker仓库用于存储Docker镜像，如[Docker Hub](https://hub.docker.com/)。`docker pull`从远程仓库中拉取所需的镜像，`docker push`则将镜像推送到仓库。

### Docker容器创建过程

假设执行以下的命令来创建一个`ubuntu`容器：

```shell
docker run -i -t ubuntu /bin/bash
```

1. 如果在本地没有`ubuntu`容器，Docker将从配置的远程仓库中拉取`docker pull ubuntu`镜像。
2. 接下来Docker会新建一个容器，`docker container create`。
3. Docker为该容器分配一个读写文件系统，其允许运行的容器在其中创建或修改文件与目录。
4. 接下来Docker将创建一个网络接口，默认情况下容器可通过主机的网络连接到外部网络。
5. Docker启动该容器，由于`-i`，`-t`命令代表该容器可交互与连接到终端，后续将直接运行`/bin/bash`命令并打开`bash`终端，接受输入，返回输出。
6. 当使用`exit`命令来退出终端时，容器虽然停止了但不会被移除，其可以被重启或者是移除。

### 参考链接

[Docker Overview](https://docs.docker.com/get-started/overview/)