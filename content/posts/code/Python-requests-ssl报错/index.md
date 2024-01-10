---
title: Python requests ssl报错
date: 2019-04-16 23:23:02
categories:
    - CODE
tags:
    - Python
    - 爬虫
---
## 问题背景
* 这是遇到的一个天坑！ ~~这么过分的坑一定要写下来！！！！~~
* 项目需求：
    * 爬取一个位于国外的网站，所以需要使用proxy，这里使用socks5代理
    * 网站是https，所以需要ssl
* 就是在这的需求之下，代码持续爆出如下错误：
    ```
    requests.exceptions.SSLError: SOCKSHTTPSConnectionPool(host='www.xxx.org', port=443): Max retries exceeded with url: /file/ (Caused by SSLError(SSLError("bad handshake: SysCallError(-1, 'Unexpected EOF')")))

    requests.exceptions.ConnectionError: SOCKSHTTPSConnectionPool(host='www.xxx.org', port=443): Max retries exceeded with url: /file/ (Caused by NewConnectionError('<urllib3.contrib.socks.SOCKSHTTPSConnection object at 0x0000021C585105C0>: Failed to establish a new connection: [Errno 11001] getaddrinfo failed'))

    requests.exceptions.SSLError: HTTPSConnectionPool(host='msft.com', port=443): Max retries exceeded with url: / (Caused by SSLError("Can't connect to HTTPS URL because the SSL module is not available."))
    
    ......
    ```
* 本文就来好好踩踩这些坑

<!--more-->

---
## SSL module is not available
* 出现如上错误表明程序未能正确加载引用ssl

* 如果你是linux系统，linux系统使用源码编译出来的python在编译过程种没有设定编译ssl库，建议重新编译安装python。或者尝试通过pip安装openssl 

* 如果你是一个win下的conda用户，首先也可以尝试使用conda安装缺失的openssl库。
    * 如果openssl本来就已经存在了，但是代码还是出现如上报错，并且你是使用某些编辑器（列如vscode）编写python脚本的，可能是因为安装好的openssl由于缺少环境变量的支持无法被程序正确的调用，你可以通过配置环境变量，或者在conda的命令行里运行你编写的脚本


---
## bad handshake: SysCallError(-1, 'Unexpected EOF')

* 这是最坑的一个错误

* 错误的握手包。

* 新版本的openssl将有漏洞的cipher禁用了，使用TSL1.0一下的cipher都无法匹配。这在新版本的requests中也是如此。
解决方法有人说是降低requests版本，还有一个方法是在代理服务器地址写成如下格式
```
# socks5h://127.0.0.1:1080

r=requests.get(URL,proxies={"https":"socks5h://127.0.0.1:1080"})
```
    
~~对就是加了个h~~

* 加h的含义：
    ```
    socks5h:// socks4a://
    ```
    表示主机名由socks服务器解析;
    ```
    socks5:// socks4://
    ```
    表示主机名在本地解析
