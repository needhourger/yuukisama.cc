---
title: mysql8的安装配置以及JDBC使用
date: 2018-12-28 16:11:13
categories: 
    - 象牙塔
    - 课设
tags: 
    - 数据库
---

## 摘要

* 基于windows下mysql 8.0.*版本安装以及配置等操作介绍
    * 基本安装
    * 初始化
    * 密码重置
    * 创建数据库
    * 创建列表
* 以及书本《Java简明教程》实列代码的采坑

![](Sqlserver.jpg)

<!--more-->

## 安装

1. 从mysql官网下载mysql社区版本

    这里选择的时二进制安装文件，就是需要配置环境变量的那种[下载](https://dev.mysql.com/downloads/mysql/)
    下载的时候选择第一个，第二个是测试版。**坑：在你点击下载之后会教你login一类的，你可以忽略它们找到下面一排英文“No thanks,just start my download”就可以正式下载**

    下载完成之后解压，将这个文件夹放到你想要的位置，列如E盘下。接着根据你选择的位置配置环境变量。如：
    ```
    E:\mysql-8.0.13-winx64\bin
    ```
    [不会配置环境变量？？？](https://jingyan.baidu.com/article/00a07f3876cd0582d128dc55.html)

2. 如果你是用可执行安装包安装，即msi文件安装请参照[这个](https://blog.csdn.net/CSDN_Liang_1991/article/details/81035293)

## 配置
1. 进入myql的安装目录（就是你放加压后文件夹的地方）,找到里面的bin文件夹。

    在该文件夹下点击窗体左上方文件 -> 找到“打开powershell” -> 选择“以管理员身份打开powershell” -> 运行如下命令
    ```
    mysqld --initialize-insecure --console
    ```
* --initialize-insecure参数是为了初始化一个没有初始密码的数据库
* --console参数显示初始化结果，方便出现问题后应对。
* 如果你初始化失败，请把错误信息给你想要请教的人看，而不是直接告诉别人你安装不了，掌握问问题的方法也很关键。


* **坑：出现缺少XXX140.dll，请安装[这个](https://www.microsoft.com/zh-CN/download/details.aspx?id=48145)**

* **类似问题安装c++运行库解决**

* **坑：初始化提示无法读写文件，创建文件问题。确保你是管理员权限的命令行**

* **坑：初始化失败，可以尝试删除mysql安装文件夹下的data文件夹重试**

* 你也可以使用这个命令初始化mysql
    ```
    mysqld --initialize --console
    ```
    这个命令在初始化过程中（如果成功）会输出一个随机密码，请务必记住他，否则你就得尝试重置密码了。

1. 安装服务

    在上一步执行没有报错之后，继续输入
    ```
    mysqld install
    ```
    安装mysql的系统服务，**再次重申，请确保你的命令行具有管理员权限！！！**

1. 启动服务
    ```
    net start mysql
    ```
    **坑:提示启动失败请尝试删除mysql服务**
        ```
        mysqld remove
        ```
        并回到第一步重新初始化（这个过程中可能还需要删除mysql安装目录中的data文件夹）
    
1. 第一次启动数据库修改密码
    
    * 如果你是使用了--initialize-insecure参数初始化的数据库的话，那么数据库默认没有密码，直接在命令行输入
        ```
        mysql -u root -p
        ```
        出现输入密码提示后按下回车即可。
    
    * 使用--initialize参数初始化的请输入初始化过程中输出的随机密码
    
1. 修改密码

    ```
    ALTER user 'root'@'local' IDENTIFIED BY 'your_password你的密码';
    ```
    **坑:数据库命令大小写其实无所谓，可以都是小写。别忘了末尾的分号**

1. 数据库的基本操作:

    * 新建数据库
    ```
    create database 你要新建的数据库的名字;
    ```
    * 展示所有数据库
    ```
    show databases;
    ```
    * 切换数据库
    ```
    use 你要切换的数据库的名字
    ```
    **坑:这里有没有分号无所谓，其他数据库语句必须分号结尾**
    
    * 新建表
    ```
    create table 表名(key1 key_type,key2 key_type,key3 key_type);
    
    实例：

    create table student(no VARCHAR(20,name VARCHAR(20),math INT,average DOUBLE);
    ```
    * 展示表中所有数据：
    ```
    select * from 表名;
    ```
    * 打印表内数据格式：
    ```
    desc 表名;
    ```

    [更多命令](http://www.runoob.com/mysql/mysql-tutorial.html)

    **坑:[数值类型详解](https://www.shadowwu.club/2018/05/14/mysql_data_type/index.html)**

    **坑:请先切换到你要创建表的数据库再执行创建表明操作**

## java连接数据库

1. 在mysql官网下载mysql connector的java sdk.请按照你的数据库版本下载对应的connector [下载地址](https://dev.mysql.com/downloads/connector/j/)

    “Select Operating System”选项请选择“Platform Independent”.

    下载第一个还是第二个只是压缩包格式的不同

    下载点击进去之后请点击“No thanks, just start my download.”开始下载

1. 下载完成之后解压，打开解压后的文件夹，找到一个*.jar后缀的文件列如：
    ```
    mysql-connector-java-8.0.13.jar
    ```
    将该文件复制到你的eclipse中的java项目里

    **坑:什么你不知道你的eclipse项目目录在哪里？？？默认就在c:\\用户\\你的用户名\\eclise-workplace里，对，打开哪个文件夹，找到你写的那个项目的名字，把这个jar文件放进去！！！**

1. 将jar文件加入你的java项目
    
    在eclipse里右键你写的项目，选择new->选择Sourse Folder，随便起个名字你开心就好->把你的那个jar文件再放进去（别问我怎么放，打开你的电脑）
    [百度知道图解（百度知道没有选择创建source folder文件夹，我建议你这里选择这个）](https://jingyan.baidu.com/article/ca41422fc76c4a1eae99ed9f.html)

1. 书本范例代码的问题
    *  新版的mysql数据库connector驱动包全名：
    ```
    com.mysql.cj.jdbc.Driver
    ```
    * 数据库连接url格式：
    ```
    jdbc:mysql://localhost:3306/数据库名称?serverTimezone=UTC
    ```
        
* **坑:mysql8.0.x新版在java调用时要增加参数化serverTimezone=UTC，设置数据库时间为世界标准时间，否则会报以下错误**

    ```
        Caused by: java.sql.SQLException: The server time zone value '�й���׼ʱ��' is unrecognized or represents more than one time zone. You must configure either the server or JDBC driver (via the serverTimezone configuration property) to use a more specifc time zone value if you want to utilize time zone support.
    ```

    * 连接数据库出现access deny字样

    ```
    DriverManger.getConnection(DB_URL,USER,PASS)
    ```
    DB_URL就是上面提到的url格式内容。
    
    USER默认"root",PASS如果为空则使用空字符串即可，有密码请确保密码正确

## 数据库连接代码示例

### 请确保已经将mysqlconnector加入项目构建目录，参照[将jar文件加入你的java项目](#将jar文件加入你的java项目)

```
public class MySQLDemo {
 
    // JDBC 驱动名及数据库 URL
    static final String JDBC_DRIVER = "com.mysql.cj.jdbc.Driver";  
    static final String DB_URL = "jdbc:mysql://localhost:3306/RUNOOB";//RUNOOB为数据库（database）名称
 
    // 数据库的用户名与密码，需要根据自己的设置
    static final String USER = "root";
    static final String PASS = "123456";
 
    public static void main(String[] args) {
        Connection conn = null;
        Statement stmt = null;
        try{
            // 注册 JDBC 驱动
            Class.forName("com.mysql.cj.jdbc.Driver");
        
            // 打开链接
            System.out.println("连接数据库...");
            conn = DriverManager.getConnection(DB_URL,USER,PASS);
        
            // 执行查询
            System.out.println(" 实例化Statement对象...");
            stmt = conn.createStatement();
            String sql;
            sql = "SELECT id, name, url FROM websites";//构建sql语句 id name url均为字段名 websites表名
            ResultSet rs = stmt.executeQuery(sql);//执行sql语句并获取返回结果
        
            // 展开结果集数据库
            while(rs.next()){
                // 通过字段检索
                int id  = rs.getInt("id");
                String name = rs.getString("name");
                String url = rs.getString("url");
    
                // 输出数据
                System.out.print("ID: " + id);
                System.out.print(", 站点名称: " + name);
                System.out.print(", 站点 URL: " + url);
                System.out.print("\n");
            }
            // 完成后关闭
            rs.close();
            stmt.close();
            conn.close();
        }catch(SQLException se){
            // 处理 JDBC 错误
            se.printStackTrace();
        }catch(Exception e){
            // 处理 Class.forName 错误
            e.printStackTrace();
        }finally{
            // 关闭资源
            try{
                if(stmt!=null) stmt.close();
            }catch(SQLException se2){
            }// 什么都不做
            try{
                if(conn!=null) conn.close();
            }catch(SQLException se){
                se.printStackTrace();
            }
        }
        System.out.println("Goodbye!");
    }
}
```
