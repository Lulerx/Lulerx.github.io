---
title: JDBC笔记
date: 2020-10-23
categories: Java核心    #分类
tags: jdbc        #标签
toc: true  #是否启用内容索引
top: false #是否置顶 true或者注释
typora-root-url: ..
---

## 引言

JDBC的全称是Java Database Connectivity，即Java数据库连接。使用JDBC对数据库进行操作，只需要三个基本工作：

+   建立与数据库的连接
+   执行SQL语句
+   获得SQL语句的执行结果

## 1. JDBC 编程步骤

**1、加载数据库驱动**

通常使用Class类的forName()静态方法来加载驱动。（这是一种反射的方式）

不同数据库使用不同的驱动，加载MySQL的驱动采用如下代码：

~~~java
//加载MySQL的驱动
Class.forName("com.mysql.jdbc.Driver");
~~~

而加载Oracle 的驱动则如下：

~~~java
//加载Oracle的驱动
Class.forName("oracle.jdbc.driver.OracleDriver");
~~~

**2、通过DriverManager获取数据库连接。DriverManager提供了如下方法：**

~~~java
//获取数据库连接
DriverManager.getConnection(String url, String username, String password);
~~~

其中，username 是数据库的连接账号，password是数据库的连接密码

而url是数据库的连接URL，

