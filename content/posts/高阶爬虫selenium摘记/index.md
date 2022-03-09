---
title: 高阶爬虫selenium摘记
date: 2019-03-08 14:46:18
categories:
    - Code
tags:
    - Python
    - 爬虫
    - selenium
---
## 概述

* 本文记录一些有关python高阶爬虫selenium+浏览器爬虫操作的踩坑记录.
* selenium

    本质是一个为了自动化测试而诞生的工具,但没想到ta自身作为一个爬虫也是极具优势的.
    
    selenium简单的说就是通过各种webdirver驱动去控制浏览器去访问网站,完成各种操作,从而达到爬取数据的目的.可以有效的规避多数反爬虫机制,列如用js动态生成的网站

![](selenium.jpeg)
<!--more-->

## Start

python version:3.5+

browser:Chrome

webdriver:chromedriver

platform:Windows 10

1. 首先安装selenium包
    ```
    pip install selenium
    ```

2. 在命令行中尝试引入selenium库,如果不报错证明安装成功
   ```
   import selenium
   ```

3. 依据你准备选用的浏览器下载对应版本的webdriver.本文以chrome浏览器为例,依据chrome浏览器版本下载对应的chromedriver


   [chromedriver download](https://npm.taobao.org/mirrors/chromedriver)

   * 关于webdriver的配置.

        webdriver事实上是一个可执行文件,为了能够让程序调用它,所以需要配置其环境变量.win下最偷懒的方法自然是把ta丢进system32里.

        当然有高阶操作可以装载指定目录中的webdriver,以及可以指定浏览器可执行文件目录等,这部分操作参见自定义webdriver以及浏览器可执行文件
4. 让我们开始第一个例子打开百度首页
   ```
   #-*- coding:utf-8 -*-
   # 从selenium包中导入webdriver
   from selenium import webdriver   

   # 声明一个chrome对象
   dirver=webdriver.Chrome()            

   # 用chrome打开百度首页
   driver.get("https://www.baidu.com/") 
   ```

5. [更多基础操作](https://blog.csdn.net/yj1556492839/article/details/79671008)
    
    ~~容我偷个懒~~

## Advance
~~其实也就是一系列踩坑行为~~

1. ### 自定义webdriver以及浏览器可执行文件
    ```
    #-*- coding:utf-8 -*-
    from selenium import webdirver
    
    # 声明一个chrome设置对象
    options=webdriver.ChromeOptions()

    # 指定浏览器可执行文件路径
    options._binary_location="./Application/chrome.exe"

    # 利用chrome设置对形象作为参数初始化webdriver对象,executable_path即webdriver路径
    chrome=webdriver.Chrome(chrome_options=options,executable_path="./chromedriver.exe")

    # 打开百度首页
    chrome.get("https://www.baidu.com/")

    ```
 
* **不指定浏览器可执行文件目录,默认启动系统浏览器.个人拙见为了软件的移植性下载32位版本的二进制可执行文件放在脚本下属目录,使用参数指定浏览器可执行文件路径**
1. ### 设置浏览器启动参数
   
   * 普通参数
   
        ```
        #-*- coding:utf-8 -*-
        from selenium import webdirver
        
        # 声明一个chrome设置对象
        options=webdriver.ChromeOptions()

        # 禁用gpu加速
        options.add_argument("--disable-gpu")
        # 允许加载不安全内容
        options.add_argument('--allow-running-insecure-content')
        # 禁用插件
        options.add_argument('--disable-extensions')
        # 使用无头浏览器,即不显示浏览器图形化界面,适用于没有图形化界面的服务器
        options.add_argument('--headless')

        chrome=webdriver.Chrome(chrome_options=options)
        ```
    * 实验性参数
        ```
        appState = { 
            "recentDestinations": [ { 
                "id": "Save as PDF", 
                "origin": "local" 
            } ], 
            "selectedDestinationId": "Save as PDF", 
            "version": 2
        } 

        profile = {
            'printing.print_preview_sticky_settings.appState': json.dumps(appState)
        } 
        
        options.add_experimental_option("prefs",profile)

        ```
    * [更多启动参数设置参见文档](https://peter.sh/experiments/chromium-command-line-switches/)

1. 执行JavaScript脚本
    ```
    #-*- coding:utf-8 -*-
    from selenium import webdirver


    chrome=webdriver.Chrome()
    # 设置js代码执行超时时间
    chrome.set_script_timeout(30)
    chrome.get("http://www.baidu.com/")
    
    # 异步执行js代码
    chrome.execute_script("alert("Halloworld");")
    # 阻塞执行js代码
    chrome.execute_async_script("alert("halloworld");")
    ```
    * execute_script函数是异步执行js代码.
    * execute_async_script是阻塞执行,即会等待js执行完成再执行接下来的程序.
    * set_script_timeout是设置js执行超时时间,对于阻塞执行的js代码,在timeout设定的时间内没有能够完成将强行终止.这个timeout值默认是30s
  
