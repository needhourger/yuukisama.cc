---
title: 多终端更新hexo博客配置方法
date: 2019-01-02 17:27:22
categories: 
    - 网站搭建
    - 小站成长日记
tags: 
    - 网站搭建
    - 小站成长日记
---

## 概述

* 本文的基础都建立在小站成长日记系列的基础上。本教程默认你参照小站成长日记系列之前的教程完成了相关配置
* 博主本人电脑双系统，考虑到在不同设备终端更新博客的需求，于是有了下面这篇文章。
* **实现方法简述：** 使用git创建新的分支用来存放hexo博客本身
* 文末附kali下的小采坑

![瞎几把找的图片可爱就完事了](sample.jpg)

<!--more-->

---
## 开始

 **坑：** 如果你使用了三方主题，请把主题文件夹中的.git（可能为隐藏文件）文件夹删除。因为这个文件夹的存在会导致你后面推送的时候无法推送成功

1. 在你的博客目录下运行命令,初始化
    ```
    git init
    ```
2. 添加远程仓库
    ```
    //host 是你的远程仓库地址
    git add remote origin git@host:blog.git
    ```

3. 新建分支并切换到新建的分支
    ```
    git checkout -b 分支名
    ```
4. 接下来是git的基本操作，添加本地文件到git，以及提交
    ```
    git add *
    git commit -m "你的提交说明"
    ```

5. 将文件提交到你所创建的分支（这里我创建的分支名为hexo）
    ```
    git push origin hexo
    ```

## 在你的git服务器上配置公钥

* 建议把准备长期使用的设备配置。（当然你不把新设备的公钥加入git服务器你也无法推送）
* 在你的新设备上生成秘钥
    ```
    ssh-keygen -t rsa -C "your_email@email.com"
    ```
* 将生成的公钥，即～/.ssh/id_rsa.pub文件的内容复制到git服务器的～/git/.ssh/authorized_keys文件中
  
* 秘钥配置可以参见本博客另一篇文章 {% post_link VPS-nginx-hexo搭建个人博客 点击查看 %}

到这里你已经完成了秘钥配置，尝试在新设备上拉取hexo分之吧。


## 嗯，在新的设备上准备写博客
1. 首先拉取hexo分支到本地
    ```
    //host你的git仓库地址
    git clone -b hexo git@host:blog.git
    ```
2. 进入到克隆下来的文件夹内，安装相应依赖
    ```
    //进入文件夹
    cd blog
    //安装相应文件依赖
    npm install
    ```

    **当然这里要保证你新的设别上具有git以及node.js框架**

3. 博客写完之后进行的操作
    ```
    //博客的编译部署操作
    hexo clean && hexo g && hexo d

    //添加文件
    git add *
    //提交
    git commit -m "你的提交说明"
    //先拉取远程仓库的文件对比合并
    git pull origin hexo
    //解决版本冲突之后推送到hexo分支
    git push origin hexo
    ```
4. 在任意一台电脑上都别忘了拉取分支确保版本的一致性哦
    ```
    git pull origin hexo
    ```

## 小插曲
 **坑** 这里任意一台新设备也是需要安装好nodejs以及npm的。这里补充一个小插曲

* 博主电脑双系统是win+kali的组合

* kali这个小奇葩自带nodejs的，在命令行直接输入node -v是由回显的。然而输入npm提示命令不存在

* 于是乎四处寻找有关修复这个bug的方法（其实也不算bug）

* 网上多数教程linux下安装npm都是去nodejs官网下载安装。即使是直接面向kali的也是

* 但是其实apt的源中包含了npm，直接使用包管理安装即可
    ```
    //确保系统最新
    apt update && apt upgrade -y && reboot
    //安装缺少的npm
    apt install npm -y
    ```
    安装完成之后再尝试输入npm就有回显说明安装完成了。

