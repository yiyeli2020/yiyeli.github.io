---
title: 计算机网络概念I
date: 2018-08-24 10:08:12
categories: 2018年8月
tags: [Computer Network]

---
# 计算机网络概念I

总结计算机网络概念



<!-- more -->



## （一）请简述TCP\UDP的区别

TCP和UDP是OSI模型中的运输层中的协议。TCP提供可靠的通信传输，而UDP则常被用于让广播和细节控制交给应用的通信传输。

两者的区别大致如下：

TCP面向连接，UDP面向非连接即发送数据前不需要建立链接
TCP提供可靠的服务（数据传输），UDP无法保证
TCP面向字节流，UDP面向报文
TCP数据传输慢，UDP数据传输快
如果还不是太了解这两者的区别，点击阅读：[TCP与UDP的区别](https://blog.csdn.net/yipiankongbai/article/details/24435977)

## （二）请简单说一下你了解的端口及对应的服务？


![image](http://ww1.sinaimg.cn/large/0071ouepgy1fuqwow0id9j30cb06gq38.jpg)

了解更多的端口，点击阅读：[常用端口号与对应的服务以及端口关闭](http://blog.sina.com.cn/s/blog_66ea0e2801011vb3.html)

## （三）说一说TCP的三次握手

在TCP/IP协议中，TCP协议提供可靠的连接服务，连接是通过三次握手进行初始化的。三次握手的目的是同步连接双方的序列号和确认号并交换 TCP窗口大小信息。

下面详细说一下三次握手（来自简析TCP的三次握手与四次分手）
![image](http://ww1.sinaimg.cn/large/0071ouepgy1fuqwq2b646j30k00mb0u1.jpg)

![image](http://ww1.sinaimg.cn/large/0071ouepgy1fuqwrg1lunj30gh07b75e.jpg)

更加深入的了解TCP的三次握手与四次分手：[简析TCP的三次握手与四次分手](https://www.jellythink.com/archives/705)


## （四）有哪些私有（保留）地址？

A类：10.0.0.0 - 10.255.255.255
B类：172.16.0.0 - 172.31.255.255
C类：192.168.0.0 - 192.168.255.255

## （五）IP地址分为哪几类？简单说一下各个分类

![image](http://ww1.sinaimg.cn/large/0071ouepgy1fuqwscdo7yj30ix06i0td.jpg)
IPv6 -- 采用128bit，首部固定部分为40字节。

## （六）在浏览器中输入网址之后执行会发生什么？


查找域名对应的IP地址。这一步会依次查找浏览器缓存，系统缓存，路由器缓存，ISPNDS缓存，根域名服务器
浏览器向IP对应的web服务器发送一个HTTP请求
服务器响应请求，发回网页内容
浏览器解析网页内容

更加详细的一种说法（以百度为例）（来自计算机网络之面试常考 - 牛客网）
![image](http://ww1.sinaimg.cn/large/0071ouepgy1fuqwt38bprj30ga0an40f.jpg)

如果你想要更加深入的了解这个过程，点击阅读：[从输入网址到显示网页的全过程分析]()
## （七）简单解释一些ARP协议的工作过程

![image](http://ww1.sinaimg.cn/large/0071ouepgy1fuqwu74fpuj30go0afjt2.jpg)

以上的说明解释来自（思想时光机），如果你想了解：[ARP协议工作原理](https://blog.csdn.net/microtong/article/details/3029931)

## （八）说一说OSI七层模型

![image](http://ww1.sinaimg.cn/large/0071ouepgy1furg8ainkrj30k00scad0.jpg)

了解OSI七层模型，请点击阅读：[OSI七层模型详解 （下面的图片来自啊该网址）](https://blog.csdn.net/yaopeng_2005/article/details/7064869)


## （九）说一说TCP/IP四层模型
如果你不了解，请直接点击阅读：[TCP/IP四层模型](http://www.cnblogs.com/BlueTzar/articles/811160.html)

## （十）HTTP 协议包括哪些请求？

GET：对服务器资源的简单请求
POST：用于发送包含用户提交数据的请求
------------以及------------

HEAD：类似于GET请求，不过返回的响应中没有具体内容，用于获取报头
PUT：传说中请求文档的一个版本
DELETE：发出一个删除指定文档的请求
TRACE：发送一个请求副本，以跟踪其处理进程
OPTIONS：返回所有可用的方法，检查服务器支持哪些方法
CONNECT：用于ssl隧道的基于代理的请求
## （十一）简述HTTP中GET和POST的区别

从原理性看：

根据HTTP规范，GET用于信息获取，而且应该是安全和幂等的
根据HTTP规范，POST请求表示可能修改服务器上资源的请求

从表面上看：

GET请求的数据会附在URL后面，POST的数据放在HTTP包体
POST安全性比GET安全性高