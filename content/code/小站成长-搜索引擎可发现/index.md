---
title: Blog搜索引擎可发现
tags:
  - 网站搭建
date: 2018-12-27 23:53:12
categories:
    - CODE
---


## 摘要

* 本文介绍了如何让搜索引擎可以搜索本站，小站成长的一步~
* 搜索引擎检索至少需要你的网站有域名（不建议没有域名）因此国内就需要进行备案等等一系列的操作，如果尚未备案的可以完成备案之后再尝试。

![谷歌娘](baidu.jpg) 

<!--more-->


## 建立站点地图

1. 在本地博客文件夹路径内运行代码如下，安装站点地图生成插件:
    ```
    npm install hexo-generator-sitemap --save
    npm install hexo-generator-baidu-sitemap --save
    ```

    分别安装hexo站点地图生成插件，对应的分别是百度的站点地图生成以及Google粑粑的站点地图。

1. 构建网站，检查在public文件夹中是否产生sitemap.xml，baidusitemap.xml。存在表明站地图生成成功。

1. 在source文件夹下新建robots.txt文件 [什么是robots.txt](https://baike.baidu.com/item/robot.txt)

    ```
    User-agent: *
    Sitemap: http://aleonchen.com/sitemap.xml
    Sitemap: http://aleonchen.com/baidusitemap.xml
    ```

1. 重新构架并发布你的网站

    ```
    hexo g -d
    ```
## 向搜索引擎注册你的网站

* 备注：这里使用html的认证方法

    [Google 提交入口](https://www.google.com/webmasters/tools/home?hl=zh-CN)

    [百度提交入口](https://ziyuan.baidu.com/linksubmit/url)

1. 下载搜索引擎提供的html文件
1. 添加到博客下的source文件夹内
1. 构建发布之后在搜索引擎提交入口完成认证,等待搜索引擎完成数据更新之后即可在搜索引擎中找到你的站点。
## **坑：**

 直接这样加入source里构建并发布会会导致在访问这个html文件的时候被hexo本身重定向。如果你出现了重定向问题，请参照如下方式:

在source文件夹下搜索引擎提供的html首部加入:
```
layout: false
---
```
这样可以标识hexo停用ta的渲染，就不会重定向了.

之后重新构建发布网站即可


