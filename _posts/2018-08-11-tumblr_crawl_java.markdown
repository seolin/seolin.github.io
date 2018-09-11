---
layout:     post
title:      "tumblr的Java爬虫基础描述--单用户版本(一)"
subtitle:   " \"这是一片有关利用Java爬取tumblr内容项目的程序简介\""
date:       2018-08-11
author:     "Aziz"
header-img: "img/in-post/tumblr_crawl_java/tumblr_crawl_java.png"
tags:
    - java
    - crawl
---

> “Java的爬虫工具 ”


## 前言

网络上有很多python的爬虫项目，但是个人偏爱Java（其实是懒得学，依旧被迫学了很多）写了一个Java的tumblr爬虫

## 项目介绍

### 简介
tumblr作为一个社交网站，上面有大量的图片以及视频，由于该网站在墙外，并且就速度来说明显的没有油管来的快，因此看tumblr是一件很痛苦的事情，
在这个前提下，准备做一个爬虫项目，爬取某个用户所有的图片与视频。

### 准备

+ 项目架构：
  * spring boot
+ 网页分析：
  * 名称：jsoup
  * 版本：1.11.2
+ 数据过滤：BloomFilter
+ socks代理：[shadowsocks](https://seolin.github.io/2018/08/16/world_out_of_wall/)

### 思路
1. 提供一个用户
2. 通过jsoup来分析网页（代理使用shadowsocks）
3. 提取网页中所有的图片或视频链接
4. 使用过滤器过滤已经存在的图片或视频链接
5. 将过滤后的图片或视频链接放入过滤器
6. 将过滤后的图片或视频链放入下载队列
7. 下载线程从中下载队列中取得图片或视频链接，并下载
8. 继续抓取下一页，重复步骤3，直到无法爬取到图片或视频链接

## 项目构建

### spring boot

spring boot或者spring cloud的教程推荐[翟永超大大](http://blog.didispace.com/)的教程，用这个的目的是为了后续的扩展（不知道有没有后续）。

spring boot 项目下载地址(SPRING INITIALIZR)[https://start.spring.io/]，构建包名，并添加web,lombok的依赖，点击下载，一个项目就完成了。

### jsoup

jsoup的教程可以查看[jsoup开发指南](http://www.open-open.com/jsoup/)，该项目提供了非常简单的Api，通过dom，css以及类似于jQuery的方式来取出和操作数据

### 数据过滤

使用BloomFilter进行大数据去重
+ 简介：布隆过滤器实际上是一个很长的二进制向量和一系列随机映射函数。
+ 原理：当一个元素被加入集合时，通过K个散列函数将这个元素映射成一个位数组中的K个点，把它们置为1。检索时，我们只要看看这些点是不是都是1就（大约）知道集合中有没有它了：如果这些点有任何一个0，则被检元素一定不在；如果都是1，则被检元素很可能在。
+ 优点：相比于其它的数据结构，布隆过滤器在空间和时间方面都有巨大的优势。布隆过滤器存储空间和插入/查询时间都是常数（O(k)）。而且它不存储元素本身，在某些对保密要求非常严格的场合有优势。
+ 缺点：一定的误识别率和删除困难。
结合以上几点及去重需求（容忍误判，会误判在，在则丢，无妨），决定使用BlomFilter。

### 流程图
![流程图](/img/in-post/tumblr_crawl_java/process.png)