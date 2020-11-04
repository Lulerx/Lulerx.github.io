---
title: SpringMVC工作原理
date: 2020-10-23
categories: [Java框架, SpringMVC]    #分类
tags: SpringMVC        #标签
toc: true  #是否启用内容索引
top: false #是否置顶 true或者注释
typora-root-url: ..
---

## 1. SpringMVC的工作原理

![springmvc.jpg](images/javaEE/springmvc.jpg)

>   1、  用户发送请求至前端控制器DispatcherServlet。
>
>   2、  DispatcherServlet收到请求调用HandlerMapping处理器映射器。
>
>   3、  处理器映射器找到具体的处理器(可以根据xml配置、注解进行查找)，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet。
>
>   4、  DispatcherServlet调用HandlerAdapter处理器适配器。
>
>   5、  HandlerAdapter经过适配调用具体的处理器(Controller，也叫后端控制器)。
>
>   6、  Controller执行完成返回ModelAndView。
>
>   7、  HandlerAdapter将controller执行结果ModelAndView返回给DispatcherServlet。
>
>   8、  DispatcherServlet将ModelAndView传给ViewReslover视图解析器。
>
>   9、  ViewReslover解析后返回具体View。
>
>   10、DispatcherServlet根据View进行渲染视图（即将模型数据填充至视图中）。
>
>   11、 DispatcherServlet响应用户。

## @RequestMapping

@RequestMapping可以接收REST风格的URL，常用的有如下几种:

+   /user/*/show：匹配 /user/aaa/show、 /user/bbb/show 等URL
+   /user/**/show：匹配 /user/aaa/show、 /user/123/bbb/show 等URL
+   /user/{userId}：匹配 /user/123、 /user/234 等URL
+   /user/**/{userId}：匹配 /user/aaa/abc/123、 /user/aaa/234 等URL

~~~java
//标准的URL格式，只匹配/user/cresteUser
@RequestMapping(value="/user/cresteUser")
public String cresteUser(User user){
  return "index"
}
~~~

~~~java
//使用占位符，URL中的{xxx}占位符可以通过@PathVariable("xxx")绑定到方法的参数中
@RequestMapping(value="/user/{userId}")
public String show(@PathVariable("userId")String id){
  return id
}
~~~

