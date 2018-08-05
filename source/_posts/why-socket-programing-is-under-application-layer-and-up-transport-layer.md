---
title: 为什么套接字编程介于应用层与传输层之间
date: 2013/8/30 10:36:31
categories:
- Shit Done
tags:
- Network 
---
#### 1.首先来说说ISO和OSI：

```
ISO：[国际标准化组织][ISO] International Organization for Standardization

OSI： [开放系统互连模型][OSI] open systems Interconnection 

关系是：ISO提出的一个试图使各种计算机在世界范围内互连为网络的标准框架，简称OSI。OSI又俗称为七层模型。
```
#### 2. 网际网协议族

```
就是我们常说的TCP/IP模型，标准称为：网际网协议族，即五层模型。OSI模型中顶上三层合为一层，称为应用层，这就是我们web客户（浏览器）、Telnet客户、web服务器等其他网络应用所在的层。
```
#### 3. 套接字编程简介

![](http://farm3.staticflickr.com/2866/11066998623_5ca1248efb.jpg)

如上图，UNP给我们讲述的套接字编程接口主要是指应用层进入传输层的接口，当然在五层模型中的传输层我们在TCP和UDP之间有间隙，表明存在网络应用绕过传输层而直接使用IPv4或者IPv6.我们称为`原始套接字（raw socket）`.

```
那为什么套接字编程介于应用层与传输层之间呢？

*   第一个理由是因为应用层太关注于网络应用的细节，而对通信细节了解的很少，底下四层对具体的网络应用关心不多，却处理所有的网络应用通信细节：发送数据，等待确认，给数据排序和计算校验和等。

*   第二个理由是顶上三层构成了操作系统所谓的`用户进程`，而底下四层通常为操作系统内核提供，另物理层和数据链路层通常还于具体的设备有关，属于驱动范围。由此可见，应用层与传输层是构件API的自然位置。
```

<!--more-->
#### 4. 协议数据单元（PDU）

PDU（protocol data unit）:计算机网络各层对等实体间交换的单位信息。也就是每对等层交换数据单元。

```
各层的PDU如下：

*   应用层： application data（应用数据）
*   传输层： message （消息）
    *  其中TCP的PDU又称为segment （TCP报文段）。
*   网络层： IP datagram (IP数据报)。

*   数据链路层： frame （帧）
*   物理层：bit （比特流）
```

注：

```
1.  MSS（maximum segment size）:指应用层与传输层的接口属性，指传输层协议规定的最大分节大小。

2.  MTU（maximum transmission unit）:指传输层与数据链路层的接口属性，指数据链路层协议规定的最大传输单元。
```
