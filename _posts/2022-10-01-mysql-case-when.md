---
layout: post
title: 【Mysql】case when 使用时的注意点
subtitle: 当心！
tags: [mysql, sql, 数据库]
---


最近在数据导入的的时候，通过 `Navicat` 将业务部门给的数据导入到数据库中，但在导入之后供应商的列表查询报错，报错提示中有串关键词是：`xx 字段导致 zero date value prohibited mysql`

以前没见过类似这样的错误，于是，我将这个关键词粘贴到 `Google` 搜索栏上，在 `stack overflow` 上找到了下面这个提示：

![search-stack-overflow](https://raw.githubusercontent.com/MaoPingZou/img_repo/master/blog/mysql-case-when.png)


看到`zeroDateTime`这个关键词，我就想起来可能是我导入的供应商数据中某个日期字段存在问题。

于是我写了个 SQL 查了下报错字段的所有字段值，发现存在`0000-00-00`这样的数据，至此，原因找到了。



解决方式有两个：

1、使用`stack overflow`网站上找到的答案，在数据库连接中添加`zeroDateTimeBehavior=convertToNull`属性。

2、将供应商表中存在`0000-00-00`这样数据的字段值通过 SQL 脚本改成 `NULL`值。



方式一比较简单，这里就不做介绍。

方式二在写 `SQL` 语句时可能会有一些坑存在，这里写下注意的点：

语句一：

```sql
update supplier_data 
set supplier_creat_date =
case 
	when supplier_creat_date = 0000-00-00
	then null
end
```

语句二：

```sql
update supplier_data 
set supplier_creat_date =
case 
	when supplier_creat_date = 0000-00-00
	then null
end
where id in (
   select * from (select sd.id from supplier_data sd where sd.supplier_creat_date = 0000-00-00) as driven
)
```

语句一与语句二的区别点在于是否存在 `where` 子句，关键点在于：

> 如果没有 `where` 子句，会将所有不为 `0000-00-00` 的也置为 `NULL` 值
>
> 如果有 `where` 子句，则只会将 `where` 语句中限制的符合条件的数据记录更新

以后使用`mysql`的 `case...then...`语句时，一定要注意加上 `where` 语句，防止对不需要更新的数据误操作产生其他问题。
