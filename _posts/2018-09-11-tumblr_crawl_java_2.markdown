---
layout:     post
title:      "tumblr的Java爬虫--单用户版本(二)"
subtitle:   " \"有关图片与视频链接的处理\""
date:       2018-08-11
author:     "Aziz"
header-img: "img/in-post/tumblr_crawl_java/tumblr_crawl_java.png"
tags:
    - java
    - crawl
---

> “Java的爬虫工具之图片与视频链接获取”


## 前言

图片与视频链接是通过jsoup来解析获取的，有关jsoup的介绍，在上期博文中已经有了，下面就看看具体如何解析tumblr的博文

## 解析方式

### 确认本地环境可以访问tumblr

1. 你得确认本地已经启用shadowsocks，并且可以访问到www.tumblr.com（默认已经会使用shadowsocks，不会的移步教程[自己在墙上搭梯子的方式](https://seolin.github.io/2018/08/16/world_out_of_wall/)）
2. 查看设置shadowsocks的本地http代理，本教程默认地址为127.0.0.1，端口为1087
3. 使用以下代码测试本地java进程是否能通过代理的方式访问tumblr
```
public class JsoupProxyTumblrTest {
    public static void main(String[] args) {
        Proxy proxy = new Proxy(Proxy.Type.HTTP, new InetSocketAddress("127.0.0.1", 1087));
        String urlStr = "www.tumblr.com";
        Connection connection = Jsoup.connect(urlStr).proxy(proxy);}
        try {
            Document document = connection.get();
            System.out.println(document.toString());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

```
### 寻找一个用户，测试图片抓取
通过热门内容随意寻找一个图片，得到用户首页地址



