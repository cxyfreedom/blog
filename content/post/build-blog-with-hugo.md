---
title: 使用 Hugo 搭建 Blog
date: 2019-08-23T22:14:05+08:00
lastmod: 2019-08-23T22:14:05+08:00
draft: false
keywords: ["Hugo", "Blog", "Github"]
description: ""
tags: ["Hugo", "Blog", "Github"]
categories: ["Blog"]
---
相比于 Hexo，Hugo 更加的轻量，从零部署也是十分简单。本文主要记录从 Hexo 迁移到 Hugo 的过程。
<!--more-->
## Hugo

### 安装Hugo

在 MacOS 下安装 Hugo 非常简单，执行下面的命令即可：
```sh
$ brew install hugo
```
验证安装是否成功：
```sh
$ hugo version
```

### 创建网站

```sh
$ hugo new site blog
```

### 目录结构

```
.
├── archetypes
├── config.toml
├── content
├── data
├── layouts
├── public
├── resources
├── static
└── themes
```

* archetypes: 存储 `Markdown` 的模板文件，优先级高于主题下的 `archetypes` 文件夹
* config.toml: 配置文件
* content: 存储所有的文章内容
* data: 存储数据文件让模板调用
* layouts: 存储 `.html` 文件的模板，优先级高于主题下的 `layouts` 文件夹
* static: 存储图片、css、js 等静态文件，优先级高于主题下的 `static` 文件夹
* themes: 存放主题

### 常用命令

```sh
# 创建新的文章
$ hugo new <file>
# 启动服务器生成静态文件
$ hugo server -D
# 启动服务器并关闭实时改动
$ hugo server --watch=false
```

## 配置

### 设置主题

我选用的是 `hugo-theme-even` 这个主题。使用方法：
```sh
$ cd blog
$ git init
$ git submodule add https://github.com/cxyfreedom/hugo-theme-even themes/even
$ echo 'theme = "even"' >> config.toml
```
为了便于修改主题，我 fork 了主题并关联到原仓库。
```sh
$ cd themes/even
$ git remote add upstream https://github.com/olOwOlo/hugo-theme-even.git
$ git fetch upstream
$ git merge upstream/master
```

该主题内置了五种颜色的主题，可以通过改变 `/src/css/_variable.scss` 中的 `$theme-color-config` 值来改变主题的颜色。

如果修改了主题的 `/src/` 目录下的任何文件，需要对主题进行重新编译。
```sh
$ cd ./themes/even/
# install dependencies
yarn install
# build
yarn build
```
如果想要删除一个子模块，执行下面的命令。
```sh
$ git submodule deinit themes/even
$ git rm themes/even
```

### URL 配置

比如从 `hexo` 迁移到 `hugo`，为了让文章链接保持一致，可以通过设置 `permalinks` 来适配 URL。
```toml
[permalinks]
  post = "/:filename/"
```
`permalinks` 支持多种不同的形式：
```toml
/:monthname/:day/:weekday/:weekdayname/:yearday/:section/:title/:slug/:filename/
```

## 部署

为了方便，没有使用 `Travis CI`，直接通过脚本的方式执行。其他的部署方式可以参考[官方文档](https://gohugo.io/hosting-and-deployment/)。
```sh
#!/bin/sh

# If a command fails then the deploy stops
set -e

printf "\033[0;32mDeploying updates to GitHub...\033[0m\n"

# Build the project.
hugo -t even # if using a theme, replace with `hugo -t <YOURTHEME>`

# Go To Public folder
cd public

# Add changes to git.
git init
git add .

# Commit changes.
msg="building site $(date)"
if [ -n "$*" ]; then
	msg="$*"
fi
git commit -m "$msg"

# Push source and build repos.
git push -f git@github.com:cxyfreedom/cxyfreedom.github.io.git master
```

## 参考

* [Host on GitHub](https://gohugo.io/hosting-and-deployment/hosting-on-github/)
* [favicon generator](https://realfavicongenerator.net/) even主题需要的文件使用该工具
* [Favicon & App Icon Generator](https://www.favicon-generator.org/)
* [子模块](https://www.git-tower.com/learn/git/ebook/cn/command-line/advanced-topics/submodules)
* [shortcodes](https://gohugo.io/content-management/shortcodes/#ref-and-relref)
* [官方文档](https://gohugo.io/documentation/)
* [Hugo 从入门到会用](https://blog.olowolo.com/post/hugo-quick-start/#%E6%A8%A1%E6%9D%BF%E9%80%89%E6%8B%A9%E9%A1%BA%E5%BA%8F)
