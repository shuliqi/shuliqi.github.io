---
title: JavaScript中的原型链
date: 2021-06-01 11:29:14
tags: JavaScript
categories: JavaScript
---

对于`JavaScript`中的原型链，一直很疑惑，看不懂，最近读到一篇文章，觉得豁然开朗。记录一下

调对象和函数对象

在 `JavaScript`中除了基本数据类型之外（引用数据类型），都是对象，但是对象也分为**普调对象** 和 **函数对象**。`Object`和`Function` 是自带的函数对象。

`typeof`能判断引用数据类型是什么类型的对象，所以我们使用`typeof `来判断一下:

```javascript
const obj1= {};
const obj2= new Object();
const obj3 = new f1();
console.log(typeof obj1); // object --> 普通对象
console.log(typeof obj2); // object --> 普通对象
console.log(typeof obj3)  // object --> 普通对象

function f1() {}
const f2 = function() {};
const f3 = new Function();
console.log(typeof Object);   // function --> 函数对象
console.log(typeof Function); // function --> 函数对象
console.log(typeof f1); // function --> 函数对象
console.log(typeof f2); // function --> 函数对象
console.log(typeof f3); // function --> 函数对象
```

上面中 `obj1`,`obj2`,`obj3`都是普调对象，内置的 `Function`，`Object`都是函数对象，`f1`,`f2`,`f3`都是函数对象。

它们之前的区分是什么？ 

# 













































