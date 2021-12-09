---
title: MYSQL密码重置详解
date: 2019-05-29 11:04:15
categories:
    - 象牙塔
tags:
    - 数据库
---

>## 摘要
* 很多同学在学习数据库操作的过程中，总会因为某些莫名奇妙的原因导致数据库密码变动，而到了不得不重置数据库密码的情况。本文主要讲解一下各个版本的MYSQL数据库重置密码的方法。

{% asset_img 1.jpg %}

<!--more-->

>## 如何查看MYSQL版本
* 打开一个命令行，在命令行里输入如下指令（如果你的MYSQL配置了环境变量的话）
    ```
    mysql --version
    ```

* 如果你的MYSQL没有配置环境变量，你可以去到mysql的安装目录下的bin文件夹里，运行上述命令。
* 如果你不知道如何改变命令行目录，可以用打开文件管理器，进入到mysql的安装目录下的bin文件夹内然后按住shift再点击鼠标右键。
* 这时你可以看到右键菜单里有个“在此处打开powershell窗口”（或者也可能时在此处打开cmd窗口一类的）
* 如果你连这些都不会，如果你不知道什么是环境变量。emmmmmm这个教程大概不适合你.

**以上均基于Microsoft Windows 10系统**

>## MYSQL 5.5+
1. 首先开启一个具有管理员权限的命令行，在其中输入如下指令关闭mysql服务
    ```
    net stop mysql
    ```

1. 接下来进入到mysql的安装文件夹下的bin目录内（如果你配置了环境变量也可以直接在命令行里运行）
    ```
    mysqld -nt --skip-grant-tables
    ```

    如果你发现命令行运行的“卡住了”，那么恭喜你可以成功进入下一步
    
    --skip-grant-tables
    你可以简单的理解为跳过密码认证

1. 接下来保证这个命令行界面开启状态，重新打开一个新的命令行窗口。运行如下命令
    ```
    mysql -u root  -p
    ```

    出现提示输入密码直接按回车即可，你会发现你已经成功给进入到了数据库内部

1. 然后使用数据库命令重置密码即可
    ```
    use mysql;  -- 切换到mysql数据库
    update user set password=password("123456") where user="root"; --修改密码为123456
    ```
    以上命令也可以用于你知道密码的情况下想要修改密码。
    不过在正常情况下执行完上述命令之后需要再增加如下命令刷新权限才可以让数据库密码更新成功（当然这里不需要）

    ```
    flush privileges;
    ```


1. 当修改命令执行成功后。输入exit退出数据库。关闭这个新的命令行，回到我们一开始启动的命令行。使用ctrl+c中止命令。然后使用如下命令重启mysql服务
    ```
    net start mysql
    ```

如果不出意外，服务启动成功，那么你的密码重置就完成了。

---
>## MYSQL 8.0+

* MYSQL 8下有两种方法可以重置密码

> ### 使用--init-file
1. 首先依旧是打开一个具有管理员权限的命令行，使用命令停止mysql服务

    ```
    net stop mysql
    ```



1. 接下来创建一个文本文件（sql.txt），指定在启动时需要执行的修改密码的命令。

    ```
    ALTER USER "root"@"localhost" IDENTIFIED BY "123456";
    ```

    将密码重置为123456

1. 最后使用如下命令启动

    ```
    mysqld --init-file=E:/sql.txt --console
    ```

    启动成功之后重新开启一个新的命令行，尝试使用重置的密码（123456）登陆数据库吧。如果没有重置成功，请检查sql.txt中的sql语句语法是否出现问题。


1. 当你使用重置的密码登陆成功之后，关闭掉新的命令行界面，回到旧的命令行界面使用ctrl+c中止命令。最后再重新启动mysql服务

    ```
    net start mysql
    ```

---
>### 使用--skip-grant-tables

1. 开启一个具有管理员权限的命令行，先关闭mysql服务

    ```
    net stop mysql
    ```



1. 使用如下命令无密码启动服务

    ```
    mysqld --console --skip-grant-tables --shared-memory
    ```


    这就是mysql5与mysql8的差别，mysql8想要使用skip-grant启动必须加入额外参数，否则服务无法启动



1. 接下来重新开启一个命令行以空密码登陆数据库

    ```
    mysql -u root -p
    ```


    提示输入密码直接按下回车即可。接下来重置密码
    ```
    update mysql.user set authenication_string="123456" where user="root" and host="localhost";
    ```

    执行成功无错误之后即表示密码重置成功

1. 接下来退出数据库，关闭新的命令行，回到之前的命令行使用ctrl+c中止命令，然后重新启动mysql服务

    ```
    net start mysql
    ```
