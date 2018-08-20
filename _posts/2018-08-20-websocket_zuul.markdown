---
layout:     post
title:      "websocket连接经过zuul网关的坑"
subtitle:   " \" 客户端的websocket连接进过zuul网关的坑 \""
date:       2018-08-16
author:     "Aziz"
header-img: "img/in-post/websocket_zuul/websocket_zuul.gif"
tags:
    - websocket
    - zuul
---

> “别用zuul网关来管理websocket”


## 前言

之前的项目中，使用zuul网关统一过滤域名，为了管理方便，想要尝试看看能不能使用zuul管理websocket连接。

## 现象描述
1. 高版本的websocket在第一次http请求后，使用的是更快速的tcp连接
2. zuul网关只能管理http请求，并且不支持tcp以及udp请求
3. websocket在经过zuul以后，就会降级会http请求（轮询的方式）

## 结论
最好是不要通过zuul来管理websocket连接，降级为轮询后，效率会降低很多。