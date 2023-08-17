---
layout: post
title: 【Spring】记一次事务问题：Transaction rolled back because it has been marked as rollback-only
subtitle:  
tags: [spring]
--- 

目录
- [遇到的问题](#遇到的问题)
- [代码示例](#代码示例)
- [排查原因](#排查原因)
- [解决方案](#解决方案)
- [知识点](#知识点)
  - [Spring 事务传播级别](#spring-事务传播级别)

## 遇到的问题

最近在项目中遇到一个报错：

> `"org.springframework.transaction.UnexpectedRollbackException: Transaction rolled back because it has been marked as rollback-only"`

根据报错信息来看是`Spring`框架中的事务管理报错了：事务已回滚，因为它已经被标记为只能回滚状态。

那为什么这个事务会被标记为只能回滚呢？

## 代码示例
首先看下代码示例。

项目中碰到问题的主要的代码场景是，两个标记有`@Transactional`的方法A和方法B，分别在`ServiceA`和`ServiceB`中。方法A对方法B进行了调用。

由于需求的原因，不管方法B发生了什么异常，方法A都需要顺利进行。于是，在代码A中将对代码B的调用进行了`try catch`处理。

代码示例如下：
```java
public interface ServiceA {
    void a();
}

public interface ServiceB {
    void b();
}

@Service
public class ServiceAImpl implements ServiceA {

    @Autowired
    private ServiceB serviceB;

    @Override
    @Transactional
    public void a() {
        System.out.println("调用了 a 方法");

        // 对数据库进行了操作

        try {
            serviceB.b();
        } catch (Exception e) {
            System.out.println("catch 到 b 方法出现的异常，但不向上抛出");
        }
        // 对数据库进行了操作
        // 继续走剩下的逻辑
    }
}

@Service
public class ServiceBImpl implements ServiceB {
    @Override
    @Transactional
    public void b() {
        System.out.println("调用了 b 方法");

        // 对数据库进行了操作

        // 模拟抛出运行时异常
        throw new RuntimeException();
    }
}
```


## 排查原因

光看报错没啥思路，于是把这个报错信息拿去 `Google `了一下，才发现报错的原因在哪里。

`Spring`框架是使用`AOP`的方式来管理事务，如果一个被事务管理的方法正常执行完毕，方法结束时`Spring`会将方法中的`sql`进行`commit`。如果方法执行过程中出现异常，则进行`rollback`。

`Spring`框架的默认事务传播方式是`PROPAGATION_REQUIRED`：如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。

在项目中，一般我们都会使用默认的事务传播方式，这样无论外层事务和内层事务有任何一个出现异常，那么所有的`sql`都不会执行。

在嵌套事务（加了事务的方法，调用了加了事务的方法）场景中，使用默认的事务传播方式，两个事务被当成一个整体。内层事务的`sql`和外层事务的`sql`会在外层事务结束时进行提交或回滚。如果内层事务抛出异常，在内层事务结束时，`Spring`会把当前事务标记为“`rollback-only`”。

这时如果外层事务`try catch`捕捉了异常，那么外层事务方法还会继续执行代码，直到外层事务也结束，方法正常执行完毕，`Spring`要`commit`当前事务时，才发现当前事务已经被标记为“`rollback-only`”，这时`Spring`就会抛出`“org.springframework.transaction.UnexpectedRollbackException: Transaction rolled back because it has been marked as rollback-only”`错误了。

## 解决方案

按照项目代码场景的需求，此处要做到的目标是：

> 在内层事务的异常出现时，内层事务自己回滚，不会影响到外层事务的正常提交。
> 内层事务不能影响外层事务，但外层事务出现异常要能影响内层事务进行回滚。

那么在这里，要达到这个目标，只需要将方法B的事务传播级别更改为 `Propagation.NESTED` 即可。

## 知识点
### Spring 事务传播级别
`Spring `的传播级别有七种，在 `org.springframework.transaction.annotation.Propagation` 枚举类中可以看到。

这七种分别为：
- `REQUIRED`：支持当前事务，如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。这是默认的传播方式。
- `SUPPORTS`：支持当前事务，如果当前没有事务，就以非事务方式执行
- `MANDATORY`：支持当前事务，如果不存在事务，抛出一个异常。
- `REQUIRES_NEW`：创建一个新事务，如果当前存在事务，则挂起当前事务。
- `NOT_SUPPORTED`：非事务的方式执行，如果当前存在事务，则挂起当前事务。
- `NEVER`：非事务的方式执行，如果当前存在事务，则引发异常。
- `NESTED`：如果存在当前事务，则在嵌套事务中执行，否则行为类似于 `REQUIRED`。