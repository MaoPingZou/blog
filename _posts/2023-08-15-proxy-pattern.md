---
layout: post
title: 【Design Pattern】设计模式之代理模式
subtitle: 对代理模式的初步认识
tags: [design pattern, proxy]
---


目录
- [前言](#前言)
- [生活中的代理](#生活中的代理)
- [程序中的代理](#程序中的代理)
- [代理模式的结构](#代理模式的结构)
- [代码示例](#代码示例)
  - [Subject](#subject)
  - [RealSubject](#realsubject)
  - [Proxy](#proxy)
  - [Client](#client)
  - [输出结果](#输出结果)
- [最后](#最后)
- [参考](#参考)

<br>

## 前言

代理模式属于**结构型**设计模式，GoF所写的书对代理模式的描述为：<br>
> 为另一个对象提供代理或占位符以控制对其的访问。（GoF 1994，第 207 页）

## 生活中的代理

「代理」这个名词不仅用在软件工程中，也用在生活中其他领域。例如房屋代理、票务代理、保险代理、代购等等。



从生活的经验来理解代理，用房屋中介的例子。中介代理委托人去处理一些事情，比如找意向购房的人，但真正重要的事情，比如在合同上签字，必须还是委托人。代理的角色只是委托人向外提供的一个过滤器，负责控制对委托人自身的访问，让外界不能直接接触到委托人。



## 程序中的代理

代理对象的使用时机为，当不想让客户端直接使用具体的对象时，则可以创建一个原对象的代理对象让客户端使用。

代理模式在软件中常见的应用包括：

- 日志记录代理：增加原对象被调用的日志记录，用于代码调试。
- 虚拟代理：根据需要创建开销很大的对象。
- 同步代理：提供多线程环境下的同步访问，它保证每个时刻只有一个线程访问实体。
- 远程代理：用于访问一个在另一个机器上的对象，它负责将请求及其参数进行编码，并向远程机器发送，以便访问该对象，并获取返回值。



## 代理模式的结构

下面是代理模式的 UML 类图。

![](https://raw.githubusercontent.com/MaoPingZou/img_repo/master/blog/proxy-pattern.png)



从上图中可以看出，代理模式有以下角色：

- Subject：声明了代理对象（Proxy）和真实对象（RealSubject）的共同接口。这样代理可以在任何使用真实对象（RealSubject）的地方使用。

- RealSubject：定义了代理对象代表的真实对象，真正的功能提供者。
- Proxy：定义了代理对象。维护一个对真实对象（RealSubject）的引用，这样代理对象可以访问、控制或扩展真实对象的功能。

- Client：客户端角色，通过 Subject 角色访问真实对象（RealSubject），它可以直接访问真实对象（RealSubject），也可以通过代理对象（Proxy）间接访问。



## 代码示例

### Subject

```java
public interface Subject {
    void doOperation();
}
```

### RealSubject

```java
public class RealSubject implements Subject {

    @Override
    public void doOperation() {
        try {
            Thread.sleep(2000L); // 2秒
            System.out.println("Do something...");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

}
```

### Proxy

```java
public class Proxy implements Subject {

    private final RealSubject subject;

    public Proxy(RealSubject subject) {
        this.subject = subject;
    }

    @Override
    public void doOperation() {
        System.out.println("doOperation() Start Time:" + new Date());
        subject.doOperation();
        System.out.println("doOperation() End Time:" + new Date());
    }
}
```

### Client

```java
public class Client {

    public static void main(String[] args) {
        Subject subject = new Proxy(new RealSubject());
        subject.doOperation();
    }

}
```

### 输出结果

```java
doOperation() Start Time:Wed Sep 06 08:10:00 CST 2023
Do something...
doOperation() End Time:Wed Sep 06 08:10:02 CST 2023
```



## 最后

以上介绍的代理模式属于静态代理。

这种代理方式的缺点有：

- 代理类需要逐个手动编写，工作量大，不易维护；
- 代理类数量多，系统复杂度高。如果每一个目标类都需要一个代理类，会导致系统类爆炸；
- 目标对象和代理对象绑定在编译期，不易扩展；
- 目标类接口改变需要修改代理类。目标类接口变更时，其对应的代理类也需要同步修改，维护代价高。



还有一种代理方式是动态代理（Dynamic Proxy），它能够在运行时（Runtime）动态生成代理对象，是一种更加灵活的创建代理的方式。



## 参考


- [OODesign 网站的 Proxy Pattern一章](https://www.oodesign.com/proxy-pattern)
- 《Java 核心卷 I》第六章 Proxies 部分











