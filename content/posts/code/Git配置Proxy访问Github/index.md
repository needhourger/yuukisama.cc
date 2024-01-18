---
title:          "Git配置Proxy访问Github"
subtitle:       ""
description:    ""
date:           2024-01-18T11:08:00+08:00
image:          ""
tags:           ["Wall"]
categories:     ["CODE"]
draft: false
---
# Git配置Proxy访问Github

## Start

一觉醒来突然发现针对Github的墙高了许多，公司新发的电脑刚安装好Archlinux还没配置Proxy,所以在此记录一下。

以下配置方案均在Linux系统上，发行版为Archlinux.配置Proxy的前提是学会天朝程序员的传统艺能：爬墙头。但在这篇文章里不会涉及这部分。

## First

Git工具主要支持两种协议
- HTTP/HTTPS
- SSH

接下来我们假定用于爬墙头的系统代理服务socks5开设在```localhost:10808```，http开设在```localhost:10809```

### HTTPS代理

Git的HTTP模式支持配置全局代理，且支持两种代理协议：HTTP协议以及socks5协议，可以使用如下命令配置：
```
# http 代理
git config --global http.proxy http://localhost:10809
git config --global https.proxy http://localhost:10809
# socks5 代理
git config --global http.proxy socks5://localhost:10808
git config --global  https.proxy socks5://localhost:10808
```

> --global参数为该设置为全局配置，如果没有该参数，且命令运行在git项目目录环境下，则此次配置仅针对该项目。

当然全局代理具有一定的局限性，因为在实际工作中除了Github还会使用很多其他的代码仓库，例如gitlab等等，并不是所有的仓库都需要走代理~~浪费流量~~

以下配置是针对Github的HTTP协议走代理
```
# http 代理
git config --global http.https://github.com.proxy http://localhost:10809
# socks5 代理
git config --global http.https://github.com.proxy socks5://localhost:10808
```

### SSH代理

Github的个人项目，以及不想每次推送代码都要输入账号密码的话肯定需要这个。

在使用ssh代理 之前，请确保你已经配置生成两ssh key,且key已经被添加到Github的settings中。

在```~/.ssh/config```文件中配置如下内容
```
Host github.com
  User git
  Hostname github.com
  IdentityFile "~/.ssh/id_rsa"
  TCPKeepAlive yes
  ProxyCommand nc -v -x localhost:10808 %h %p
```
nc即[netcat](https://zh.wikipedia.org/zh-sg/Netcat)号称网络瑞士军刀的一个小工具，如果没有需要安装一下。

但是需要注意的是在Archlinux的软件源中，netcat有两个版本，一个```openbsd-netcat```一个```gnu-netcat```，以上```ProxyCommand```中的命令仅仅适用于```openbsd-netcat```版本的netcat

---

如果是windows,ProxyCommand可能需要替换成如下样式.其中```connect```是一个Git在windows下都会自带的一个程序，如果提示connect找不到你可能需要指定connect的完整路径，亦或是配置环境变量。
```
Host github.com
  User git
  Hostname github.com
  IdentityFile "~/.ssh/id_rsa"
  TCPKeepAlive yes
  ProxyCommand connect -S localhost:10808 -a none %h %p
```

### Test

在配置完成之后可以尝试使用https协议clone一个项目，例如
```
git clone https://github.com/nginx/nginx.git
```

ssh协议可以使用如下命令测试
```
ssh -T git@github.com
Connection to github.com 22 port [tcp/ssh] succeeded!
Hi xxxx! You've successfully authenticated, but GitHub does not provide shell access.
```


