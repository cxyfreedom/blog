---
title: Helm 使用教程
date: 2018-09-13T21:30:37+08:00
description: ""
tags: ["kubernetes", "Helm"]
categories: ["DevOps"]
---
Helm 是 Kubernetes 的一个包管理器，用来简化 Kubernetes 应用的部署和管理。类似于 CentOS 的 yum 工具 或者是 Ubuntu 的 apt-get 工具。
<!--more-->

## 概览

Helm 是 Kubernetes 的一个包管理器，用来简化 Kubernetes 应用的部署和管理。类似于 CentOS 的 yum 工具 或者是 Ubuntu 的 apt-get 工具。

Helm 的架构图

{{% figure class="center" src="/img/helm-arch.jpg" alt="helm-arch" %}}

关于 Helm 的详细介绍可以看一下 [Helm - Application deployment management for Kubernetes](https://www.slideshare.net/alexLM/helm-application-deployment-management-for-kubernetes)中的内容

**1. 三个基本概念**

* Chart：Helm的应用，包括应用所有的 manifest 模版
* Repository：存储仓库
* Release：Chart的部署实例

## 安装 Helm

<p id="div-border-left-red">注意：安装 Helm 之前首先需要安装并成功运行 Kubernetes。</p>

```sh
wget https://storage.googleapis.com/kubernetes-helm/helm-v2.10.0-linux-amd64.tar.gz
tar zxvf helm-v2.10.0-linux-amd64.tar.gz
mv liniux-amd64/helm /usr/local/bin
```
执行 `helm version` 能看到如下结果，说明客户端能够正常使用
```sh
Client: &version.Version{SemVer:"v2.10.0", GitCommit:"9ad53aac42165a5fadc6c87be0dea6b115f93090", GitTreeState:"clean"}
```

Tiller 是 Helm 的 server，由于国内节点无法直接访问 `gcr.io/kubernetes-helm/tiller`，因此这里使用国内阿里云的源来安装。

**首先配置 Tiller 的 RABC 权限**
```sh
vi rbac-config.yaml
```
输入以下内容：
```sh
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```
最后执行
```sh
kubectl create -f rbac-config.yaml
```

上述配置成功以后，执行以下命令即可完成安装
```sh
# 使用google官方源
helm init --service-account tiller --upgrade -i gcr.io/kubernetes-helm/tiller:v2.10.0
# 使用国内阿里云的源
helm init --service-account tiller --upgrade -i registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.10.0 --stable-repo-url https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
```
参数 `--stable-repo-url` 用于拉取 charts 所在的源位置。

Tiller 默认部署在 Kubernetes 集群中 kube-system 这个 namespace 下。
```sh
kubectl get pod -n kube-system -l app=helm
```

如果希望添加其他的 charts 地址，可以通过下面的方式来添加
```sh
# 格式
helm repo add <name> <url>
# 示例
helm repo add extra https://burdenbear.github.io/kube-charts-mirror/
```

## 常用命令

```sh
# 查询某个chart
helm search <name>
# 列出所有部署应用
helm list --all
# 删除实例
helm delete --purge <name>
# 删除 helm
helm reset
```

## 参考资料

* [官方文档](https://docs.helm.sh/using_helm/#role-based-access-control)
* [helm-安装与使用](https://blog.csdn.net/kozazyh/article/details/79537996)