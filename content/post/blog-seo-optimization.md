---
title: Hexo 博客 SEO 优化
date: 2018-09-10T20:31:51+08:00
description: ""
tags: ["Hexo", "SEO"]
categories: []
---
为了能让我们的博客被搜索引擎更好的收录，我们需要对 SEO 进行优化。
<!--more-->

## 准备工作

### 安装插件

为了能够同时被 Google 和 Baidu 收录，我们需要两个插件
```sh
npm install hexo-generator-sitemap --save
npm install hexo-generator-baidu-sitemap --save
```

### 修改配置文件

修改<span id="inline-blue">站点配置文件</span>，将以下内容添加进去
```
sitemap:
  path: sitemap.xml
baidusitemap:
  path: baidusitemap.xml

url: https://reishin.me
```

添加完后执行 `hexo g`，然后就能在 `public` 目录下找到 `sitemap.xml` 和 `baidusitemap.xml`。

## 添加蜘蛛协议

在站点的 `source` 目录下创建 `robots.txt` 并添加以下内容：
```
User-agent: *
Allow: /
Allow: /archives/
Allow: /categories/
Allow: /tags/
Disallow: /vendors/
Disallow: /js/
Disallow: /css/
Disallow: /fonts/

Sitemap: https://reishin.me/sitemap.xml
Sitemap: https://reishin.me/baidusitemap.xml
```
<p id="div-border-left-green">域名需要替换成你自己的</p>

## 提交站点到 Google

1. 打开 [Google Search Console](https://search.google.com/search-console)，添加博客地址

2. 站点验证

    官方给出了多种验证的方式，我选择的是使用 `HTML` 标记的方法。将官方给出的代码复制到 `themes/next/layout/_partials/head/head.swig` 中
    ```
    <meta charset="UTF-8"/>
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=2"/>
    <meta name="theme-color" content="{{ theme.android_chrome_color }}">
    # 下面这行就是我们添加的内容
    <meta name="google-site-verification" content="xxxxxxxx" />
    ```

3. 测试 robots.txt

    选择左边菜单栏中的 `抓取 > robots.txt测试工具` 进行测试即可。

4. 提交站点地图

    选择左边菜单栏中的 `抓取 > 站点地图` ，在打开的页面中首先选择右上角的 `添加/测试站点地图` 按钮，测试通过后就可以提交。

5. Google 抓取

    选择左边菜单栏中的 `抓取 > Google 抓取工具` ，填写需要抓取的 URL，点击抓取，最后点击请求编入索引即可。这样我们的博客就成功提交到 Google。

## Baidu 主动提交链接方法

为了能够加快被百度收录，采用自动提交链接的方式。

1. 安装插件
    ```
    npm install hexo-baidu-url-submit --save
    ```

2. 站点验证

    验证方法和 Google 的类似。

3. 配置

    修改<span id="inline-blue">站点配置文件</span>，将以下内容添加进去
    ```
    baidu_url_submit:
      count: 1 // 代表提交最新的三个链接
      host: https://reishin.me // 在百度站长平台中注册的域名
      token: your_token // token
      path: baidu_sitemap.txt // 文本文档的地址

    deploy:
    - type: baidu_url_submitter
    ```
    最后部署的时候，就会自动提交。
