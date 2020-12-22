---
title: mysql-Explain性能分析
categories: [mysql]   #分类
tags: [mysql]         #标签
toc: true  #是否启用内容索引
top: false #是否置顶 true或者注释
date: 2020-12-22
typora-root-url: ..
---

### 概念

使用EXPLAIN 关键字可以模拟优化器执行SQL 查询语句，从而知道MySQL 是如何处理你的SQL 语句的。分
析你的查询语句或是表结构的性能瓶颈。

**用法：Explain+SQL 语句；**

Explain 执行后返回的信息：

![img](https://img2020.cnblogs.com/blog/1506061/202012/1506061-20201222164544897-296518114.png)

### explain 字段分析

| 字段            | 含义            |
| ------------- | ------------- |
| id            | 选择标识符         |
| select_type   | 表示查询的类型       |
| table         | 输出结果集的表       |
| partitions    | 匹配的分区         |
| type          | 表示表的连接类型      |
| possible_keys | 表示查询时，可能使用的索引 |
| key           | 表示实际使用的索引     |
| key_len       | 索引字段的长度       |
| ref           | 列与索引的比较       |
| rows          | 扫描出的行数(估算的行数) |
| filtered      | 按表条件过滤的行百分比   |
| Extra         | 执行情况的描述和说明    |

### 字段属性分析

**一、id**

select 查询的序列号,包含一组数字，表示查询中执行select 子句或操作表的顺序。

1.   **id 相同**，执行顺序由上至下；
2.   **id 不同**，如果是子查询，id 的序号会递增，**id 值越大优先级越高，越先被执行**；
3.   **id 有相同也有不同**，id 如果相同，可以认为是一组，从上往下顺序执行；在所有组中，id 值越大，优先级越高，越先执行衍生( DERIVED )

>   【关注点】id 号每个号码，表示一趟独立的查询。一个sql 的查询趟数越少越好。

---



**二、select_type**

select_type 代表查询的类型，主要是用于区别普通查询、联合查询、子查询等的复杂查询。

+   **SIMPLE**：简单的select 查询,查询中不包含子查询或者UNION
+   **PRIMARY**：  查询中若包含任何复杂的子部分，最外层查询则被标记为Primary
+   **DERIVED**：  在FROM 列表中包含的子查询被标记为DERIVED(衍生)。MySQL 会递归执行这些子查询, 把结果放在临时表里。
+   **SUBQUERY**：  在SELECT或WHERE列表中包含了子查询
+   **DEPEDENT SUBQUERY**：在SELECT或WHERE列表中包含了子查询,子查询基于外层
+   **UNCACHEABLE SUBQUERY**：  无法使用缓存的子查询
+   **UNION**：  若第二个SELECT出现在UNION之后，则被标记为UNION；若UNION包含在FROM子句的子查询中,外层SELECT将被标记为：DERIVED
+   **UNION RESULT**：从UNION表获取结果的SELECT

---

**三、table**

表示这个数据是基于哪张表的。



**四、type**

type 是查询的访问类型。是较为重要的一个指标，结果值从最好到最坏依次是：

system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > **range** > **index** > **ALL**

**不过在实际应用中常用到的只有：system > const > eq_ref > ref > range > index > ALL**

+   **system**：表只有一行记录（等于系统表），这是 const 类型的特列，平时不会出现，这个可以忽略不计
+   **const**：表示通过索引一次就找到了, const 用于比较 primary key 或者 unique 索引。因为只匹配一行数据，所以很快。例如将主键置于 where 列表中，MySQL 就能将该查询转换为一个常量。
+   **eq_ref**：唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描。
+   **ref**：非唯一性索引扫描，返回匹配某个单独值的所有行
+   **range**：只检索给定范围的行,使用一个索引来选择行。key 列显示使用了哪个索引一般就是在你的 where 语句中出现
    了 between、<、>、in 等的查询这种范围扫描索引扫描比全表扫描要好，因为它只需要开始于索引的某一点，而
    结束语另一点，不用扫描全部索引。
+   **index**：出现 index 是 sql **使用了索引但是没用通过索引进行过滤**，一般是使用了**覆盖索引**或者是**利用索引进行了排序分组**。
+   **ALL**：Full Table Scan，将遍历全表以找到匹配的行。

>   【提示】
>
>   一般来说，得保证查询至少达到 range 级别，最好能达到 ref。

---

**五、 possible_keys**

显示可能应用在这张表中的索引，一个或多个。查询涉及到的字段上若存在索引，则该索引将被列出，**但不一定被查询实际使用**。



**六、key**

实际使用的索引。如果为NULL，则没有使用索引。



**七、key_len**

表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度。key_len 字段能够帮你检查是否充分的利用上了索引。**ken_len 越长，说明索引使用的越充分。**



**八、ref**

显示索引的哪一列被使用了，如果可能的话，是一个常数。**哪些列或常量被用于查找索引列上的值**。



**九、rows**

**rows 列显示MySQL 认为它执行查询时必须检查的行数。越少越好！**



**十、Extra**

该列包含不适合在其他列显示但十分重要的额外信息。有以下几种情况：

+   Using filesort：说明mysql 会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取。**MySQL 中无法利用索引完成的排序操作称为“文件排序”**。
+   Using temporary：使了用临时表保存中间结果, MySQL 在对查询结果排序时使用临时表。常见于排序 order by 和分组查询 group by。
+   Using index：Using index 代表表示相应的 select 操作中使用了覆盖索引(Covering Index)，避免访问了表的数据行，效率不错！如果同时出现 using where，表明索引被用来执行索引键值的查找;如果没有同时出现 using where，表明索引只是用来读取数据而非利用索引执行查找。
+   Using where：表明使用了where 过滤。
+   Using join buffer：使用了连接缓存。
+   impossible where：where 子句的值总是 false，不能用来获取任何元组。
+   select tables optimized away：在没有 GROUPBY 子句的情况下，基于索引优化 MIN / MAX 操作或者对于 MyISAM 存储引擎优化 COUNT(*) 操作，不必等到执行阶段再进行计算，查询执行计划生成的阶段即完成优化。

>   **覆盖索引**：select 的数据列只用从索引中就能够取得，不必读取数据行，MySQL 可以利用索引返回列表中的字段，而不必根据索引再次读取数据文件，即查询列要被所建的索引覆盖。

