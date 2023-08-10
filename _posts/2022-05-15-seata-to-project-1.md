---
layout: post
title: 【Seata】集成Seata分布式事务到项目中（一）
subtitle: 分布式事务Seata
tags: [seata]
---

{: .box-success}
本文将讲解如果将 **Seata** 的**XA事务模式**作为分布式事务解决方案融合到现有的项目中。

目录
- [Server配置与启动](#server配置与启动)
  - [认识存储模式（store.mode）](#认识存储模式storemode)
  - [各模式对应步骤](#各模式对应步骤)
    - [场景一：使用的是`file`模式](#场景一使用的是file模式)
    - [场景二：使用的是`db`模式](#场景二使用的是db模式)
    - [场景三：使用的是`redis`模式](#场景三使用的是redis模式)
- [Client客户端配置](#client客户端配置)
  - [添加依赖](#添加依赖)
  - [`undo_log`建表](#undo_log建表)
  - [配置参数](#配置参数)
  - [数据源代理](#数据源代理)
  - [初始`GlobalTransactionScanner`化](#初始globaltransactionscanner化)
  - [实现`xid`跨服务传递](#实现xid跨服务传递)
- [注意事项](#注意事项)
    - [导入`seata`依赖与项目本身依赖可能产生的冲突问题](#导入seata依赖与项目本身依赖可能产生的冲突问题)
    - [事务分组的问题](#事务分组的问题)
- [参考](#参考)


## Server配置与启动

### 认识存储模式（store.mode）

在启动`server`前，需要了解一下 `server` 的存储模式（**store.mode**）。

`seata`目前提供有`file`、`db`、`redis`三种模式。

- `file` 

  `file`模式无需改动，直接启动即可。

  `file`模式为单机模式，全局事务会话信息内存中读写并持久化本地文件`root.data`，性能较高;

- `db` 

  需要在库中建立三张表：`global_table`（全局事务表）、`branch_table`（分支事务表）、`lock_table`（全局锁表）

  该模式为高可用模式，全局事务会话信息通过`db`共享，相应性能差些;

- `redis` 

  `Seata-Server 1.3`及以上版本支持，性能较高，存在事务信息丢失风险，需要提前配置合适当前场景的`redis`持久化配置.

以上三种存储模式的配置，在 `seata-server` 启动包中 **conf** 目录下的 **file.conf** 文件中。路径为：
 `seata-server-1.4.2 --> conf --> file.conf` ， 其中的 `store.mode`参数。

### 各模式对应步骤

#### 场景一：使用的是`file`模式

使用`file`模式的话，只需要在下载好[seata-server启动包](https://github.com/seata/seata/releases)之后，找到`bin`目录下启动脚本，直接执行启动命令即可，无需修改任何配置文件。

#### 场景二：使用的是`db`模式

使用`db`时，需要

- 步骤一：**修改`server`的`file.conf`配置文件**，
- 步骤二：**在数据库中建立三张表：`global_table`（全局事务表）、`branch_table`（分支事务表）、`lock_table`（全局锁表）**

修改 **store.mode** 参数，指定具体的存储模式为`db`；然后根据使用的存储模式修改对应的参数，**store.db**。

以使用**db模式**为例，参数如下：

```
  ## db存储模式参数设置
  db {
    ## javax.sql.DataSource的实现, 比如DruidDataSource(druid)/BasicDataSource(dbcp)/HikariDataSource(hikari) 等等。
    datasource = "druid"
    ## 使用的数据库类型。如mysql/oracle/postgresql/h2/oceanbase等等。
    dbType = "mysql"
    ## 如果使用的mysql8，需要将该参数值设为：com.mysql.cj.jdbc.Driver
    driverClassName = "com.mysql.jdbc.Driver"
    ## 将url、user、password三个参数设置成自己的数据库
    ## 如果使用的是mysql存储数据，推荐将 rewriteBatchedStatements=true 放到jdbc连接参数中。
    url = "jdbc:mysql://127.0.0.1:3306/seata?rewriteBatchedStatements=true"
    user = "mysql"
    password = "mysql"
    minConn = 5
    maxConn = 100
    ## 指定使用的全局事务表、分支事务表、全局锁表表名
    globalTable = "global_table"
    branchTable = "branch_table"
    lockTable = "lock_table"
    queryLimit = 100
    maxWait = 5000
  }
```

#### 场景三：使用的是`redis`模式

暂未使用过该模式，请查阅资料了解。

## Client客户端配置

### 添加依赖

- `spring-cloud-starter-alibaba-seata`推荐依赖配置方式如下：

```xml
           <dependency>
                <groupId>io.seata</groupId>
                <artifactId>seata-spring-boot-starter</artifactId>
                <version>最新版</version>
            </dependency>
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
                <version>2.2.1.RELEASE</version>
                <exclusions>
                    <exclusion>
                        <groupId>io.seata</groupId>
                        <artifactId>seata-spring-boot-starter</artifactId>
                    </exclusion>
                </exclusions>
            </dependency>
```

### `undo_log`建表

XA模式并不需要建立`undo_log`表。

### 配置参数

Client客户端也需要配置一些参数，可以参考官方给的配置文件示例，如 [使用Spring 的配置文件配置 Seata参数示例](https://github.com/seata/seata/tree/1.4.0/script/client/spring) 、或者[使用 conf 文件配置示例](https://github.com/seata/seata/tree/1.4.0/script/client/conf)

对各个配置参数的说明描述，请查看文档：[seata 参数配置](https://seata.io/zh-cn/docs/user/configurations.html)

###  数据源代理

因为`seata`依赖使用的是`seata-spring-boot-starter`，所以在使用**自动代理数据源**时，使用XA模式还需要调整配置文件。

在`application.yml`文件中添加如下配置：

```xml
seata:
  data-source-proxy-mode: XA
```

### 初始`GlobalTransactionScanner`化

引入`seata-spring-boot-starterjar`依赖后，已经内置了`GlobalTransactionScanner`自动初始化功能。

### 实现`xid`跨服务传递

引入`spring-cloud-starter-alibaba-seata`依赖后，其内部已经实现`xid`跨服务传递。



## 注意事项

#### 导入`seata`依赖与项目本身依赖可能产生的冲突问题

`Seata`依赖本身会引入一些传递依赖进入项目，如果项目中的其他依赖版本与`Seata`引入的依赖版本之间有差异，可能会产生冲突。

这个问题也好解决，找到符合条件的对应版本替换原有的依赖版本即可解决冲突。

#### 事务分组的问题

Client客户端中的 `application.yml` 配置文件中还需要添加下面一行配置：

```xml
seata:
  tx-service-group: my_test_tx_group
```

如果没有配置该参数，项目启动时会在日志控制台中不断输出以下内容：

```java
can not get cluster name in registry config 'service.vgroupMapping.xxxxxx-seata-service-group', please make sure registry config correct
```

其中`xxxxxx`对应你的`spring.application.name`值

参考`seata`官网文档：[事务分组专题](https://seata.io/zh-cn/docs/user/txgroup/transaction-group.html)

---

## 参考

[Seata官方文档-部署-新人文档-部署指南](https://seata.io/zh-cn/docs/ops/deploy-guide-beginner.html)

[分布式事务如何实现？深入解读 Seata 的 XA 模式](https://seata.io/zh-cn/blog/seata-xa-introduce.html)

[Seata 官方 Github 仓库](https://github.com/seata/seata)