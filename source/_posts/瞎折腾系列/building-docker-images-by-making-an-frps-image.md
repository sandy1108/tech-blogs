---
title: 通过frps镜像的制作学习Docker镜像制作
categories: 
  - 瞎折腾系列
excerpt: 在本文中，我记录了如何通过 Dockerfile 脚本来为 frps 服务制作一个 Docker 镜像。我详细描述了从编写 Dockerfile、构建镜像，到如何通过端口映射或使用 host 网络模式运行容器的完整步骤。此外，我还补充了如何进入容器内部，以及将容器当前状态直接保存为新镜像等实用操作。
date: 2022-09-22 11:25:47
tags:
  - Docker
  - Dockerfile
  - FRP
  - 镜像构建
  - 容器
---

## 本文源码

https://github.com/sandy1108/frps-docker

## 以Dockerfile脚本的形式制作Docker镜像

### 制作步骤

1.  下载ubuntu基础镜像(这里采用了19.04版)

    docker pull ubuntu:19.04

2.  查看本地镜像列表

    docker image ls -a

    docker images

3.  编写Dockerfile

```
    FROM ubuntu:19.04

    LABEL maintainer="sandy1108 <sandy1108@163.com>"

    ADD <https://github.com/fatedier/frp/releases/download/v0.27.1/frp_0.27.1_linux_amd64.tar.gz> /tmp/

    RUN tar -xzvf /tmp/frp\_0.27.1\_linux\_amd64.tar.gz -C / \
    && mv /frp\_0.27.1\_linux\_amd64 /frp

    CMD /frp/frps -c /frp/frps.ini
```

4.  构建镜像

    docker build -t frps:0.27.1

## 由镜像生成容器

1.  运行镜像到容器

    docker run -itd -p \[要映射到的宿主机端口]:\[容器内需要被映射服务端口] --name \[要新建的容器ID或名称] \[镜像ID或名称] /bin/bash

其中，d参数代表后台运行容器，返回容器ID；不添加d参数，则会在当前会话执行容器的CMD。

例如：

```
    docker run -itd -p 9191:7000 -p 9191:7000/udp --name myfrp-test-08 frps:0.27.1
```

2.  （另一种情况）运行镜像到直接使用宿主机网络的容器（这样frp可以随意开放端口了，只要防火墙开放就行，个人认为比较适用于frp服务。host模式解释<https://docs.docker.com/network/host/）>

    docker run -itd --network=host --name myfrp-test-08 frps:0.27.1

## 进入容器

```
    docker exec -it [container名称或者ID] /bin/bash
```

## 附：以容器当前状态保存为镜像（一般是测试或者备份使用）

1.  命令解释

```
    # docker commit \[OPTIONS] CONTAINER \[REPOSITORY\[:TAG]]
```
```
    \-a :提交的镜像作者；
    \-c :使用Dockerfile指令来创建镜像；
    \-m :提交时的说明文字；
    \-p :在commit时，将容器暂停。
```

2.  示例

```
    docker commit -a "sandy1108" -m "just for test" container\_ID\_OR\_NAME ImageName\:tagv1
```

3.  查看镜像会发现本地多了一个镜像

```
    docker images
```
