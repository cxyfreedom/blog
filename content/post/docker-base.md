---
title: Docker 容器技术基础总结
date: 2019-01-16T18:10:40+08:00
description: ""
tags: ["Docker"]
categories: ["DevOps"]
---
Docker是一个开源的引擎，可以轻松的为任何应用创建一个轻量级的、可移植的、自给自足的容器。开发者在笔记本上编译测试通过的容器可以批量地在生产环境中部署。

<!--more-->

## 简介

### 为什么要使用 Docker

* **更快速的交付和部署**。开发人员可以使用镜像快速构建一套标准的开发环境。然后测试和运维可以使用相同环境进行测试。Docker 可以快速创建和删除容器，实现快速迭代，节约开发、测试、部署的大量时间。
* **更高效的资源利用**。Docker 是内核级的虚拟化，可以实现更高的性能，同时对资源的额外需求很低。
* **更轻松的迁移和扩展**。几乎可以在任意的平台运行，同时支持主流的操作系统发行版本。
* **更简单的更新管理**。使用 Dockerfile，通过很小的配置修改，就可以以增量的方式被分发和更新，从而实现自动化并且高效的容器管理。

★ Docker 和虚拟机比较

特性|容器|虚拟机
--- | --- | ---
启动速度 | 秒级 | 分钟级
硬盘使用 | 一般为 MB | 一般为 GB
性能 | 接近原生 | 较弱
内存代价 | 很小 | 较多
运行密度 | 单机支持上千个容器 | 一般几十个
隔离性 | 安全隔离 | 完全隔离 | 
迁移性 | 优秀 | 一般 |

传统方式是在硬件层面实现虚拟化，需要有额外的虚拟机管理应用和虚拟机操作系统层。Docker 容器是在操作系统层面上实现虚拟化，直接复用本地主机的操作系统，因此更加轻量级。

## 核心概念

★ 镜像

Docker 镜像类似于虚拟机镜像，可以将其理解为一个只读的模板。镜像是创建 Docker 容器的基础。

★ 容器

镜像和容器的关系，就像是面向对象程序设计中的类和实例一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。

容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的 命名空间。因此容器可以拥有自己的 root 文件系统、自己的网络配置、自己的进程空间，甚至自己的用户 ID 空间。容器内的进程是运行在一个隔离的环境里，使用起来，就好像是在一个独立于宿主的系统下操作一样。

★ 仓库

类似于代码仓库，是 Docker 集中存放镜像文件的场所。可以分为公开仓库和私有仓库。

### 安装

★ Ubuntu

如果是 Ubuntu 16.04 LTS 版本，要让 Docker 使用 aufs 存储，推荐安装如下软件包：
```sh
$ sudo apt-get update
$ sudo apt-get install -y linux-image-extra-$(uname -r) linux-image-extra-virtual
```

添加镜像源（以 16.04 为例，非该版本需要替换对应的系统代号）
```sh
$ sudo apt-get update
$ sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
# 添加源的 gpg 密钥
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
# 添加官方软件源
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu xenial stable"
$ sudo apt-get update
```

安装 Docker
```sh
$ sudo apt-get install -y docker-ce
```

★ CentOS 7

添加软件源以及支持 devicemapper 存储类型
```sh
$ sudo yum update
$ sudo yum install -y yum-utils device-mapper-persistent-data lvm2
$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

安装 Docker
```sh
$ sudo yum update
$ sudo yum install -y docker-ce
```

★ 脚本安装

```sh
$ curl -fsSL https://get.docker.com/ | sh
# 或者
$ wget -qO- https://get.docker.com/ | sh
```

如果 Docker 服务不正常，可以通过命令 `journalctl -u docker.service` 查看相关日志信息。

## 镜像

### 获取镜像

命令格式：`docker [image] pull NAME[:TAG]`。

NAME 是镜像仓库名称，一般是两段式。即`<用户名>/<软件名>`。如果不给出用户名，则默认是`library`，也就是官方镜像。TAG 是镜像的标签，通常用来表示版本信息。如果不显示指定，默认选择 `latest` 标签。

示例：`docker pull ubuntu:18.04`。

可选参数：
```sh
# 是否获取仓库中的所有镜像，默认为否。
-a, --all-tags=true|false
# 取消镜像的内容校验，默认为真。
--disable-content-trust
```

### 查看镜像

命令格式：`docker images` 或 `docker image ls`。

可选参数：
```sh
# 列出所有镜像文件，默认为否
-a, --all=true|false
# 列出镜像的数字摘要值，默认为否
--digests=true|false
# 过滤列出的镜像
-f, --filter=[]
# 控制输出格式，比如.ID代表ID信息
--format="TEMPLATE"
# 截断输出结果，默认为是
--no-trunc=true|false
# 仅输出ID信息，默认为否
-q, --quiet=true|false
```

★ 使用 tag 命令添加标签

命令格式：`docker tag <src-tag> <new-tag>`

★ 获取镜像详细信息

命令格式：`docker [image] inspect <image>`

★ 查看镜像历史

命令格式：`docker history <image>`

### 查询镜像

命令格式：`docker search [option] keyword`

可选参数：
```sh
# 过滤输出内容
-f, --filter filter
# 格式化输出内容
--format string
# 限制输出结果个数，默认为 25 个
--limit int
# 不截断输出结果
--no-trunc
```

### 清理镜像

命令格式：`docker rmi` 或者 `docker image rm`

可选参数：
```sh
# 强制删除镜像
-f, -force
# 不清理未带有标签的父镜像
-no-prune
```

删除镜像可以通过标签，也可以通过镜像 ID。

使用 Docker 后，系统中可能会遗留一些临时的镜像文件以及一些没有被使用的镜像，可以通过 `docker image prune` 来进行清理。

可选参数：
```sh
# 删除所有无用镜像
-a, -all
# 只清理符合给定过滤器的镜像
-filter filter
# 强制删除镜像
-f
```

### 创建镜像

★ 基于已有容器创建

命令格式：`docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]`

可选参数：
```sh
# 作者信息
-a, --author=""
# 提交的时候执行 Dockfile 指令
-c, --change=[]
# 提交消息
-m, --message=""
# 提交时暂停容器运行
-p, --pause=true
```

示例：
```sh
# 创建一个容器并做修改
$ docker run -it ubuntu:16.04 bash
root@100040bb9a1c:/# touch test
root@100040bb9a1c:/# exit
exit

# 创建新的镜像
$ docker commit -m "Added a new file" -a "Docker Newbee" 100040bb9a1c test

# 查看新创建的镜像
docker images
```

★ 基于本地模板导入

可以使用 [OpenVZ](https://download.openvz.org/template/precreated/) 提供的模板来创建。

示例：
```sh
cat ubuntu-18.04-x86_64-minimal.tar.gz |docker import - ubuntu:18.04
```

★ 基于 Dockerfile 创建

示例：
```sh
FROM centos
RUN yum install -y vim
```

执行 `docker build -t centos-vim .` 即可。

### 存出和载入镜像

★ 存出镜像

比如将本地的 ubuntu:18.04 镜像导出为文件 ubuntu_18.04.tar，执行以下命令：
```sh
$ docker save -o ubuntu_18.04.tar ubuntu:18:04
```

★ 载入镜像

```sh
$ docker load -i ubuntu_18.04.tar # 方法一
$ docker load < ubuntu_18.04.tar # 方法二
```

## 容器

### 创建容器

★ 新建容器

命令格式：`docker [container] create`

示例：
```sh
$ docker create -it ubuntu:latest
```

★ 新建并启动容器

```sh
$ docker run ubuntu:16.04 /bin/echo "Hello World"
```

通常如果无法正常执行会直接退出，可以查看退出的错误代码。

* 125: Docker daemon 执行出错，例如指定了不支持的 Docker 命令参数。
* 126: 所指定命令无法执行，例如权限出错。
* 127: 容器内命令无法找到。

★ 守护态执行

```sh
$ docker run -d ubuntu:18.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
```

★ 查看容器输出

命令格式：`docker [container] logs`

可选参数：
```sh
# 打印详细信息
-details
# 持续保持输出
-f, -follow
# 输出从某个时间开始的日志
-since string
# 输出最近的若干日志
-tail string
# 显示时间戳信息
-t, -timestamps
# 输出某个时间之前的日志
-until string
```

### 停止容器

```sh
# 暂停容器
docker pause <container>
# 恢复暂停的容器
docker unpause <container>
# 终止容器
docker stop <container>
# 重新启动容器
docker restart <container>
```

### 进入容器

★ attach

命令格式：`docker attach <container>`

可选参数：
```sh
# 指定退出 attach 模式的快捷键序列，默认 ctrl-p ctrl-q
--detach-keys[=[]]
# 是否关闭标准输入，默认是保持打开
--no-stdin=true|false
# 是否代理收到的系统信号给应用进程，默认为 true
--sig-proxy=true|false
```

★ exec

示例：
```sh
$ docker exec -it <container> /bin/bash
```

可选参数：
```sh
# 在容器中后台执行命令
-d, --detach
# 指定将容器切回后台的按键
--detach-keys=""
# 指定环境变量列表
-e, --env=[]
# 打开标准输入接受用户输入命令，默认为false
-i, --interactive=true|false
# 是否给执行命令以高权限，默认为false
--privileged=true|false
# 分配伪终端。默认为false
-t, --tty=true|false
# 执行命令的用户名或ID
-u, --user=""
```

### 删除容器

命令格式：`docker rm <container>`

可选参数：
```sh
# 强制终止并删除一个运行中的容器
-f, --force=false
# 删除容器的连接，但保留容器
-l, --link=false
# 删除容器挂载的数据卷
-v, --volumes=false
```

### 导出和导入容器

★ 导出容器

命令格式：`docker [container] export [-o|--output=""]] <container>`

```sh
$ docker export 72a > ubuntu_container.tar
```

★ 导入容器

命令格式：`docker [container] import <container> - [repository[:tag]]`

示例：
```sh
$ docker import ubuntu_container.tar - cxyfreedom/ubuntu:v1.0
```

### 查看容器

★ 查看容器详情

命令格式：`docker inspect <container>`

★ 查看容器进程

命令格式：`docker [container] top <container>`


★ 查看统计信息

命令格式：`docker [container] stats <container>`

可选参数：
```sh
# 输入所有容器统计信息，默认尽在运行中
-a, -all
# 格式化输出信息
-format string
# 不持续输出，默认会自动更新持续实时结果
-no-stream
# 不截断输出信息
-no-trunc
```

### 其他命令

★ 复制文件

命令格式：`docker [container] cp <container>:<src_path> dest_path|-`

可选参数：
```sh
# 打包模式，复制文件会带有原始的uid/gid信息
-a, -archive
# 跟随软连接。当原路径为软连接时，默认只复制链接信息，使用该选项会复制链接的目标内容
-L, -follow-link
```

示例：
```sh
# 将本地路径的 data 复制到 test 容器的 /tmp 路径下
docker cp data test:/tmp/
```

★ 查看变更

命令格式：`docker [container] diff <container>`

★ 查看端口映射

命令格式：`docker [container] port <container>`

★ 更新配置

命令格式：`docker [container] update [options] <container>`

支持很多的参数，这里不具体展开，可以查看相关文档。

## 数据管理

### 数据卷

数据卷是一个可供容器使用的特殊目录，它将主机操作系统目录直接映射进容器，类似于 Linux 中的 mount 行为。

★ 创建数据卷

示例：
```sh
# 在 /var/lib/docker/volumes 下创建
$ docker volume create -d local test
```

★ 绑定数据卷

在启动容器时，可以使用 `--mount` 来使用数据卷。支持三种类型的数据卷：

* volume：普通数据卷，映射到主机 `/var/lib/docker/volumes` 路径下。
* bind: 绑定数据卷，映射到主机指定路径下。
* tmpfs: 临时数据卷，只存在于内存中。

示例：使用 training/webapp 镜像创建一个 Web 容器，并创建一个数据卷挂载到容器的 /opt/webapp 目录。
```sh
$ docker run -d -P --name web --mount type=bind,source=/webapp,destination=/opt/webapp training/webapp python app.py
```

等同于使用 `-v`:
```sh
$ docker run -d -P --name web -v /webapp:/opt/webapp training/webapp python app.py
```

挂载数据卷的默认权限是读写，也可用通过 ro 指定为只读。
```sh
$ docker run -d -P --name web -v /webapp:/opt/webapp:ro training/webapp python app.py
```

### 数据卷容器

如果需要在多个容器之间共享一些持续更新的容器，最简单的方式就是使用数据卷容器。

示例：
首先创建一个数据卷容器 dbdata，并在其中创建一个数据卷挂载到 /dbdata。
```sh
$ docker run -it -v /dbdata --name dbdata ubuntu
```

然后可以在其他容器中使用 `--volumes-from` 参数来挂载 dbdata 容器中的数据卷。
```sh
$ docker run -it --volumes-from dbdata --name db1 ubuntu
$ docker run -it --volumes-from dbdata --name db2 ubuntu
```

此时，容器 db1 和 db2 都挂载同一个数据卷到相同的 /dbdata 目录，三个容器任何一方在该目录下的写入，其他容器都可以看到。

如果删除了挂载的容器，数据卷并不会被自动删除。如果需要删除数据卷，需要在删除最后一个还挂载着它的容器时显式使用 `docker rm -v` 来制定同时删除关联的数据卷。

★ 迁移数据

备份
```sh
$ docker run --volumes-from dbdata -v ${pwd}:/backup --name worker ubuntu tar cvf /backup/backup.tar /dbdata
```

首先创建一个 worker 容器并挂载 dbdata 容器的数据卷，然后挂载本地目录到容器中的 /backup 目录下。启动后，使用打包压缩的方式将数据备份到容器内的 /backup/backup.tar，即本地当前目录下的 backup.tar。

恢复
```sh
# 创建一个带有数据卷的容器 dbdata2
$ docker run -v /dbdata --name dbdata2 ubuntu /bin/bash
# 恢复
$ docker run --volumes-from dbdata -v ${pwd}:/backup busybox tar xvf /backup/backup.tar
```

## 端口映射和容器互联

★ 端口映射

通过 `-p` 或者 `-P` 来指定端口映射。如果使用 `-P`，会随机映射一个 49000～49900 的短裤到内部容器开放的网络端口。使用 `-p` 时，可以有多种支持格式：

* IP:HostPort:ContainerPort 映射到指定地址的指定端口
* IP::ContainerPort 映射到指定地址的任意端口
* HostPort:ContainerPort 映射所有接口地址

★ 容器互联

通过 `--link` 参数可以让容器之间安全地进行交互。参数格式为：`--link name:alias`，其中 name 是要链接的容器的名称，alias 是别名。

## Dockerfile

Dockerfile 主题内容分为四个部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令。

### 指令说明

★ ARG: 定义创建镜像过程中使用的变量

格式：`ARG <name>[=<default value>]`
示例：`ARG VERSION=9.3`

★ FROM: 指定所创建镜像的基础镜像

格式：`FROM <image> [AS <name>]`
示例：`FROM ubuntu:18.04`

★ LABEL: 添加元数据标签信息

格式：`LABEL <key>=<value> ...`
示例：`LABEL author="cxyfreedom"`

★ EXPOSE: 声明镜像内服务监听的端口

格式：`EXPOSE <port> [<port>/<protocol>...]`
示例：`EXPOSE 22 80 443`

★ ENV: 指定环境变量，在镜像生成过程中会被 RUN 指令使用，在启动的容器中也会存在

格式：`ENV <key> <value>` 或 `ENV <key>=<value> ...`
示例：`ENV APP_VERSION=1.0.0`

运行时会被覆盖掉。

★ ENTRYPOINT: 指定镜像的默认入口命令，该入口命令会在启动容器时作为根命令执行，所有传入值作为该命令的参数

格式：

* `ENTRYPOINT ["executable", "param1", "param2"]`: exec 调用执行
* `ENTRYPOINT command param1 param2`: shell 中执行

每个 Dockerfile 中只能有一个 ENTRYPOINT，当指定多个时，只有最后一个起效。

★ VOLUME: 创建一个数据卷挂载点

格式：`VOLUME ["/data"]`

★ USER: 指定运行容器时的用户名或UID

格式：`USER <用户名>[:<用户组>]`
示例：
```sh
RUN groupadd -r postgres && useradd --no-log-init -r -g postgres postgres
USER postgres
```
★ WORKDIR: 为后续的 RUN、CMD、ENTRYPOINT 指令配置工作目录

格式：`WORKDIR <workdir>`
推荐该指令中只使用绝对路径。

★ ONBUILD: 指定当基于所生成镜像创建子镜像时，自动执行的操作指令

格式：`ONBUILD [instruction]`

由于 ONBUILD 指令是隐式执行的，推荐在使用它的镜像标签中进行标注。

★ STOPSIGINAL: 指定所创建镜像启动的容器接收退出的信号值

格式：`STOPSIGNAL signal`

★ HEALTHCHECK: 配置容器如何进行健康检查

* `HEALTHCHECK [option] CMD command`: 根据所执行命令返回值是否为 0 来判断
* `HEALTHCHECK NONE`: 禁止基础镜像中的健康检查

可选参数：
```sh
# 隔多久检查一次，默认 30s
-interval=DURATION
# 每次检查等待结果的超时，默认 30s
-timeout=DURATION
# 重试几次认为失败，默认 3 次
-retries=N
```

★ SHELL: 指定其他命令使用 shell 时的默认 shell 类型

格式：`SHELL ["executable", "parameters"]`，默认值为 `["/bin/sh", "-c"]`

★ RUN: 运行指定命令

格式：`RUN <command>` 或 `RUN ["executable", "param1", "param2"]`。前者默认在 shell 终端中执行，后者使用 exec 执行，不会启动 shell 环境。

每条 RUN 指令将在当前镜像基础上执行指定命令，并提交为新的镜像层。

★ CMD: 指定启动容器时默认执行的命令

格式：

* `CMD ["executable", "param1", "param2"]`: 推荐方式
* `CMD command param1 param2`: 在默认的 shell 中执行
* `CMD ["param1", "param2"]`: 提供给 ENTRYPOINT 的默认参数

每个 Dockfile 只能有一条 CMD 命令。如果指定了多条命令，只有最后一条会被执行。如果 `docker run` 指定了其它命令，`CMD` 命令被忽略。

★ ADD: 添加内容到镜像

格式：`ADD <src> <dst>`

将指定的 `<src>` 路径下内容复制到容器中的 `<dst>` 路径下。其中 `<src>` 可以是 Dockfile 所在目录的一个相对路径；也可以是一个 URL；也可以是一个 tar 文件（会自动解压为目录）。`<dst>` 可以是镜像内的绝对路径，也可以是相对于工作目录的相对路径。

★ COPY: 复制内容到镜像

格式：`COPY <src> <dst>`

将指定的 `<src>` 路径下内容复制到容器中的 `<dst>` 路径下。目标路径不存在时，会自动创建。

### 创建镜像

★ 多步骤创建

示例：
```sh
FROM golang:1.9 as builder
RUN mkdir -p /go/src/test
WORKDIR /go/src/test
COPY main.go .
RUN CGO_ENABLED=0 GOOS=linux go build -o app .

FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /go/src/test/app .
CMD ["./app"]
```

### 最佳实践

* 精简镜像用途
* 选用合适的基础镜像
* 提供注释和维护者信息
* 正确使用版本号
* 减少镜像层数
* 恰当使用多步骤创建（17.05+版本支持）
* 使用 `.dockerignore` 文件
* 及时删除临时文件和缓存文件
* 提高生成速度：比如合理使用 cache，减少内容目录下的文件等。
* 调整合理的指令顺序
* 减少外部源的干扰