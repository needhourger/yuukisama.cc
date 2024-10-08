---
title:          "计算机存储单位详解"
subtitle:       ""
description:    ""
date:           2023-03-07T13:54:47+08:00
image:          ""
tags:           []
categories:     []
draft: false
---
# 计算机存储单位详解

本文将通过较为接近基础原理的方式讲解计算机中有关存储容量单位的相关知识点。

  _本章节阅读前需要的知识储备：九年之义务教育的数学计算能力以及语文阅读能力。_

不论你是不是计算机相关专业的人员，相信你都听说过一句话：**计算机世界是1和0的世界**（~~不是那个01啊喂！~~），因此本文将会涉及到非常多的2的次方运算，准确的说，计算机存储容量的换算方式就是基于2的多少次方进行的，即2的幂运算，即二进制。

## 二进制

我们常规的十进制运算是满10进1，理解这个概念后可以类比2进制，即满2进1。例如如果我们用十进制数数是这样的：
```
0，1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13， 14, 15, 16 ……
```
如果用二进制的方式数数就是（下面我们数到了16,和上面一样，每一位都可以对照起来看）：
```
0, 1, 10, 11, 100, 101, 110, 111, 1000, 1001, 1010, 1011, 1100, 1101, 1110, 1111, 10000 ……
```

## 位 bit 比特

**位（bit）计算机中最小的计量单位——比特**

我们在电脑中所看到的任何东西，文本，网页，图片，音频，视频……它们归根结底存储在计算机的磁盘中都是一串0和1组成的数字。最早的时候，在第一台计算机诞生时，我们用打孔纸带表示01，现在我们在计算机里用电压高地，逻辑门开关表示01。

计算机中，一位数字0,或者一位数字1，即一个二进制位，我们称之为 “**位**”，英文写作**bit**，中文写作“**比特**”（**必须是小写的b，大写B和小写b是两个不同的单位**），可以简写成 1 bit。1bit的数据就是一位只能表示0,或者1。

## 字节Byte

**8个二进制位，即 8 bit 记为1Byte， 1字节**

在早期计算机中，计算机科学家们定义了一套编码表，把二进制数字和一系列字母符号对应起来并一直沿用至今称之为“**ASCII码表**”（ American Standard Code for Information Interchange，美国信息交换标准代码），就像电报一样，我们规定三长三短表示SOS,在ASCII码表里，我们规定 1000001 表示大写字母 A。 以此类推，将大小写英文字母以及一些常用符号映射成了二进制。而恰好这个编码表每一个数据的长度都是8个二进制位，即8bit表示一个字母，所以我们把 8bit 记作 1Byte字节。

*题外话，综上在某些时候的计算机内一个字母一个字符的长度就是1 Byte，8 bit的长度。而早期汉字编码则是使用两个字节来表示一个汉字，即 2Byte, 16bit*

[详细的ASCII码表](http://c.biancheng.net/c/ascii)

## 基础容量单位换算

在弄明白 bit 和 Byte 后，恭喜这个只是点的地基已经被打好了。在这两个单位基础上我们总结一下常见计算机单位容量换算机制：

-|-|-
---|---|---
8 bit（8比特）| 1Byte | 1字节 
1024 Byte | 1 KB （KiloByte）| 1千字节
1024 KB | 1 MB （MegaByte）|1兆字节
1024 MB | 1 GB  （GigaByte）|1吉字节
1024 GB | 1 TB （TeraByte）|1太字节
1024 TB | 1 PB （PetaByte）|1拍字节
1024 PB | 1 EB （ExaByte）| 1艾字节
1024 EB | 1 ZB （ZetaByte）| 1泽字节
1024 ZB | 1 YB （YottaByte）| 1尧字节
1024 YB | 1 BB（Brontobyte）|1珀字节
1024 BB | 1 NB （NonaByte） |1诺字节
1024 NB | 1 DB （DoggaByte）|1刀字节

目前常用到的单位是从bit到PB. 除去bit到字节是 8 的换算，其他都是 1024 倍的关系，即 2^10 二的十次方。

## 实际硬盘容量

在说到内存，或者是现存大小的时候，以上单位大小是准确的。但是常接触硬盘的人可能会有疑问，因为标称1TB的硬盘到手总是可能只有900GB左右。

这点是因为硬盘厂商为了方便硬盘的制造，都有自己的换算方式（~~缺斤少两~~）。比如我们购买一个标称1TB的硬盘，到手可能只有930GB左右。硬盘厂商实际上是这么算的。

- 同样是以8bit为1Byte,到这里还没有问题。
- 但是到了Byte 到 KB的时候，以及KB 到 MB, MB到GB的时候厂商都是以1000计的，而不是1024。于是乎我们可以列出公式得出：
  ```
  厂商： 1TB = 1Byte * 1000 * 1000 * 1000 * 1000 = 10^12 = 1000000000000 Byte
  ```
  而这 10^12次方 Byte 用真正的 1024 的方法换算即:

  ```
  10^12 Byte / 1024 = 976562500 KB

  976562500 KB / 1024 = 953674.31640625 MB

  953674.31640625 MB /1024 = 931.3225746154785 GB
  ```

于是乎你就做到了买1TB的硬盘实际到手 931GB。

## 速度单位

在之前的章节之中，我们强调了

**Byte 和 bit, 大写B和小写b是两个完全不同的单位！**

**Byte 和 bit, 大写B和小写b是两个完全不同的单位！**

**Byte 和 bit, 大写B和小写b是两个完全不同的单位！**

这里重要的事情说三遍！！！

在描述计算机网络速度的时候，我们不再以字节Byte，大B为基准，而是以比特bit,小b为基准。并由此诞生了一系列的新的速度单位：

- bps（bit per second）比特每秒，也称作比特率
- Kbps （kilobit per second）千比特每秒
- Mbps （Megabit per second）兆比特每秒
- Gbps
- Tbps
- ……

当然其实有时候为了更方便常规的认知，我们也有以Byte，大写B为基准的速度单位：

- Byte/s （Byte per second）字节每秒
- KB/s （KiloByte per second）千字节每秒
- MB/s  （MegaByte per second）兆字节每秒
- ……

其他单位以此类推。

**综上所述，kb和kB是两个完全不同的单位！一个是千比特一个是千字节（k的大小写其实是无所谓的）Mb和MB也是两个完全不同的单位，一个是兆比特，一个是兆字节。**

## 速率换算

在有了之前的容量换算的基础后，我们可以很轻松的推理速率换算之间的关系。首先你需要搞清楚其单位究竟是大B还是小b,是基于字节Byte的还是基于bit的。除去比特和字节的差异，千K到兆M，兆M到吉G还是和容量单位一样基于1024换算。

**8 bit = 1 Byte**

-|-|-
---|---|---
1 Byte/s | 8 bps
1 KB/s | 1024 * 8 =  2^10 * 2^3 = 2^13 bps| 8 Kbps
1 MB/s | 1024 * 1024 * 8 = 2^23 bps | 8Mbps
1 GB/s | 1024^3 * 8 = 2^33 bps | 8Gbps

*Kbps也可以写作Kb/s,但是和KB/s不是同一个单位，同样数字的情况下后者是前者的8倍大，当然KB/s也可以写作KBps,其他单位以此类推。*

以此类推，我们可以得到，大写B到小写b单位的转换都是以8为单位换算的。

## 运营商的速度概念

### 千兆宽带

在日常生活里我们常常有听到这样的宣传词，尤其是办理宽带的时候：

    我们的宽带是千兆宽带，万兆宽带，或者是百兆宽带。

其实这里运营商们玩了个文字游戏，这里说的兆是指Mb而非MB

让我们来用所谓的千兆宽带为例，即运营商嘴里的1000M宽带，来算算其真正的实际速率。

```
这里的 1000M 即 1000Mbps

1000 Mbps = 1000 * 1024 （kbps） = 1000 * 1024 * 1024 bps

而 8 bit = 1 Byte， 所以：

1000 Mbps = 1000 * 1024 * 1024 / 8（MB/s）= 125 MB/s


当然其实可以简单点直接除以8,即：
1000Mb = 1000 / 8 = 125MB/s
```

### 2.5G网口

第二个离子是2.5G网口，即我们目前可以买到的较为新的电脑基本都配备了一个2.5G网口，那么这个网口的真实最大下载速度是多少呢？

```
2.5G网口即 2.5Gbps 网口

2.5Gbps = 2.5 * 1024 (Mbps) = 2.5 * 1024 / 8 (MB/s) = 320MB/s
```

### 万兆

那如果是万兆呢，即10000Mbps
```
10000 Mbps = 10000 / 8 （MB/s） = 1250 MB/s
```
~~勉强达到了1GB每秒的下载速度~~


### 速率总结

我们常规听到的宣传网络速度，网络带宽的单位事实上都是基于比特bit的单位，例如：

- 5G网络的峰值速度是10-20Gbps。
- 宽带速度是1000Mbps
- 服务器带宽是10Mbps





