---
layout:     post
title:      "自己在墙上搭梯子的方式"
subtitle:   " \"Virtual Private Network Unit \""
date:       2018-08-16
author:     "Aziz"
header-img: "img/in-post/world_out_of_wall/world_out_of_wall.jpg"
tags:
    - 梯子
---

> “翻墙梯”


## 前言

网上有很多类似的教程，这里只是类似于重复造轮子的一个总结，本片实战环境是centOS 7(但是推荐centOS 6)

## 内容概述
+ 购买服务器
+ 安装ss服务
+ 下载客户端试用
+ 启用锐速加速
+ 启用多用户管理（可以跳过）

## 具体过程

以下相应工具支持centOS 6，7，Debina，Ubuntu，最好不要使用centOS 7(但是笔者实战过的就是centOS 7，拥有许多的坑)，用centos 6是比较好的，教程也比较多

### 1.购买服务器
+ [vultr](https://www.vultr.com/)
+ [搬瓦工](http://banwagong.cn/)
+ [hostShare](http://www.hostshare.cn/)

### 2.安装ss服务

#### 适用环境
+ 系统支持：CentOS 6，7，Debian，Ubuntu
+ 内存要求：≥128M

#### 安装方法
用root用户登陆，运行以下命令
```
wget --no-check-certificate -O shadowsocks.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh
chmod +x shadowsocks.sh
./shadowsocks.sh 2>&1 | tee shadowsocks.log
```
然后根据引导输入相应的信息。
+ 安装完成后，脚本提示如下：
    ```
    Congratulations, Shadowsocks-python server install completed!
    Your Server IP        :your_server_ip
    Your Server Port      :your_server_port
    Your Password         :your_password
    Your Encryption Method:your_encryption_method
    
    Welcome to visit:https://teddysun.com/342.html
    Enjoy it!
    ```

#### 默认配置
+ 服务器端口：自己设定（如不设定，默认从 9000-19999 之间随机生成）
+ 密码：自己设定（如不设定，默认为 teddysun.com）
+ 加密方式：自己设定（如不设定，默认为 aes-256-gcm）
+ 备注：脚本默认创建单用户配置文件，如需配置多用户，安装完毕后参照下面的教程示例手动修改配置文件后重启即可。

#### 卸载方法
用root用户登陆，运行以下命令
```
./shadowsocks.sh uninstall
```

#### 配置文件路径
/etc/shadowsocks.json
+ 单用户：
```
{
    "server":"0.0.0.0",
    "server_port":your_server_port,
    "local_address":"127.0.0.1",
    "local_port":1080,
    "password":"your_password",
    "timeout":300,
    "method":"your_encryption_method",
    "fast_open": false
}
```
+ 多用户：
```
{
    "server":"0.0.0.0",
    "local_address":"127.0.0.1",
    "local_port":1080,
    "port_password":{
         "8989":"password0",
         "9001":"password1",
         "9002":"password2",
         "9003":"password3",
         "9004":"password4"
    },
    "timeout":300,
    "method":"your_encryption_method",
    "fast_open": false
}
```
#### 使用命令
+ 启动：/etc/init.d/shadowsocks start
+ 停止：/etc/init.d/shadowsocks stop
+ 重启：/etc/init.d/shadowsocks restart
+ 状态：/etc/init.d/shadowsocks status

### 客户端下载
+ windowsPC版本（官方）
https://github.com/shadowsocks/shadowsocks-windows/releases
+ mac版本
https://github.com/shadowsocks/ShadowsocksX-NG/releases
+ Android版本（官方）
https://github.com/shadowsocks/shadowsocks-android/releases
+ IOS版本
截止2018年8月17号，还能在中国appStore上下载到SsrConnectPro。
如果下载不到了可以参考：https://github.com/j-proxy/iossos，大概思路是通过pp助手安装以前版本的ss客户端

### 启用锐速加速
#### 概述
锐速原版是需要钱的，这里提供的是锐速破解版本的安装方式，不支持OpenVZ虚拟技术架构

#### 内核检查
锐速的安装对系统内核有一定的要求，目前支持的内核如下
+ CentOS-6.8：2.6.32-642.el7.x86_64
+ CentOS-7.2：3.10.0-327.el7.x86_64
+ CentOS：4.4.0-x86_64-linode63
+ Ubuntu_14.04：4.2.0-35-generic
+ Debian_8：3.16.0-4-amd64

#### 内核修改
如果内核不再上述范围内，则需要对内核进行修改
+ centOS 6 支持内核 2.6.32-504.3.3.el6.x86_64
```
uname -r #查看当前内核版本
rpm -ivh http://xz.wn789.com/CentOSkernel/kernel-firmware-2.6.32-504.3.3.el6.noarch.rpm
rpm -ivh http://xz.wn789.com/CentOSkernel/kernel-2.6.32-504.3.3.el6.x86_64.rpm --force
```
+ centOS 7 支持内核 3.10.0-327.el7.x86_64
```
rpm -ivh http://xz.wn789.com/CentOSkernel/kernel-3.10.0-229.1.2.el7.x86_64.rpm --force
rpm -qa | grep kernel #查看内核是否安装成功
```
+ 执行命令“rpm -qa | grep kernel”，查看内核是否安装成功

+ 重启，查看是否修改陈工
```
reboot #重启
uname -r #当前使用内核版本
```

#### 锐速破解版安装方法  
以root用户登陆，执行以下命令
```
wget -N --no-check-certificate https://github.com/91yun/serverspeeder/raw/master/serverspeeder.sh && bash serverspeeder.sh
```

#### 锐速破解版卸载方法
以root用户登陆，执行以下命令
```
﻿chattr -i /serverspeeder/etc/apx* && /serverspeeder/bin/serverSpeeder.sh uninstall -f
```

#### 锐速常用命令
+ service serverSpeeder start #启动
+ service serverSpeeder stop #停止
+ service serverSpeeder reload #重新加载配置
+ service serverSpeeder restart #重启
+ service serverSpeeder status #状态
+ service serverSpeeder stats #统计
+ service serverSpeeder renewLic #更新许可文件
+ service serverSpeeder update #更新

## 多用户管理
使用python写的工具：bsp（bsp是shadowsocks多用户、流量、限时管理接口）
### 安装

#### 查看系统是否安装git
```
git --version
```
如果出现未找到git命令
则运行
```
yum install git 
```
#### 安装bsp

执行
```
git clone https://github.com/edboffical/bsp.git
cd bsp
chmod 775 install
./install
```
运行 bsp --version 如果出现版本号则安装成功

#### 配置文件路径
/etc/byte/ss_byte.json
```
{
    "ssconf_path":"/etc/shadowsocks-python/config.json", ----shadowsocks多用户配置文件路径
    "ress_cmd":"/etc/init.d/shadowsocks-python restart", ----shadowsocks重启命令
    "update_time":"180",                                 ----bsp流量使用情况更新时间（ss重启/端口生效时间），单位s
    "port_limit": {                                      ----端口limit表
      "8989":"100"                                                                  
    }
}
```
#### 配置文件修改
如果你用的是本教程的shadowsocks教程，则将配置文件改成如下
```
{
    "ssconf_path": "/etc/shadowsocks.json",
    "ress_cmd": "/etc/init.d/shadowsocks restart",
    "update_time": "180",
    "port_limit": {
    }
}
```

#### 操作命令
+ 增加8989端口上网账户密码为123456限额100MB
bsp -p 8989 -P 123456 -s 100 -a -A -j
+ 删除命令
bsp -p 8989 -d -D -R

+ 从配置文件的limit表里遍历端口并增加规则（会将原有的流量信息清空）
bsp -r

+ 后台监听，自动删除对应账号和规则
bsp --start

#### 卸载
```
git clone https://github.com/edboffical/bsp.git
cd bsp
chmod 775 uninstall
./uninstall
```

#### centOS 7 的防火墙问题
centOS 7的默认防火墙使用的是firewall，但是bsp工具主要针对的是iptables命令的操作，所以需要变更防火墙

+ 直接关闭防火墙
```
systemctl stop firewalld.service #停止firewall
systemctl disable firewalld.service #禁止firewall开机启动
```
+ 设置iptables service
yum -y install iptables-services
+ 设置iptables service开启启动
systemctl restart iptables.service #重启防火墙使配置生效
systemctl enable iptables.service #设置防火墙开机启动
+ 最后重启系统

## 参考
+ 秋水逸冰python版本安装教程（需翻墙）：https://teddysun.com/342.html
+ shadowsocks客户端下载：https://shadowsocks.org/en/download/clients.html
+ 锐速加速教程：https://www.wn789.com/4678.html
+ 如何修改centOS的内核版本：https://www.wn789.com/4689.html
+ 多用户管理教程：https://github.com/edboffical/bsp
