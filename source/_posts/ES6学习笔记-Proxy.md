---
title: ES6学习笔记- Proxy
date: 2018-03-05 12:00:01
tags: ES6
categories: 学习笔记

---



之前深入了解了**[Object.defineProperty](http://localhost:4000/2018/02/19/%E6%B7%B1%E5%85%A5%E4%BA%86%E8%A7%A3Object-defineProperty/)**。 发现ES6有一个新的对象（Proxy）也能实现属性拦截器的作用，但是功能更加强大。所以这里也学习学习，总结总结。

学习都是来自[阮一峰老师](http://www.ruanyifeng.com/)。

<!--more-->

### Proxy概述

**Proxy**对象在目标对象的外层设置一层拦截器，外界对目标的某些操作，必须通过这层拦截

```javascript
const proxy = new Proxy(target, handler)
```

**new proxy**表示生成一个proxy实例；

**target：**表示拦截的目标对象；
**handler：** 表示拦截的行为， 也是一个对象；

**注意：**要使得**Proxy **起作用， 就必须针对**Proxy **实例（上面的例子就是**proxyObj**对象）进行操作，而不是针对目标对象（上面的例子就是(**obj对象**)进行操作。

### proxy拦截操作--handler

##### get()方法

拦截对象属性的读取。 比如 **proxyObj.name** 和 **proxyObj[name]**

```javascript
get(target, propKey, recevier){}
```

get()方法接受三个参数

1. **target:** 目标对象
2. **propKey:** 读取的属性名
3. **recevier：**Proxy实例本身

```javascript
const obj = {
  name: "shuliqi",
};
const proxyObj = new Proxy(obj, {
  get(target, propKey, recevier) {
    if (target[propKey]) {
      return target[propKey];
    } else {
      console.log("找不到该属性的值");
    }
  },
});
console.log(proxyObj.name); // shuliqi
console.log(proxyObj.age);  // 找不到该属性的值
```

这个例子代码表示， 如果访对象不存在的属性， 会打印“找不到该属性的值”。 如过没有这个拦截， 访问不存在的属性，， 只会返回undefined。

##### set()方法

拦截某个属性的赋值操作。比如： 比如 **proxyObj.name = "shuliqi"** 和 **proxyObj[name] =. "shuliqi"**

```javascript
set(target, propKey，value, recevier){}
```

 set()方法有三个参数：

1. **target:** 目标对象
2. **propKey:** 读取的属性名
3. **value：** 属性值
4. **recevier：**Proxy实例本身

```javascript
const obj = {
  name: "shuliqi",
};
const proxyObj = new Proxy(obj, {
  set(target, propKey, value, recevier) {
    if (propKey === "age") {
      if (!Number.isInteger(value)) {
        // 如果不是整数
        console.log("age需要是一个整数");
      } else {
        console.log("符合赋值");
        target[propKey] = value;
      }
    }
  },
});
proxyObj.age = 12.1; // age需要是一个整数
proxyObj.age = 25; // 符合赋值
console.log(obj.age); // 25
```

这个例子如果age赋值的不是整数，会拦截到并且log“age需要是一个整数”。如果符合的话log“符合赋值” . 并且成功赋值。

##### apply()方法

拦截函数的调用，**call** 和 **apply**的操作。

```javascript
apply(target, context, args){}
```

apply()方法有三个参数：

1. **target：** 目标函数对象
2. **context:** 目标函数对象的上下文对象(this)
3. **args：**目标函数对象的参数数组

```javascript
const targetFn = () => "我是目标函数对象";
const proxyFn = new Proxy(targetFn, {
  apply(_proxyFntarghet, context, args) {
    console.log("我目标函数代理对象");
  },
});
proxyFn.call(); // 我目标函数代理对象
proxyFn.apply();// 我目标函数代理对象
proxyFn(); // 我目标函数代理对象

```

例子里面proxyFn 是 targetFn的实例，当被调用的时候，就会被apply方法拦截到。返回“我目标函数代理对象”



##### has() 方法

拦截判断某个对象是否具有某个属性时(hasProperty)， 这个方法会生效。 典型的操作比如 in 操作

```javascript
has(targe, propKey){}
```

has() 方法 有两个参数：

1. **target：** 目标对象‘
2. **propKey:** 需要查询的属性名；

```javascript
  const target = { name: "shuliqi" };
  const proxyObj = new Proxy(target, {
    has(target, key) {
      // 拦截的时候， 我们都给判断未false
      return false;
    },
  });
  "name" in target; // true
  "name" in proxyObj; // false
```

例子里面， 我们代理对象用has()方法拦截判断对象是否具有某个属性时， 拦我们拦截到的时候都个片段未false， 所以使用代理对象判断时返回的都是false( "name" in target)。如果我们没有使用代理对象的话，那么是对象target 是具有name属性的。

**注意：**has()方法只能拦截hasProperty， 而不是hasOwnProperty. 所以是不能判断一个属性是否是一个对象的自身属性。

 还有注意的就是：has 对 for...in... 是不管用的的











未完， 待学习....

