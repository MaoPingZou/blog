---
layout: post
title: 【Spring】记录策略模式的使用
subtitle: 拒绝大量的分支逻辑代码
tags: [spring, design pattern]
---

## 前言
今天在整理项目代码时，发现曾经写的两个需求都用到了策略模式，但实现方式有点不一样。

在这里记录一下，简单写了一个小demo，看看这两种实现方式。

## 重写 ApplicationContextAware 接口的方式
```java
/**
 * 策略工厂类：实现 ApplicationContextAware ，重写 setApplicationContext 方法，为策略 map 赋值
 *
 * @author MaoPing Zou
 */
@Component
public class StrategyFactory2 implements ApplicationContextAware {

    /**
     * key为策略实现类组件名称，value为对应策略实现类
     */
    private final Map<String, IStrategy> strategyMap = new ConcurrentHashMap<>();

    /**
     * 重写 setApplicationContext 方法
     *
     * @param applicationContext 容器上下文对象
     * @throws BeansException bean 获取异常
     */
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        // 通过 applicationContext 获取所有 ManualDockStrategy 的实现类
        Map<String, IStrategy> beansMap = applicationContext.getBeansOfType(IStrategy.class);
        // 清除
        this.strategyMap.clear();
        // 往 strategyMap 填充
        this.strategyMap.putAll(beansMap);
    }

    /**
     * 根据组件名称获取对应策略实现类
     *
     * @param componentName 组件名称
     * @return 策略实现类
     */
    public IStrategy getStrategy(String componentName) {
        IStrategy iStrategy = strategyMap.get(componentName);
        if (iStrategy == null) {
            throw new RuntimeException("No strategy defined!");
        }
        return iStrategy;
    }
}
```

## 使用 @Autowired 自动注入的方式
```java
/**
 * 策略工厂类：使用 @Autowired 注入所有实现了策略接口的策略实现类
 *
 * @author MaoPing Zou
 */
@Component
public class StrategyFactory1 {
    /**
     * key为策略实现类组件名称，value为对应策略实现类
     */
    private final Map<String, IStrategy> strategyMap = new ConcurrentHashMap<>();

    /**
     * 使用 @Autowired 注入所有实现了IStrategy接口的Bean
     *
     * @param strategyMap 所有实现了 IStrategy 的策略实现类组成的Map
     */
    @Autowired
    public void setStrategyContextMap(Map<String, IStrategy> strategyMap) {
        // 清除Map
        this.strategyMap.clear();
        // 往策略Map中set容器中
        this.strategyMap.putAll(strategyMap);
    }

    /**
     * 根据组件名称获取对应策略实现类
     *
     * @param componentName 组件名称
     * @return 策略实现类
     */
    public IStrategy getStrategy(String componentName) {
        IStrategy iStrategy = strategyMap.get(componentName);
        if (iStrategy == null) {
            throw new RuntimeException("No strategy defined!");
        }
        return iStrategy;
    }
}
```

## 最后
其实上面两种实现方式的最终目标都是为了给策略Map这个变量赋值，只是使用的方式略微有点不同，不过最终都是依赖了Spring容器提供的Bean管理能力。

在平常做需求时，碰到要根据不同的判断条件走不同的逻辑时，最好使用策略模式实现。

因为策略模式可以将这些判断条件逻辑从主要的业务逻辑中分离出来，更灵活，扩展性强，也让代码后期维护起来不那么困难。


## Github地址
文中展示的是部分代码，如果需要查看全部代码，访问这个 [Github 地址](https://github.com/MaoPingZou/spring_strategy_demo)