---
title: C++与SQLite
date: 2019-05-14 08:47:25
categories:
    - C/C++
tags:
    - C/C++
    - 数据库
---

## 摘要
* 本文主要讲解了C++对接sqlite3数据库的配置以及相关操作
* IDE：Visual Studio 2017
* SQLite3

![](sqlite.jpg)

<!--more-->

## 碎碎念

敲代码不接触数据库几乎是不可能的。但是很多学校开设的课程都是从Mysql开始的，导致很多学生一些项目就是Mysql。毫无疑问Mysql是很强大的，但是有的时候我们真的很需要因地制宜选择更加合适的数据库。~~其实还是因为C++&Mysql的模式本垃圾配置不来~~

## SQLite3 简介
* SQLite是一个进程内的库，实现了完全的自给自足的，无服务器的，零配置的，事务性的SQL数据库引擎。
* 零配置意味着你不需要在任何一台机器上像mysql一样进行复杂的配置
* SQLite正如其名，轻量级，操作方便。毫无疑问相对于功能强大的mysql它缺少了很多相应的其他功能，但是如果你不准备采取分布式结构分离数据应用，复杂的数据库功能，只是需要一个轻量化便捷的数据库基础功能，SQLite是你最好的选择
* ~~嗯，sqlite的c++API也不像mysql的那么变态~~

## 下载SQLite3
* 依据你的需要的系统版本在SQLite官方下载源代码版本 [官方下载界面](https://www.sqlite.org/download.html)

    ![](1.png)

    SQLite官方给出了编译好的动态链接库方便开发，不过本教程以源代码入手（因为只要配置正确相较于动态链接库文件，本方法调用更为简单，并且效率更高）

* 如图我们选择第一个下载。下载完成之后我们解压到当前文件夹，并修改文件夹为sqlite。把文件夹移动到我们自己的项目里，并在IDE中引入该文件夹。（具体怎么引入瞎几把发挥啦，随你是添加额外项目还是添加额外包含目录，搞通就行hhh）

## SQLite3的基础操作
* sqlite遵循最基本的通用sql语句。所以大部分操作与mysql等一系列其他数据库无差异
* 你可以从sqlite3官网下载exe可执行文件用以访问sqlite生成的数据库文件，在可执行文件内，基础的支持命令如下：
    ```
    sqlite3.exe ./test.db //在命令行内使用该命令连接指定数据库文件，不存在则创建
    ```

    | 命令               | 作用                                                                                |
    | ------------------ | ----------------------------------------------------------------------------------- |
    | .backup ?DB? FILE  | 备份 DB 数据库（默认是 "main"）到 FILE 文件。                                       |
    | .bail ON           | OFF                                                                                 | 发生错误后停止。默认为 OFF。                                                           |
    | .databases         | 列出数据库的名称及其所依附的文件。                                                  |
    | .dump ?TABLE?      | 以 SQL 文本格式转储数据库。如果指定了 TABLE 表，则只转储匹配 LIKE 模式的 TABLE 表。 |
    | .echo ON           | OFF                                                                                 | 开启或关闭 echo 命令。                                                                 |
    | .exit              | 退出 SQLite 提示符。                                                                |
    | .explain ON        | OFF                                                                                 | 开启或关闭适合于 EXPLAIN 的输出模式。如果没有带参数，则为 EXPLAIN on，及开启 EXPLAIN。 |
    | .header(s) ON      | OFF                                                                                 | 开启或关闭头部显示。                                                                   |
    | .help              | 显示帮助消息。                                                                      |
    | .import FILE TABLE | 导入来自 FILE 文件的数据到 TABLE 表中。                                             |
    | .indices ?TABLE?   | 显示所有索引的名称。如果指定了 TABLE 表，则只显示匹配 LIKE 模式的 TABLE 表的索引。  |
    | .load FILE ?ENTRY? | 加载一个扩展库。                                                                    |
    | .log FILE          | off                                                                                 | 开启或关闭日志。FILE 文件可以是 stderr（标准错误）/stdout（标准输出）。                |
    | .mode MODE         | 设置输出模式，MODE 可以是下列之一：                                                 |
    * csv 逗号分隔的值
    * column 左对齐的列
    * html HTML 的 \<table\> 代码
    * insert TABLE 表的 SQL 插入（insert）语句
    * line 每行一个值
    * list 由 .separator 字符串分隔的值
    * tabs 由 Tab 分隔的值
    * tcl TCL 列表元素

    | -                     | -                                                                          |
    | --------------------- | -------------------------------------------------------------------------- |
    | .nullvalue STRING     | 在 NULL 值的地方输出 STRING 字符串。                                       |
    | .output FILENAME      | 发送输出到 FILENAME 文件。                                                 |
    | .output stdout        | 发送输出到屏幕。                                                           |
    | .print STRING...      | 逐字地输出 STRING 字符串。                                                 |
    | .prompt MAIN CONTINUE | 替换标准提示符。                                                           |
    | .quit                 | 退出 SQLite 提示符。                                                       |
    | .read FILENAME        | 执行 FILENAME 文件中的 SQL。                                               |
    | .schema ?TABLE?       | 显示 CREATE 语句。如果指定了 TABLE 表，则只显示匹配 LIKE 模式的 TABLE 表。 |
    | .separator STRING     | 改变输出模式和 .import 所使用的分隔符。                                    |
    | .show                 | 显示各种设置的当前值。                                                     |
    | .stats ON             | OFF                                                                        | 开启或关闭统计。        |
    | .tables ?PATTERN?     | 列出匹配 LIKE 模式的表的名称。                                             |
    | .timeout MS           | 尝试打开锁定的表 MS 毫秒。                                                 |
    | .width NUM NUM        | 为 "column" 模式设置列宽度。                                               |
    | .timer ON             | OFF                                                                        | 开启或关闭 CPU 定时器。 |

## 使用C++连接SQLite并执行一条sql语句
```
#include<iostream>
#include<sqlite3.h>
using namespace std;

//需要执行的sql语句
const char sql[] = {    
	"CREATE TABLE IF NOT EXISTS Users("
	"ID INTEGER PRIMARY KEY NOT NULL,"
	"Stu_ID INTEGER,"
	"Password TEXT"
	");"
};


int main(){
    sqlite3 *db=NULL;   //数据库句柄
    char *ErrMsg=NULL;  //错误信息指针
    int rc=0;           //函数执行返回值
    DATABASE="./test.db"//需要连接的数据库路径
    
    //连接数据库，并将返回的数据库句柄交给db
    rc=sqlite3_open(DATABASE,&db);
    if (rc){
        //如果链接失败关闭句柄退出程序
        sqlite3_close(db);
        cout<<"Cannot connect to "<<DATABASE<<endl;
        return 0;
    } 
    
    //执行sql语句
    rc=sqlite3_exec(db,sql,NULL,NULL,&ErrMsg);
    if (rc){
        如果执行失败关闭句柄，打印出错误信息，退出程序
        sqlite3_close(db);
        cout<<"sql execute failed..."<<endl;
        cout<<ErrMsg<<endl;
        return 0;
    }
    cout<<"table create"<<endl;
    //关闭数据库句柄释放内存
    sqlite3_close(db);
    return 0;
}
```
* sqlite3_open(const char *filename,sqlite3 **ppDb)

    该函数用以打开指定数据库（不存在则创建），并返回一个数据库连接对象。函数返回值表示执行成功与否。
    **如果filename为:memory:，sqlite将会在内存里创建一个数据库**

* sqlite3_exec(sqlite3* db,const char *sql,sqlite_callback,void *data,char **ErrMsg)

    该函数用以执行sql语句，sql语句可以是一条也可以是多条。
    * sqlite3* db 数据库连接句柄
    * const char *sql 需要执行的sql语句
    * sqlite_callback 执行成功后的回调函数
    * void *data 回调函数参数
    * char **ErrMsg 错误信息
    
* sqlite3_close(sqlite3* db)

    该函数用以关闭数据库连接，释放内存。所有相关sql语句都应该在连接关闭之前完成，如果仍有sql语句在执行，该函数将返回SQLITE_BUSY错误

## sqlite3回调函数

* [什么是回调函数](https://blog.csdn.net/qq_29924041/article/details/74857469)

* SQLite中的回调函数可以让我们自定义sql语句执行完成之后的操作。sqlite3_exec中执行的sql语句的执行结果会被默认传递给回调函数（如果你定义了的话）

```
sqlite3_exec(db,"select * from users;",callback,"helloworld",&ErrMsg)

//该回调函数将会在sql语句执行时每一行数据返回时被执行
static int callback(void *data,int argc,char **argv,char **azColName){
    //同时会打印由sqlite3_exec传入的参数"helloworld"
    printf("%s",(char*)data);
    for(int i=0;i<argc;i++){
        printf("%s : %s\n",azColName[i],argv[i]?argv[i]:"NULL");
    }
    return 0;
}
```

## 不用回调函数获取sql结果

* sqlite的回调函数尽管已经很方便，但是对于某些情景的应用时却力不从心，有什么办法可以不适用回调函数也获取sql的执行结果么？答案是有的

```
void func(){
    sqlite3 *db=NULL;       //定义数据库句柄
    sqlite3_stmt* stmt=NULL;//Statement handle 能令sqlite3_step()执行的编译好的准备语句的指针。调用sqlite3_finalize()删除它。
    const char *zTail=NULL; //Pointer to unusedportion of zSql 当zSql在遇见终止符或者达到设定的nByte结束后，如果还有剩余的内容，那么这些剩余的内容将被存放到pzTail中，不包含终止符
    const char sql[]="select ID from users;"
    int rc;
    
    rc=sqlite3_open("./test.db",&db);
    if (rc) return;

    //准备sql语句
    if (sqlite3_prepare_v2(db,sql,strlen(sql),&stmt,&zTail)==SQLITE_OK){
        //步进执行，并获取执行结果
        while (sqlite3_step(stmt)==SQLITE_ROW)
            print("%d\n",sqlite3_column_int(stmt,0));
            /*
            sqlite3_column_int(stmt,0) 第二个参数表示sql语句返回的数据的列数，从0开始，本例子中的sql语句只选取了ID那一列，所以这里只用数字0
            sqlite3_column_text()用以获取字符串类型返回值
            sqlite3_column_int64()用以获取int64大小的返回值
            */
    }
    //释放资源
    sqlite3_finalize(stmt);
    sqlite3_close(db);
    return;
}
```

sqlite还有很多其他的函数用以获取sql语句执行结果，详情可以参见官方文档

## 构造sql语句并执行
* 尽管一旦程序中有接收用户参数动态构造sql语句的程序实现，就通常意味着sql注入漏洞的存在，但是数据库与用户输入数据的交互几乎是不可能不存在的。所以下面我们介绍有关构造sql语句的方法

1. 使用字符串处理函数（个人安利）

    sql语句本质是一段字符串，所以我们可以直接使用c++自带的字符串处理函数用以绑定参数，动态构造sql语句

    ```
    int id=1;
    
    //首先定义一个字符数组用以存放动态生成的sql语句
    char sql[1024]={0};

    //使用相对安全的没有缓冲区溢出漏洞的sprintf_s构造sql语句
    sprintf_s(sql,1024,"select * from users where ID=%d",id);

    //如果你的编译器不支持sprintf_s()函数，也可使用相对不安全的sprintf()函数
    sprintf(sql,"select * from users where ID=%d",id);

    //执行sql语句，这里函数换成sqlite3_prepare_v2()也是一样
    sqlite3_exec(db,sql,NULL,NULL,&Errmsg);
    ```

1. 使用sqlite3自带的绑定参数函数
    ```
    int id=1;
    char sql[1024]="select * from users where ID=?";
    sqlite3_stmt *stmt=NULL;

    sqlite3_prepare_v2(db,sql,strlen(sql),&stmt,NULL);

    //绑定整形参数到对应位置，参数位置从1开始
    sqlite3_bind_int(stmt,1,id);
    /*
    sqlite3_bind_text(stmt,pos,"halloworld",strlen("halloworld"),SQLITE_STATIC)
    */
    ...
    ```
    ~~据说~~使用sqlite3_bind_*()函数相对使用字符串处理函数更为节省内存，有一定程度可以规避sql注入攻击的效果（这里我感觉不是很苟同，这段是网上说的）

## 中文字符编码问题

Visual studio等一众Win下的IDE，编辑器默认编码不是utf-8，而sqlite3默认的字符编码是utf-8.于是又到了我们喜闻乐见的字符编码问题。这里给大家安利一波函数，方便的进行字符格式转换。

**请将如下代码全部纳入你的项目中因为函数之间存在相互调用问题**

```
//UTF-8转Unicode 
std::wstring Utf82Unicode(const std::string& utf8string) {
	int widesize = ::MultiByteToWideChar(CP_UTF8, 0, utf8string.c_str(), -1, NULL, 0);
	if (widesize == ERROR_NO_UNICODE_TRANSLATION)
	{
		throw std::exception("Invalid UTF-8 sequence.");
	}
	if (widesize == 0)
	{
		throw std::exception("Error in conversion.");
	}
	std::vector<wchar_t> resultstring(widesize);
	int convresult = ::MultiByteToWideChar(CP_UTF8, 0, utf8string.c_str(), -1, &resultstring[0], widesize);
	if (convresult != widesize)
	{
		throw std::exception("La falla!");
	}
	return std::wstring(&resultstring[0]);
}


//unicode 转为 ascii 
std::string WideByte2Acsi(std::wstring& wstrcode) {
	int asciisize = ::WideCharToMultiByte(CP_OEMCP, 0, wstrcode.c_str(), -1, NULL, 0, NULL, NULL);
	if (asciisize == ERROR_NO_UNICODE_TRANSLATION)
	{
		throw std::exception("Invalid UTF-8 sequence.");
	}
	if (asciisize == 0)
	{
		throw std::exception("Error in conversion.");
	}
	std::vector<char> resultstring(asciisize);
	int convresult = ::WideCharToMultiByte(CP_OEMCP, 0, wstrcode.c_str(), -1, &resultstring[0], asciisize, NULL, NULL);
	if (convresult != asciisize)
	{
		throw std::exception("La falla!");
	}
	return std::string(&resultstring[0]);
}



//utf-8 转 ascii 
std::string UTF_82ASCII(std::string& strUtf8Code) {
	using namespace std;
	string strRet = "";
	//先把 utf8 转为 unicode 
	wstring wstr = Utf82Unicode(strUtf8Code);
	//最后把 unicode 转为 ascii 
	strRet = WideByte2Acsi(wstr);
	return strRet;
}



//ascii 转 Unicode 
std::wstring Ascii2WideByte(std::string& strascii) {
	using namespace std;
	int widesize = MultiByteToWideChar(CP_ACP, 0, (char*)strascii.c_str(), -1, NULL, 0);
	if (widesize == ERROR_NO_UNICODE_TRANSLATION)
	{
		throw std::exception("Invalid UTF-8 sequence.");
	}
	if (widesize == 0)
	{
		throw std::exception("Error in conversion.");
	}
	std::vector<wchar_t> resultstring(widesize);
	int convresult = MultiByteToWideChar(CP_ACP, 0, (char*)strascii.c_str(), -1, &resultstring[0], widesize);
	if (convresult != widesize)
	{
		throw std::exception("La falla!");
	}
	return std::wstring(&resultstring[0]);
}


//Unicode 转 Utf8 
std::string Unicode2Utf8(const std::wstring& widestring) {
	using namespace std;
	int utf8size = ::WideCharToMultiByte(CP_UTF8, 0, widestring.c_str(), -1, NULL, 0, NULL, NULL);
	if (utf8size == 0)
	{
		throw std::exception("Error in conversion.");
	}
	std::vector<char> resultstring(utf8size);
	int convresult = ::WideCharToMultiByte(CP_UTF8, 0, widestring.c_str(), -1, &resultstring[0], utf8size, NULL, NULL);
	if (convresult != utf8size)
	{
		throw std::exception("La falla!");
	}
	return std::string(&resultstring[0]);
}


//ascii 转 Utf8 
std::string ASCII2UTF_8(std::string& strAsciiCode) {
	using namespace std;
	string strRet("");
	//先把 ascii 转为 unicode 
	wstring wstr = Ascii2WideByte(strAsciiCode);
	//最后把 unicode 转为 utf8 
	strRet = Unicode2Utf8(wstr);
	return strRet;
}
```


