---
title: k-shell算法Java实现
date: 2018-12-28 15:06:28
updated: 2018-12-19 17:02:28
categories:
    - Code
tags:
    - 算法
    - 象牙塔
---

## 摘要

* 本文章是基于个人对k-shell算法的理解，若有偏颇还望指正。
* 基于java实现的k-shell算法，文末附源码，仅供学习。

![](xiaoling.jpg)

<!--more-->
## k-shell算法
* 从数据结构图中逐步删除节点权值小于某阈值的节点以及其相关路径的算法

* K-shell 方法递归地剥离网络中度数小于或等于 k 的节点,具体划分过程如下: 假设网络中不存在度数为 0 的孤立节点。从度指标的角度分析,度数为 1的节点是网络中最不重要的节点,因此首先将度数为 1 的节点及其连边从网络中删除。删除操作进行之后的网络中会出现新的度数为 1 的节点,接着将这些新出现的度数为 1 的节点及其连边删除。重复上述操作,直到网络中不再新出现度数为 1的节点为止。此时所有被删除的节点构成第一层,即 1-shell,节点的 Ks 值等于 1。剩下的网络中,每个节点的度数至少为2。继续重复上述删除操作,得到 Ks 值等于 2 的第二层,即 2-shell。依此类推,直到网络中所有的节点都被赋予 Ks 值。
[出处](https://blog.csdn.net/DreamHome_S/article/details/78830943)

* 关于k-shell算法的一些小问题，目前为止本人未能找到足够官方的文章讲解。若有人能指正，感激不尽。

    节点的度在图的定义中可以理解为节点与其他节点的路径数目。在我们删除对应度数节点的过程中势必会又节点度数更大的节点变为度数小的节点。这样节点的度数就会改变，对于这种改变发生而产生的低于当前处理度数k的新节点又应该怎样处理？
    
    本代码对这类节点，这个代码处理时在删除当前所有节点度数小于等于ks值的节点，并把删除节点加入ks值。针对此次提供的数据ks层数最大值为12。实验数据以及实验结果将会附在源代码末尾

## 数据结构安排

**以下代码使用vscode+java环境编写**

* 因为是使用的java语言。这里用一个hashmap来存储图。hashmap中key，value分别为string，以及hashset。
key值中存储图节点，对应value（hashset型）存储与该节点有连接的节点。
    ```
    HashMap<String,HashSet<String>> map=new HashMap<String,HashSet<String>>();
    ```
* 用另一个hashmap来存储对应k值的网络层,key值用来存放k值，value的hashset里面存放从网络中剔除的节点。
    ```
    HashMap<Integer,HashSet<String>> kmap=new HashMap<Integer,HashSet<String>>();
    ```
* 首先第一步从文件中读取数据,这里操作时按行读取。[java文件操作](https://blog.csdn.net/brushli/article/details/12356695)
    ```
    private static void loadfile(String filename){
        map.clear();                                                //将图清空重置
        try {
            File f=new File(filename);                              //尝试打开文件
            FileInputStream fstream=new FileInputStream(f);         
            InputStreamReader ireader=new InputStreamReader(fstream);
            BufferedReader breader=new BufferedReader(ireader);
            String line=""; //存储每一行的数据
            String a="";    //存储逗号左边
            String b="";    //存储逗号右边
            String[] points;//存储行数据分割之后的内容
            HashSet<String> temp=null;
            
            while ((line=breader.readLine())!=null){
                points=line.split(","); //以逗号分割字符串
                a=points[0];
                b=points[1];
                temp=map.get(a);
                if (temp!=null){    //这里的读取操作有点绕
                    temp.add(b);    //将逗号前面的节点当作key值在map中检索，如果存在对应的value
                }                   //就把逗号后面的节点存入这个key值所对应的value中
                else{               //如果不存在，证明map中还没有这个节点的信息。
                    temp=new HashSet<String>(); 
                    temp.add(b);    //则将逗号前面的节点作为key
                    map.put(a, temp);   //逗号后面的节点作为value中的元素存入map中
                }

                temp=map.get(b);    //这里将逗号前后对换位置重复一遍，保证map的key值中包含所有的节点
                if (temp!=null){
                    temp.add(a);
                }
                else{
                    temp=new HashSet<String>();
                    temp.add(a);
                    map.put(b, temp);
                }
            }
            breader.close();
        } catch (Exception e) {
            //TODO: handle exception
            e.printStackTrace();
        }
        
    }
    ```

* 为了方便查看图的变化这里写了一个map的打印函数：
    ```
    private static void mapprint(){
        Iterator iter=map.entrySet().iterator();
        while (iter.hasNext()){
            Map.Entry<String,HashSet<String>> entry=(Map.Entry<String,HashSet<String>>) iter.next();
            System.out.println(entry.getKey()+" "+entry.getValue().toString());
        }
    }
    ```
* 以及存储对应k值剥离出来的网络节点的打印函数
    ```
    private static void printkmap(){
        Iterator iter=kmap.entrySet().iterator();
        while(iter.hasNext()){
            Map.Entry<Integer,HashSet<String>> entry=(Map.Entry<Integer,HashSet<String>>) iter.next();
            System.out.println(entry.getKey().toString()+" "+entry.getValue().toString());
        }
    }
    ```
* k-shell算法。
    * 以图（hashmap为空为终止条件）为空为终止条件的循环。每次删除节点权值小于k的节点以及相应路径，在图（hashmap）一轮遍历完成之后k值+1。
    * 删除的时候要注意，这样的存储结构的设计模式，我们删除一个节点时，要同时把该节点从其他与该节点相连接的节点的value中删除。因此这里的删除函数显得很复杂。
    
    [Hashmap的遍历删除操作](https://www.cnblogs.com/zhangnf/p/HashMap.html)
    
    ```
    private static void kshell_func(){
        int k=1;
        kmap.clear();
        while (!map.isEmpty()){
            HashSet<String> ktemp=new HashSet<String>();
            for (Iterator<Map.Entry<String,HashSet<String>>> iter = map.entrySet().iterator(); iter.hasNext();){
                Map.Entry<String,HashSet<String>> item = iter.next();
                if (item.getValue().size()<=k){
                    ktemp.add(item.getKey())
                    for (String i:item.getValue()){
                        HashSet<String> temp=map.get(i);
                        temp.remove(item.getKey());
                    }
                    iter.remove();
                }
            }
            
            // Iterator iter=map.entrySet().iterator();
            // Map.Entry<String,HashSet<String>> entry=(Map.Entry<String,HashSet<String>>) iter.next();
            // while (iter.hasNext()){
            //     if (entry.getValue().size()<=k){
            //         ktemp.add(entry.getKey());
            //         for (String index:entry.getValue()){
            //             HashSet<String> temp=map.get(index);
            //             temp.remove(entry.getKey());
            //         }
            //         String keytemp=entry.getKey();
            //         entry=(Map.Entry<String,HashSet<String>>) iter.next();
            //         map.remove(keytemp);
            //         System.out.println("delete ...");
            //     }
            // }
            kmap.put(Integer.valueOf(k),ktemp);
            k=k+1;
        }
    }
    ```

## 源码

[数据文件](data.txt)

[结果文件](result.txt)

```
import java.io.BufferedReader;
import java.io.File;
import java.io.FileInputStream;
import java.io.InputStreamReader;
import java.util.HashMap;
import java.util.HashSet;
import java.util.Iterator;
import java.util.Map;
import java.lang.Integer;

public class kshell{
    private static HashMap<String,HashSet<String>> map=new HashMap<String,HashSet<String>>();
    private static HashMap<Integer,HashSet<String>> kmap=new HashMap<Integer,HashSet<String>>();
    private static void loadfile(String filename){
        map.clear();
        try {
            File f=new File(filename);
            FileInputStream fstream=new FileInputStream(f);
            InputStreamReader ireader=new InputStreamReader(fstream);
            BufferedReader breader=new BufferedReader(ireader);
            String line="";
            String a="";
            String b="";
            String[] points;
            HashSet<String> temp=null;
            
            while ((line=breader.readLine())!=null){
                points=line.split(",");
                a=points[0];
                b=points[1];
                temp=map.get(a);
                if (temp!=null){
                    temp.add(b);
                }
                else{
                    temp=new HashSet<String>();
                    temp.add(b);
                    map.put(a, temp);
                }
                temp=map.get(b);
                if (temp!=null){
                    temp.add(a);
                }
                else{
                    temp=new HashSet<String>();
                    temp.add(a);
                    map.put(b, temp);
                }
            }
            breader.close();
        } catch (Exception e) {
            //TODO: handle exception
            e.printStackTrace();
        }
        
    }

    private static void mapprint(){
        Iterator iter=map.entrySet().iterator();
        while (iter.hasNext()){
            Map.Entry<String,HashSet<String>> entry=(Map.Entry<String,HashSet<String>>) iter.next();
            System.out.println(entry.getKey()+" "+entry.getValue().toString());
        }
    }

    private static void kshell_func(){
        int k=1;
        kmap.clear();
        
        while (!map.isEmpty()){
            HashSet<String> ktemp=new HashSet<String>();
            for (Iterator<Map.Entry<String,HashSet<String>>> iter = map.entrySet().iterator(); iter.hasNext();){
                Map.Entry<String,HashSet<String>> item = iter.next();
                if (item.getValue().size()<=k){
                    ktemp.add(item.getKey())
                    for (String i:item.getValue()){
                        HashSet<String> temp=map.get(i);
                        temp.remove(item.getKey());
                    }
                    iter.remove();
                }
            }
            
            // Iterator iter=map.entrySet().iterator();
            // Map.Entry<String,HashSet<String>> entry=(Map.Entry<String,HashSet<String>>) iter.next();
            // while (iter.hasNext()){
            //     if (entry.getValue().size()<=k){
            //         ktemp.add(entry.getKey());
            //         for (String index:entry.getValue()){
            //             HashSet<String> temp=map.get(index);
            //             temp.remove(entry.getKey());
            //         }
            //         String keytemp=entry.getKey();
            //         entry=(Map.Entry<String,HashSet<String>>) iter.next();
            //         map.remove(keytemp);
            //         System.out.println("delete ...");
            //     }
            // }
            kmap.put(Integer.valueOf(k),ktemp);
            k=k+1;
        }
    }

    private static void printkmap(){
        Iterator iter=kmap.entrySet().iterator();
        while(iter.hasNext()){
            Map.Entry<Integer,HashSet<String>> entry=(Map.Entry<Integer,HashSet<String>>) iter.next();
            System.out.println(entry.getKey().toString()+" "+entry.getValue().toString());
        }
    }
    public static void main(String[] args) {
        loadfile("./data.txt");
        //mapprint();
        kshell_func();
        //mapprint();
        printkmap();
        System.out.println("complete");
        
    }
}
```