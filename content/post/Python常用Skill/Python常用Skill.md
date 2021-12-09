---
title: Python常用Skill
date: 2019-01-25 17:01:19
update: 2019-04-11 14:58:00
categories:
    - Python
tags:
    - Python
---
>## Python常用技能汇总
>这里是一些写python脚本时常用的小技巧,长期更新
~~咕咕咕~~

{% asset_img Python.jpg Python娘,来源于萌娘百科 %}

<!--more-->

---
>### 获取脚本目录

* 脚本运行时相对路径时基于命令行的路径.这样直接在脚本里使用相对路径会出现问题.我们可以使用如下方法获得脚本所在的绝对路径,以及脚本本身的文件名.
    ```
    import os
    WORK_PATH,FILE_NAME=os.path.split(os.path.abspath(__file__))
    ```
    * WORK_PATH中就存储了脚本所在的绝对路径
    * FILE_NAME中就是脚本名称

---
>### 获取Windows用户目录
* 有时我们需要获取用户默认的下载目录,或者是文档目录等等
    ```
    import os
    USER_PATH=os.path.expanduser("~")
    DOWNLOAD_PATH=ps.path.join(os.path.expanduser("~"),"Download")
    ```
    * USER_PATH中存储了用户目录的绝对路径
    * DOWNLOAD_PATH中存储了用户默认下载目录(如果用户没有自己重命名这个文件夹的话)

---
>### Python参数传递中的*args以及**kwargs
* 事实上真正的python参数传递语法是* 和 **。*args和\*\*kwargs是我们一种约定俗成的写法。


* >#### *args
    * *args用来表示函数接受可变长度的 __非关键字__ 参数作为函数的输入
        ```
        def test(normal_arg, *args):
            print("first normal arg:"+normal_arg)

            for i,x in enumerate(args):
                print("{} arg is {}".format(i+1,x))
        
        test("normal","a","b","c","d")
        ```

    * 样例输出
        ```
        first normal arg:normal
        1 arg is a
        2 arg is b
        3 arg is c
        4 arg is d
        ```

* >#### **kwargs
    * **kwargs表示函数接受可变长度的关键字参数字典作为参数，即可以简单的理解为传入的是字典参数
        ```
        def test(**kwargs):
        if kwargs is not None:
            for key, value in kwargs.iteritems():
                print("{} = {}".format(key,value))
            # Or you can visit kwargs like a dict() object
            # for key in kwargs:
            #    print("{} = {}".format(key, kwargs[key]))
        test(name="python", value="5")
        ```
    * 样例输出
        ```
        name = python
        value = 5
        ```

* >#### What's more
    * 我们也可以使用这两个参数来调用一般参数格式的函数
        ```
        def func(arg1,arg2,arg3):
            print("arg1: " + arg1)
            print("arg2: " + arg2)
            print("arg3: " + arg3)
        
        args_list=("python",2,3)
        func(*args_list)
        print("==================")

        kwargs_dict={"arg3": 3, "arg1": "python", "arg2": 2}
        func(**kwargs_dict)
        ```

    * 样例输出:

        ```
        arg1: python
        arg2: 2
        arg3: 3
        ==================
        arg1: python
        arg2: 2
        arg3: 3
        ```

* 本段参考简书 [https://www.jianshu.com/p/be92113116c8](https://www.jianshu.com/p/be92113116c8)
