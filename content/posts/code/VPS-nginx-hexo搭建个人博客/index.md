---
title: VPS+nginx+hexo搭建个人博客
date: 2018-12-27 15:54:11
categories: 
    - CODE
tags: 
    - 建站
comments: true
---

## 开始前的碎碎念

### 为了搭建这个小站，搜索了很多资料，踩了很多坑。这里努力写点有用的东西防止后人踩坑。希望阅读的读者遇到问题时思考自己是否是符合本样例的情况。

这是一篇旨在使用linux服务器+nginx+hexo+git实现搭建个人博客的教学性质文章。

<!--more-->
## requires
* 一台服务器，本例子是linux centos,阿里的云服务器

    ```
    [root@host~]# lsb_release -a
    LSB Version:    :core-4.1-amd64:core-4.1-noarch
    Distributor ID: CentOS
    Description:    CentOS Linux release 7.6.1810 (Core)
    Release:        7.6.1810
    Codename:       Core
    ```
* 本地电脑：window 10
* nginx ; node.js ; hexo ; git ; npm
---
## 本地电脑
1. 安装git

    * windows [下载](https://git-scm.com/downloads)
        
    
    * linux使用包管理器安装git ~~(不了解包管理器的同学就不要看下去了吧)~~
        
        ```
        yum install git -y
        ```
    
    * 更多[Git基本配置工作](https://git-scm.com/book/zh/v1)


1. 安装node.js (node.js自带npm)

     [什么是npm，本垃圾觉得大概类似python的pip吧~~什么你不知道pip是啥？？？这是wiki打不开就算了~~](https://zh.wikipedia.org/wiki/Npm)
    * windows
    [下载](https://nodejs.org/zh-cn/download/)

    * linux
    下载对应版本的node.js以及安装操作
    [下载](https://nodejs.org/zh-cn/download/)

1. 安装hexo

    使用node.js的npm安装hexo，这里windows可以先设置一下npm的默认安装目录，本强迫症表示你默认装在c盘我很难受。打开命令行，运行:
    
    **坑：原教程里写了路径末尾加了node_modules，事实证明并不需要，这个命令会在目标目录生成一个node_modules。我这里设置默认安装路径在node.js本体的安装路径，因为这个路径里本身就有一个用于安装npm本身的nnode_modules文件夹。当然即使这样做了，npm运行缓存还是会存在c盘，需要继续修改设置的请移步[这里](https://www.jianshu.com/p/645c758d4428)**

    ```
    npm config set prefix "E:\node.js\"
    ```

    * 安装hexo
        ```
        npm install -g hexo-cli
        ```
        **坑：-g意为global，全局安装就会安装到你设置的安装目录里，没有这个参数默认会安装在你npm当时命令行运行的目录里创建node_moudels并进行安装**
    
1. 搭建博客
    
    **后续关于博客搭建的章节可以参照[官方教程](https://hexo.io/zh-cn/docs/setup)**

    * 创建一个文件夹用于做博客的目录
    * 切换到这个文件夹所在目录，运行命令：
        ```
        hexo init <folder name>
        ```
    * 进入该文件夹：
        ```
        npm install
        ```
    **坑：以上两步必不可缺，否则会产生缺少依赖问题，列入不生成静态文件（如x.html）的问题**

1. 配置你的站点：
    * 修改博客目录下的_congfig.yml文件[官方教程](https://hexo.io/zh-cn/docs/configuration)

1. 书写你的第一篇文章：
    
    * 在你的博客文件夹内运行命令

        ```
        hexo new "Hallow world"
        ```
    * 会在source文件夹内的默认_posts文件夹下生成一个md文件，接下来就可以参照markdown语法愉快的敲代码了

1. 生成你的网站：
    * 在你的博客文件夹内运行命令

        ```
        hexo generate
        ```
        或者使用快捷缩写
        ```
        hexo g
        ```

    * 在本地预览你的博客网站

        ```
        hexo server
        ```
        或者使用快捷缩写
        ```
        hexo s
        ```
    **坑：如果generate出错请检查博客文件夹下_config.yml内的配置是否正确，每一个冒号后面必须跟一个空格！！！**
    
    **坑：如果没能本地预览可能是因为缺少hexo-server，请运行npm install hexo-server --save**

1. 安装hexo-deployer-git
    
    安装该模块方便使用hexo最牛逼的推送功能

    ```
    npm install hexo-deployer-git --save
    ```
1. 本地配置git生成私钥以及公钥
    
    在git安装配置没有出错的情况下，环境变量完成配置，用户信息设置完全的情况下：
    ```
    ssh-keygen -t rsa -C "your_email@email.com"
    ```
    该命令会在c:\user\(你的电脑用户名)\.ssh文件加下生成密钥对。id_rsa文件为私钥，id_rsa.pub为公钥。
    **在后续的服务器配置过程中，我们会把公钥设置在服务器上，这样git推送的时候就可以免去密码认证**

---
## 服务器配置（linux）

1. 安装nginx
    ```
    yum install nginx -y
    ```

    你也可以使用lnmp一类的一键脚本安装web环境，这里不举例。

1. 配置nginx

    * 基本配置

        找到nginx的配置文件，默认在/etc/nginx/nginx.conf编辑内容

        基本只需要简单的修改默认配置中的网站根目录以及域名的设置，找到如下字段

        ```
        listen       80 default_server;     # 默认端口
        listen       [::]:80 default_server;
        server_name  _;                     # 你的域名，没有的话根目录即可
        root         /home/www/blog;        # 网站根目录,教程后续创建
        ```

    * 启动nginx：
        ```
        service nginx start
        service nginx reload
        ```

    * 设置开机自启：
        ```
        chkconfig nginx on
        ```
        或者
        ```
        systemctl enable nginx.servcice
        ```

1. 开放防火墙放行80端口

    * 运行
        ```
        iptables -A INPUT -p tcp --dport 80 -j ACCEPT
        ```
    * 保存防火墙设置
        ```
        service iptables save
        ```
        **坑：本人这条命令并没有执行成功，这个版本的linux似乎不支持这条命令，最后还是执行的service iptables reload**

    * 查看防火墙规则:
        出现这条内容意味着防火墙设置完成:
        ```
        [root@host ~]# iptables --list -n
        Chain INPUT (policy ACCEPT)
        target     prot opt source               destination
        ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:80
        ```
    **坑：云服务器供应商还会提供安全组一类的东西，请在那里面也放行对应端口**
    
1. 测试nginx是否配置成功

    * 服务器本地访问又返回结果：
        ```
        curl 127.0.0.1
        ```
    * 远程访问服务器网页端口，即在你的浏览器里输入服务器ip或者域名出现nginx默认页面即表示配置完成。

1. 配置git

    * 如果服务器没有git请参照上面安装git
    * 新建git用户,并设置用户密码

        ```
        useradd git
        passwd git
        ```
    * 新建网站目录,并修改网站根目录的用户拥有者为git用户

        ```
        cd /home
        mkdir www
        cd www
        mkdir blog
        chown git:git blog
        ```
    * 切换为git用户，进入其家目录创建.ssh文件夹，并进入
        ```
        su git
        cd ~
        mkdir .ssh
        cd .ssh
        ```
    * 创建authorized_keys文件，讲你本地电脑上的公钥内的文件内容复制到该文件内。

        ~~这里怎么操作自我发挥吧，把公钥文件传上来该跟名字，或者你开心就好~~
        
        设置密钥文件只读，文件夹权限:
        ```
        chmod 600 authorized_keys
        chmod 700 /home/git/.ssh
        ```
    ---
    * 配置ssh的配置文件/etc/ssh/sshd_config，修改如下词条

    ```
    RSAAuthentication yes       # 开启rsa密钥认证
    PubkeyAuthentication yes    # 开启公钥认证
    AuthorizedKeysFile  .ssh/authorized_keys    # 设置存贮文件
    ```
    ---
    * 在你的git用户家目录下新建你的博客仓库,并初始化为git裸仓库
        ```
        mkdir blog.git
        git init --bare blog.git
        ```

    * 配置裸仓库:进入仓库文件夹，在hooks文件夹内新建post_receive。文件写下一下内容

        ```
        #!/bin/bash
        git --work-tree=/home/www/blog --git-dir=/home/git/blog.git checkout -f
        ```
        这是一个linux shell规定了当接受到post请求之后的动作，把文件内容放到网站根目录里
    
    **在你的本地！！！** 测试ssh 免密码连接
        ```
        ssh -T git@your_host
        ```
        如果没有提示输入密码证明配置成功
    
---
## 回到本地电脑

1. 编辑blog下的配置文件_config.yml

    ```
    # Deployment
    ## Docs: https://hexo.io/docs/deployment.html
    deploy: 
     type: git
     repo: git@your_host:blog.git
     branch: master
    ```

1. 愉快的测试推送把！~ ~~或者死于bug error hhh~~
    
    ```
    hexo deploy
    ```    
    
    或者

    ```
    hexo d
    ```

**坑：不安装hexo-deployer-git是不可能推送的** 

**坑：如果访问页面出现cantnot GET/ 请检查主题设置，网上很多教程的设置基于hexo next主题，涉及到主题文件的请看你的主题的作者自己的文档，而不是看某dn** 


    

