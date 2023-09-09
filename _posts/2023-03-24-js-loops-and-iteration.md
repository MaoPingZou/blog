---
layout: post
title: 【JavaScript】js 循环与迭代
subtitle:  
tags: [javascript]
---

目录
- [for 语句](#for-语句)
- [do...while 语句](#dowhile-语句)
- [while 语句](#while-语句)
- [labeled 语句](#labeled-语句)
- [break 语句](#break-语句)
- [continue 语句](#continue-语句)
- [for...in语句](#forin语句)
- [for...of 语句](#forof-语句)



## for 语句

跟Java和C的 for 循环类似。

语法：

```js
for (initialization; condition; afterthought) {
  statement
}
```

## do...while 语句

语法：

```js
do {
  statement
}
while (condition);

```

## while 语句

语法：

```js
while (condition) {
  statement
}
```

> 一定要小心退出条件，避免出现无限循环。

## labeled 语句

语法：

```js
label:
  statement
```

## break 语句

语法：

```js
break; // 结束loop中最里面的或者结束switch
// or 
break label; // 结束 labeled 语句
```

结束 labeled 语句的示例：

```js
let x = 0;
let z = 0;
labelCancelLoops: while (true) {
  console.log("Outer loops:", x);
  x += 1;
  z = 1;
  while (true) {
    console.log("Inner loops:", z);
    z += 1;
    if (z === 10 && x === 10) {
      break labelCancelLoops; // 满足if条件就跳出 label 语句
    } else if (z === 10) {
      break;
    }
  }
}

```

## continue 语句

语法：

```js
continue;
continue label;
```

继续 labeled 语句的示例：

```js
let i = 0;
let j = 10;
checkiandj: while (i < 4) {
  console.log(i);
  i += 1;
  checkj: while (j > 4) {
    console.log(j);
    j -= 1;
    if (j % 2 === 0) {
      continue checkj; // 满足条件，继续执行 checkj 标签
    }
    console.log(j, "is odd.");
  }
  console.log("i =", i);
  console.log("j =", j);
}
```



## for...in语句



语法：

```js
for (variable in object) {
  statement
}
```

示例：

```js
function dumpProps(obj, objName) {
  let result = "";
  for (const i in obj) { // 这里使用到了 for...in 语句
    result += `${objName}.${i} = ${obj[i]}<br>`;
  }
  result += "<hr>";
  return result;
}
```



> 注意：使用 for 语句循环迭代数组会比使用 for...in 语句迭代数组要好。



## for...of 语句

for...of 语句给可迭代对象（包括Array、Map、Set、arguments等等对象）创建循环迭代。

语法：

```js
for (variable of object) {
  statement
}
```

> 注意：for...in 和 for...of 的区别在于：for...in 迭代属性名，for...of 迭代属性值。

代码示例：

```js
const arr = [3, 5, 7];
arr.foo = "hello";

for (const i in arr) {
  console.log(i);
}
// "0" "1" "2" "foo"

for (const i of arr) {
  console.log(i);
}
// Logs: 3 5 7
```


