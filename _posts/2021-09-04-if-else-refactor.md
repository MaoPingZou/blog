---
layout: post
title: 【重构】如何正确编写以及重构if-else语句
subtitle: 改善代码“坏味道” 
tags: [重构]
---


## 前言

在日常的开发工作中，面对某些业务需求，在编写代码时，很有可能会使用大量的判断语句。代码一层一层的嵌套，一开始写起来虽然方便，但写着写着你会发现，自己可能都被绕晕进去了……

如果自己真这样写代码，那后来维护代码的人（很有可能是你自己），也许真的要一边骂X一边满脸痛苦的表情看代码了……

为了让自己减少编写出“垃圾”代码的概率，有必要学习如何正确编写并优化`if-else`语句了。

接下来，一起看看如何正确编写以及如何优化`if-else`语句吧！

## 优化方式

### 方式一：提前return

  如果`if-else`代码块包含`return`语句，可以考虑提前`return`，这样可以把多余的`else`分支去掉，代码更加简洁。
  
  ```java
  if (condition)
  	// 部分流程……
  else 
  	return "abc";
  // 其余代码……
  ```
  
  把条件反转，提前`return`，就变为：
  
  ```java
  if (!condition)
  	return "abc";
  // 部分流程……
  // 其余代码……
  ```
### 方式二： 使用三目运算符

1.使用`if-else`判断对变量进行赋值时，如：
   ```java
  String name = "";
  if (isHarry)
  	name = "Harry";
  else
  	name = "Bob";
   ```
  此时，可以利用三目运算符，优化成一行代码：
  ```java
  String name = isHarry ? "Harry" : "Bob";
  ```

2.使用`if-else`判断对方法的调用，如：
  ```java
  Set<String> set1 = new HashSet<>();
  Set<String> set2 = new HashSet<>();
  if(flag)
      set1.add(id);
  else
      set2.add(id);
  ```
  利用三目运算符，可以优化成：
  ```java
  Set<String> set1 = new HashSet<>();
  Set<String> set2 = new HashSet<>();
  (flag ? set1 : set2).add(id);
  ```
### 方式三：使用枚举
当碰到有的业务需求需要判断很多中状态时，可以使用枚举进行优化，这样封装好的枚举类也可以多次复用，减少重复代码。

```java
String orderStatusDesc;
if (orderStatus == 0)
    orderStatusDesc ="订单已生成运单";
else if (orderStatus == 1)
    orderStatusDesc ="订单已付款";
else if (orderStatus == 2)
   orderStatusDesc ="订单已完成"; 
...
```

​先定义一个枚举：
```java
​
public enum OrderStatusEnum {
    GENERATE_WAYBILL(0, "订单已生成运单"),
    PAID(1, "订单已付款"),
    DONE(2, "订单已完成");
	
	// 订单状态
    private int status;
    // 订单描述
    private String desc;

    public int getStatus() {
        return status;
    }

    public String getDesc() {
        return desc;
    }

    OrderStatusEnum(int status, String desc) {
        this.status = status;
        this.desc = desc;
    }
    
    // 匹配状态
    OrderStatusEnum convert(int orderStatus) {
        for (OrderStatusEnum value : OrderStatusEnum.values()) {
            if (value.getStatus() == orderStatus)
                return value;
        }
        return null;
    }
}
```
​
定义好上述枚举之后，原来那段代码就变成了以下一段简洁的代码：
```java
String orderStatusDesc = OrderStatusEnum.convert(orderStatus).getDesc();
```

### 方式四：使用Map

如果遇到类似以下这种代码，可以考虑使用`Map`来优化。

```java
    public String getCourse(String day) {
        if ("Monday".equals(day))
            return "周一上英语课";
        else if ("Tuesday".equals(day))
            return "周二上语文课";
        else if ("Wednesday".equals(day))
            return "周三上数学课";
        else if ("Thursday".equals(day))
            return "周四上音乐课";
        else if ("Friday".equals(day))
            return "周五上编程课";
        else
            ...;
    }
```

我们需要在类中合适的位置定义一个静态变量`Map`，如下所示：
```java
​
	// 这里使用了Google Guava里面的一个类来生成一个不可变的Map对象。可以保证线程安全。
    public static final Map<String, String> dayAndCourseMap = ImmutableMap.<String, String>builder()
            .put("Monday", "周一上英语课")
            .put("Tuesday", "周二上语文课")
            .put("Wednesday", "周三上数学课")
            .put("Thursday", "周四上音乐课")
            .put("Friday", "周五上编程课")
            .build()

```

这样，就可以在先前使用`if-else`的方法里面，优化成：
```java
  public String getCourse(String day) {
	return dayAndCourseMap.get(day);
  }
```

### 方式五：合并条件表达式
如果`if-else`中有一系列的条件返回的是一样的结果，可以使用“合并条件表达式”的方式，让代码更简洁，逻辑更清晰。例如：
```java
// 获取折扣价格
public double getDiscount() {
	if (isStudent) return 0.75;
	if (isOlderMan) return 0.75;
	if (isSolider) return 0.75;
	return 0;
}
```
优化后，可以写成：
```java
public double getDiscount() {
	if (isStudent || isOlderMan || isSolider) return 0.75;
	return 0;
}
```
更进一步重构，可以把三个条件抽取出来变成一个方法调用：
```java
public double getDisocunt() {
	if (meetTheConditions()) return 0.75;
	return 0;
}
private boolean meetTheConditons() {
	return isStudent || isOlderMan || isSolider;
}
```

### 方式六：使用Stream流API
`jdk1.8`的新特性`Stream`流中提供了以下三个API：

- anyMatch：任意一个值满足条件，返回true；
- allMatch：所有都满足条件，返回true；
- noneMatch：所有都不满足条件，返回true；

这样，当我们在遇到多种条件判断语句时，如：
```java
if(StringUtils.isEmpty(str1) || StringUtils.isEmpty(str2) 
	|| StringUtils.isEmpty(str3) || StringUtils.isEmpty(str4) 
	|| StringUtils.isEmpty(str5) || StringUtils.isEmpty(str6)
){
	// 业务实现……
}
// 其他业务逻辑
```

就可以考虑使用`Stream`流的`API`来优化条件选择，如：
```java
// 这里使用anyMatch
boolean isSatisfied = Stream.of(str1, str2, str3, str4, str5, str6).anyMatch(
						i -> StringUtils.isEmpty(i));
if (isSatisfied)
	// 业务实现……
// 其他业务逻辑
```

以上是针对或条件的，当遇到与条件时，同样可以使用`Stream`的`API`进行优化，如：
```java
if(StringUtils.isEmpty(str1) && StringUtils.isEmpty(str2)
    && StringUtils.isEmpty(str3) && StringUtils.isEmpty(str4)
    && StringUtils.isEmpty(str5) && StringUtils.isEmpty(str6)
){
    // 业务实现……
}
// 其他业务逻辑
```

使用`Stream`优化：

```java
// 这里使用allMatch
boolean isSatisfied = Stream.of(str1, str2, str3, str4, str5, str6).allMatch(
						i -> StringUtils.isEmpty(i));
if (isSatisfied)
	// 业务实现……
// 其他业务逻辑
```
### 方式七：使用Optinal
平常代码编写过程中，可以看到使用如下方式调用对象的方法获取信息，看起来没什么问题，但如果其中某个属性值获取结果为`null`，那`NullPointerException`异常直接就蹦出来了。
```java
String name = school.getGrades().getStudent().getName();
```

一般如果使用层层嵌套的`if-else`语句，会把上面语句改成这样：
```java
String name = null;
if(school != null){
    Grades grade = school.getGrades();
    if(grade != null){
        Student student = grade.getStuendt();
        if(student != null){
          name = student.getName();
        }
    }
}
```

这样的代码看起来就绕，嵌套的也比较深。这个时候，可以使用`jdk1.8`的新特性`Optional`类来优化这段`if-else`语句。如下：
```java
String name = Optional.ofNullable(school)
  .flatMap(School::getGrades)
  .flatMap(Grades::getStuendt)
  .map(Student::getName)
  .orElse(null);
```

## 最后
以上是一些关于如何优化`if-else`的方法。在平时的编写代码实践中，我也用过其中的一些方法，确实能有效的减少过多的流程判断语句，灵活运用这些方式也可以使代码可读性更强，后期维护起来也更轻松。

作为一名程序员，编写清晰易读、可维护的代码是重要的能力之一。
​

Reference:

1. [项目代码 if/else 过多，引起程序猿口吐莲花](https://cloud.tencent.com/developer/article/1793536)

2. [6个实例详解如何把if-else代码重构成高质量代码](https://blog.csdn.net/qq_35440678/article/details/77939999)

3. [if-else代码优化的八种方案](https://juejin.cn/post/6844904083665453063#heading-5)

4. [Java编程技巧：if-else优化实践总结归纳](https://www.cnblogs.com/zhujiqian/p/14918020.html)
