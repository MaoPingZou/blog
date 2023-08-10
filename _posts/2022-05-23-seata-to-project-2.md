---
layout: post
title: 【Seata】集成Seata分布式事务到项目中（二）
subtitle: 分布式事务Seata
tags: [seata]
---

{: .box-success}
本文将梳理如何将`Nacos`作为`Seata-Server` 和 `Client` 端的注册中心融入到项目中的步骤。

目录
- [前言](#前言)
- [`Seata-Server`服务端](#seata-server服务端)
  - [一、将`nacos`作为服务端注册中心](#一将nacos作为服务端注册中心)
  - [二、将全局事务存储模式改为`db`模式](#二将全局事务存储模式改为db模式)
- [`Client`客户端](#client客户端)
  - [一、增加`Maven`依赖](#一增加maven依赖)
  - [二、配置注册中心](#二配置注册中心)
- [最后](#最后)

## 前言
上一篇文章[集成Seata分布式事务到项目中（一）](https://maopingzou.github.io/blog/2022-05-15-seata-to-project-1)讲到使用最简单的 **`file` 模式**作为 `Seata-Server` 的全局事务会话信息存储模式。

在这篇文章中将引入 **`Nacos`** 作为 **`Seata-Server`** 端和 **`Client`** 端的注册中心，并将全局事务存储模式改为 **`db`** 模式。

在启动服务之前要确保后台已经启动 了 **`Nacos`** 服务。如果你还不熟悉 **`Nacos`** 的基本使用的话，可先参考 [Nacos 快速入门](https://nacos.io/zh-cn/docs/quick-start.html) 熟悉一下。

## `Seata-Server`服务端

### 一、将`nacos`作为服务端注册中心

将 **`Nacos`** 作为 **`Seata-Server`** 服务端的注册中心，并且将存储模式改为 **`db`** 模式，只需要修改两个配置文件即可。

一个是 `file.conf` 文件，一个是 `register.conf` 文件，两个文件都在 `bin` 目录下的 `conf` 目录中。

先看看 `register.conf` 文件，这个文件主要包含两部分内容： `registry` 和 `config` 。

其中， `registry` 用于配置**注册中心**，默认是 `file` 模式，可选的其他方式有`nacos`、`eureka`、`redis`、`zk`、`consul`、`etcd3`、`sofa`等；而 `config` 用于配置**配置中心**，默认也是 `file` 模式，可选的其他方式有`nacos` 、`apollo`、`zk`、`consul`、`etcd3`等。

现在只关注**注册中心**的配置，并且打算使用 `Nacos` ，那么最终 `registry.conf` 文件中关于 `registry` 的配置参数如下：

```java
registry {
  # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
  type = "nacos"

  nacos {
    # 注册到nacos的服务名称
    application = "seata-server"
    # nacos的服务地址
    serverAddr = "127.0.0.1:8848"
    # nacos的group组
    group = "SEATA_GROUP"
    # 所属命名空间，这里为空使用的是nacos保留的public命名空间。
    namespace = ""
    # 集群名称
    cluster = "default"
    # nacos的用户名
    username = ""
    # nacos的密码
    password = ""
  }
}
# config 配置参数略
```

### 二、将全局事务存储模式改为`db`模式

这个修改需要在 `file.conf` 文件中进行。并且，还需要在对应的库中建立三张表。

1.  `file.conf` 文件中具体的配置参数如下：
（已去除 `file`、`redis` 模式下的配置参数，只保留了 `db` 模式下所需配置）。

    ```java
       ## transaction log store, only used in seata-server
       ## 事务日志存储，只用于seata-server
       store {
         ## store mode: file、db、redis
         ## 存储模式：file、db、redis，这里我们使用db的模式
         mode = "db"
         ## rsa decryption public key
         publicKey = ""

         ## database store property  db模式存储的属性参数设置
         db {
           ## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp)/HikariDataSource(hikari) etc.
           ## 选择javax.sql.DataSource数据源的实现，如 druid、dbcp、hikari等等
           datasource = "druid"
           ## mysql/oracle/postgresql/h2/oceanbase etc.
           ## 数据库类型 mysql/oracle/postgresql/h2/oceanbase 等等
           dbType = "mysql"
           ## 驱动类名称。这里使用的是mysql8的驱动。
           driverClassName = "com.mysql.cj.jdbc.Driver"
           ## if using mysql to store the data, recommend add rewriteBatchedStatements=true in jdbc connection param
           ## 数据库连接的三个属性：url、user、password
           url = "jdbc:mysql://127.0.0.1:3306/seata?rewriteBatchedStatements=true"
           user = "mysql"
           password = "mysql"
           minConn = 5
           maxConn = 100
           ## 全局事务表
           globalTable = "global_table"
           ## 分支事务表
           branchTable = "branch_table"
           ## 全局锁表
           lockTable = "lock_table"
           queryLimit = 100
           maxWait = 5000
         }
       }
    ```

2. 建立三张全局事务相关的表

   从上面的配置中可以看到这三张表分别是：全局事务表（`global_table`）、分支事务表（`branch_table`）、全局锁表（`lock_table`）

   如果你使用的 `mysql` 数据库，那么建表的sql我已经贴在下面，如果你使用别的数据库，请参考[seata官方github仓库中sever建表语句](https://github.com/seata/seata/tree/1.4.0/script/server/db)

   ```sql
   -- -------------------------------- The script used when storeMode is 'db' --------------------------------
   -- the table to store GlobalSession data
   CREATE TABLE IF NOT EXISTS `global_table`
   (
       `xid`                       VARCHAR(128) NOT NULL,
       `transaction_id`            BIGINT,
       `status`                    TINYINT      NOT NULL,
       `application_id`            VARCHAR(32),
       `transaction_service_group` VARCHAR(32),
       `transaction_name`          VARCHAR(128),
       `timeout`                   INT,
       `begin_time`                BIGINT,
       `application_data`          VARCHAR(2000),
       `gmt_create`                DATETIME,
       `gmt_modified`              DATETIME,
       PRIMARY KEY (`xid`),
       KEY `idx_gmt_modified_status` (`gmt_modified`, `status`),
       KEY `idx_transaction_id` (`transaction_id`)
   ) ENGINE = InnoDB
     DEFAULT CHARSET = utf8;
   
   -- the table to store BranchSession data
   CREATE TABLE IF NOT EXISTS `branch_table`
   (
       `branch_id`         BIGINT       NOT NULL,
       `xid`               VARCHAR(128) NOT NULL,
       `transaction_id`    BIGINT,
       `resource_group_id` VARCHAR(32),
       `resource_id`       VARCHAR(256),
       `branch_type`       VARCHAR(8),
       `status`            TINYINT,
       `client_id`         VARCHAR(64),
       `application_data`  VARCHAR(2000),
       `gmt_create`        DATETIME(6),
       `gmt_modified`      DATETIME(6),
       PRIMARY KEY (`branch_id`),
       KEY `idx_xid` (`xid`)
   ) ENGINE = InnoDB
     DEFAULT CHARSET = utf8;
   
   -- the table to store lock data
   CREATE TABLE IF NOT EXISTS `lock_table`
   (
       `row_key`        VARCHAR(128) NOT NULL,
       `xid`            VARCHAR(96),
       `transaction_id` BIGINT,
       `branch_id`      BIGINT       NOT NULL,
       `resource_id`    VARCHAR(256),
       `table_name`     VARCHAR(32),
       `pk`             VARCHAR(36),
       `gmt_create`     DATETIME,
       `gmt_modified`   DATETIME,
       PRIMARY KEY (`row_key`),
       KEY `idx_branch_id` (`branch_id`)
   ) ENGINE = InnoDB
     DEFAULT CHARSET = utf8;
   
   ```



做好以上两步，就已经将 `Seata-Server` 的注册中心配置为 `Nacos` ，并且将 `Seata-Server` 的全局事务存储模式改为db模式。

**Congratulation！**

接下来就看看如何将 `Client` 端的注册中心也改为 `Nacos` 吧！

---

## `Client`客户端

将 `Nacos` 作为 `Client` 端的注册中心配置非常简单，分成两步，第一步是增加 `Maven` 依赖，第二步是配置注册中心。

### 一、增加`Maven`依赖

第一步需要将 `nacos-client` 的 Maven 依赖添加到项目 `pom.xml` 文件中

```java
<dependency>
    <groupId>com.alibaba.nacos</groupId>
    <artifactId>nacos-client</artifactId>
    <version>1.2.0及以上版本</version>
</dependency>
```

### 二、配置注册中心

如果使用的是 Srping 应用，那么在 [**application.yml**](https://github.com/seata/seata/blob/develop/script/client/spring/application.yml) 中加入对应的配置中心即可。其他的配置方式可以参考[配置参考](https://github.com/seata/seata/tree/develop/script/client)

```yml
seata:
  registry:
  	## 注册中心类型
    type: nacos
    ## nacos相关配置
    nacos:
      application: seata-server
      server-addr: 127.0.0.1:8848
      # 分组，确保跟seata-server的分组保持一致，不然会出现找不到服务的情况。
      group : "SEATA_GROUP"
      # 命名空间，确保跟seata-server的namespace保持一致，不然会出现找不到服务的情况。
      namespace: ""
      username: "nacos"
      password: "nacos"
```

完成以上两步，就算已经配置好 `Nacos` 作为 `Client` 端的注册中心了！

**Congratulation！**

---

## 最后

先启动 `Seata-Server` 服务端，`Server` 端的服务将出现在 `Nacos` 控制台中的注册中心列表中。示例如下：

![nacos](https://raw.githubusercontent.com/MaoPingZou/img_repo/master/blog/nacos.png)

`Client` 配置完成后启动应用就可以开始愉快的使用 `Seata` 服务啦！

**Congratulation！**