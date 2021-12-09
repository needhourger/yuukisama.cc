---
title: VC++调用外部exe并获取程序运行输出
date: 2019-01-17 12:16:36
categories: 
    - C/C++
tags:
    - C/C++
---

>## 摘要
* 最近在采坑某个项目，需要用到调用外部写好的exe运行，项目基于c语言，于是就有了关于**VC++调用外部程序并获取程序命令行运行输出的问题**
* 本文将围绕windows下调用外部exe程序的常见的几种方法展开。部分方法可能也适用于linux平台。
* ~~虽然到最后我的项目也没能具体应用上这项技术~~

* 本文将会涉及到的主要函数:
    * system()
    * winexec()
    * ShellExecute()
    * CreateProcess()
    

{% asset_img lobei.jpg 萝卜啊！赐予我力量！ %}
<!--more-->

>## 综述
在调用外部exe程序的同时还想要获得程序运行过程中的输出内容。本文介绍的主要是使用管道的方法。不涉及内存共享等一类的方法。

本文中假定所有的父程序即parent.exe；子程序为child.exe

* 子程序源码
    ```
    #include<iostream>
    using namespace std;

    int main(int argc, char* arg[]){
    	int x=argc;
    	cout<<x<<endl;
    	for(int i=0;i<x;i++) 
    	cout<<arg[i]<<endl;
	    return 0;
    }
    ```

>## System()
* 这是c/c++的标准函数之一，可以执行命令行命令。也可以在linux下调用。[详情文档参见cplusplus](http://www.cplusplus.com/reference/cstdlib/system/)

* 函数原型

    ```
    int system(const char* command)
    ```

    * 调用命令处理器执行命令，win下可以理解为调用cmd.exe。如果执行的命令参数为空，该函数会检测命令处理器是否可用。

    * 返回值：

        如果命令参数为空，如果命令处理器可用该函数返回一个非零值。反之返回0

        如果命令参数不为空，返回值取决于你所执行的命令。
    
* 调用子程序并获得其回显
    
    这里使用的是默认的命令行将所有的命令执行的输出结果重定向到result.txt中，之后我们只需要使用文件操作读取result.txt中的内容即可。
    
    **坑：** 命令行重定向输出到文件采用的是末尾追加的打开方式，为了保证程序每次执行结果不受上次执行结果的干扰，建议每次在读取完result.txt中的内容之后将文件清空。
    ```
    #include<iostream>
    #include<cstdlib>
    #include<fstream>
    using namespace std;
    int main(){
        char buffer[4096]={0};  //用于文件读取
        system(".\\child.exe argv1 argv2 argv3 >>.\\result.txt");//执行子程序并传入参数

        fstream file(".\\result.txt");  //读取结果文件
        while(file.is_open() && !file.eof()){
            file.read(buffer,4096);
            cout<<buffer<<endl;
        }
        file.clear();   //清空文件内容方便下次读取
        file.close();   //关闭文件
    }
    ```

* 父程序输出样例
    ```
    4
    .\child.exe
    argv1
    argv2
    gv3
    ```
    参数0为程序本身，参数1为传入的参数
    **坑：** 为森魔丢了argv3的ar？？？我也不知道，大佬了解的可以解答一下。result.txt里是全的，读取文件出了问题？？？

>## WinExec()
* Windows API 函数[（个人建议微软爸爸的函数还是看微软爸爸的文档最合适）](https://docs.microsoft.com/en-us/windows/desktop/api/winbase/nf-winbase-winexec)
* 函数原型：
    ```
    UINT WinExec(
        LPCSTR lpCmdLine,   //你所要执行的命令
        UINT   uCmdShow     //显示模式，设置不同的参数可以实现控制台隐藏等等
    );
    ```
    >返回值：如果函数成功，则返回值大于31。 如果函数失败，则返回值会表示错误类型，详情参见文档

* 调用子程序并获得其回显

    总的来说这个命令和system类似，只是多了一个显示模式的参数，可以做到隐藏控制台等等一系列的操作（具体参见微软文档）获取回显的方式依然是使用输出重定向到文本文件并读取的方式。注意点同上，因为使用的是文末追加的方式，最好每次读取完之后清空输出的文本内容。

    ```
    #include<iostream>
    #include<windows.h> //WinExec函数头文件
    #include<fstream>
    using namespace std;
    int main(){
        char buffer[4096];
        WinExec(".\\child.exe argv1 argv2 argv3 >>result.txt",SW_SHOW);
        fstream file(".\\result.txt");
        
        while (file.is_open() && !file.eof()){
            file.read(buffer,4096);
            cout<<buffer<<endl;
        }
        
        file.clear();   //清空输出文件内容
        file.close();
        return 0;
    }
    ```
* 父程序输出样例
    ```
    5
    .\child.exe
    argv1
    argv2
    argv3
    >>result.txt
    ```
    可以看得出来system()和WinExec()的差别，WinExec()会把最后重定向输出到文本文件也作为一个参数。

>## ShellExecute()
* Window API函数，[详情文档参见微软官方文档](https://docs.microsoft.com/en-us/windows/desktop/api/shellapi/nf-shellapi-shellexecutea)
    
* 函数原型
    ```
    HINSTANCE ShellExecuteA(
        HWND   hwnd,        //程序的窗口句柄，可以使用GetDesktopWindow()函数获取，也可以填写NULL
        LPCSTR lpOperation, //你所要执行的操作，Shellexecute不光是可以运行外部程序那么简单，它还可以操作文件等等，具体取决于这个参数的设置
        LPCSTR lpFile,      //你所要操作的文件名称
        LPCSTR lpParameters,//操作传入的参数
        LPCSTR lpDirectory, //你所要操作的文件所在的文件夹，可以理解为工作目录
        INT    nShowCmd     //显示参数
    );
    ```
    **坑：** 你可以在lpFile参数中直接写全文件路径，这样的话文件的工作目录就会是父程序的工作目录。如果不屑写全的话可以在lpFile中写上文件名，lpDirectory中写明文件所在路径。这样的话启动的子程序的默认工作路径也得到了指定。
    > 返回值：
    函数返回一个HINSTANCE(实质是一个无符号的整形)，大于32表明子程序调用成功，小于32则表明调用失败。微软官方文档有详情解释每个错误的返回值的含义。

* 调用子程序
    ### **但并不能获取回显！！！** 
    网上某些论坛博客里有人用这个函数调用外部exe并使用将输出重定向到文本文件的方式获取输出的返回内容，但是本人实践并未成功。如有大佬知道原因欢迎指正！
    ```
    #include<iostream>
    #include<windows.h>     //WinExec函数头文件
    using namespace std;

    int main(){
        HINSTANCE ret;      
	    ret=ShellExecute(
	    	GetDesktopWindow(), //获取窗口句柄 
	    	"open",             //指定操作
	    	"child.exe",        //程序文件名称
	    	"argv1 agr2 argv3",    //命令行参数
	    	".\\",               //程序所在目录
	    	SW_HIDE             //子程序显示模式
	    );
        return 0;
    }
    ```

>## ShellExecuteEx()
* Window API函数，[详情文档](https://docs.microsoft.com/zh-cn/windows/desktop/api/shellapi/nf-shellapi-shellexecutew)
    * 这个函数是ShellExecute的扩展函数
    * 可以实现阻塞调用子程序
* 函数原型
    ```
    BOOL ShellExecuteExW(
        SHELLEXECUTEINFOW *pExecInfo    //指向SHELLEXECUTEINFO结构体的指针，该结构体中包含你所要调用的程序的相关信息
    );
    ```
    >返回值：显而易见，True表示执行成功，反之，失败.

* 调用子程序示例

    **同样不可以使用重定向输出到文件的方式获取程序执行输出**
    ```
    #include<iostream>
    #include<windows.h>     //shellapi.h你也可以使用这个头文件，shellapi.h包含在windows.h这个头文件内。
    using namespace std;
    int main(){
        SHELLEEXECUTEINFO shellinfo;                    
        ZeroMemory(&shellinfo,sizeof(SHELLEXECUTEINFO));    //初始化结构体数据
        shellinfo.cbSize=sizeof(SHELLEXECUTEINFO);          //cbSzie存储结构体大小
        shellinfo.fMask=SEEK_MASK_NOCLOSEPROCESS;           //标志表明其他结构成员的内容和有效性,SEEK_MASK_NOCLOSEPRCESS指明使用hProcess接收进程句柄
        shellinfo.hwnd=GetDesktopWindow();                  //获取窗口句柄
        shellinfo.lpVerb="open";                            //指定操作类型，同上面的lpOperation
        shellinfo.lpFile="child.exe";                       //指定文件名
        shellinfo.lpParameters="agrv1 argv2 argv3";         //传入子程序的参数
        shellinfo.lpDirectory=".\\";                        //子程序工作目录
        shellinfo.nShow=SW_HIDE;                            //子程序窗体显示，SW_HIDE隐藏
        shellinfo.hInstApp=NULL;                            //如果SEEK_MASK_NOCLOSEPROCESS参数被设置，该参数用于接收返回子程序的执行情况，类似ShellExecute函数的返回值

        BOOL ret=ShellExecuteEx(&shellinfo);                   
        WaitForSingleObject(shellinfo.hProcess,INFINITE);   //阻塞等待子进程执行完毕

        return 0;
    }
    ```

>## CreatePipe() && CreateProcess()
* Window API函数，理论上调用外部exe并且获得其执行输出的最佳方案，但是函数很复杂，参数众多。[依旧是微软爸爸的文档](https://docs.microsoft.com/en-us/windows/desktop/api/processthreadsapi/nf-processthreadsapi-createprocessa)
    
    ~~并且本人的项目中致死都没有成功使用这个函数，个人认为是因为项目使用的SDK在多线程支持方面存在问题？？？~~
    
    **本文重头戏，使用函数CreatePipe创建匿名管道将子程序输出重定向。（这里仅仅重定向输出，微软官方样例是输入输出都重定向了）**[微软官方样例](https://docs.microsoft.com/en-us/windows/desktop/procthread/creating-a-child-process-with-redirected-input-and-output)
* 函数原型
    ```
    BOOL WINAPI CreatePipe(
        _Out_    PHANDLE               hReadPipe,       //接收管道读取句柄的变量指针
        _Out_    PHANDLE               hWritePipe,      //接收管道写句柄的变量指正
        _In_opt_ LPSECURITY_ATTRIBUTES lpPipeAttributes,   //指向SECURITY_ATTRIBUTES结构体的指针，该结构确定子进程是否可以继承返回的句柄。如果lpPipeAttributes为NULL，则无法继承句柄。
        _In_     DWORD                 nSize        //管道缓冲区大小
    );

    ```
    >返回值：如果函数成功，则返回值为非零。

    ```
    BOOL CreateProcessA(
        LPCSTR                lpApplicationName,    //子程序的名称或者完整路径
        LPSTR                 lpCommandLine,        //子程序的完整路径加上命令行参数全部，如果你设置了上一个参数，这个参数只需要传入命令行参数即可。只有当上一个参数为空的时候你才需要填写子程序完整路径
        LPSECURITY_ATTRIBUTES lpProcessAttributes,  //子程序运行的子进程相关设置参数
        LPSECURITY_ATTRIBUTES lpThreadAttributes,   //子程序运行的子线程相关设置参数
        BOOL                  bInheritHandles,      //新进程是否继承父进程相关权限
        DWORD                 dwCreationFlags,      //子程序优先级相关是核定
        LPVOID                lpEnvironment,        //指向新进程的环境块的指针。NULL，则新进程使用调用进程的环境。
        LPCSTR                lpCurrentDirectory,   //当前进程的完整目录
        LPSTARTUPINFOA        lpStartupInfo,        //子进程启动信息
        LPPROCESS_INFORMATION lpProcessInformation  //子进程信息
    );
    ```
    >返回值：如果函数成功，则返回值为非零。


    这是两个相当复杂的参数，其中包含多个结构体，每个结构体我们还需要单独讲解。接下来直接以源码搭配注释理解

* 使用匿名管道重定向外部exe输出示例：

```
#include<iostream>
#include<windows.h>
using namespace std;

int main(){
	HANDLE hRead,hWrite;    //管道的读写句柄
	SECURITY_ATTRIBUTES sa; //管道安全属性相关结构体
	
	sa.nLength=sizeof(SECURITY_ATTRIBUTES); //结构体长度赋值
	sa.lpSecurityDescriptor=NULL;           //NULL管道默认安全描述符,管道的安全属性将继承与父程序
	sa.bInheritHandle=TRUE;                 //一个布尔值，指定在创建新进程时是否继承返回的句柄。如果此成员为TRUE，则新进程将继承该句柄。

	if(!CreatePipe(&hRead,&hWrite,&sa,0)){  //尝试创建管道，失败则弹出提示并退出
		MessageBox(NULL,"Error on CreatePipe()","WARNING",MB_OK);
		return 1;
	}
	
	STARTUPINFO si;         //启动信息结构体
	PROCESS_INFORMATION pi; //进程信息结构体
	si.cb=sizeof(STARTUPINFO);  //初始化启动信息结构体大小
	GetStartupInfo(&si);        //获取父进程的启动信息，利用这个函数我们可以只需要修改较少的参数值
	si.hStdError=hWrite;        //重定向错误信息输出到管道
	si.hStdOutput=hWrite;       //重定向标准输出新信息到管道
	si.wShowWindow=SW_HIDE;     //设定子进程窗体是否隐藏
	si.dwFlags=STARTF_USESHOWWINDOW | STARTF_USESTDHANDLES; //wShowWindow成员将包含其他信息；hStdInput，hStdOutput和hStdError成员包含其他信息。
	if (!CreateProcess(
        ".//child.exe",         //子进程完整目录
        "argv1 argv2 argv3",    //命令行参数
        NULL,NULL,              
        TRUE,                   //新进程继承父进程相关权限
        NULL,NULL,NULL,
        &si,                    //启动信息结构体指针
        &pi)                    //进程信息结构体指针
        ){
		MessageBox(NULL,"Error on CreateProcess()","WARNING",MB_OK);
		return 1;
	}
	CloseHandle(hWrite);    //关闭管道写入句柄
	
	string result;          
	char buffer[4096]={0};
	DWORD bytesRead;
	while(1){   //读取管道内的数据
		if (ReadFile(hRead,buffer,4095,&bytesRead,NULL)==NULL) break;
		result+=buffer;
		Sleep(200);
	}
	
	cout<<result<<endl;
    retrun 0;
}
```
* 程序运行结果
```
3
argv1
argv2
argv3
```
emmm  很有意思啊，系统认定的传入参数数量越来越少咯~