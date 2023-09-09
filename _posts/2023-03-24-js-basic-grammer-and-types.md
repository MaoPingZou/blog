---
layout: post
title: 【JavaScript】js 基础、语法和类型
subtitle:  
tags: [javascript]
---

目录
- [基础](#基础)
- [注释](#注释)
- [变量声明](#变量声明)
- [数据结构和类型](#数据结构和类型)
  - [数据类型转换](#数据类型转换)
  - [Numbers and the '+' operator](#numbers-and-the--operator)
  - [把字符串转换成数值](#把字符串转换成数值)
- [字面量（Literals）](#字面量literals)
  - [Array literals](#array-literals)
  - [Boolean literals](#boolean-literals)
  - [Numeric literals](#numeric-literals)
    - [Integer 字面量](#integer-字面量)
    - [Floating-point 字面量](#floating-point-字面量)
  - [Object literals](#object-literals)
  - [RegExp literals](#regexp-literals)
  - [String literals](#string-literals)
    - [Using special characters in strings](#using-special-characters-in-strings)
    - [Escaping characters](#escaping-characters)


## 基础
JavaScript 从 Java、C 和 C++ 中借用了大部分语法，但它也受到了 Awk、Perl 和 Python 等编程语言的影响。

js 是大小写敏感的（case-sensitive），并使用 Unicode 字符集。

单行语句不需要使用semicolon，但如果想要多行语句在同一行，则必须使用semicolon进行分隔。最佳实践是总是使用semicolon。

## 注释
注释的规则跟Java很像。如下：
```js
// a one line comment

/* this is a longer,
 * multi-line comment
 */
```

## 变量声明

JavaScript 有三种变量声明：

- var：声明一个变量，可以选择将其初始化为值。
- let：声明块范围的局部变量，可以选择将其初始化为值。
- const：声明一个块范围的**只读**命名常量，必须初始化。

const的注意点：它只用于防止变量被重新赋值，但不防止对对象内部的属性进行修改。

示例一：

```js
const MY_OBJECT = { key: "value" };
MY_OBJECT.key = "otherValue";
```
示例二：
```js
const MY_ARRAY = ["HTML", "CSS"];
MY_ARRAY.push("JAVASCRIPT");
console.log(MY_ARRAY); // ['HTML', 'CSS', 'JAVASCRIPT'];
```



## 数据结构和类型

总共8种数据类型：

- 7种基本数据类型

  - Boolean：true、false

  - null：由于JS的大小写敏感特性，null值不等同于 Null，NULL

  - undefined：值未定义的顶级属性。

  - Number：整数或浮点数。如42 或 3.1818

  - BigInt：具有任意精度的整数。

  - String：表示文本值的字符序列。

  - Symbol：其实例是唯一且不可变的数据类型。

- 一种对象

### 数据类型转换

js是一种dynamically typed 语言。也就是说，声明的时候不需要指定变量的数据类型。这也意味着数据类型会在脚本执行过程中按需自动转换。

### Numbers and the '+' operator

将数字和字符串使用“+”操作符，则会将数值转换成字符串，结果为拼接在一起。

但在使用其他的操作符时，js不会将数值转换成字符串。如 "37" - 7 的结果为30，"37" * 7 的结果为 259。

### 把字符串转换成数值

- 使用 `parseInt()` 或 `parseFloat()` 方法。

  `parseInt()` 只返回整数，所以它的使用会丢失精度。（这个方法的最佳实践是，每次在入参时都加上 `radix` 参数，表示使用多少进制的数字系统，比如二进制、8进制、10进制。）

- 使用 `+(unary play)` 

  ```js
  "1.1" + "1.1" // '1.11.1'
  (+"1.1") + (+"1.1"); // 2.2
  // Note: the parentheses are added for clarity, not required.
  ```

  

## 字面量（Literals）

### Array literals

使用`[]` 括起来的0个或多个表达式，每个表示一个数组值。

> Array 字面量创建了一个 Array 对象。

数组中间如果有额外逗号出现，log出来是一个emtpy。使用数组方法时，这样的值会被跳过，但使用索引访问时，结果依然是 undefined。

理解额外逗号的行为对于理解JavaScript作为一种语言非常重要。

如果一定要出现额外逗号，最佳实践如下：

```js
const myList = ["home", /* empty */, "school", /* empty */, ];
```

### Boolean literals

> 不要弄混了基本类型 Boolean的true 和 false 值 和 Boolean 对象的true和false值。
>
> Boolean 对象是对基本类型的包装。

### Numeric literals

包含不同基数的 integer 字面量和基本10的浮点字面量。

#### Integer 字面量

基数有：decimal（10进制）、hexadecimal（16进制）、octal（8进制）、binary（2进制）

- decimal ：正常的数字，没有0开头的。
- 0 或者 0o 开头表示 octal 8进制
- 0x 或0X 开头表示 hexadecimal 16进制
- 0b 或 0B 开头表示 binary 二进制
- 结尾有个 n 表示是 BigInt 字面量。

#### Floating-point 字面量

如：

```js
3.1415926
.123456789
3.1E+12
.1e-23
```

### Object literals

这个示例有点意思：

```js
const car = { manyCars: { a: "Saab", b: "Jeep" }, 7: "Mazda" };

console.log(car.manyCars.b); // Jeep
console.log(car[7]); // Mazda
```

这里的第二个log语句这样使用的原因：之所以要使用 car[7] 而不是 car.7 ，是因为在 JavaScript 中，不能将不是**有效标识符**的属性名称作为点 （.） 属性进行访问。

对象属性的访问可以通过两种方式:

1. 对象名.属性名 ：的方式访问普通的属性。如 car.manyCars.b 可以访问到内部对象 manyCars 的属性 b。
2. 对象名[表达式] ：的方式可以根据一个变量或表达式来动态访问属性。

### RegExp literals

正则表达式字面量。使用slashes包围的模式，示例：

```js
const re = /ab+c/; // 表示匹配一个a在开头，中间最少一个b，末尾一个c的字符串
```

### String literals

字符串字面量使用双引号（"）或单引号（'）包围的0个或多个字符表示。

>  应使用字符串字面量，除非特别需要才使用 String 对象。

在String字面量值中可以使用任何 String对象的方法。JS会自动将String字面量转换成临时的String对象，调用方法，然后丢弃临时的String对象。



> 使用 back-tick（`）符号包围的是模版字面量（Template literals）
>
> 示例：
>
> ```js
> // String interpolation
> const name = 'Lev', time = 'today';
> `Hello ${name}, how are you ${time}?` // 结果：'Hello Lev, how are you today?'
> ```



#### Using special characters in strings

示例，最常见的：

```js
"one line \n another line"; // \n 表示新的一行或者换行
```

类似这样的符号还有很多，可以搜索查看。

#### Escaping characters

使用 "\\" 符号进行转义的作用。

比如：

```js
const home = "c:\\temp"; // 表示 c:\temps
```

