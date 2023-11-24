---
layout: post
title: 【MySQL】EXPLAIN 之 key_len 列值计算规则
subtitle: 
tags: [mysql, sql]
---

# key_len 列值是什么

`explain` 关键字列出的执行计划中有一个列为 `key_len`。`MySQL` [官方文档][1]对其作出的解释为：

> The key_len column indicates the length of the key that MySQL decided to use. The value of key_len enables you to determine how many parts of a multiple-part key MySQL actually uses. If the key column says NULL, the key_len column also says NULL.
> 
> Due to the key storage format, the key length is one greater for a column that can be NULL than for a NOT NULL column.

意思就是说，`key_len` 列表明了 `MySQL` 决定要使用的索引的长度。它的值让你能够确定 `MySQL` 实际使用了多部分索引中的哪些部分。

这里的 `multiple-part` 应该指的是多个字段组合成的索引。

有时候因为最左前缀原则，导致只有部分索引被使用。这个时候就可以通过 `key_len` 的长度来判断到底走了哪些列的索引。

# 如何计算

那这个列的值是怎么被计算出来的呢？官网并没有找到详细的解释。

经过 Google 查询之后，找到一些网友分享的经验，在此做个记录。

在计算 `key_len` 时，需要考虑以下几个点：

- 索引字段的数据类型是变长还是定长；
- 当索引字段为定长数据类型时，如 `char`、`int`、`datetime`，需要有是否为空的标记，这个标记占用 1 个字节（对于 `NOT NULL` 来说不需要这 1 个字节）；
- 当索引字段为变长数据类型时，如 `varchar`，除了是否为空的标记外，还需要有长度信息，需要占用 2 个字节；
- 对于 `char`，`varchar`，`blob`，`text` 等，`key_len` 的长度还和字符集有关，`latin1` 一个字符占用 1 个字节，`gbk` 一个字符占用 2 个字节，`utf8` 一个字符占用 3 个字节.

一些常见的数据类型具体的计算公式总结如下：

对于变长字段：

- `varchar(X)` 变长字段且 允许为 `NULL` ：`X * (字符集长度: latin1=1, gbk=2, utf8mb4=4) + 1(NULL值标记) + 2（变长字段长度信息）字节`
- `varchar(X)` 变长字段且 不允许为 `NULL` ：`X * (字符集长度: latin1=1, gbk=2, utf8mb4=4) + 2（变长字段长度信息）字节`

对于定长字段：
- `char(X)` 定长字段且 允许为 `NULL`：`X * (字符集长度: latin1=1, gbk=2, utfmb4=4) + 1(NULL值标记) 字节`
- `char(X)` 定长字段且 不允许为 `NULL`：`X * (字符集长度: latin1=1, gbk=2, utf8mb4=4) 字节`
- `int` 定长字段且 允许为 `NULL`： `4 字节`
- `int` 定长字段且 不允许为 `NULL`： `4 + 1 字节`
- `typeint` 定长字段且 允许为 `NULL`： `1 字节`
- `typeint` 定长字段且 不允许为 `NULL`： `1 + 1 字节`
- `date` 定长字段且 允许为 `NULL`：`3 个字节`
- `date` 定长字段且 不允许为 `NULL`：`3 + 1 个字节`
- `datetime` 定长字段且 允许为 `NULL`：`5 个字节`
- `datetime` 定长字段且 不允许为 `NULL`：`5 + 1 个字节`

# 实践
以 `utf8mb4` 字符集为例进行测试，其他字符集可以使用同样的方法进行测试。

## 创建表
```sql
CREATE TABLE `test` (
  `id` int NOT NULL,
  `name` varchar(20) NOT NULL COMMENT '姓名',
  `sex` tinyint NOT NULL COMMENT '性别,1：男，2：女',
  `email` varchar(20) DEFAULT NULL,
  `age` tinyint default 0,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```
## 创建索引
```sql
CREATE INDEX NAME_IDX USING BTREE ON test (name);
CREATE INDEX SEX_IDX USING BTREE ON test (sex);
CREATE INDEX EMAIL_IDX USING BTREE ON test (email);
CREATE INDEX AGE_IDX USING BTREE ON test (age);
CREATE INDEX name_age_sex_IDX USING BTREE ON test (name,age,sex);
```

## 插入测试数据
```sql
INSERT INTO test (id,name,sex,email,age) VALUES
	 (1,'tom',1,'tom@163.com',16),
	 (2,'lucy',2,'lucy@163.com',18),
	 (3,'nancy',2,'nancy@163.com',22),
	 (4,'lilly',2,'lilly@163.com',25),
	 (5,'letter',1,'letter@163.com',35);
```

## EXPLAIN 分析
1、根据主键 id 分析，由于 id 为 int 类型，设置的 NOT NULL ，key_len 为 4

从下面的 `explain` 执行计划可以看出，`key_len` 确实为 4。

```sql
mysql> explain select * from test where id = 1;
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | test3 | NULL       | const | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL  |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
```

2、根据 email 分析，由于 email 为 varchar 类型，长度为 20，默认为 NULL，因此 key_len 为 20*4+2+1=83

从下面的 `explain` 执行计划可以看出，`key_len` 确实为 83。

```sql
mysql> explain select * from test where email like 'tom%';
+----+-------------+-------+------------+-------+---------------+-----------+---------+------+------+----------+-----------------------+
| id | select_type | table | partitions | type  | possible_keys | key       | key_len | ref  | rows | filtered | Extra                 |
+----+-------------+-------+------------+-------+---------------+-----------+---------+------+------+----------+-----------------------+
|  1 | SIMPLE      | test  | NULL       | range | email_IDX     | email_IDX | 83      | NULL |    1 |   100.00 | Using index condition |
+----+-------------+-------+------------+-------+---------------+-----------+---------+------+------+----------+-----------------------+
```

3、根据 sex 分析，由于 sex 为 tinyint 类型，设置为 NOT NULL，因此 key_len 为 1

从下面的 `explain` 执行计划可以看出，`key_len` 确实为 1。

```sql
mysql> explain select * from test where sex = 1;
+----+-------------+-------+------------+------+---------------+---------+---------+-------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+---------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | test  | NULL       | ref  | sex_IDX       | sex_IDX | 1       | const |    1 |   100.00 | NULL  |
+----+-------------+-------+------------+------+---------------+---------+---------+-------+------+----------+-------+
```

4、测试复合索引

前提：将单个字段的索引去除后进行测试
```sql
ALTER TABLE test DROP INDEX age_IDX;
ALTER TABLE test DROP INDEX name_IDX;
ALTER TABLE test DROP INDEX sex_IDX;
```
4.1、根据 name 和 age 分析，name 的索引长度为 20*4+2=82, age 的索引长度为 1+1=2，因此 key_len 长度为 84

从下面的 `explain` 执行计划可以看出，当只用到 name_age_sex_IDX 索引中的 name 和 age 两个 key_len 确实为 84

```sql
mysql> explain select * from test where name like 'l%' and age > 20 and sex = 2;
+----+-------------+-------+------------+-------+------------------+------------------+---------+------+------+----------+-----------------------+
| id | select_type | table | partitions | type  | possible_keys    | key              | key_len | ref  | rows | filtered | Extra                 |
+----+-------------+-------+------------+-------+------------------+------------------+---------+------+------+----------+-----------------------+
|  1 | SIMPLE      | test  | NULL       | range | name_age_sex_IDX | name_age_sex_IDX | 84      | NULL |    3 |    20.00 | Using index condition |
+----+-------------+-------+------------+-------+------------------+------------------+---------+------+------+----------+-----------------------+
```

4.2、在 4.1 的基础上，修改一下 SQL 语句，将 age > 20 改成 age >= 20，这样就能用上三个字段的联合索引，此时，key_len 应该为 85

从下面的 `explain` 执行计划可以看出，当完全用到 name_age_sex_IDX 索引时， key_len 确实为 85

```sql
mysql> explain select * from test where name like 'l%' and age >= 20 and sex = 2;
+----+-------------+-------+------------+-------+------------------+------------------+---------+------+------+----------+-----------------------+
| id | select_type | table | partitions | type  | possible_keys    | key              | key_len | ref  | rows | filtered | Extra                 |
+----+-------------+-------+------------+-------+------------------+------------------+---------+------+------+----------+-----------------------+
|  1 | SIMPLE      | test  | NULL       | range | name_age_sex_IDX | name_age_sex_IDX | 85      | NULL |    3 |    20.00 | Using index condition |
+----+-------------+-------+------------+-------+------------------+------------------+---------+------+------+----------+-----------------------+
```

# 总结

通过了解 `key_len` 的计算规则， 就可更加清楚 `MySQL` 在决定复合索引时具体使用了哪些索引，从而更好的为进行索引优化提供依据。


[1]:https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain_key_len