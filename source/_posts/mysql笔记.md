---
typora-root-url: D:\Devsoft\hexo\source
title: mysql笔记
categories: 随笔    #分类
tags: 基础          #标签
toc: true  #是否启用内容索引
---

# mysql笔记

## 常用数据类型

数值类型

| 类型      | 有符号            | 无符号      |
| ------- | -------------- | -------- |
| tinyint | -128~127       | 0~255    |
| int     | -2^31 ~ 2^31-1 | 0~2^32-1 |

浮点型

| 类型     | 大小   |
| ------ | ---- |
| float  | 4字节  |
| double | 8字节  |

时间日期

| 类型        | 格式                            |
| --------- | ----------------------------- |
| Datetime  | YYYY-mm-dd HH:MMss            |
| Date      | YYYY-mm-dd                    |
| Time      | HH:MM:ss                      |
| Timestamp | 时间戳，从1970/1/1开始，格式与Datetime一致 |

字符串型

| 格式      | 长度              |
| ------- | --------------- |
| char    | 最大长度255         |
| varchar | 可变字符串，最大长度65536 |
| Text    | 通常超过255就会使用     |

---



## 数据库语法

**查询已有数据库列表：**

~~~mysql
show databases;
~~~

**创建数据库：**

~~~mysql
create database 数据库库名 [选项];
~~~

**查看建库语句：**

~~~mysql
show create database 数据库名;
~~~

**修改数据库(数据库名不可更改)：**

~~~mysql
alter database 数据库名 [选项];
~~~

**删除数据库：**

~~~mysql
drop database [if exists] 数据库名;
~~~

**进入/使用数据库：**

~~~mysql
use 数据库名;
~~~



## 数据库约束

| 约束类型 | 语法             | 举例            |
| ---- | -------------- | ------------- |
| 非空约束 | not null       |               |
| 唯一约束 | unique         |               |
| 主键约束 | primary kry    |               |
| 条件约束 | check          | $ check(id>0) |
| 默认值  | default        | $ default ‘ ‘ |
| 自增约束 | auto_increment |               |

>表级约束：
>
>字符集：charser/ character set		具体字符集
>
>校对集：collate	具体校对集
>
>存储引擎：engine		具体存储引擎（InnoDB   MyISAM）

---



## 表语法

**例子：**

~~~mysql
create table t_user(
	id int primary key auto_increment,
  	name char(32) default '',
  	age int
)engine=InnoDB charset=utf8
~~~

**查看所有表：**

~~~mysql
show tables;
~~~

**查询带关键字的表：**

~~~mysql
show tables like '%关键字%';
~~~

**查看建表语句：**

~~~mysql
show create table 表名;
~~~

**查看表结构：**

~~~mysql
#三种方式:
desc 表名;
describe 表名;
show columns from 表名;
~~~

**删除表（可多表删除）：**

~~~mysql
drop table [if exists] 表名1，表名2...;
~~~



**修改表名：**

~~~mysql
rename table 旧表名 to 新表名;
~~~

**修改表选项（字符集，校对集，存储引擎）:**

~~~mysql
alter table 表名 表选项=值;
~~~

**新增表字段：**

~~~mysql
alter table 表名 add 字段名 数据类型 [列属性] [位置];
~~~

**修改表字段：**

~~~mysql
alter table 表名 change 旧字段 新字段名 数据类型 [列属性] [位置];
~~~

**删除字段：**

~~~mysql
alter table 表名 drop 字段名;
~~~

---



## 表数据操作（增删改查）

**插入数据：**

~~~mysql
insert into 表名(字段列表) values(对应字段值列表1),(对应字段值列表2)...
~~~

**删除数据：**

~~~mysql
delete from 表名 [where 条件];
~~~

**更新数据：**

~~~mysql
update 表名 set 字段=值 [where 条件];
~~~

**查询数据：**

~~~mysql
select */字段列表 from 表名 [where 条件];
~~~

---



## 查询操作

### **where条件查询**

where的字符串查询不区分大小写，若要区分则用**binary**

~~~mysql
select * from t_user where binary name = 'zhangsan';
~~~

### **like模糊查询**

~~~mysql
select * from t_user where name like 'zhang%';
~~~

>占位符：
>
>​	% : 匹配0个或多个字符
>
>​	_ : 匹配1个字符

### **order by 排序**

默认为升序 ASC，倒序使用 DESC

~~~mysql
select * from t_user order by name desc;
select * from t_user order by age desc name ASC;
~~~

> 使用多个字段排序时，按先后顺序
>
> 按第一字段规则排序时，有重复的数据，再根据第二字段规则排序，以此类推。
>
> 且多个字段时，要显式引用ASC/DESC，否则为默认ASC

### in 包含

~~~mysql
select * from t_user where id in(1,3,5);
~~~

### between and

~~~mysql
#查询id在1到5之间的记录（包含1，包含5）
select * from t_user where id between 1 and 5;
~~~

### limit 分页

~~~mysql
#查询6~15行数据
select * from t_user limit 5 10;
~~~



### **distinct 去除重复**

~~~mysql
select distinct * from t_user;
~~~

### 聚合函数

| 函数名   | 作用      |
| ----- | ------- |
| AVG   | 返回平均值   |
| count | 返回总记录条数 |
| sum   | 返回数值总和  |
| min   | 返回最小值   |
| max   | 返回最大值   |

> 聚合函数无法与where 一起使用

### group by 分组函数

分组函数是根据一个或多个字段的唯一组合结果集进行分组

通常结合**聚合函数**一起使用

<img src="/images/table1.jpg">

~~~mysql
#统计男女人数
select sex as '性别',count(sex)as '人数' 
from t_user 
group by sex;
~~~

结果：

![group1](/images/group1.jpg)

> 根据性别的唯一组合进行分组，分成 男 女 两组，再结合count()函数进行统计



**对字段分组：**

~~~mysql
#统计不同年龄的男女人数
select sex as '性别',age as '年龄',count(sex)as '人数' 
from t_user 
group by sex,age;
~~~

结果：

![group2](/images/group2.png)

> 根据sex, age两个字段进行分组，根据不同值进行唯一组合，再结合聚合函数进行统计

### having 条件

因为聚合不能与where一起使用，所以有了having

having是用于分组后返回满足条件的数据

~~~mysql
#统计不同年龄的男女人数,并要求年龄不小于19
select sex as '性别',age as '年龄',count(sex)as '人数' 
from t_user 
group by sex,age having age>=19;
~~~

![having1](/images/having1.jpg)

### 连接查询



**内连接 inner join**

![inner2](/images/inner2.png)

**左外连接 left join**

![left_join](/images/left_join.png)

**右外连接 right join**

![right_join](/images/right_join.png)



以员工表和部门表为例，dept代表该员工所在的部门：

t_employee表

![inner1](/images/inner1.png)

t_dept表

![t_dept](/images/t_dept.png)

内连接查询：

~~~mysql
 #查询出员工姓名以及其对应的部门名称
 select 
 e.empName,d.deptName
 from t_employee e
 INNER JOIN t_dept d
 ON e.dept = d.id;
~~~

结果：

![inner3](/images/inner3.png)

在内连接中赵七没有被查出来，因为他没有对应的部门，现在想要把赵七也查出来，就要使用**左外连接查询**：

~~~mysql
 #查询所有员工姓名以及他所在的部门名称
 select e.empName,d.deptName
 from t_employee e
 left join t_dept d 
 on d.id = e.dept;
~~~

结果：

![left2](/images/left2.png)

**右外连接查询**与做查询同理，只是基准表的位置发生了变化：

~~~mysql
 SELECT e.empName,d.deptName
 from t_employee e
 right join t_dept d 
 on d.id = e.dept;
~~~

结果：

![right2](/images/right2.png)

>总结：
>
>左连接查询时，左表数据全部显示，若右表没有对应数据，则显示为null；
>
>右连接查询时，右表数据全部显示，若左表没有对应数据，则显示为null；



**自连接查询**

自连接查询就是当前表与自身的连接查询，关键点在于虚拟化出一张表给一个别名

~~~mysql
#查询员工以及他的上司的名称，由于上司也是员工，所以这里虚拟化出一张上司表
SELECT e.empName,b.empName
from t_employee e
LEFT JOIN t_employee b
ON e.bossId = b.id;
~~~

结果：

![self1](/images/self1.png)

**全外连接 union**

> mysql是没有全外连接的(mysql中没有full outer join关键字)，想要达到全外连接的效果，可以使用union关键字连接左外连接和右外连接

---



## 视图

**视图就是一张数据库表，根据表或者视图为基准创建视图。视图本身不存储任何数据，只是将结果集组合在一起。**

**视图的优点：**

1. 提高重用性，简化复杂查询。将复杂的查询结果创建成视图，以后直接查询视图
2. 对数据表进行重构，却不影响程序运行。因为某需求将表拆分后，可通过视图将拆分的表结果结合在一起，而不用去更改后端代码。
3. 提高了安全性能。可以根据不同的用户权限，设定不同的视图显示。
4. 让数据更清晰。想要什么样的数据，就创建什么样的视图。

**缺点：**

1. 性能相对较差。查询数据相对慢一些，特别是当视图基于其他视图创建的。
2. 表依赖关系。当表结构更改时，就必须要更改视图。

### 视图语法

> 视图名一般以“v_"开头，便于区分数据库表

**创建视图：**

~~~mysql
create view 视图名(字段1，字段2...)
AS select 字段1，字段2...
from ...(表查询语句)
~~~

**修改视图：**

~~~mysql
create or replace view 视图名(字段1，字段2...)
AS select 字段1，字段2...
from ...(表查询语句)
~~~

> 视图创建后就相当于一张数据库表，可用表的语法对其进行修改操作

> 注意：更改视图的数据，基表的数据也会随之改变



## 索引

**排好序的快速查找数据结构**

