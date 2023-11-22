---
layout: post
title: 【MySQL】MySQL 优化之索引条件下推
subtitle: 
tags: [mysql, sql]
---

# 前言
在这篇短文中，主要了解 MySQL 索引优化中常见的 索引条件下推。

# 定义
根据 MySQL 官方文档中 [8.2.1.6 Index Condition Pushdown Optimization][1] 的描述，索引下推是指：

> Index Condition Pushdown (ICP) is an optimization for the case where MySQL retrieves rows from a table using an index.
> 索引条件下推 （ICP） 是针对 MySQL 使用索引从表中检索行的情况的优化。

这项优化技术可以将 Where 条件的匹配从 MySQL 服务器层下推到存储引擎层，从而提高查询性能。

# 原理
要了解索引下推优化（ICP）的工作原理，首先需要看一下在没有使用索引条件下推时索引扫描是怎么进行的：

- 1、获取下一行，读取索引，找到关联主键rowid，然后通过回表找到完整的行数据
- 2、对整个行数据进行 WHERE 条件的匹配，根据匹配结果获取最终结果数据。

当使用索引条件下推时，索引扫描是这样进行的：

- 1、获取下一行，读取索引
- 2、将 Where 条件中包含在索引中的字段进行匹配，如果不匹配，继续找下一行的索引
- 3、通过回表，继续对 Where 条件中剩余的字段进行匹配，根据匹配结果获取最终结果数据。

# 实践
## 建表和索引
建立一张 people 表，建表语句如下：
```sql
CREATE TABLE `people` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'id',
  `user_name` varchar(10) DEFAULT NULL COMMENT '用户名',
  `age` int DEFAULT NULL COMMENT '年龄',
  `gender` int DEFAULT NULL COMMENT '性别（0-女 1-男）',
  PRIMARY KEY (`id`),
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb3;
```

对 user_name、age 字段建立联合索引
```sql
CREATE INDEX USER_NAME_AGE_IDX USING BTREE ON people (user_name,age);
```

插入5条数据
```sql
INSERT INTO hope.people (user_name,age,gender) VALUES
	 ('刘晓晓',23,0),
	 ('刘佳佳',25,0),
	 ('潘致远',55,1),
	 ('朱杰宏',59,1),
	 ('刘明杰',38,1);
```

## 关闭索引条件下推
通过 `SELECT @@optimizer_switch;` 语句可以看到 `index_condition_pushdown` 是 `on` 也就是开启的状态。
![查看][3]

现在把这个选项关闭进行测试。通过 `SET optimizer_switch = 'index_condition_pushdown=off';` 语句即可关闭索引条件下推。如下图：
![关闭索引下推][2]

现查询姓刘的并且年龄小于等于30岁的所有人的信息，SQL 语句如下：
`select * from people where user_name like '刘%' and age <= 30;`

使用 explain 关键字查看这条 SQL 的执行计划，如下：
![explain 1][4]

## 打开索引条件下推
通过 `SET optimizer_switch = 'index_condition_pushdown=on';` 语句打开索引条件下推

再次使用 explain 关键字查看，如下：
![explain 2][5]

在 Extra 字段中出现了 `using index condition`，说明使用了索引条件下推

## using index condition
官方对 Extra 中出现 `using index condition` 的解释是：

> ![extra info][6]
> 通过访问索引元组并首先测试它们以确定是否读取完整的表行来读取表。这样，除非有必要，否则索引信息用于延迟（“下推”）读取整个表行。

## 执行步骤对比

在关闭索引条件下推时，执行步骤如下：
- 1、客户端发起查询请求，存储引擎根据索引树找出 name like '刘%' 的用户id，分别是：1、2、5
- 2、存储引擎根据 3 个 id ，回表 3 次查询到所有相关数据，返回给服务层
- 3、服务层过滤掉不符合 age <= 30 的数据，最后返回两行数据给客户端

开启索引条件下推时，执行步骤如下：
- 1、客户端发起查询请求，存储引擎根据索引树匹配 name like '刘%' ，并且过滤符合 age <= 30 的用户id，分别是：1、2
- 2、存储引擎根据 2个 id，回表 2 次查询到所有相关数据，返回给服务层
- 3、没有 Where 条件需要进一步过滤，服务层将数据返回给客户端

可以看到，在开启索引条件下推时，回表查询次数减少了。这在大数据量的查询情况下，能有效减少总的回表查询次数，提高查询效率。

# 总结

- 在 InnoDB 引擎中，索引条件下推（ICP）只适用于二级索引。索引条件下推（ICP）的目标是减少全行读取的次数，也就减少了回表次数，从而减少 I/O 操作。对于 InnoDB 聚集索引，因为完整的行记录都已经存在 B+ 树中了，无需回表，所以并不会减少 I/O 操作。

- 索引下推可以有效减少回表的次数，大大提升查询的效率。



[1]:https://dev.mysql.com/doc/refman/8.2/en/index-condition-pushdown-optimization.html
[2]:https://raw.githubusercontent.com/MaoPingZou/img_repo/master/history/202311221115990.png
[3]:https://raw.githubusercontent.com/MaoPingZou/img_repo/master/history/202311221117174.png
[4]:https://raw.githubusercontent.com/MaoPingZou/img_repo/master/history/202311221209611.png
[5]:https://raw.githubusercontent.com/MaoPingZou/img_repo/master/history/202311221210345.png
[6]:https://raw.githubusercontent.com/MaoPingZou/img_repo/master/history/202311221218621.png