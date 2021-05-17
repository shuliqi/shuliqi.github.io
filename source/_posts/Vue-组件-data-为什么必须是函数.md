---
title: Vue 组件 data 为什么必须是函数
date: 2018-02-14 14:58:36
tags: Vue
categories: Vue
---

在使用`vue`开发的时候，有一点觉得非常奇怪；使用`new Vue()`的时候，`data`是可以传入一个对象的；但是在组件中`data`就必须为一个函数；`vue`的官方文档是这么写的：

> 通过`Vue`构造器传入的各种选项大多数是可以在组件里使用。`data`是一个例外，它必须是一个函数。如果定义成了一个对象，那么`Vue`就会停止。并且会在控制台发出警告，告诉你在组件中`data`必须是一个函数。

```js
// new Vue() ---- 实例
const app = new Vue({
  el: "#app",
  data: {
  	name: shuliqi,
  	age: 12
  }
})

// 组件
Vue.compoment("people", {
	data: function () {
		return {
        name: shuliqi,
        age: 12
		}
	}
})
```

这是为什么？为什么在组件中`data`就必须为一个函数。而`new Vue()`的`data`可为函数，可为对象

 <!--more-->

## vue 组件的data 必须为函数的原因

最根本的原因是是我们**js对于对象是传引用地址的**。 如果我们传了一个对象进去。那么依此配置初始化多个实例之后，这个对象是这些多个实例共享的。

例子1：如果 `data` 是一个对象

```javascript
function Ppeople (){};
Ppeople.prototype.data = {
  name: "shuliqi",
  age: 12
}
const p1 = new Ppeople();
const p2 = new Ppeople();
console.log(p1.data === p2.data, p1.data.name, p2.data.name); 
// true shuliqi shuliqi

p2.data.name = "舒丽琦";
console.log(p1.data.name, p2.data.name); // 舒丽琦 舒丽琦
```

我们可以看出来：`p1` 和 `p2` 的原型都是` People`。也就是会继承原型的属性；**因为 原型上的`data`是普通对象，属于引用类型**。所以`P1` 和`p2`的`data`其实都是指向同一块内存地址。严格运算符判断都是相等的，说明他们的值相等，内存地址也相同，所以修改其中一个也会影响另外一个。

例子2：如果`data`是一个函数

```javascript
function Ppeople (){
  this.data = this.data();
};
Ppeople.prototype.data = () => {
  return  {
    name: "shuliqi",
    age: 12
  }
}
const p1 = new Ppeople();
const p2 = new Ppeople();
console.log(p1 === p2, p1.data.name, p2.data.name ); 
// false shuliqi shuliqi
p2.data.name = "舒丽琦";
console.log(p1.data.name, p2.data.name); // shuliqi 舒丽琦
```

我们在执行 `new`的时候，`Ppeople`其实充当了`constructor`。这时候的`this.data`还是一个函数，还没执行的函数。所以调用一下`this.data()`让函数返回一个值然后重新赋值给`this.data`。

`Ppeople`原型的`data`使用了`function`。使用了`function`后，`data`都被锁定在当前的`function`作用域中，然后被`return`出去。 **相当于创建了另外一个对象，所以多个实例之间互相不影响**

## new Vue()的data为对象为函数都可行的原因

我们看如下的例子：

```javascript

function Ppeople ({ data }){
  this.data = data;
};

const p1 = new Ppeople({ data:{ name:"shuliqi", age: 12 } });
const p2 = new Ppeople({ data:{ name:"shuliqi", age: 12 } });
console.log(p1 === p2, p1.data.name, p2.data.name ); // shuliqi shuliqi
p2.data.name = "舒丽琦";
console.log(p1.data.name, p2.data.name); // shuliqi 舒丽琦
```

这里的变量声明方式是直接放在了构造函数中，并不是通过原型链来查找的，这也就是为什么`new Vue`的时候 `data`说是可以直接为非函数。在构造函数执行的时候，`data`就已经相互隔离。

















