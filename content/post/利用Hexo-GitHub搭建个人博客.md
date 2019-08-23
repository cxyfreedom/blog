---
title: 利用 Hexo + GitHub 搭建个人博客
date: 2015-11-06T15:35:56+08:00
description: ""
tags: ["hexo", "Blog"]
categories: ["Blog"]
---
网上看到各式各样的博客，因此，自己也想拥有一个属于自己的独立博客。利用搜索引擎，最终选择了 Hexo + Github 的方式来搭建一个静态博客。搭建完成后，决定把自己从开始到搭建完成的整个过程给记录下来，也可以给以后搭建博客的人一个参考。

<!--more-->

## 安装前提

* [Git](http://git-scm.com/)
* [Node.js](https://nodejs.org/en/)

我是在Ubuntu平台上搭建的，所以就以Ubuntu下为例：
### 安装 Git

```shell
$ sudo apt-get install git-core
```

### 安装 Node.js

* 脚本安装

```shell
$ curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.31.2/install.sh | bash
# or
$ wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.31.2/install.sh | bash
```
* 手动安装

```shell
export NVM_DIR="$HOME/.nvm" && (
  git clone https://github.com/creationix/nvm.git "$NVM_DIR"
  cd "$NVM_DIR"
  git checkout `git describe --abbrev=0 --tags --match "v[0-9]*" origin`
) && . "$NVM_DIR/nvm.sh"

# 添加以下内容到~/.bashrc或者~/.profile或者~/.zshrc
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" # This loads nvm
```
安装完毕后，重启终端，然后执行以下命令
```
$ nvm install v6.2.2
$ nvm alias default v6.2.2
```

## 开始搭建博客

### 安装 Hexo

```shell
$ npm install -g hexo-cli
$ npm install hexo --save
```
如果上述命令无法安装，可以将官方源替换成[淘宝npm源](https://npm.taobao.org/)
```shell
$ npm install -g cnpm --registry=https://registry.npm.taobao.org
```
查看 Hexo 版本
```shell
$ hexo version
hexo: 3.2.2
hexo-cli: 1.0.2
os: Linux 3.13.0-32-generic linux ia32
http_parser: 2.7.0
node: 6.2.2
v8: 5.0.71.52
uv: 1.9.1
zlib: 1.2.8
ares: 1.10.1-DEV
icu: 57.1
modules: 48
openssl: 1.0.2h
```

### 配置 Hexo

```shell
$ hexo init <folder>
$ cd <folder>
$ sudo npm install
```

### 部署 Hexo

执行以下命令，启动本地服务器。然后在浏览器中输入 [http://localhost:4000](http://localhost:4000) 即可访问
```shell
$ hexo s    #启动本地服务器
```

## 博客部署到 GitHub

### 新建一个博客的项目仓库

`Repository name` 为`your_github_name.github.io`

### 配置本地的 SSH KEY 到 GitHub

```shell
$ ssh-keygen -t rsa -C "your email address" # 默认生成的文件名为id_rsa和id_rsa.pub
# or
$ ssh-keygen -t ras -C "your email address" -f "file_name" # 生成的文件名就为file_name和file_name.pub
```
然后在GitHub上新建一个SSH KEYS，把 *.pub 的内容拷贝进去。可以使用以下命令来测试是否设置成功
```shell
$ ssh -T git@github.com
```

### 把本地 Hexo 部署到 GitHub
首先，需要修改站点的配置文件 `_config.yml`, 在文件中添加
```
deploy:
  type: git
  repository: https://github.com/cxyfreedom/cxyfreedom.github.io.git
  branch: master
```

然后执行以下命令，就能够很容易把 Hexo 部署到 GitHub 上面
```shell
$ hexo clean    #清除缓存
$ hexo g    # 生成静态文件
$ hexo d    # 部署网站
```
正常情况下，稍微等一会儿，输入`XXX.github.io`就能看到搭建好的博客了～

## 绑定独立域名

1. 购买独立域名
域名购买我是选择通过 [Godaddy](https://www.godaddy.com/), 找个优惠码注册，购买，一般没有什么问题

2. 域名解析
因为Godaddy的关系，我把域名解析移到了国内，我选择了 [DNSPOD](https://www.dnspod.cn/), 目前来说还是比较稳定的

3. 绑定域名
在你博客根目录下的 `source` 中，新建一个 `CNAME` 文件。在文件中填写你的域名。例如
> reishin.me

最后部署到 GitHub 上面

## 博客同步和备份

由于我们可能会在不同的 PC 上进行文章的编写，那么自然而然的我们需要同步我们的博客。
因为有些配置文件涉及到一些隐私数据，所以为了安全起见，我选择了用 `bitbucket` 上建立一个私有仓库，然后把整个 Hexo 的文件用 git 来备份到仓库。因为 Hexo 的主题文件默认是不会同步的，为了便于博客更好的同步更新，我另外使用了一个 `git-backup` 的插件，这个插件可以备份整个博客，包括主题。

1.新建仓库并配置 SSH KEY 到 bitbucket 上

2.安装插件

```shell
$ npm install hexo-git-backup --save    # install base on 3.x.x
```

3.配置文件

在根目录下的 `_config.yml` 添加以下内容

```
backup:
    type: git
    theme: `theme`
    message: `commit的内容`
    repository:
        bitbucket: git@bitbucket.org:xxx/xxx.git,branchName
```

4.备份博客

```shell
$ hexo b
```

5.同步博客

```shell
# 依次安装 git 和 nodejs
$ npm install -g hexo-cli
$ git clone git@bitbucket.org:cxyfreedom/hexo-blog.git
$ cd hexo-blog
$ npm install hexo --save
$ npm install
```

## 总结

第一次搭建博客的确花了不少时间，虽然搭建的过程大同小异，但是自己也遇到了很多配置上面的问题，所以只有耐心尝试，才能真正的搭建自己的博客。相比其他搭建博客的方式，Hexo + GitHub 这种方式的确更加简便，可塑性也非常高。

目前还只是初步搭建完成，还有很多细节的地方需要完善。博客也只是个平台，要坚持写文章才能让博客有意义，现在还只是个开始。

这篇文章只是大致描述了搭建的过程，如果能够对你有帮助，真是再好不过了～
欢迎相互交流～

## 参考链接

[1] [如何搭建一个独立博客——简明 Github Pages与 jekyll 教程](http://cnfeat.com/blog/2014/05/10/how-to-build-a-blog/)
[2] [Hexo 静态博客使用指南](http://www.jianshu.com/p/73779eacb494#)
[3] [利用 git 解决 hexo 博客多PC间同步问题](http://chitanda.me/2015/06/18/hexo-sync-in-multiple-pc/)
[4] [使用 hexo，如果换了电脑怎么更新博客？](http://www.zhihu.com/question/21193762)
[5] [Hexo 官网](http://hexo.io/)

## 更新日志

* [2016.01.15] 增加一些步骤的细节和同步的方法
* [2016.07.04] 更新主题版本，修正一些细节