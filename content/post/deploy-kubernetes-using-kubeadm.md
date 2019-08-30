---
title: 使用 kubeadm 安装 Kubernetes 1.12
date: 2018-11-22T17:26:21+08:00
description: ""
tags: ["Kubernetes", "kubeadm"]
categories: ["DevOps"]
---
kubeadm 是 Kubernetes 官方提供的用于快速部署 Kubernetes 集群的工具，本文主要记录自己通过 kubeadm 部署 Kubernetes 过程中遇到的问题和踩过的坑，便于自己和他人查阅。

<!--more-->

### 准备

k8s至少需要一个 `master` 节点和一个 `node` 节点才能组成一个可用的集群。

本文搭建一个 `master` 节点和一个 `node` 节点。

* 主机环境：Mac OS Sierra
* 虚拟机环境：`CentOS 7.5`
* 虚拟机工具 `vagrant` + `VirtualBox`

创建两台虚拟机，IP和主机名分别如下：
```
10.0.0.100 k8s-master
10.0.0.111 k8s-node1
```
服务器系统均为 `CentOS/7`

> 使用 vagrant 创建虚拟机可以参考 [Mac OS 使用 Vagrant 管理虚拟机（VirtualBox）](http://www.cnblogs.com/xishuai/p/macos-use-vagrant-with-virtualbox.html)

注意：Kubernetes 几乎所有的安装组件和 Docker 镜像都放在 google 自己的网站上，这对国内的同学可能是个不小的障碍。建议是：网络障碍都必须想办法克服，不然连 Kubernetes 的门都进不了。

参考文章
* [Shadowsocks + Privoxy 搭建 http 代理服务](https://blog.csdn.net/qianghaohao/article/details/80383434)
* [centos7 docker使用https_proxy 代理配置](https://my.oschina.net/tinkercloud/blog/638960)

### 系统配置

#### 修改主机名

分别使用 `hostname` 把主机名称设置为 `k8s-master` 和 `k8s-node1`
```sh
hostnamectl set-hostname k8s-master
hostnamectl set-hostname k8s-node1
```
然后修改每一台主机的 hosts
```sh
vim /etc/hosts
# 加入以下内容
10.0.0.100 k8s-master
10.0.0.101 k8s-node1
```

#### 开放端口

如果各个主机启用了防火墙， 需要开放 Kubernetes 各个组件所需要的端口，可以查看 [Installing kubeadm](https://kubernetes.io/docs/setup/independent/install-kubeadm/#check-required-ports)中的“Check required ports”一小节。这里为了方便起见在各个节点直接禁用防火墙：
```sh
systemctl stop firewalld
systemctl disable firewalld
```

> 如果在生产环境，不建议直接关闭防火墙。只需要把需要的端口开放即可。

#### 禁用 SELINUX

目的是让容器可以读取主机文件系统

**临时方法**
```sh
setenforce 0
```
**永久方法**
```sh
vim /etc/selinux/config
# 修改以下内容后，重启服务器
SELINUX=disabled
```

#### 开启 iptables

创建 `/etc/sysctl.d/k8s.conf` 文件并添加相关内容
```sh
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```
然后执行下面命令即可
```sh
sysctl --system
```

> 实际可以参考官方文档[Installing kubeadm, kubelet and kubectl](https://kubernetes.io/docs/setup/independent/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl)一小节中的内容

#### 关闭系统 swap

Kubernetes 1.8 开始要求关闭系统的Swap，如果不关闭，默认配置下 kubelet 将无法启动。
可以通过 kubelet 的启动参数 `--fail-swap-on=false` 更改这个限制。

**方法一**

此方法是直接关闭系统的 Swap

首先执行
```sh
swapoff -a
```
然后修改 `/etc/fstab` 文件，注释掉 SWAP 的自动挂载，防止机子重启后 swap 启用
```sh
vi /etc/fstab
```
输出如下：
```sh

#
# /etc/fstab
# Created by anaconda on Wed Feb 28 17:48:43 2018
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/VolGroup00-LogVol00 /                       xfs     defaults        0 0
UUID=df1e67ab-46b0-4b49-a861-e7d10736d467 /boot                   xfs     defaults        0 0
/dev/mapper/VolGroup00-LogVol01 swap                    swap    defaults        0 0
```
注释 swap 这一行后保存退出
```sh
# /dev/mapper/VolGroup00-LogVol01 swap                    swap    defaults        0 0
```
确认 swap 是否已经关闭，执行命令
```sh
free -m
```
输出如下：
```sh
[root@master vagrant]# free -m
              total        used        free      shared  buff/cache   available
Mem:           1838         670         134           9        1033         946
Swap:             0           0           0
```
swap为0说明已经关闭。

接着调整 `/etc/sysctl.d/k8s.conf` 配置文件的 `swappiness` 参数
```sh
vi /etc/sysctl.d/k8s.conf
# 添加下面的内容
vm.swappiness=0
```
最后执行
```sh
sysctl -p /etc/sysctl.d/k8s.conf
```
让修改生效。

**方法二**

由于主机上可能还需要运行其他的服务，因此直接关闭swap可能会对其他服务产生影响。
可以通过修改 kubelet 的启动参数来去掉这个限制。
```sh
vi /etc/sysconfig/kubelet
```
加入以下内容
```sh
KUBELET_EXTRA_ARGS=--fail-swap-on=false
```


### 安装Docker

#### CentOS

> Kubernetes 1.12已经针对Docker的1.11.1, 1.12.1, 1.13.1, 17.03, 17.06, 17.09, 18.06等版本做了验证，需要注意 Kubernetes 1.12 最低支持的Docker版本是 1.11.1。

```sh
# 首先确保 docker 环境干净
sudo yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate
sudo yum remove docker-logrotate docker-selinux docker-engine-selinux docker-engine
# 管理 yum 源
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# 添加 yum 源
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 更新 yum 索引
sudo yum makecache fast
# 安装 docker
$ yum install docker-ce
# 启动 docker
$ systemctl start docker
# 将 docker 添加到开机启动服务
$ systemctl enable docker
```

如果需要安装特定的 Docker 版本，执行

```sh
# 查找指定 Docker-CE 的版本
yum list docker-ce.x86_64 --showduplicates | sort -r
# 安装指定版本的Docker-CE
sudo yum -y install docker-ce-[VERSION]
```

#### Ubuntu

```sh
# 安装系统工具
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
# 安装GPG证书
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
# 写入软件源信息
sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
# 更新并安装 Docker-CE
sudo apt-get -y update
sudo apt-get -y install docker-ce
# 查找指定版本
apt-cache madison docker-ce
```

#### 可能遇到的问题

##### 执行 `yum install docker-ce` 出现 `[Errno 256] No more mirrors to try`

执行以下命令
```sh
$ yum clean all
$ yum makecache fast
```

##### `Package docker-ce-selinux is obsoleted by docker-ce, but obsoleting package does not provide for requirements`

错误原因：
> on a new system with yum repo defined, forcing older version and ignoring obsoletes introduced by 17.06.0

通过执行以下命令可以解决该问题
```sh
$ yum install --setopt=obsoletes=0 docker-ce-17.03.2.ce docker-ce-selinux-17.03.2.ce
```

### 使用 kubeadm 部署

#### 安装kubeadm，kubelet和kubectl

**非国内源安装**

如果你的网络能够访问 `packages.cloud.google.com`，可以采用该种方式进行安装

```sh
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
systemctl enable kubelet
```

> 运行 `kubelet --help` 会发现之前大多数的命令行参数配置都已经 DEPRECATED 了。
>
> 目前官方推荐使用 --config 来指定配置文件，并在配置文件中指定相关内容。具体内容可以参考 [Set Kubelet parameters via a config file](https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/).

**国内源安装（阿里云）**

```sh
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
        http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
systemctl enable kubelet
```

#### 准备镜像

由于 Kubernetes 的安装过程中，相关组件的镜像都是托管在 Google Container Registry 上的。国内镜像加速通常都是针对 DockerHub 来的，因此国内的服务器无法直接通过 gcr 来安装镜像。

因此最好的解决方法就是在可访问 `gcr.io/google-containers` 的国外节点中，把 gcr.io 的镜像拉到可访问的镜像仓库中，比如 DockerHub 或者阿里云等等。

##### 创建仓库

本文使用阿里云的镜像仓库作为示例。
阿里云镜像仓库地址：`https://cr.console.aliyun.com/cn-shanghai/repositories`

创建步骤：
1. 点击创建镜像仓库按钮
2. 填写相关的表单内容
    ![repo.jpg](https://i.loli.net/2018/11/22/5bf66cb9dfe2d.jpg)
3.选择本地仓库

这样我们就能够成功创建一个我们自己的公开的镜像仓库。

地址类似这样：
```
registry.cn-shanghai.aliyuncs.com/cxyfreedom/k8s
```

登录方式：
```
docker login --username=<username> registry.cn-shanghai.aliyuncs.com
```

登录成功的话，会返回 `Login Succeeded`

##### 获取版本号

获取每个版本的镜像版本号可以通过 [Overview of kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm/#custom-images) 所写内容来调整自己脚本中的一些参数。

比如我现在要获取 `1.12.2` 版本的所有相关镜像可以通过如下命令：
```sh
kubeadm config images list
```
输出结果为：
```sh
k8s.gcr.io/kube-apiserver:v1.12.2
k8s.gcr.io/kube-controller-manager:v1.12.2
k8s.gcr.io/kube-scheduler:v1.12.2
k8s.gcr.io/kube-proxy:v1.12.2
k8s.gcr.io/pause:3.1
k8s.gcr.io/etcd:3.2.24
k8s.gcr.io/coredns:1.2.2
```

##### 准备镜像仓库

**方法一：利用脚本推送镜像**

在国外节点上面创建一个脚本
```sh
vim push.sh
```
内容如下：
```sh
#!/bin/bash
set -o errexit
set -o nounset
set -o pipefail

KUBE_VERSION=v1.12.2
KUBE_PAUSE_VERSION=3.1
ETCD_VERSION=3.2.24
DNS_VERSION=1.2.2

GCR_URL=gcr.io/google-containers
ALIYUN_URL=registry.cn-shanghai.aliyuncs.com/cxyfreedom

images=(kube-proxy-amd64:${KUBE_VERSION}
kube-scheduler-amd64:${KUBE_VERSION}
kube-controller-manager-amd64:${KUBE_VERSION}
kube-apiserver-amd64:${KUBE_VERSION}
pause:${KUBE_PAUSE_VERSION}
etcd-amd64:${ETCD_VERSION}
coredns:${DNS_VERSION})

for imageName in ${images[@]}
do
  docker pull $GCR_URL/$imageName
  docker tag $GCR_URL/$imageName $ALIYUN_URL/$imageName
  docker push $ALIYUN_URL/$imageName
  docker rmi $ALIYUN_URL/$imageName
done
```

成功运行脚本后，在我们自己的阿里云镜像仓库就存在了对应的包。

**方法二：使用别人已经做好的镜像仓库**

国内目前做的比较好的 k8s 的镜像仓库是安家的。

[Google Container Registry(gcr.io) 中国可用镜像(长期维护)](https://anjia0532.github.io/2017/11/15/gcr-io-image-mirror/)
[安家的Github](https://github.com/anjia0532/gcr.io_mirror)
[DockerHub仓库](https://hub.docker.com/u/anjia0532/)

#### 获取镜像

在 master 节点上面创建一个脚本
```sh
vim pull.sh
```
内容如下：
```sh
#!/bin/bash
KUBE_VERSION=v1.12.2
KUBE_PAUSE_VERSION=3.1
ETCD_VERSION=3.2.24
DNS_VERSION=1.2.2
username=registry.cn-shanghai.aliyuncs.com/cxyfreedom

images=(kube-proxy-amd64:${KUBE_VERSION}
kube-scheduler-amd64:${KUBE_VERSION}
kube-controller-manager-amd64:${KUBE_VERSION}
kube-apiserver-amd64:${KUBE_VERSION}
pause:${KUBE_PAUSE_VERSION}
etcd-amd64:${ETCD_VERSION}
coredns:${DNS_VERSION}
    )

for image in ${images[@]}
do
    docker pull ${username}/${image}
    docker tag ${username}/${image} k8s.gcr.io/${image}
    #docker tag ${username}/${image} gcr.io/google_containers/${image}
    docker rmi ${username}/${image}
done
```

执行脚本成功以后,通过 `docker images` 可以看到需要的镜像应该都正常被拉取下来。

#### 使用 kubeadm init 初始化集群（只在主节点执行）

首先需要确保在主节点环境中没有设置 `http_proxy` 和 `https_proxy`

然后执行以下命令
```sh
kubeadm init --kubernetes-version=v1.12.2 --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=<host ip> --ignore-preflight-errors=Swap --token-ttl 0
```

常见 Optional
```sh
--apiserver-advertise-address: Api server 服务 IP 位址
--apiserver-bind-port: Api server 服务端口号，默认 6443
--pod-network-cidr: Pod 的虚拟 IP 范围 (CIDR)
--kubernetes-version: 指定 Kubernetes 版本，建议 1.6+
```

因为选择 flannel 作为 Pod 的网络插件，所以在命令中需要指定 `--pod-network-cidr=10.244.0.0/16`。

`--apiserver-advertize-address`是apiserver的通信地址，一般使用的是 master 节点的 IP 地址。

初始化成功以后，会提供节点加入 k8s 集群的 token。默认 token 的有效期是 24 小时。过期后需要重新生成，比较麻烦，设置 `--token-ttl 0` 表示永不过期。

初始化成功示例
```
Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join 10.0.0.100:6443 --token q5egpf.zjw0a7bgnd11dw6d --discovery-token-ca-cert-hash sha256:29118e58fcd32d3234fd53584d376c05c7d36d4e1be011dbac92260cbd007963
```

#### 可能遇到的问题

##### 卡在 preflight/images

由于在 init 的过程中，k8s 会自动从 google 服务器中拉取相关的 docker 镜像。首先会去检查代理服务器，确定和 kube-apiserver 的 https 连接方式，如果有代理，会发出警告。

如果一直卡在 preflight/images
```
[preflight/images] Pulling images required for setting up a Kubernetes cluster
[preflight/images] This might take a minute or two, depending on the speed of your internet connection
[preflight/images] You can also perform this action in beforehand using 'kubeadm config images pull'
```
可以尝试上面小节的方式，事先准备好相关的镜像。

##### WARNING HTTPProxyCIDR

如果遇到如下的警告
```
 [WARNING HTTPProxy]: Connection to "https://10.0.0.10" uses proxy "http://127.0.0.1:8118". If that is not intended, adjust your proxy settings
    [WARNING HTTPProxyCIDR]: connection to "10.96.0.0/12" uses proxy "http://127.0.0.1:8118". This may lead to malfunctional cluster setup. Make sure that Pod and Services IP ranges specified correctly as exceptions in proxy configuration
    [WARNING HTTPProxyCIDR]: connection to "10.244.0.0/16" uses proxy "http://127.0.0.1:8118". This may lead to malfunctional cluster setup. Make sure that Pod and Services IP ranges specified correctly as exceptions in proxy configuration
```
那么需要设置 `no_proxy`。
执行 `vim /etc/profile`，在最后添加
```sh
export no_proxy='127.0.0.1,10.0.0.100,k8s.gcr.io,10.96.0.0/12,10.244.0.0/16'
```
最后执行 `source /etc/profile` 使命令生效

##### failed to read kubelet config file "/var/lib/kubelet/config.yaml"

如果遇到这种问题，大多数情况是因为在 `kubeadm init` 之前就运行了 `systemctl start kubelet`。

解决方法就是在成功运行 `kubeadm init` 之后，再运行 `systemctl start kubelet`。

##### cni config uninitialized

```
Unable to update cni config: No networks found in /etc/cni/net.d
KubeletNotReady runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:docker: network plugin is not ready: cni config uninitialized
```
如果遇到上述错误，原因是因为 kubelet 配置了 `network-plugin=cni` ，但是还未安装，所以状态是 NotReady，也有可能是国内节点无法获取 flannel 这个镜像，如果不想看到这个报错或者不需要该网络，可以通过修改配置的方式或者提前拉取镜像。

**方法一：拉取镜像（推荐）**

在国外节点上面创建一个脚本
```sh
vim push_pod.sh
```
内容如下：
```sh
#!/bin/bash
set -o errexit
set -o nounset
set -o pipefail

ALIYUN_URL=registry.cn-shanghai.aliyuncs.com/cxyfreedom

docker pull quay.io/coreos/flannel:v0.10.0-amd64
docker tag quay.io/coreos/flannel:v0.10.0-amd64 ${ALIYUN_URL}/flannel:v0.10.0-amd64
docker push ${ALIYUN_URL}/flannel:v0.10.0-amd64
docker rmi quay.io/coreos/flannel:v0.10.0-amd64
docker rmi ${ALIYUN_URL}/flannel:v0.10.0-amd64
```

执行 `sh push_pod.sh`

然后在国内节点也创建一个脚本
```sh
vim pull_pod.sh
```
内容如下
```sh
#!/bin/bash
set -o errexit
set -o nounset
set -o pipefail

ALIYUN_URL=registry.cn-shanghai.aliyuncs.com/cxyfreedom

docker pull ${ALIYUN_URL}/flannel:v0.10.0-amd64
docker tag ${ALIYUN_URL}/flannel:v0.10.0-amd64 quay.io/coreos/flannel:v0.10.0-amd64
docker rmi ${ALIYUN_URL}/flannel:v0.10.0-amd64
```

执行 `sh pull_pod.sh`

**方法二：修改配置文件**

执行
```sh
vim /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```
删除最后一行中的 `$KUBELET_NETWORK_ARGS`。
最后，重启 kubelet 并重新初始化
```sh
systemctl enable kubelet && systemctl start kubelet
kubeadm reset
kubeadm init --kubernetes-version=v1.12.2 --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=<host ip> --token-ttl 0
```

##### net/http: TLS handshake timeout

如果比如请求 `kubeadm version` 告诉端口无法连接、超时或者拒绝连接等。一般来说是因为我们设置了 `http_proxy` 和 `https_proxy` 做代理翻墙导致无法访问自身和内网。

执行 `vim /etc/profile`，在最后添加
```sh
export no_proxy='127.0.0.1,10.0.0.10,k8s.gcr.io,10.96.0.0/12,10.244.0.0/16'
```
如果有多个集群，需要把集群中每一个节点的IP也添加上去。

最后执行 `source /etc/profile` 使命令生效。

如果还是失败，可以重新进行初始化

简便
```sh
kubeadm reset
kubeadm init
```

完整
```sh
kubeadm reset
ifconfig cni0 down
ip link delete cni0
ifconfig flannel.1 down
ip link delete flannel.1
rm -rf /var/lib/cni/
```

关于初始化中的错误可以另外起一个终端来进行跟踪
```sh
journalctl -f -u kubelet.service
```

#### 正确输出

如果 `kubeadm init` 后没有出错，正常情况下会返回类似如下结果
```sh
Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join 10.0.0.10:6443 --token 2g0sf1.17rr8z5kvrsg12ha --discovery-token-ca-cert-hash sha256:69c0763f251e09db08a650ae2c3677f501ebbb7a9fed8961b8bf376a4c8c8ed2
```

关键点：
* 生成的 token 需要记录下来，后续使用 `kubeadm join` 往集群中添加节点需要使用
* 下面的命令是为了配置常规用户如何使用 kubectl 来访问集群
```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### 安装 Pod Network（只在主节点执行）

执行
```sh
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
kubectl apply -f kube-flannel.yml
```

可以通过 cat 来查看 kue-flannel.yml 这个文件里的 flannel 的镜像版本
```sh
cat kube-flannel.yml|grep quay.io/coreos/flannel
```

如果 Node 有多个网卡的话，目前需要在 `kube-flannel.yml`  中使用 `—iface` 参数指定集群主机内网网卡的名称，否则可能会出现 dns 无法解析。需要将 `kube-flannel.yml` 下载到本地，flanneld 启动参数加上 `—iface=`
```sh
containers:
      - name: kube-flannel
        image: quay.io/coreos/flannel:v0.10.0-amd64
        command:
        - /opt/bin/flanneld
        args:
        - --ip-masq
        - --kube-subnet-mgr
        - --iface=eth1
```

使用 `kubectl get pod --all-namespaces -o wide` 确保所有的 Pod 都处于 Running 状态。
如果有状态错误的 Pod，可以通过 `kubectl describe pod <pod-name> -n kube-system` 来查看错误原因。

#### master 节点参与工作负载

默认情况下 master 节点不能被用来部署 pod。如果想让 master 节点也能够部署 pod，执行以下命令
```sh
kubectl taint nodes <node name> <node label>:NoSchedule-
```

示例
```sh
$ kubectl get nodes --show-labels
NAME                 STATUS     ROLES     AGE       VERSION   LABELS
k8s-master   Ready    master   16m   v1.12.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=k8s-master,node-role.kubernetes.io/master=
$ kubectl taint nodes k8s-master,node-role.kubernetes.io/master:NoSchedule-
```

#### 从集群中移除 Node

假设现在集群中有一个节点叫做 `node1`，在 master 节点上执行：
```sh
kubectl drain node1 --delete-local-data --force --ignore-daemonsets
kubectl delete node node1
```

然后在 `node1` 上执行：
```sh
kubeadm reset
ifconfig cni0 down
ip link delete cni0
ifconfig flannel.1 down
ip link delete flannel.1
rm -rf /var/lib/cni/
```

### 安装常用组件

#### 安装 dashboard 监控界面（只在主节点执行）

**所需要的镜像**

`k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.0`


执行命令
```sh
curl -O https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
kubectl create -f kubernetes-dashboard.yaml
```
如果执行成功，会输出以下内容
```sh
secret/kubernetes-dashboard-certs created
serviceaccount/kubernetes-dashboard created
role.rbac.authorization.k8s.io/kubernetes-dashboard-minimal created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard-minimal created
deployment.apps/kubernetes-dashboard created
service/kubernetes-dashboard created
```

由于网络原因，所以我们可以事先先把镜像来去下来，使用本地的镜像来部署。
在 `kubernetes-dashboard.yaml` 中找到 `image` 那一行，在下面添加
```sh
imagePullPolicy: Never
```
`imagePullPolicy` 可选值有：`Always`、`IfNotPresent`、`Never`。


查看 dashboard 的镜像是否正常运行
```sh
[root@master ~]# kubectl get pod --namespace=kube-system
NAME                                         READY     STATUS    RESTARTS   AGE
coredns-78fcdf6894-rt2qh                     1/1       Running   0          7h
coredns-78fcdf6894-zb4jh                     1/1       Running   0          7h
etcd-master.example.com                      1/1       Running   0          5h
kube-apiserver-master.example.com            1/1       Running   0          5h
kube-controller-manager-master.example.com   1/1       Running   0          5h
kube-flannel-ds-amd64-4vbqk                  1/1       Running   0          5h
kube-flannel-ds-amd64-fnkdr                  1/1       Running   0          7h
kube-proxy-97s6f                             1/1       Running   0          7h
kube-proxy-qkjrr                             1/1       Running   0          5h
kube-scheduler-master.example.com            1/1       Running   0          5h
kubernetes-dashboard-6948bdb78-grshl         1/1       Running   0          3m
```

##### 访问方式

**1. kubectl proxy**

如果需要本地访问 dashboard，执行 `kubectl proxy` 即可。然后通过 `http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy` 就能访问 dashboard 的 UI 了。

![k8s-dashboard.jpg](https://i.loli.net/2018/11/22/5bf66cbd8c4c7.jpg)

可以使用 `--address` 和 `--accept-hosts` 参数来允许外部访问：
```sh
kubectl proxy --address='0.0.0.0'  --accept-hosts='^*$'
```

> 备注：**虽然从其它地址可以访问到该界面，但是无法登录，因为 dashboard 只允许本地进行 HTTP 连接访问，其它地址只允许使用 HTTPS**。

**2. NodePort**

该方法只建议在开发环境进行使用

编辑 `kubernetes-dashboard.yaml`，将 `type: ClusterIP` 改为 `type: NodePort`。如果没有，则直接添加该语句。

通过下面的语句可以查看对外的端口
```sh
kubectl -n kube-system get service kubernetes-dashboard
```

>注意事项：由于证书问题，无法直接访问，需要在部署的同时指定有效的证书才可以访问。因此，在生产环境中，不建议使用 NodePort 的方式。

**3. API Server**

访问地址
```
https://<master-ip>:<apiserver-port>/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/
```

使用 `client-certificate-data` 和 `client-key-data` 来生成一个p12文件，执行命令如下：
```sh
# 生成client-certificate-data
grep 'client-certificate-data' ~/.kube/config | head -n 1 | awk '{print $2}' | base64 -d >> kubecfg.crt

# 生成client-key-data
grep 'client-key-data' ~/.kube/config | head -n 1 | awk '{print $2}' | base64 -d >> kubecfg.key

# 生成p12
openssl pkcs12 -export -clcerts -inkey kubecfg.key -in kubecfg.crt -out kubecfg.p12 -name "kubernetes-client"
```

>注意事项：对于生产系统，我们应该为每个用户应该生成自己的证书，因为不同的用户会有不同的命名空间访问权限。

##### 身份认证

登录 dashboard 的时候支持 kubeconfig 和 token 两种认证方式，kubeconfig 中也依赖 token，因此生成 token 是必不可少的。

**使用 kubeconfig**

默认的 kubeconfig 在 `$HOME/.kube/config` 下，也可以创建我们自己的 kubeconfig 文件。

> 生成的 kubeconfig 文件中没有 token 字段，需要在最后手动添加。

**创建 token**

需要创建一个用户并将 admin 角色进行绑定，这样就能赋予其管理员权限。

创建一个 yaml 文件叫做 `kubernetes-dashboard-rbac.yaml`（文件名可以随便取），内容如下：

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
```

然后执行以下命令即可：
```sh
kubectl create -f kubernetes-dashboard-rbac.yaml
```

最后获取我们需要的 token
```sh
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
```

### 参考资料

* [使用kubeadm安装Kubernetes 1.12](https://blog.frognew.com/2018/10/kubeadm-install-kubernetes-1.12.html#33-%E4%BD%BF%E7%94%A8helm%E9%83%A8%E7%BD%B2dashboard)
* [kubernetes-dashboard(1.8.3)部署与踩坑](https://www.cnblogs.com/RainingNight/p/deploying-k8s-dashboard-ui.html)
* [Kubernetes Dashboard中的身份认证详解](https://jimmysong.io/posts/kubernetes-dashboard-upgrade/)