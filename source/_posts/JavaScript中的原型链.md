---
title: JavaScript中的原型链
date: 2018-06-01 11:29:14
tags: JavaScript
categories: JavaScript
---

对于`JavaScript`中的原型链，一直很疑惑，看不懂，最近读到一篇文章，觉得豁然开朗。记录一下。关于`javaScript`为什么会有原型链，可以点击 [Javascript继承机制的设计思想](http://www.ruanyifeng.com/blog/2011/06/designing_ideas_of_inheritance_mechanism_in_javascript.html)讲得非常清楚

# 调对象和函数对象

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

上面中 `obj1`,`obj2`,`obj3`都是普通对象，内置的 `Function`，`Object`都是函数对象，`f1`,`f2`,`f3`都是函数对象。

它们之前的区分是什么？ 

> 凡是通过`new Function()`创建的对象都是函数对象，其他的都是普通对象。

F3 通过 `new Function `创建的，所以是函数对象，`Object`和`Function` 是自带的函数对象。`f1`,` f2`归根到底通过`new Function` 创建的。可能会觉得`f1`,` f2`会觉得有点疑惑。 我们验证一下：

```javascript
function f1() {}
const f2 = function() {};
console.log(f1 instanceof Function); //  true
console.log(f2 instanceof Function); //  true
```

# 构造函数

```javascript

function MyPerson(name, age) {
  this.name = name;
  this.age = age;
}
const person1 = new MyPerson("舒丽琦", 18);
const person2 = new MyPerson("shuliqi", 20);
console.log(person1.constructor === MyPerson); // true
console.log(person2.constructor === MyPerson); // true
```

这个例子中 `MyPerson` 我们叫构造函数， `person1` 和` person2` 是构造函数  `MyPerson` 的实例`person1` 和` person2` 这两个实例都有一个 `constructor `属性，这个属性是一个指针，指向构造函数` MyPerson`

# 原型对象

在`JavaScript`中， 每当定义一个对象（函数也是对象）的时候， 对象中都会有一些预定义的属性。其中每个**函数对象**都会有`prototype`属性。这个属性指向个对象的**原型对象**

```javascript

// 定义了一个对象 MyPerson
function MyPerson(name, age) {
  this.name = name;
  this.age = age;
}
// 定义的这个对象的有一个prototype， 指向对象的原型
MyPerson.prototype = {
  type: "人类",
  job: "高高高级开发",
  getName: function() {
    return this.name;
  }
}
const person1 = new MyPerson("舒丽琦", 18);
const person2 = new MyPerson("shuliqi", 20);
console.log(person1.getName()); // 舒丽琦
console.log(person2.getName()); // shuliqi
```

上面代码中， 我们定义了一个对象（`MyPerson`），因为是定义的函数对象，所以有`prototype`属性。这个属性指向对象原型（`MyPerson.prototype`的就是原型对象 ）。

 很容易看出原型对象就是一个普通的对象:

```javascript
function MyPerson() {}
console.log(typeof MyPerson.prototype ); // object
```

而普通对象是由构造器 `Object`构造的， 下面会讲到构造器。

## constructor

> 默认情况下，所有的原型对象都会自动获得一个` constructor` 属性（构造函数属性），这个属性是一个指针，指向`prototype` 属性所在的函数（`MyPerson`）

```javascript
// 定义了一个对象 MyPerson
function MyPerson(name, age) {
  this.name = name;
  this.age = age;
}
// 对象 MyPerson 的原型的 constructor 属性
console.log(MyPerson.prototype.constructor === MyPerson); // true
```

在上面的小节中，我们实例的`constructor`（构造函数属性）指向构造函数`person1.constructor === MyPerson`。 这两个指向是一样的。

```javascript

// 定义了一个对象 MyPerson
function MyPerson(name, age) {
  this.name = name;
  this.age = age;
}

// MyPerson 的实例 person1
const person1 = new MyPerson("舒丽琦", 18);

// 对象 MyPerson 的原型的 constructor 属性
console.log(MyPerson.prototype.constructor === MyPerson); // true

// 实例的 constructor 属性
console.log(person1.constructor === MyPerson); // true
```

`person1`为什么会有` constructor` 属性？ 那是因为` person1 `是`MyPerson `的实例。

但是`MyPerson`的原型对象（`MyPerson.prototype`）为什么有 `constructor` 属性？ 同理，`MyPerson.prototype` 应该就是 MyPerson 的一个实例。

也就是说明在定义一个函数对象的时候， 创建了一个它自己的实例对象，并且赋值给它的原型对象。类似如下的过程：

```javascript
 var A = new Person();
 Person.prototype = A;
// 注：上面两行代码只是帮助理解，并不能正常运行
```



那么我们可以得到这样结论：

> 原型对象是构造函数的一个实例

**注意：**如果我们改写了 对象原型，那么原型的`constructor`就不是`MyPerson`了， 已经被改写了

## Function.prototype

以上我们可以知道，原型对象是一个普通的对象，但是 `Function.prototype`是一个例外，它不是普通对象而是函数对象？ 并且它没有`prototype`属性（前面讲过：函数对象都有`prototype`属性）为什么？ 

根据上面`constructor`这个小节可知：原型对象是构造函数的一个实例。那么`Function.prototype`就是 `Function`的一个实例。类似下面这样的过程：

```javascript
 var A = new Function ();
 Function.prototype = A;
```

而上面我们提到**凡是通过 new Function() 产生的对象都是函数对象**。上面的`A`是函数对象，所以`Function.prototype`是一个函数对象。

## 原型的作用

那原型对象是用来最什么的呢？ 它的主要做作用是是继承。

````javascript
function MyPerson(name, age) {
  this.name = name; // 这个this 指的是 new 出来的实例
  this.age = age; // 这个this 指的是 new 出来的实例
}
MyPerson.prototype = {
  getName: function() {
    return this.name;
  }
}
const person1 = new MyPerson("舒丽琦", 18);
console.log(person1.getName()); // 舒丽琦
````

上面代码中， 我们改了 `MyPerson`的原型，使其有一个 `getName`方法。`person1`是`MyPerson`的一个实例。这个实例继承了对象原型的方法。

关于是如何继承， 我们就需要讲到下面的原型链了。

#  \__proto__

`JavaScript`在创建对象的时候（不论是函数对象还是普通对象），都有一个叫做`__proto__`属性。这属性指向创建它的构造函数的原型对象。

那根据以上全部我们可以得到 一个实例与它构造函数，构造函数的原型对象之间的关系如下：

```javascript
// 定义了一个对象 MyPerson
function MyPerson(name, age) {
  this.name = name;
  this.age = age;
}
// MyPerson 的实例 person1
const person1 = new MyPerson("舒丽琦", 18);

console.log(person1.__proto__ === MyPerson.prototype); // true
console.log(person1.constructor === MyPerson); // true
console.log(MyPerson.prototype.constructor === MyPerson); // true
```

可以得出很重要的一点：**实例与构造原型对象之间是通过`__proto__`来进行链接**

# 构造器

我们创建一个对象的时候：

```javascript
var obj = {};
```

它等同于

```javascript
var obj = new Object();
```

`obj `是构造函数 `Object` 的一个实例； 所以：

```javascript
obj.constructor === Object
obj.__proto__ === Object.prototype
```

新对象`obj`是使用` new `操作符后跟一个构造函数来创建的。构造函数`Object` 本身就是一个函数对象。它和我们之前的构造函数`MyPerson`是差不多的。只不多这个函数是出自于创建新对象的目的而定义的。

同理，可以创建对象的构造器不仅仅有`Object`还有其他的构造器如：`Array`，`Date`，`Function`等。

用不同的构造器来创建不同的对象：

```javascript

const obj = {}
console.log(obj.constructor === Object); // true
console.log(obj.__proto__ === Object.prototype); // true

const obj = new Object();
console.log(obj.constructor === Object); // true
console.log(obj.__proto__ === Object.prototype); // true

const a = new Array();
console.log(a.constructor === Array);  // true
console.log(a.__proto__ === Array.prototype);  // true

const s = new String();
console.log(s.constructor === String);  // true
console.log(s.__proto__ === String.prototype); // true

const d = new Date();
console.log(d.constructor === Date); // true
console.log(d.__proto__ === Date.prototype); // true

const f = new Function();
console.log(f.constructor === Function); // true
console.log(f.__proto__ === Function.prototype);  // true

const n = new Number();
console.log(n.constructor === Number);  // true
console.log(n.__proto__ === Number.prototype);  // true

const b = new Boolean();
console.log(b.constructor === Boolean);  // true
console.log(b.__proto__ === Boolean.prototype);  // true
```

而这些构造器都是函数对象：

{% asset_img 1.png %}

所以我们自己写构造函数（ MyPerson）和构造器都会有`Function`构造的。

# 原型链



总结一下全部的知识点

- 每个函数对象都有`prototype`属性，指向自己的原型、

- 每个对象都有`__proto__`属性， 指向自构造自己的构造函数的的原型。

- 而构造函数的原型也是对象， 也有`__proto__`属性，构造函数和构造器一样都是函数对象，是由`Function`构造的。所以构造函数的原型的`__proto__`属性指向`Function.prototype`。

- 而`Function.prototype`也是一个对象，也有`__proto__`属性。在`JavaScript`中万物皆对象。所以`Function.prototype`是由`Object`构造的。所以`Function.prototype`的`__proto__`属性指向 `Object`的原型。prototype`。

  ```
  Function.prototype.__proto__ === Object.prototype; // true
  ```

  Function.prototype是一个特殊的对象，它是函数对象，并且没有`prototype`属性。 

- `Object.prototype`也是一个对象，也有`__proto__`属性。但是它指向 `null`， 指原型链的终点。

​                                                                                                                    

而上面的对象也原型之前是通过`__proto__`来进行链接。这样就构成了原型链。



根据以上内容，我们来做做下面的测试：

```javascript
function MyPeople() {};
const person1 = new MyPeople();

```

1. `person1.__proto__ === ?`
2. `MyPeople.__proto__ === ?`
3. `Person.prototype.__proto__ === ?`
4. `Object.__proto__ === ?`
5. `Function.prototype === ?`
6. `Object.prototype.__proto__  === ?`



- `person1.__proto__ === ?`:

  每个对象都有`__proto__`属性，指向当前对象的构造函数的原型，`person1`的构造函数是`MyPeople`。所以：`person1.__proto__ === MyPeople.prototype`

- `MyPeople.__proto__ === ?`:

  `Person` 也是一个对象, 所以它也有`__proto__`属性， 指向当前对象的构造函数的原型。我们自己写的构造函数和自带的构造器是一样的都是函数对象，所以`MyPeople`的构造函数是`Function`。所以：`MyPeople.__proto__ === Function.prototype`。

  

- `Person.prototype.__proto__ === ？`:

  构造函数的原型也是一个对象，也有`__proto__`属性。构造函数的原型都是普调对象。普通对象的话那就是由构造器`Object` 构造的。 所以：`Person.prototype.__proto__` === Object.prototype 

- `Object.__proto__ === ?`

  构造器`Object`也是一个对象，也有`__proto__`属性。 构造器跟我们自己写的构造函数一样， 都是函数对象，所以`Object`是由`Function`构造的。 所以：`Object.__proto__ === Function.prototype`。

- `Function.prototype === ?`

  `Function.prototype`也是一个对象。但是它比较特，他是函数对象（前面讲过构造函数的原型就是一个普通函数）。并且它没有`prototype`属性。在`JavaScript`中万物皆对象。所以`Function.prototype`是由`Object`构造的。所以`Function.prototype__proto__ === Object.prototype`。

- `Object.prototype.__proto__  === ?`

  `Object.prototype`也是一个对象，也有`__proto__`属性。但是它比较特殊，它为`null`,`null`为原型链的顶端。



所以现在来看这个原型链图应该明白很多：

{% asset_img 3.png %}





参考文章：

- https://www.jianshu.com/p/dee9f8b14771

- https://www.jianshu.com/p/652991a67186

- https://www.jianshu.com/p/a4e1e7b6f4f8























