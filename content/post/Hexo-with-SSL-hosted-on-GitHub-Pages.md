---
title: GitHub Pages 启用 SSL/TLS
date: 2016-08-25T03:14:00+08:00
description: ""
tags: ["SSL", "Github"]
categories:
---
通过本文，可以让 GitHub Pages 从 http 转变为 https，从而让数据更加安全。

<!--more-->
## 注册 CloudFlare

注册 CloudFlare 后，按照提示在 Add Websites 中添加自己网站的域名，然后点击 `Begin Scan`

通常情况一路默认即可。最后，CloudFlare 会分配两个 `NameServer` ，比如：

> *.ns.cloudflare.com
>
> *.ns.cloudflare.com



## 修改域名服务商的 NameServer

理论上可以添加多个 `NameServer`，但混用多个可能会导致一些问题，所以建议直接将原有的 NS 替换

为 CloudFlare 提供的 NS。例如 `f1g1ns1.dnspod.net` 和 `f1g1ns2.dnspod.net`。添加完毕后，等待

片刻，在 CloudFlare 中点击 `Recheck Nameservers`，如果激活成功，会提示 `Status: Active`。

到此为止，已经可以用 `https` 来访问你的网站。



## 强制网页跳转 HTTPS

现在我们需要手动在地址栏输入 `https` 才能访问我们加密的网站。为了能够访问网站的任何一个页面

时，都能够以加密方式进行访问，最为便捷的方式就是使用 CloudFlare 提供的 `Page Rules`。设置规

则类似于如下：

{{% figure class="center" src="/img/rules.png" alt="rules" %}}

## 参考链接

* [为 Github的 Hexo 博客启用 SSL/TLS](https://g2ex.github.io/2015/10/14/Hexo-with-SSL-Hosted-on-Github-Page/)
* [在 GitHub Pages上使用 CloudFlare https CDN](https://blog.chionlab.moe/2016/01/28/github-pages-with-https/)
* [再谈域名 DNS 服务器不能混用多个 NS 的问题](http://wangye.org/blog/archives/196/)
* [CloudFlare](https://www.cloudflare.com)