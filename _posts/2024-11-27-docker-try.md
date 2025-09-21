---
layout: post
title: docker 初尝试
categories: [docker]
description: docker 初尝试
keywords: docker
---

# 启动镜像

装一个 22.04

```bash
docker pull dockerpull.org/ubuntu:22.04
```

好不容易找到的网址，我日，怎么全封了

然后把这个镜像搞成 ！

```bash
docker run -itd --name minirts --privileged -v D:/code/ELF1120:/root/ELF1120 dockerpull.org/ubuntu:22.04
```

不知道为什么这里必须是 `-itd` 不能是 `-d`，否则启动不起来，原因不明。

参数说明：

- **-i**: 交互式操作。
- **-t**: 终端。
- **-d**: 参数默认不会进入容器
- **-v**: 路径映射
- **--privileged**: 
  - 使用该参数，container内的root拥有真正的root权限。
  - 否则，container内的root只是外部的一个普通用户权限。
  - privileged启动的容器，可以看到很多host上的设备，并且可以执行mount。
  - 甚至允许你在docker容器中启动docker容器。 

# 配置环境

## 假如说环境是这样的

在容器中更新包列表并安装 `wget`：

> ```bash
> apt-get update
> apt install -y wget
> apt install -y build-essential # 包含了 gcc g++ 11.4.0
> apt install -y git
> apt install -y libtbb-dev
> apt install -y cmake
> ```
>
> 接下来装一下 conda 和 python
>
> ```bash
> # 需要python环境，python3.7
> wget https://repo.anaconda.com/archive/Anaconda3-5.3.0-Linux-x86_64.sh
> chmod +x Anaconda3-5.3.0-Linux-x86_64.sh
> ./Anaconda3-5.3.0-Linux-x86_64.sh
> source .bashrc
> # base 环境就是3.7
> ```

## docker 把容器保存为镜像

把容器保存为镜像，其中 `35b4e664c341e3d287e5093b273da2f1640270e703715caf2d56d29ef02c8d53`。

```bash
$ docker commit -m="minirts" -a="poplpr" 35b4e664c341e3d287e5093b273da2f1640270e703715caf2d56d29ef02c8d53 poplpr/minirts
```

## docker 镜像保存：

```bash
$ docker save -o minirts.tar dockerpull.org/ubuntu:minirts
```

## docker 镜像加载：

```bash
$ docker load -i minirts.tar
```

# 运行环境

```bash
$ docker run -itd --name minirts2 -p 8848:8848 -p 8000:8000 --privileged dockerpull.org/ubuntu:minirts
```

进入容器环境运行 bash

```
docker exec -it minirts2 /bin/bash
```

然后切到 `~/minirts/rts/build` 目录下：

```bash
cd ~/minirts/rts/build
./minirts-backend humanplay --vis_after 0
```

如果不执行下面的 python 转发的话，打开网页：`./ELF/rts/frontend/minirts.html`

否则 python 转发，然后浏览器输入 `localhost:8848`

```bash
python3 -m http.server 8848 --directory ~/minirts/rts/frontend/
```

# docker 重启关闭的容器

首先如果不知道关闭的容器名称的话，输入下面的命令查看名称：

```bash
docker ps -a
```

然后输入下面的命令启动容器：

```bash
docker start <容器名称 or Id>
```