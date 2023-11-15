---
layout: post
title: 【Java】IDEA 配置远程 Debug
subtitle: 
tags: [idea, java]
---

目录
- [前言](#前言)
- [添加 `jar` 包启动参数](#添加-jar-包启动参数)
- [如何查看远程服务的 `address`](#如何查看远程服务的-address)
- [IDEA配置 `Remote JVM Debug`](#idea配置-remote-jvm-debug)
  - [添加新配置](#添加新配置)
  - [详细配置](#详细配置)
  - [测试配置是否成功](#测试配置是否成功)
- [注意点](#注意点)
- [远程`DEBUG`原理](#远程debug原理)
- [参考](#参考)

## 前言

当把一个项目部署到远程服务器后有可能出现意想不到的错误，日志打印过多或者过少都影响系统出现问题时排查的效率，这个时候可以通过远程调试的方式快速定位bug，提升工作效率。

## 添加 `jar` 包启动参数

```java
-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005
```

- `agentlib:jdwp` 表示启动`Java Debug Wire Protocol (JDWP)` 代理。
- `transport=dt_socket` 表示使用`socket`作为调试通信传输协议。
- `server=y` 表示以`server`模式开启,等待外部调试客户端连接。
- `suspend=n` 表示在启动时不暂停`Java`程序执行。
- `address=5005` 表示开启调试端口为5005。

其中，`address`的值各个`jar`包启动时是需要设置成不同端口号的。

## 如何查看远程服务的 `address`

使用 `ps aux | grep java` 或 `ps -ef | grep java` 命令可以查看远程服务器中运行的jar包的启动参数（如果程序启动添加了此参数的话）

![picture 1](https://raw.githubusercontent.com/MaoPingZou/img_repo/master/history/202311151702869.png)

## IDEA配置 `Remote JVM Debug`

### 添加新配置

![picture 2](https://raw.githubusercontent.com/MaoPingZou/img_repo/master/history/202311151705924.png)

### 详细配置

![picture 3](https://raw.githubusercontent.com/MaoPingZou/img_repo/master/history/202311151706350.png)

- ️1️⃣：自定义名称
- 2️⃣：远程服务器的`ip`地址
- 3️⃣：远程服务器中`jar`包运行时设置的`address`对应的值，修改后会影响 `Command line arguments for remote JVM` 框框中`address`的值
- 4️⃣：选择程序运行使用的`JDK`版本
- ️5️⃣：选择需要调试的服务对应的模块`classpath`

### 测试配置是否成功

通过上面配置好的配置，用debug模式启动，控制台会输入如下信息，表示配置成功，可以在本地打断点进行远程调试了。

![picture 4](https://raw.githubusercontent.com/MaoPingZou/img_repo/master/history/202311151706145.png)

## 注意点

- 本地代码需要与远程代码**相同**，不一样则会出现无法进入断点的情况。
- 改了本地代码不可以直接 `debug`，需要将改动后的代码部署到远程服务器之后才能再次远程`debug`。

## 远程`DEBUG`原理

原理：本机和远程主机的两个 `VM` 之间使用 `Debug` 协议通过 `Socket` 通信，传递调试指令和调试信息。

## 参考
美团技术团队-Java动态调试技术原理及实践：https://tech.meituan.com/2019/11/07/java-dynamic-debugging-technology.html