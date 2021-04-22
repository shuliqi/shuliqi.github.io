---
title: 关于this的指向问题
date: 2018-07-02 15:07:02
tags: JavaScript
---



this关键字是`JavaScript`中最复杂的机制之一，是一个特别的关键字，被自动定义在所有函数的作用域中，但是相信很多`JavaScript`开发者并不是非常清楚它究竟指向的是什么。听说你很懂this,是真的吗？

 <!--more-->

## 普通函数

**函数的`this`指向在函数定义的时候是不能确定的。只有在函数执行的时候才能确定函数的`this`指向谁**

总结普通函数的绑定规则主要分为以下这几种：

### 默认绑定

默认绑定是在不能应用其他绑定规则的时候使用的默认规则， 通常是独立函数调用。

例子1:

```javascript
function getName() {
    console.log(this.name)
}
var name = 'shuliqi';
getName(); // shuliqi
```

在调用`getName`的时候，使用了默认绑定，`this`指向了全局对象。严格模式下，`this`指向`undefined`，``undefined`上没有this对象，会抛出错误。

这里例子是`window`调用了独立的函数，`this`就是指向`window`（非严格模式就是`windom `）

这里的例子可以这么看：

```javascript
function getName() {
    console.log(this.name)
}
var name = 'shuliqi';
window.getName(); // shuliqi
```

### 隐式绑定

函数的调用时在某个对象上触发的。即调用位置上存在上下文对象。典型的形式为` XXX.fun()`。

例子2:

```javascript
function getName() {
    console.log(this.name)
}
var person = {
    name:'shuliqi2222',
    getName: getName,
}
var name = 'shuliqi11111';
person.getName(); // shuliqi2222
```

是对象`person`调用函数`logName`， 所以`this`指向`person`。

`getName`函数在外部声明， 严格来说并不属于`person`，但是在调用`getName`时,调用位置会使用`person`的上下文来引用函数，隐式绑定会把函数调用中的`this`(即此例`getName`函数中的this)绑定到这个上下文对象（即此例中的`person`）.

注意：**对象属性链中只有最后一层会影响到调用位置**

例子3:

```javascript
function getName() {
    console.log(this.name)
}
var person1 = {
    name:'shuliqi11111',
    getName: getName,
}
var person2 = {
    name:'shuliqi2222',
    person: person1,
}
var name = 'shuliqi';
person2.person.getName(); // shuliqi11111
```

因为只有最后一层会确定`this`指向的是什么，不管有多少层，在判断this的时候，我们只关注最后一层，即此处的`person`。

**隐式绑定**有一个很大的缺陷，就是很容易丢失（即容易给我们造成误导，我们以为`this`指向的是什么，但是实际上并非如此）

例子4:

```javascript

function getName() {
    console.log(this.name)
}
var person1 = {
    name:'shuliqi11111',
    getName: getName,
}
var person2 = {
    name:'shuliqi2222',
    person: person1,
}
var name = 'shuliqi';
var logName = person2.person.getName;
logName(); // shuliqi
```

为什么结果是 `shuliqi`？

这是因为l`ogName` 直接指向了`getName`的引用， 跟`person`没关系。针对此类问题，我建议大家只需牢牢继续这个格式:`XXX.fn();fn()`前如果什么都没有，那么肯定不是隐式绑定。

除了上面这种丢失之外，隐式绑定的丢失是发生在回调函数中(事件回调也是其中一种)，我们来看下面一个例子:

```javascript
function sayHi(){
    console.log('Hello,', this.name);
}
var person1 = {
    name: 'YvetteLau',
    sayHi: function(){
        setTimeout(function(){
            console.log('Hello,',this.name);
        })
    }
}
var person2 = {
    name: 'Christina',
    sayHi: sayHi
}
var name='Wiliam';
person1.sayHi();
setTimeout(person2.sayHi,100);
setTimeout(function(){
    person2.sayHi();
},200);
```

结果为:

```
Hello, Wiliam
Hello, Wiliam
Hello, Christina
```

- 第一条输出很容易理解，`setTimeout`的回调函数中，`this`使用的是默认绑定，非严格模式下，执行的是全局对象

- 第二条输出是不是有点迷惑了？说好`XXX.fun()`的时候，`fun`中的`this`指向的是`XXX`呢，为什么这次却不是这样了！`Why`?

  其实这里我们可以这样理解: `setTimeout(fn,delay){ fn(); }`,相当于是将`person2.sayHi`赋值给了一个变量，最后执行了变量，这个时候，`sayHi`中的`this`显然和`person2`就没有关系了。

- 第三条虽然也是在`setTimeout`的回调中，但是我们可以看出，这是执行的是`person2.sayHi()`使用的是隐式绑定，因此这是`this`指向的是`person2`，跟当前的作用域没有任何关系。

### 显示绑定

显示绑定就是通过`apply`，`call`，`bind`的方法， 显示的指定`this`所指的对象。

`apply`，`call`，`bind`的第一个参数都是该函数this所指的对象。 `call`，`apply` 是一样的， 都会立即执行，只是传参方式不同。`call`和`apply`都会执行对应的函数，而`bind`方法不会。`apply`的第二个参数是一个数组。

```javascript
function sayHi(){
    console.log('Hello,', this.name);
}
var person = {
    name: 'YvetteLau',
    sayHi: sayHi
}
var name = 'Wiliam';
var Hi = person.sayHi;
Hi.call(person); //Hi.apply(person)
```

结果：`Hello, YvetteLau`。因为明确将`this`绑定在了`person`上

那么，使用了硬绑定，是不是意味着不会出现隐式绑定所遇到的绑定丢失呢？显然不是这样的，不信，继续往下看。

```javascript
function sayHi(){
    console.log('Hello,', this.name);
}
var person = {
    name: 'YvetteLau',
    sayHi: sayHi
}
var name = 'Wiliam';
var Hi = function(fn) {
    fn();
}
Hi.call(person, person.sayHi); 
```

输出的结果是 H`ello, Wiliam.` 原因很简单，`Hi.call(person, person.sayHi)`的确是将`this`绑定到Hi中的`this`了。但是在执行`fn`的时候，相当于直接调用了`sayHi`方法(记住`: person.sayH`i已经被赋值给`fn`了，隐式绑定也丢了)，没有指定`this`的值，对应的是默认绑定。

如果我们现在希望绑定不要丢失， 我们该怎么做？ 很简单， 在调用`fn`的时候也是用显示绑定。

```javascript
function sayHi(){
    console.log('Hello,', this.name);
}
var person = {
    name: 'YvetteLau',
    sayHi: sayHi
}
var name = 'Wiliam';
var Hi = function(fn) {
    fn.call(this);
}
Hi.call(person, person.sayHi);
```

此时，输出的结果为`: Hello, YvetteLau`，因为`person`被绑定到`H`i函数中的`this`上，fn又将这个对象绑定给了`sayHi`的函数。这时，`sayHi`中的`this`指向的就是`person`对象。



如果我们将`null`或者是`undefined`作为`this`的绑定对象传入`call`、`apply`或者是`bind`,这些值在调用时会被忽略，实际应用的是默认绑定规则。

```javascript
function getName() {
    console.log(this.name)
}
var person = {
    name:'shuliqi11111',
    getName: getName,
}
var hi = function(fn) {
    fn()
}
var name = 'shuliqi'
hi.call(null, person.getName);  //  shuliqi
```



### new 绑定

`new`的原理：

- 创建一个空对象
- 将空对象的原型指向构造函数的原型属性，从而继承构造原型上的方法（`newObj.__proto__ === Fn.prototype`）
-  将构造函数的`this`指向新建的对象，并且执行构造函数的代码，从而获得构造函数的私有属性
- 最后看构造函数是否是返回一个对象，如果是直接返回该对象，如果不是， 则返回我们新建对象

```javascript
function Fn() {
    this.name = "shuliqi";
}
var newFn = new Fn();
console.log(newFn.name); // shuliqi
```

这里`newFn`对象子所以可以点`name`出来，是因为`new`关键字可以改变`this`指向。

**实现new:**

```javascript

function MyNew(Fn, ...args)  {
  // 1.创建一个空对象
  // 2. 将空对象的原型指向构造函数的原型属性，从继承构造函数原型的方法（obj.__proto__ === Fn.prototype）
  const obj = Object.create(Fn.prototype);

  // 3. 将构造函数的thia，指向新建的对象，并且执行构造函数，从而获得私有属性
  const result = Fn.apply(obj, args);

  // 4.如果构造函数返回的是对象， 我们就返回此对象，不然返回新建的对象
  return result instanceof Object  ? result : obj;
}
```

**测试：**

````javascript
function People(name) {
  this.name = name;
}
People.prototype.getName = function() {
  console.log(this.name)
}
const MyPeople  = MyNew(People, 'shuliqi');
console.log(MyPeople); // People { name: 'shuliqi' }
MyPeople.getName(); // shuliqi
````



### 普通函数的绑定优先级

我们知道了this有四种绑定规则，但是如果同时应用了多种规则，怎么办？

显然，我们需要了解哪一种绑定方式的优先级更高，这四种绑定的优先级为:

`new`绑定 > 显式绑定 > 隐式绑定 > 默认绑定



## 箭头函数

**箭头函数`this`的定义： 箭头函数的`this`在定义的时候绑定。而不是在执行的时候绑定。它的`this`指向在定义的时候继承自外层第一个普通函数的`this`。**

```javascript
var person = {
    name:'shuliqi11111',
    getName() {
        var logName = () => {
            console.log(this.name)
        }
        logName();
    },
}
person.getName(); // shuliqi11111
```

内层箭头函数`logName`本身并没有`this`对象。它的`this`对象来自于外层作用域。`logName`函数的外层函数`getName`是一个普通的函数。 它是有`this`值的指向`person`对象。所以`logName`函数的`this`指向`person`对象。

