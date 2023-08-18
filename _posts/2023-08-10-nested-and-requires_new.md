---
layout: post
title: 【Spring】对事务传播属性 NESTED 与 REQUIRES_NEW 的探究
subtitle: 用代码实际验证！
tags: [spring]
--- 

目录
- [前言](#前言)
- [代码验证一](#代码验证一)
  - [部分代码](#部分代码)
  - [测试思路](#测试思路)
  - [观察结果](#观察结果)
  - [代码验证一总结](#代码验证一总结)
- [代码验证二](#代码验证二)
  - [部分代码](#部分代码-1)
  - [测试思路](#测试思路-1)
  - [观察结果](#观察结果-1)
  - [代码验证二总结](#代码验证二总结)
- [应用场景](#应用场景)
- [源码链接](#源码链接)
- [Reference](#reference)

## 前言

NESTED 与 REQUIRES_NEW 之间到底有怎么样的区别？

在 `StackOverflow` 找到的一个 [`Answer`][1]，是这么回答两者之间的区别的：

> PROPAGATION_REQUIRES_NEW
> 为给定范围启动一个新的、独立的“内部”事务。此事务将完全独立于外部事务提交或回滚，具有自己的隔离作用域、自己的锁集等。外部事务将在内部事务开始时挂起，并在内部事务完成后恢复。
> 
> PROPAGATION_NESTED
> 启动一个“嵌套”事务，这是现有事务的真正子事务。将发生的情况是在嵌套事务开始时采用保存点。如果嵌套事务失败，我们将回滚到该保存点。嵌套事务是外部事务的一部分，因此它只会在外部事务结束时提交。

## 代码验证一
这部分实测一的代码在外层函数调用内层函数时，没有进行try catch操作。这样内层方法throw的异常会直接传递到外层方法中。此时，使用 NESTED 和 REQUIRES_NEW 的区别是什么？
### 部分代码
```java
@Service
public class DBServiceImpl implements DBService {

    @Autowired
    private UserMapper userMapper;
    @Autowired
    private DBAnotherService dbAnotherService;

    @Override
    @Transactional(rollbackFor = Exception.class, propagation = Propagation.REQUIRED)
    public void outterMethod() {

        // 插入一个用户
        User user = new User();
        user.setUsername("methodA-username");
        user.setPassword("methodA-password");
        userMapper.insert(user);

        // 调用 DBServiceB 的 methodB() 方法
        dbAnotherService.innerMethod();

        // 模拟抛出运行时异常
//        throw new RuntimeException();
    }

}

@Service
public class DBAnotherServiceImpl implements DBAnotherService {

    @Autowired
    private UserMapper userMapper;

    @Override
    // 使用  REQUIRES_NEW
//    @Transactional(rollbackFor = Exception.class, propagation = Propagation.REQUIRES_NEW)
    // 使用 NESTED
    @Transactional(rollbackFor = Exception.class, propagation = Propagation.NESTED)
    public void innerMethod() {

        User user = new User();
        user.setUsername("methodB-username");
        user.setPassword("methodB-password");
        userMapper.insert(user);

        // 模拟抛出运行时异常
//        throw new RuntimeException();
    }
}

// 测试代码
@SpringBootTest
class DBExampleTest {

    @Autowired
    public UserMapper userMapper;

    /**
     * 每次测试前，先清空数据库表
     */
    @BeforeEach
    public void setUp() {
        userMapper.delete(null);
    }

    @Autowired
    public DBService dbService;

    @Test
    void test() {
        dbService.outterMethod();
    }
}
```

### 测试思路
<!-- 主要是看看外层函数抛出异常，会不会让嵌套的事务也触发数据库回滚，再看看内层函数抛出异常会不会让外部事务触发数据库回滚。 -->

在测试时，保持外层函数的事务传播属性为 propagation = Propagation.REQUIRED 不变，这也是Spring 默认事务传播属性。

通过改变内层函数的事务传播属性 propagation，以及抛出异常的位置，进行观察。主要分为以下四种场景：
- 场景一：设置内层方法的 propagation = Propagation.REQUIRES_NEW，在外层函数的最后加上 throw new RuntimeException();
- 场景二：设置内层方法的 propagation = Propagation.REQUIRES_NEW，在内层函数的最后加上 throw new RuntimeException();
- 场景三：设置内层方法的 propagation = Propagation.NESTED，在外层函数的最后加上 throw new RuntimeException();
- 场景四：设置内层方法的 propagation = Propagation.NESTED，在内层函数的最后加上 throw new RuntimeException();

### 观察结果
- 场景一的结果：内层函数的事务正常commit，新增一条数据；外层事务rollback，没有新增数据。
- 场景二的结果：内层函数的事务rollback，没有新增数据；外层事务rollback，没有新增数据。
- 场景三的结果：内层函数的事务rollback，没有新增数据；外层事务rollback，没有新增数据。
- 场景四的结果：内层函数的事务rollback，没有新增数据；外层事务rollback，没有新增数据。

### 代码验证一总结
- 无论是Propagation.REQUIRES_NEW，还是Propagation.NESTED，只要是内部事务抛出异常数据库都会回滚;
- Propagation.REQUIRES_NEW是一个全新开启的事务，即使外部事务抛出异常发生数据库回滚，也不影响内部事务的提交。
- Propagation.NESTED则更像是一个事务的子事务，受外部事务的影响。

## 代码验证二
如果外层方法在调用内层方法时，对内层方法做了try catch，捕获到异常之后，不做任何处理。这时使用 NESTED 和 REQUIRES_NEW 的区别是什么？
### 部分代码
代码部分唯一不同的地方就是在外层方法调用内层方法的地方，加上了 try catch 语句捕获异常，但不进行处理。测试代码也单独加了一个。
```java
// 其他部分代码跟实测一的代码一致，此处省略
    try {
        // 调用 DBServiceB 的 methodB() 方法
        dbAnotherService.innerMethod();
    } catch (Exception e) {
        log.error("将异常吞掉，不向上抛出异常");
    }

// 测试代码
    /**
     * 对异常进行了 try catch 的测试
     */
    @Test
    void testForTryCatch() {
        dbService.outterMethodForTryCatch();
    }
```

### 测试思路
这里的测试思路与代码验证一的测试思路一样。

### 观察结果
- 场景一的结果：内层函数的事务正常commit，新增一条数据；外层事务rollback，没有新增数据。
- 场景二的结果：内层函数的事务rollback，没有新增数据；外层事务正常commit，新增一条数据。
- 场景三的结果：内层函数的事务rollback，没有新增数据；外层事务rollback，没有新增数据。
- 场景四的结果：内层函数的事务rollback，没有新增数据；外层事务正常commit，新增一条数据。

### 代码验证二总结
在 try catch 消化了内部异常的情况下：
- 无论是Propagation.REQUIRES_NEW，还是Propagation.NESTED，内部事务抛出异常，都不会影响外层事务正常commit。
- 在内层函数的事务为 Propagation.REQUIRES_NEW 时，内外层方法出现异常事务独立commit和rollback，互不影响。
- 在内层函数的事务为 Propagation.NESTED 时，外层事务出现异常，会将内层事务产生的commit进行rollback，而内层事务rollback，不会影响外层事务commit。


## 应用场景
此处示例来自 [Spring事务传播行为][2]。

假设我们有一个注册的方法，方法中调用添加积分的方法，如果我们希望添加积分不会影响注册流程（即添加积分执行失败回滚不能使注册方法也回滚），我们会这样写：

```java
   @Service
   public class UserServiceImpl implements UserService {
        
        @Transactional
        public void register(User user){
                   
            try {
                membershipPointService.addPoint(Point point);
            } catch (Exception e) {
                // 不向上抛出异常
            }
            //省略...
        }
        //省略...
   }
```
我们还规定注册失败要影响addPoint()方法（注册方法回滚添加积分方法也需要回滚），那么addPoint()方法就需要这样实现：
```java
   @Service
   public class MembershipPointServiceImpl implements MembershipPointService{
        
        @Transactional(propagation = Propagation.NESTED)
        public void addPoint(Point point){
                   
            try {
                recordService.addRecord(Record record);
            } catch (Exception e) {
                // 不向上抛出异常
            }
            //省略...
        }
        //省略...
   }
```
我们注意到了在addPoint()中还调用了addRecord()方法，这个方法用来记录日志。他的实现如下：
```java
   @Service
   public class RecordServiceImpl implements RecordService{
        
        @Transactional(propagation = Propagation.NOT_SUPPORTED)
        public void addRecord(Record record){
            // 记录逻辑...
        }
        //省略...
   }
```
可以注意到addRecord()方法中propagation = Propagation.NOT_SUPPORTED，因为对于日志无所谓精确，可以多一条也可以少一条。

所以addRecord()方法本身和外围addPoint()方法抛出异常都不会使 register() 方法回滚，并且addRecord()方法抛出异常也不会影响外围addPoint()方法的执行。

通过这个例子可以对事务传播行为的使用有更加直观的认识，通过各种属性的组合确实能让业务实现更加灵活多样。



## 源码链接
本文代码放在了下面的Github仓库地址中：

[https://github.com/MaoPingZou/spring-propagation-test/tree/master](https://github.com/MaoPingZou/spring-propagation-test/tree/master)


##  Reference
1. [Propagation.NESTED 和Propagation.REQUIRES_NEW的区别](https://codeantenna.com/a/G5blYLvwZm)
2. [spring事务传播行为][2]



[1]: https://stackoverflow.com/questions/12390888/differences-between-requires-new-and-nested-propagation-in-spring-transactions "Differences in behaviour of REQUIRES_NEW and NESTED propagation in Spring transactions"
[2]: https://blog.51cto.com/u_13540373/4858289?u_atoken=47ce92c5-35b3-4440-a4a0-9cf6fff5cde7&u_asession=01AH8SeqIlhT1ANaxGxCmuDAUYWlM55cD8-cNotZbJ4YNhclrDW3Tl1TaXnmkmYeM9X0KNBwm7Lovlpxjd_P_q4JsKWYrT3W_NKPr8w6oU7K_SfK-S5OOj0_tdG2CMrkNAx4ZXIQqVT8if6s_DdipgNWBkFo3NEHBv0PZUm6pbxQU&u_asig=056p5cIcPIKtRzQlYqVpNFBW76rFqtWnb2ox-knohmxp7xmseMU_341pQXmGAKRFlSsobQFKULnyyhxMI4i5vThY6pXCI_EE3SE5ZJj7QBgt33b3wQoqCdHz8ymCx_Nfai3_4y0FFV19LGcLb9JrtnniQ3GGkPYjflI78K5smO_nv9JS7q8ZD7Xtz2Ly-b0kmuyAKRFSVJkkdwVUnyHAIJzYRE4D3dTd0YpCOWIOuCSk8Ciq15yDHbiog-A8JCPdHezs3JSbShbdcDfBG2DkIlju3h9VXwMyh6PgyDIVSG1W9ieCddmAsC1AI5cE6aAjxr3Lxxiw1Iyz2TSblidq6Hsx2x46vi6EdYksJ1mLFbIijEdzLDPl6YKrJWQYHNa9_ymWspDxyAEEo4kbsryBKb9Q&u_aref=J2Slv9xeA6sH2QdWZVFroso86do%3D#item-3-6