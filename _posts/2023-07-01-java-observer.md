---
layout: post
title: 【Java】Java内置的观察者设计模式支持
subtitle:  
tags: [java, design pattern]
--- 

目录
- [前言](#前言)
- [Observer 接口](#observer-接口)
- [Observable 类](#observable-类)
- [缺陷](#缺陷)


## 前言
观察者设计模式如此广泛的应用在程序设计中，甚至 `Java` 在 `JDK 1.0` 时就内置支持了观察者设计模式。

**`Java`** 内置支持的观察者设计模式在 **`java.util`** 包中，一个是 **`Observer`** 接口，一个是 **`Observable`** 类。

## Observer 接口
**`Observer`** 接口用于给观察者们实现，每一个实现了该接口的类都相当于是一个 “订阅者”。
```java
public interface Observer {
    /**
     * 每当被观察的对象（继承了 Observable 类的对象）发生变化，这个方法就会被调用。
     * @param o 被观察对象
     * @param arg 传入Observable 对象中 notifyObservers 方法的入参
     */
    void update(Observable o, Object arg);
}
```
该接口里面只有一个方法 `update` 方法，观察者们需要实现这个方法，写自己在观察到变化后需要做的逻辑。


## Observable 类
**`Observable`** 类用于给被观察对象继承，每一个继承了该类的子类都相当于是一个“发布者”。
```java
public class Observable {
    // 被观察对象是否产生变化
    private boolean changed = false;
    // 保存所有的观察者
    private Vector<Observer> obs;

    /**
     * 添加观察者
     */
    public synchronized void addObserver(Observer o) {
        if (o == null)
            throw new NullPointerException();
        if (!obs.contains(o)) {
            obs.addElement(o);
        }
    }
        
    /**
     * 移除观察者
     */
    public synchronized void deleteObserver(Observer o) {
        obs.removeElement(o);
    }

    /**
     * 通知所有观察者
     */
    public void notifyObservers(Object arg) {
        Object[] arrLocal;

        synchronized (this) {
            if (!changed)
                return;
            arrLocal = obs.toArray();
            clearChanged();
        }

        for (int i = arrLocal.length-1; i>=0; i--)
            ((Observer)arrLocal[i]).update(this, arg);
    }

    /**
     * 是否发生改变
     */
    protected synchronized void setChanged() {
        changed = true;
    }
        
    // ...
}
```
每当被观察对象发生变化需要通知观察者时，需要先调用 `setChanged()` 方法设置变化状态，然后再调用 `notifyObservers()` 通知所有观察者。

## 缺陷
`Java` 内置支持的观察者设计模式有一些缺陷：

1. `Observable` 是一个类，只能通过子类化这个类来让一个类成为被观察对象。如果一个已存在的类继承了其他的父类，那么这个类就不能再继承 `Observable` 类了。这一点限制了代码重用的可能性。
    也违反了面向对象设计原则之一：面向接口编程，而不是面向实现编程（programming to interfaces not implementations）。
    
2. `Observable` 的API中，`setChanged()` 是一个受保护的方法。这样的话，除非你继承 `Observable` 类，否则就访问不到`setChanged()` 方法了。
    这意味着你不能创建 `Observable` 类的实例并且将它跟其他对象进行组合。这一点违反了面对对象设计原则之一：最好用组合（`Composition`）而不是继承（`inheritance`）。
