---
title: 深入浅出Nodejs读书笔记
date: 2021-07-13 00:06
tags: [总结,nodejs,读后感]
---

为了系统的学习Node，三思之后买了本《深入浅出Nodejs》，由国内Nodejs布道者”朴灵“大神编写。 虽然本书是13年编写，距离现在已经有8年之久，但是阅读下来，发现还是相当有启发（可能之前对Node了解确实太少了）。

看了本书，不仅对Node能有个比较深刻的了解，甚至还补充了前端的一些知识：如V8内存、TCP、UDP协议等。很有收获

<! -- more -->

# 正文

## 第一章： Nodejs简介

Nodejs由2009年3月诞生于Ryan Dahl之手。起初目的是为了开发一个高性能的web服务器。Ryan Dahl在众多语言中选中JavaScript有其中几点原因：

1. 在浏览器大战中脱颖而出的V8，给JavaScript带来了足够的性能
2. Javascript非常适合Ryan Dahl在寻求的事件驱动、非阻塞I/O
3. JavaScript在服务器环境历史几乎为0，没有历史包袱。

### node的跨平台

起初，Nodejs是只支持\*nux环境，但后来微软看到了Node的潜力，组建了个团队，让Node兼容windows平台。

随着Node的发展。Nodejs底层使用`libuv`来实现跨平台的兼容。所以当面试问道Node事件循环时，离不开`libuv`的原因。

## 第二章：模块机制

看完这一篇，终于彻底的理解了commonJS、AMD、CMD的规范了。

服务端开发，是离不开模块化的。而Javascript则没有模块化的概念。于是社区总结出一套commonJS规范

commonJS概念大概是：

1. 每个js文件都是一个模块
2. 每个模块会有module、export、require对象
3. 每个模块相互独立
4. 加载过后的模块Node会进行缓存。

commonJS与AMD、CMD

modules、export、require从哪来的

模块加载顺序

## 第三章：异步I/O

Node异步的实现，不是操作系统的异步IO、是利用线程实现的虚假异步IO

定时器、process.nextTick、setImmediate的关系，以及执行流程

## 第四章：异步编程

高阶函数与偏函数的概念，并不是常见的。是js特有的。

提到了Promise/A与其前辈Promise/Deffered

顺便介绍介绍目前以及沉淀了很久的express与koa2？

## 第五章：内存控制

node的内存限制（来自V8的限制）

具体的回收过程（新生代，老生代）

不影响V8内存的办法：Buffer与stream（堆外内存）



## 第六章：理解Buffer

类似数组，一个元素是16进制的两位数，也就是0~255；

slab分配机制，以8KB为单位分配内存。也就是8\*1024。
大于8KB的是大对象，反之是小对象。也就是`new Buffer(8*1024)`以上是大对象

再理解 highWaterMark参数

## 第七章：网络编程

什么是套接字？（虽然文中没有解释，但是有空的话可以彻底解决这个问题）

tcp服务demo

tcp存在发送延迟的问题。因为Nagle算法。

nc命令

udp demo

http demo

websocket简介

TSL/SSL与网路安全（整数作用、加密过程）

## 第八章：构建web应用

说的是比较常规的web开发。

可以把koa与express的开发结合本文说一下。

cookie、session

路由解析

缓存

数据上传

中间件

模板引擎

## 第九章：玩转进程

比较难，比较底层

child_process与cluster

## 第十章： 测试

可以简单说一下：TDD、BDD两种区别

Node调试放到这个地方、

## 第十一章：产品化

这个没啥好说了。

## 总结

