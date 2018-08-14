---
layout:     post
title:      "tumblr的Java爬虫基础描述--多用户版本"
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
  * mybatis
+ 网页分析：
  * 名称：jsoup
  * 版本：1.11.2
+ 数据管理：mysql
+ 数据过滤：BloomFilter
+ socks代理：shadowsocks

### 思路
+ 提供一个起始用户
+ 用shadowsocks作为代理，通过jsoup来分析网页
+ 提取网页中所有的图片与视频，并查找下一页，直到页尾
+ 下载得到的用户图片与视频
+ 下载图片与视频到本地
+ 通过上一步得到的用户，继续重复该循环

