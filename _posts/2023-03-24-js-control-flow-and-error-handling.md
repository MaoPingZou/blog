---
layout: post
title: 【JavaScript】js 控制流和错误处理
subtitle:  
tags: [javascript]
---

目录：
- [块语句 Block statement](#块语句-block-statement)
- [条件语句 Conditional statements](#条件语句-conditional-statements)
  - [if...else 语句](#ifelse-语句)
    - [虚假值 Falsy values](#虚假值-falsy-values)
  - [switch 语句](#switch-语句)
    - [break的坑](#break的坑)
- [异常处理语句Exception handling statements](#异常处理语句exception-handling-statements)
  - [异常类型](#异常类型)
  - [throw 语句](#throw-语句)
  - [try...catch 语句](#trycatch-语句)
    - [finally 语句块](#finally-语句块)
  - [利用Error对象 Utilizing Error objects](#利用error对象-utilizing-error-objects)


## 块语句 Block statement

最基本的语句就是块语句，由一堆尖括号包围。

> 注意 var 声明变量不是块范围的，而是函数或脚本范围的。也就是说，块代码里面对块语句外面用var定义的变量进行再赋值，会改变外部的var变量的值。
>
> 这跟java 就不同了，java的块语句中的变量是局部变量，不会影响到外面。

## 条件语句 Conditional statements

JS支持两种条件语句，一种是 `if...else` ，另一种是 `switch`

### if...else 语句

如果逻辑条件为 true，则使用 if 语句执行语句。如果条件为 false，则使用可选的 else 子句执行语句。

#### 虚假值 Falsy values

下面的值都会被当作 false：

- false
- undefined
- null
- 0
- NaN
- 空字符串（`""`）

所有其他的值，包括所有的对象，当传递到条件语句中时，都会被计算成 true

>  注意一点：千万不要搞混淆了基本类型boolean值 true 和false 与 Boolean 对象的 true 和false 值
>
> ```js
> const b = new Boolean(false);
> if (b) {
>   // this condition evaluates to true
> }
> if (b == true) {
>   // this condition evaluates to false
> }
> ```

### switch 语句

代码结构如下：

```js
switch (expression) {
  case label1:
    statements1;
    break;
  case label2:
    statements2;
    break;
  // …
  default:
    statementsDefault;
}
```

#### break的坑

因为break是一个可选项，所以如果在写switch的case语句时，少了break关键字，那么将会逐个执行case语句，直到碰到break或者default，执行完才会跳出switch语句。



## 异常处理语句Exception handling statements

可以使用 throw 语句抛出异常，并且使用 try...catch 语句捕获它们。

### 异常类型

抛出具有指定目的的异常通常是更高效的方式。

异常类型参考：

- [ECMAScript exceptions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error#error_types)
- [`DOMException`](https://developer.mozilla.org/en-US/docs/Web/API/DOMException) and [`DOMError`](https://developer.mozilla.org/en-US/docs/Web/API/DOMError)  

### throw 语句

语法：`throw expression`

可以抛出任何类型的异常，示例：

```js
throw "Error2"; // String type
throw 42; // Number type
throw true; // Boolean type
throw {
  toString() {
    return "I'm an object!";
  },
};
```

### try...catch 语句

try 用于包裹着可能产生异常的语句，catch用来执行在捕获到异常之后该如何处理的语句。

换句话说就是希望try语句块能成功，如果不能成功，那么将控制权交给catch语句块。

> 注意，在打印 catch 语句块的日志信息时，使用 console.error() 有助于调试。

示例：

```js
try {
  throw "myException"; // generates an exception
} catch (err) {
  // statements to handle any exceptions
  logMyErrors(err); // pass exception object to error handler
}
// 注意这里的catch语句括号的值，不需要像Java一样声明类型
```

#### finally 语句块

finally 语句块中包含了将在try...catch 语句块执行之后执行的语句。此外，finally 语句块又在 try...catch...finally 后面的语句之前执行。

finally 语句块不管异常是否发生都会执行。这个功能可以使得finally很适合处理当一些脚本占用了资源需要释放时使用。

示例代码：

```js
openMyFile();
try {
  writeMyFile(theData); // This may throw an error
} catch (e) {
  handleError(e); // If an error occurred, handle it
} finally {
  closeMyFile(); // Always close the resource
}
```



> 注意：如果 finally 语句块中有return一个值，那么这个值将变成整个 try...catch...finally 语句块的返回值，而不管try或catch语句块中的任何return返回值结果。



### 利用Error对象 Utilizing Error objects

可以使用 Error 对象的 name 和 message 属性获取更精准的错误信息。

示例：

```js
function doSomethingErrorProne() {
  if (ourCodeMakesAMistake()) {
    throw new Error("The message");
  } else {
    doSomethingToGetAJavaScriptError();
  }
}

try {
  doSomethingErrorProne();
} catch (e) {
  // Now, we actually use `console.error()`
  console.error(e.name); // 'Error'
  console.error(e.message); // 'The message', or a JavaScript error message
}
```

