---
layout: post
title: 【Mysql】Group by 的一次使用
subtitle: 加深对 Group By 的了解 
tags: [mysql, sql, 数据库]
---

​
### 工作中碰到的一个小需求
今天工作中碰到了一个查询指定数据的需求，大致如下：
> 查询出每辆车承运的所有运单中各条合同运输线路分别跑了多少次。

这条SQL在查了资料后写好了，并不难写，主要是自身对一些基础知识的还不是特别熟悉，总会因为一些小细节导致耽误了不少时间。（**越来越意识到基础的重要性！**）

在看到这个需求后，我马上想到使用group by来进行分组就行。但是当我在实际编写sql的时候，我发现，我对group by的用法不是很了解。于是，我便想记录下我对所查资料的总结。

### GROUP BY基础知识
我查阅《MySQL必知必会》这本书，想要先弄明白最基础的关于分组的知识。这本书讲到，在使用GROUP BY之前，需要知道的一些**规定**：
- **GROUP BY子句可以包含任意数目的列**。这使得能对分组进行嵌套，为数据分组提供更细致的控制。
- 如果在GROUP BY子句中**嵌套了分组**，数据将在最后规定对分组上进行汇总。换句话说，在建立分组时，指定对所有列都一起计算（所以不能从个别的列取回数据）
- **GROUP BY子句中列出的每个列都必须是检索列或有效的表达式（但不能是聚集函数）**。如果在SELECT中使用表达式，则必须在GROUP BY子句中指定相同的表达式。不能使用别名。
- **除聚集计算语句外，SELECT语句中的每个列都必须在GROUP BY子句中给出。**
- **如果分组列中具有NULL值，则NULL将作为一个分组返回。如果列中有多行NULL值，它们将分为一组。**
- GROUP BY子句必须出现在WHERE子句之后，ORDER BY子句之前。

这里的规定一就是我解决我遇到的需求的关键，但是我对这个规定又不是特别理解。于是，我又针对这一点查了一些资料。

当GROUP BY 单个字段 X，表示什么意思呢？
> GROUP BY X 的意思是将所有具有相同X字段值的记录放到一个分组里。

那当GROUP BY两个字段X，Y时，又表示什么意思呢？
> GROUP BY X 的意思是将所有具有相同X字段值和Y字段值当记录放到一个分组里。

以上两点参考 stackoverflow： [Using group by on multiple columns](https://stackoverflow.com/questions/2421388/using-group-by-on-multiple-columns)

### 如何过滤分组
在需要过滤分组的时候，可以使用**HAVING子句**。
看看《MySQL必知必会》中举的例子，就知道如何使用了：
```sql
-- 这条SQL根据cust_id分组，最后，加上HAVING子句过滤出 COUNT(*) >= 2的那些分组。
SELECT cust_id COUNT(*) AS orders
FROM orders
GROUP BY cust_id
HAVING COUNT(*) >= 2;
```
#### HAVING和WHERE的差别
> WHERE在**数据分组前**过滤，HAVING在**数据分组后**进行过滤。这是一个重要的区别，WHERE排除的行不包括在分组中。这可能会改变计算值，从而影响HAVING子句中基于这些值过滤掉的分组。
