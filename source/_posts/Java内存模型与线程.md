---
title: java内存模型与线程
date: 2020-09-24
categories: [JVM,java内存模型与线程]    #分类
tags: [JVM,java线程]          #标签
toc: true  #是否启用内容索引
top: false #是否置顶 true或者注释
typora-root-url: ..
---

## 概述

&nbsp;&nbsp;&nbsp;&nbsp; 由于计算机的存储设备与处理器的运算速度有几个数量级的差距，所以现代计算机系统都不得不加入一层读写速度尽可能接近处理器运算速度的**高速缓存**（Cache）来作为内存与处理器之间的缓冲：**将运算需要使用到的数据复制到缓存中，让运算能快速进行，当运算结束后再从缓存同步回内存之中，这样处理器就无须等待缓慢的内存读写了**

>   在多处理器系统中，每个处理器都有自己的高速缓存，而它们又共享同一主内存。



![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxNy85LzQvMjBhOTk2ODc0NmFmYTJhZmRlNGIzNzE2YmFiZjU1Y2U_aW1hZ2VWaWV3Mi8wL3cvMTI4MC9oLzk2MC9mb3JtYXQvd2VicC9pZ25vcmUtZXJyb3IvMQ)

## Java内存模型

&nbsp;&nbsp;&nbsp;&nbsp; Java虚拟机规范中试图定义一种Java内存模型（Java Memory Model,JMM）来屏蔽掉各种硬件和操作系统的内存访问差异，以实现让Java程序在各种平台下都能达到一致的内存访问效果。



<img src="https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxNy85LzQvOGY5ODMzMGRjOGFmNGNlOGNmNTM5N2EwMTMzMDhlYzI_aW1hZ2VWaWV3Mi8wL3cvMTI4MC9oLzk2MC9mb3JtYXQvd2VicC9pZ25vcmUtZXJyb3IvMQ">

>   这里所讲的主内存、工作内存与我们通常所讲的Java内存区域中的Java堆、栈、方法区等并不是同一个层次的内存划分，这两者基本上是没有关系的，如果两者一定要勉强对应起来，那从变量、主内存、工作内存的定义来看，主内存主要对应于Java堆中的对象实例数据部分，而工作内存则对应于虚拟机栈中的部分区域。



### 主内存与工作内存间的交互操作

&nbsp;&nbsp;&nbsp;&nbsp; java内存模型中定义了以下8种操作来完成，虚拟机实现时必须保证下面提及的每一种操作都是原子的、不可再分的。

| 操作          | 作用对象 | 解释                                       |
| ----------- | ---- | ---------------------------------------- |
| lock (锁定)   | 主内存  | 把一个变量标识为一条线程独占的状态                        |
| unlock (解锁) | 主内存  | 把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定        |
| read (读取)   | 主内存  | 把一个变量的值从主内存传输到线程工作内存中，以便 load 操作使用       |
| load (载入)   | 工作内存 | 把 read 操作从主内存中得到的变量值放入工作内存中              |
| use (使用)    | 工作内存 | 把工作内存中一个变量的值传递给执行引擎，每当虚拟机遇到一个需要使用到变量值的字节码指令时将会执行这个操作 |
| assign (赋值) | 工作内存 | 把一个从执行引擎接收到的值赋接收到的值赋给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作 |
| store (存储)  | 工作内存 | 把工作内存中的一个变量的值传送到主内存中，以便 write 操作         |
| write( 写入)  | 主内存  | 把 store 操作从工作内存中得到的变量的值放入主内存的变量中         |

### volatile 关键字

>   关键字 volatile 是 Java 虚拟机提供的最轻量级的同步机制

当一个变量定义为volatile之后，它将具备两种特性:

1.  **保证此变量对所有线程的可见性**
2.  **禁止指令重排序优化**

>   “内存可见性”是指当一条线程修改了这个变量的值，新值对于其他线程来说是可以立即得知的。而普通变量不能做到这一点，普通变量的值在线程间传递均需要通过主内存来完成。
>
>   
>
>   禁止指令重排序优化是指：通过插入内存屏障保证一致性。

