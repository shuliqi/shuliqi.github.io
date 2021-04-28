---
title: 深入了解Object.defineProperty
date: 2018-02-19 14:42:44
tags: JavaScript
categories: JavaScript
---



了解到vue的实现原理是使用了Object.defineProperty()方法。 之前对这个方法没深入了解。知道的就只有这个方法可以设置对象的的属性。现在了解了之后，感觉还是有很多点很值得记录一下。

 <!--more-->

## 语法

`Object。defineProperty()` 方法会直接在一个对象上添加新的属性， 或者修改一个对象的现有属性。然后返回这个对象。

```javascript 
Object.defineProperty(obj, pro, descripor)
```

**obj：** 要定义属性的对象

**prop:** 要定义的属性名或者是要修改的属性的名字

**descriptor:**要 定义的属性或者是要修改的属性的描述

**返回值**：返回此对象

```javascript
  const obj = {
    name: "asjdha"
  }
  Object.defineProperty(obj, "age", {
    value: 12,
    configurable: true,
    writeable: true,
    enumerable: true;
  })
  console.log(obj.age); // 12
```

其中**obj** 和 **prop** 参数没什么好说。 主要是这个descriptor(描述符)参数有比较深究的东西

## 描述符

javacript 有三种类型的属性

1. 命名数据属性：拥有一个确定的值的属性。这也是最常见的属性
2. 命名访问器属性：通过`getter`和`setter`进行读取和赋值的属性
3. 内部属性：由JavaScript引擎内部使用的属性，不能通过JavaScript代码直接访问到，不过可以通过一些方法间接的读取和设置。比如，每个对象都有一个内部属性`[[Prototype]]`，你不能直接访问这个属性，但可以通过`Object.getPrototypeOf()`方法间接的读取到它的值。虽然内部属性通常用一个双吕括号包围的名称来表示，但实际上这并不是它们的名字，它们是一种抽象操作，是不可见的，根本没有上面两种属性有的那种字符串类型的属性

在js中可以访问到的对象中有两种属性： **数据属性** 和 **访问器属性**； 所以描述符也分为**数据描述** 和**访问器描述**

**注意：** 在使用描述符时， 必须是两种描述符之一，且两者是不能并存的。

## 数据描述符

数据描述符是一个具有值的属性，该值可写可不写

具有以下的键值：

1. **configurable：** 该属性表示是否可以通过delete删除，是否能够修改属性的特征或者访问器属性。默认值是false；
2. **enumerable:** 属性述否可枚举。 即 可否通过for..in 访问属性；默认值为 false
3. **value:** 属性的值； 默认为undefined；
4. **writable：** 该属性的值是否可写。 默认值为false。

举个例子：

```javascript
  const obj = {
    name: "asjdha"
  }
  Object.defineProperty(obj, "age", {
    value: 12,
    writable: false, // 不可写
    configurable: false, // 不可删除，不可修改属性的特性和防访问器属性
    enumerable: false, // 不可枚举
  })
  obj.age = 15;
  delete obj.age;
  for (const key in obj) {
    console.log(key);   // name, 没有把age 枚举出来
  }
  console.log(obj.age); // 12，  值没有被删除
```

由例子可以知道：当我们设置不可写， 不可删除，不可枚举的时候， 我们是分别改变不了它们的值，删除不了， 枚举不了的。

```javascript
 const obj = {
    name: "asjdha"
  }
  Object.defineProperty(obj, "age", {
    value: 12,
    writable: true, // 可写
    configurable: true, // 可删除，可修改属性的特性和防访问器属性
    enumerable: true, // 可枚举
  })
  obj.age = 15;
  for (const key in obj) {
    console.log(key);   // name, age   把age 枚举出来
  }

  console.log(obj.age); // 15， 值被改变了
  delete obj.age;

  console.log(obj.age); // undefined  值已经被删除了
```

由例子可以知道：当我们设置可写， 可删除，可枚举的时候， 我们是分别可改变它们的值，可删除， 可枚举不的。

## 访问器描述符

1. **configurable：** 该属性表示是否可以通过delete删除，是否能够修改属性的特征或者访问器属性。默认值是false；
2. **enumerable:** 属性述否可枚举。 即 可否通过for..in 访问属性；默认值为 false
3. **get：**在读取属性的时候调用的函数， 默认值为undefned
4. **set：**在写入属性时调用的函数， 默认值是undefined

```javascript
  const obj = {
    name: "asjdha",
    _age: null, // 私有变量
  }
  Object.defineProperty(obj, "age", {
    set(value) {
      console.log('set', value);
      this._age = value;
    },
    get() {
        console.log('get');
        return this._age;
    },
  });
  obj.age = 12
  console.log(obj.age);
}
```



## 注意的点

 数据描述符 和访问器描述符是不能同时是使用的

例如：

```javascript
{
  const obj = {
    name: "asjdha",
    _age: null, // 私有变量
  }
  Object.defineProperty(obj, "age", {
    configurable: true, //  数据描述符 和 访问描述符 共同有的
    enumerable: true, // 数据描述符 和 访问描述符 共同有的
    writable: true, // 数据描述符单有的
    value: "shuliqi", // 
    set(value) {    // 访问器描述符
      console.log('set', value);
      this._age = value;
    },
    get() { // 访问器描述符
        console.log('get');
        return this._age;
    },
  });
  obj.age = 12
  console.log(obj.age);
}
```

结果是页面报错了

{% asset_img 1.jpg %}