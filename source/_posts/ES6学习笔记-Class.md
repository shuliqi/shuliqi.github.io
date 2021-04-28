---
title: ES6学习笔记-Class
date: 2018-4-10 14:26:30
tags: ES6
categories: ES6
---

`ES6`的`Class`很多大部分的功能，`ES5`都是可以实现的， 但是为什么还有出`Class`呢? 这是因为`ES5`中生成实例对象的方法是通过构造函数生成的。 这与我们所接触的很多语言（C++）差异很大。于是`ES6`就提供了跟传统语言的写法。可以说`Class`是一个语法糖。

<!--more-->

# 类与ES5

`ES6`的`Class`与`ES5`的构造函数有什么不一样呢？

我们先看`ES5`如何生成实例：

```javascript
function People(name, age) {
  // 父类实例对象属性：name
  this.name = name;
   // 父类实例对象属性：age
  this.age = age;
}
// 父类原型方法
People.prototype.getAge = function() {
  console.log(this.age)
}
// 父类原型方法
People.prototype.getNane = function() {
  console.log(this.name)
}
// 通过new 操作符生成实例对象people
const people = new People('shuliqi', 18);
console.log(people)
people.getAge();  // 18
people.getNane(); // shuliqi
```

上面代码我们有一个构造函数`People` ；`this`关键字代表实例对象，实例对象上有两个属性（`name`, `age` ） 在原型有两个方法（`getName`, `getAge`）。通过`new`的方式生成实例对象`people`；

我们把上面的代码改成用`class`来写：

```javascript
// 改成class
class People {
  // 构造函数
  constructor(name, age) {
     // 父类实例对象属性：name
    this.name = name;
     // 父类实例对象属性：age
    this.age = age
  }
  // 父类原型方法
  getNane() {
    console.log(this.name);
  }
  // 父类原型方法
  getAge() {
    console.log(this.age);
  }
}
// 通过new 操作符生成实例对象people
const people = new People('shuliqi', 18);
people.getAge();  // 18
people.getNane(); // shuliqi
```

上面代码我们定义来看一个 类叫`People`；里面的`constructor`方法就是构造方法、`this`表示实例对象(也就是`new People`返回的对象)的本身。`People`类里有 `getName`,`getAge`原型方法。**类里面的方法都是在类的原型上创建的**我们在**类里面写方法的时候是不需要写`function`关键字**；并且**方法与方法之间是不需要逗号分隔的，加了反而会报错**

总归来说， `ES6`的类就是构造函数的另外一种写法。为什么这么说呢？我们对比一下`ES5`:

* **类的数据类型就是函数。类本身就指向构造函数**

  ```javascript
  // ES5
  function People() {};
  console.log(typeof People); // function
  console.log(People.prototype.constructor === People); // true
  
  // ES6
  class People{}
  console.log(typeof People); // function
  console.log(People.prototype.constructor === People); // true
  ```

* **使用类的时候都是直接通过`new`的命令来使用**

  ```javascript
  // ES5
  function People() {};
  const people = new People();
  
  // ES6
  class People{}
  const people = new People();
  ```

* **`ES6`的类也有在`prototype`属性。类中定义的所有方法都是定义在类的`prototype`属性上面**；

  首先我们先来了解`ES5`的一些概念：

  **构造函数：**

  js规定，每个构造函数都有一个`prototype`属性，指向另外一个对象（我们也叫原型对象或者`prototype对象`），这个对象的所有属性和方法都会被构造函数所拥有，我们通常把不变的方法方法定义在`prototype对象`上，这样构造函数生成的实例对象就可以共享这些方法。

  **实例对象：**

  构造函数生成的实例对象有一个属性`__proto__`，指向构造函数的原型对象；所有构造函数生成的实例对象的`__proto__`与构造函数的`prototype`是等价的；

  实例对象方法和属性的查找规则：先在对象自己身上找，没有再去构造函数的原型对象上查找，还是没有就继续沿着原型链查找。

  回到正题：我们`ES6`的`类`也是有如`ES5`这么一套逻辑的(即有自己的prototype属性)。类中定义的方法都是定义在类的`prototype对象上 `的。

  ```javascript
  class People{
    constructor(){}
    getName() {}
    getAge() {}
  }
  // 等价
  People.prototype.constructor = function(){}
  People.prototype.getName = function(){}
  People.prototype.getAge = function(){}
  ```

  因此我们类生成的实例上调用的方法其实都是调用`prototype`对象上的方法。 

  ```javascript
  class People{
    constructor(){}
    getName() {}
    getAge() {}
  }
  const people = new People();
  console.log(people.constructor === People.prototype.constructor); // true
  console.log(people.getName === People.prototype.getName);  // true
  console.log(people.getAge === People.prototype.getAge);  // true
  ```

  

  

但是`ES6`的类与`ES5`的构造函数还是有不一样的地方的，我们看

* **类中定义的方法都是不可枚举的**

  ```javascript
  class People {
    constructor(name, age) {
      this.name = name;
      this.age = age;
    }
    getName() {
      return this.name;
    }
    getAge() {
      return this.age;
    }
  }
  Object.keys(People.prototype); // []
  Object.getOwnPropertyNames(People.prototype); // [ 'constructor', 'getName', 'getAge' ]
  ```

  可以看出不可枚举。但是`ES5`写的构造函数定义的方法是可以枚举的

  ```javascript
  
  function People(name, age) {
    this.name = name;
    this.age = age;
  }
  People.prototype.getName = function() {
    return this.name;
  }
  People.prototype.getAge = function() {
    return this.age;
  }
  Object.keys(People.prototype); // [ 'getName', 'getAge' ]
  Object.getOwnPropertyNames(People.prototype); // [ 'constructor', 'getName', 'getAge' ]
  ```


-----



# constructor

我们在定义类的时候， 必须有`constructor`方法，`constructor`方法是默认方法，如果没有显式的定义，一个空的`constructor`方法会默认添加。

```javascript
class People {}
// 等同于
class People {
  constructor () {}
}
```

上面代码中我们定义了一个空类 `People`，js引擎会自动给它添加`constructor`方法。



使用`new`生成对象的时候， 会自动调用该方法。`constructor`方法默认返回实例对象(即`this`)

```javascript
class People{
 constructor() {
   console.log('使用new命令生成实例的时候， 会被自动调用')
 }
}
const people = new People();
// 使用new命令生成实例的时候， 会被自动调用
```

我们使用`new`命令生成实例的时候，`constructor`方法会被调用。



类使用的时候必须使用**new 调用，否则会报错。**

```javascript
class People{
 constructor() {
   console.log('使用new命令生成实例的时候， 会被自动调用');
 }
}
const people =  People();
// TypeError: Class constructor People cannot be invoked without 'new'
```

但是普通构造函数式可以不用`new`的。

```javascript
function People(name, age) {
  this.name = name;
  this.age = age;
  console.log('调用')
}
People(); // 调用
```



# 类的实例

生成一个类的实例，也是使用一个`new`命令。如果不用，将会报错（上一节说过）；

```javascript
class People {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  getName() {
    return this.name;
  }
  getAge() {
    return this.age;
  }
}
//  Class constructor People cannot be invoked without 'new'
const people = People('shuliqi', 18)
// 生成类 People 的实例people
const people = new People('shuliqi', 18)
```



在一个类中， 属性除非是定义在本身(即定义在this对象上)。否则都是定义在原型上。

```javascript
class People {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  getName() {
    return this.name;
  }
  getAge() {
    return this.age;
  }
}
const people = new People('shuliqi', 18)
console.log(  people.getAge()  ); // 18
console.log(  people.hasOwnProperty('name')  ) //  true
console.log(  people.hasOwnProperty('age')  ) //  true
console.log(  people.hasOwnProperty('getName')  ) //  false
console.log(  people.hasOwnProperty('getAge')  ) //  false
console.log(  people.__proto__.hasOwnProperty('getAge')  ) //  true
```

这代码我们定义了一个类叫`People`，属性`name`,`age`实例对象`people`的自身属性(因为定义在`this` 上), 所有`hasOwnProperty()`方法返回了`true`。而`getName`,`getAge`是原型对象的属性（因为定义在类上）。所以`hasOwnProperty()`返回`false`。



跟上面讲过的一样，类的实例共享一个原型对象。

```javascript
class People {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  getName() {
    return this.name;
  }
  getAge() {
    return this.age;
  }
}
const p1 = new People('shuliqi', 12);
const p2 = new People('shulina', 22);
console.log( p1.__proto__ === p2.__proto__ ); // true
```



# 属性表达式

类的属性名可以采用表达式，和`ES5`也是一样的

```javascript
// 方法名变量(表达式)
const getName = 'getName';
class People {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  [getName]() {
    return this.name;
  }
  getAge() {
    return this.age;
  }
}

const people = new People('shuliqi', 18)
console.log(  people.getName()  ); // shuliqi
```

上面代码中，`People`类的方法名`getName`，是从表达式得到的。



## 取值函数(getter)和存值函数(setter)

在类的内部可以使用`get`,`set`关键字。对一个属性进行设置值和取值。

```javascript

class People {
  constructor(name){
    this._name = name;
  }
  get name() {
    return this._name;
  }

  set name(value) {
    this._name = value;
  }
}
const people = new People();
people.name = 'shuliqi';
console.log(people.name); // shuliqi
```

**注意：** 我们存取值函数的函数名和里面的变量名不能一致。如：

```javascript
class People {
  constructor(name){
    this.name = name;
  }
  get name() {
    return this.name;
  }

  set name(value) {
    this.name = value;
  }
}
const people = new People();
people.name = 'shuliqi';
console.log(people.name); // shuliqi
```

这样写是会报错的。这是因为，在构造函数中执行`this.name=name`的时候，就会去调用`set name`，在set name方法中，我们又执行`this.name = name`，进行无限递归，最后导致栈溢出(RangeError)。



## Class 表达式

类也可以使用表达式的方式定义。

```javascript

const People = class MyPeople {
  constructor(name) {
    // MyPeople 只能在类的内部使用
    MyPeople.name = name;
  }
}

// 在外部只能使用类的引用名：People
const people = new People('shuliqi')
```

如上，我们使用表达式的方式定义了一个类。这个类的名字是`MyPeople`。但是`MyPeople`只能在类的内部使用，指向的是当前类。在类的外部只能使用`People`引用。



当然， 如果内部不用， 可以省略类名。

```javascript
const People = class {
  constructor(name) {
    this.name = name;
  }
}
// 在外部只能使用类的引用名：People
const people = new People('shuliqi')
```



采用表达式定义类， 可以写出立即执行的`Class`。

```javascript
const people = new class {

  constructor(name) {
    this.name = name;
  }
  getName() {
    console.log(this.name)
  }
}('shuliqi')
people.getName();
// shuliqi
```





# Class 注意的点



## 严格模式

类和模块的内部默认就是严格模式，所以我们不需要使用`use strict`指定运行模式。只要你的代码写在类和模块当中。就只有严格模式可以使用。

## 不存在提升

类是不存在提升的（这和`ES5`完全是不一样的）

```javascript
// 不存在声明提升， 在一个类定义好之前使用，直接报错
const people = new People();
class People {}
// ReferenceError: Cannot access 'People' before initialization
```

上面的代码 `People`类使用在前，定义在后，这样使用是会报错的。因为`ES6` 不会把类的声明提升到代码块的顶部的。那为什么有这样的规定呢？ 这是有原因：这和继承有关，必须保证子类在父类之后定义；

```javascript
let People = class {};
class MyPeople extends People {}
```

上面的代码不会出错，因为`Mypeople`在继承`People`的时候，`People`已经定义了。但是如果存在类的声明提升，上面的代码就会出错了，这是因为`class` 会被提升到代码块的顶部， 但是`let`不会提升，就会导致`Mypeople`在继承`People`的时候。`People`没有定义。

## name 属性

`name`属性总是返回紧跟在`class`关键字后面的类名。

```javascript
class  People {};
console.log(People.name); // People
```

## this 指向

类的达内部如果有`this`，它默认指向类的实例，但是单独使用该方法的时候，就可能会报错。

```javascript
class People {
  constructor (name) {
    this.name = name;
  }
  getName() {
    console.log(this.name);
  }
}
const people = new People('shuliqi');
people.getName();  // shuliqi
const { getName } = people;
getName(); // TypeError: Cannot read property 'name' of undefined
```

类`People`的方法`getName`中的`this`默认指向`People`的实例。但是这个方法提取出来单独调用，`this`会指向该方法运行时所在的环境（由于 class 内部是严格模式，所以 this 实际指向的是`undefined`），从而导致找不到name属性而报错。

解决的办法可以使用箭头函数。如下这么写：

```javascript
class People {
  constructor (name) {
    this.name = name;
  }
  getName = () => {
    console.log(this.name);
  }
}
const people = new People('shuliqi');
people.getName();  // shuliqi
const { getName } = people;
getName(); // shuliqi
```

箭头函数内部的`this`在定义的时候就确定好了，箭头函数的`this`总是指向外层第一个普通函数的`this`。上面代码中，箭头函数构造函数内部，它的定义生效的时候，是在构造函数执行的时候。这时，箭头函数所在的运行环境，肯定是实例对象，所以`this`会总是指向实例对象。关于this指向的问题可以看这篇文章

[关于this的指向问题](https://shuliqi.github.io/shuliqi.github.io/2018/07/02/%E5%85%B3%E4%BA%8Ethis%E7%9A%84%E6%8C%87%E5%90%91%E9%97%AE%E9%A2%98/)



# 静态方法

类相当于实例的原型，所有在类中定义的方法，都会被继承，如果我们不想让一个方法被继承，可以在方法名前加`static`关键字，该方法就就不会被实例继承，只能通过类直接调用。

```javascript
class People {
  // 静态方法 getName
  static getName() {
    console.log('shuliqi');
  }
}
People.getName(); // shuliqi
const people = new People();
people.getName(); // people.getName is not a function
```

上面定义了一个类`People`, 类`People`里面定义了一个静态方法`getName`, 该方法只能类`People直接调用`，实例`people`调用的话直接报错（该方法不存在）



**注意：** 如果在静态方法中有`this`关键字，这个`this`指向类，而不是实例。

```javascript
class People {
  constructor() {
    // 实例属性
    this.age = 19;
  }
  // 静态方法，只能类调用，实例不能调用
  static getAge() {
    // 静态方法中的this 指向 类，而不是实例
    console.log(this.age);
  }
}
// 类属性
People.age = 900;
People.getAge(); // 900

```



静态方法可以与非静态方法重名

```javascript
class People {
  // 静态方法
  static getAge() {}

  // 普通方法
  getAge() {}
}
const people = new People();
```



父类的静态方法是可以被子类继承的

```javascript
class People {
  // 静态方法
  static getAge() {
    console.log('子类可以继承父类的静态方法')
  }
}
class MyPeople extends People {}
MyPeople.getAge(); // 子类可以继承父类的静态方法

```

# 实例属性的新写法 

实例属性除了可以定义在`constructor`方法中的`this`上，也可以直接定义在类的最顶层。这样的写法看上去比较整齐

```javascript
class People {
  name = 'shuliqi';
  constructor() {
    this.age = 10;
  }

  get() {
    console.log(this.name,  this.age)
  }
}
const people = new People();
people.get(); // shuliqi 10

```



# 静态属性

静态属性是指 `Class` 本身的属性.即`Class.propName`，而不是定义在对象（`this`）的属性。

```javascript
class People {
 getAge() {
   console.log(this.age)
 }
}
People.age = 20;
console.log(People.age); // 20

const people = new People();
people.getAge(); // undefined
```

这表示给类`People`添加了静态属性`age`。只能`People.age`调用。 在`this` 是取不到的（`people.getAge(); // undefined`）



第二种方式，在实例属性前面添加`static`关键字

```javascript
class People {
 static name = 'shuliqi';
 getName() {
   console.log(this.name)
 }
}
People.name = 'shulina';
console.log(People.name); // shulina
const people = new People();
people.getName(); // undefined
```



# 类的继承

类可以通过`extends`关键字实现继承。比ES5 那么多继承方式方便的多。

```javascript
class People {
  constructor(name) {
    this.name = name;
  }
  get() {
    console.log(this.name,  this.age)
  }
}

class MyPeople extends People {
  constructor() {
     super(...arguments)
  }
  getName() {
    console.log(super.name)
  }
}
const people = new MyPeople('shuliqi');
// ReferenceError:: Must call super constructor in derived class before accessing 'this' or returning from derived constructor

```

这段代码定义了一个类 `MyPeople`, 通过关键字`extends`继承了 `People`类的所有属性和方法。我们看到``MyPeople`的构造函数和`getName`使用了`super`。那`super`这代表什么呢? 我们之后下面有讲到。



需要注意： 在子类的`construtor`中必须是用`super`方法。否则在新建实例的时候会报错。为什么呢？这是因为子类的`this`对象必须先通过父类的构造函数完成塑造，得到与父类同样的实例属性和方法。如果不调用`super`方法的话，子类就将得不到`this`对象。

```javascript
class People {
  constructor(name) {
    this.name = name;
  }
  get() {
    console.log(this.name,  this.age)
  }
}

class MyPeople extends People {
  constructor() {
     // super(...arguments)
  }
  getName() {
    console.log(super.name)
  }
}
const people = new MyPeople('shuliqi');
// ReferenceError:: Must call super constructor in derived class before accessing 'this' or returning from derived constructor

```

这代码中，子类`MyPeople`继承了父类`People`, 但是在它的构造函数中没有调用`super`方法，导致在新建实例的时候报错。

**必须先调用super方法的原因：**

ES5的继承方式：先创造子类的实例对象的`this`。然后再把父类的属性和方法加到this上面来。

ES6的继承方式：先将父类实例的属性和方法加到this上面来(所以必须先调用super方法)。然后再用子类的构造函数修改`this`



 跟定义类一样，如果子类没有定义construtor方法，那么这个方法就会被默认加上。

```javascript
 class MyPeople extends People {}

 // 等同于
 class MyPeople extends People {
   constructor() {}
 }
```



在子类的构造函数中， 只有调用了` super` 方法之后，才可以使用this关键字，不然就会报错。这是因为子类的实例的创建基于父类实例，只有`super方法才能调用父类实例。

```javascript
class People {
  constructor(name) {
    this.name = name;
  }
  get() {
    console.log(this.name,  this.age)
  }
}

class MyPeople extends People {
  constructor(name) {
    // 在调用super()之前使用this
    this.name = name
    super(...arguments)
  }
}
const people = new MyPeople('shuliqi');
// ReferenceError: Must call super constructor in derived class before accessing 'this' or returning from derived constructor
```

上面的代码在子类`MyPeople`的`constructor`方法中在使用`super()`之前是用了`this`。结果在创建实例的时候报错。



上面有讲过父类的静态方法也会被子类继承

```javascript
class People {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  // 父类的静态方法
  static getName() {
    console.log('shuliqi')
  }
}
class MyPeople extends People {
  constructor(name, age, sex) {
    // 调用super方法
    super(name, age);
    this.sex = sex; 
  }
}
// 子类继承了父类的静态方法，所以可以调用父类的静态方法
MyPeople.getName(); //  shuliqi
```

# Object.getPrototypeOf()

`Object.getPrototypeOf()`该方法用于从子类获取父类。

```javascript
class People {
  constructor(name) {
    this.name = name;
  }
  get() {
    console.log(this.name,  this.age)
  }
}

class MyPeople extends People {
  constructor(name) {
    // 在调用super()之前使用this
    this.name = name
    super(...arguments)
  }
}
console.log(Object.getPrototypeOf(MyPeople) === People) // true
```

因此，可以使用这个方法判断，一个类是否继承了另一个类。

# super 关键字

super关键字可以当函数使用，也可以当对象使用。这里可以分成下面这几类来：

## super作为函数使用

`super`作为函数使用时，代表父类的构造函数，ES6要求，子类的构造函数必须执行一次super函数；

```javascript
class People {
  constructor() {
    console.log('父类的构造函数');
  }
}
class MyPeople extends People {
  constructor() {
    // super作为函数使用时，代表父类的构造函数
    super();
  }
}
const people = new MyPeople();
// 父类的构造函数
```

这段代码子类`MyPeople`的`constructor`函数中，**`super`作为函数调用，代表父类的构造函数**。



**注意：**作为函数使用， 只能在子类的构造函数中使用，在其他地方使用的话是会报错的。

```javascript
class People {
  constructor() {
    console.log('父类的构造函数');
  }
}
class MyPeople extends People {
  constructor() {
    // super当函数使用时，代表父类的构造函数
    super();
  }
  getName() {
    // super作为函数调用时，只能在constructor中使用
    // 在其他地方使用报错
    super(); // SyntaxError: 'super' keyword unexpected here
  }
}
const people = new MyPeople();

```

## super作为对象在普通方法中使用是指向父类的原型对象

```javascript
class People {
  constructor(name) {
    // 父类实例属性
    this.name = '父类实例属性';
  }
  // 父类原型方法（类的的所有方法都是在类的原型上）
  getName() {
    console.log('父类原型方法')
  }
}

class MyPeople extends People {
  constructor() {
    super();
  }
  // 子类原型普通方法（类的的所有方法都是在类的原型上）
  getName() {
    // super在子类作为对象使用，并且在普通方法中，那么super指向父类的原型对象
    super.getName();
  }
}
const people = new MyPeople();
people.getName(); // 父类原型方法
```

`super`作为对象在普通方法中使用是指向父类的原型对象（`prototype`对象）。



**注意：**由于这种情况，`super`指向父类的原型对象，所以定义在父类实例上的属或者方法是没有办法通过`super`调用的

```javascript

class People {
  constructor(name) {
    // 父类实例属性
    this.name = '父类实例属性';
  }
  // 父类原型方法（类的的所有方法都是在类的原型上）
  getName() {
    console.log('父类原型方法')
  }
}

class MyPeople extends People {
  constructor() {
    super();
  }
  // 子类原型普通方法（类的的所有方法都是在类的原型上）
  getName() {
    // super在子类作为对象使用，并且在普通方法中，那么super指向父类的原型对象
    // 所以定义在父类实例上的属性或者方法是不能通过super调用的
    console.log(super.name)
  }
}
const people = new MyPeople();
people.getName(); // undefined
```

调用`super.name`返回`undefined`。



如果是定义在父类的原型上就可以取到了。

```javascript
class People {
  constructor(name) {
    // 父类实例属性
    this.name = '父类实例属性';
  }
  // 父类原型方法（类的的所有方法都是在类的原型上）
  getName() {
    console.log('父类原型方法')
  }
}
// 父类原型对象属性
People.prototype.name = '父类原型对象属性'

class MyPeople extends People {
  constructor() {
    super();
  }
  // 子类原型普通方法（类的的所有方法都是在类的原型上）
  getName() {
    // super在子类作为对象使用，并且在普通方法中，那么super指向父类的原型对象
    // 如果name定义在父类原型对象上，那么久可以取到
    console.log(super.name)
  }
}
const people = new MyPeople();
people.getName(); // 父类原型对象属性
```

## super作为对象在普通方法中调用父类原型的方法，该方法内部的this指向子类的实例(this) 

这句话看着好大，但是我们可以分析简单理解下。父类的方法都是定义在原型对象上面，而super作为对象在普通方法中使用时指向父类的原型，所以不用说的那么复杂，可以说成**super作为对象在普通函数中调用方法，改方法内部的this指向子类的实例(即this)**

```javascript
class People {
  constructor(name) {
    // 父类实例属性
    this.name = '父类实例属性';
  }
  // 父类原型方法（类的的所有方法都是在类的原型上）
  getName() {
    console.log(this.name)
  }
}
// 父类原型对象属性
People.prototype.name = '父类原型对象属性'
class MyPeople extends People {
  constructor() {
    super();
    this.name = '子类实例属性'
  }
  // 子类原型普通方法（类的的所有方法都是在类的原型上）
  getName() {
    // super在子类作为对象使用，并且在普通方法中，调用方法，那么改方法中的this指向子类的实例（this）
    super.getName();
  }
}
const people = new MyPeople();
people.getName(); // 子类实例属性
```

上面代码中，`  super.getName();`虽然调用的是`People.prototype.getName()`。 但是`People.prototype.getName()`中的`this` 指向的是子类`MyPeople`的实例。 导致输出的是"子类实例属性" 而不是“父类实例属性”

## super作为对象在静态方法中是指向父类。

```javascript

class People {
  constructor() { 
    this.age = '父类实例属性';
  }
  // 父类静态方法（只能 People.getName()调用）
  static getName() {
    console.log('父类静态方法');
  }
}
People.prototype.age = '父类原型属性'
People.age = '父类属性'

class MyPeople extends People {
  constructor() {
    super();
  }
  // 子类静态方法（只能MyPeople.getName()调用）
  static getName() {
    // 作为对象在子类静态方法中使用，super指向父类(People)，而不是父类的原型(Pepple.prototype)
    super.getName();
    console.log(super.age);
    
  }
}
MyPeople.getName(); 
// 父类静态方法
// 父类属性
```

super作为对象在子类的静态方法使用，super指向父类(People)，而不是父类的原型(People.prototype)。

## super作为对象在静态方法中调用父类的方法时，方法内部的`this`指向当前的子类，而不是子类的实例。

```javascript
class People {
  constructor() { 
    this.age = '父类实例属性';
  }
  // 父类静态方法（只能 People.getName()调用）
  static getName() {
    console.log(this.age);
  }
}
class MyPeople extends People {
  constructor() {
    super();
    this.age = '子类实例属性';
  }
  // 子类静态方法（只能MyPeople.getName()调用）
  static getName() {
    // 作为对象在子类静态方法中使用，super指向父类(People)，而不是父类的原型(Pepple.prototype)
    // 调用父类方法，方法中的this指向子类(MyPeople)，而不是子类的原型(MyPeople.prototype)
    super.getName();
    
  }
}
MyPeople.prototype.age = '子类原型属性';
MyPeople.age = '子类属性';
MyPeople.getName();  // 子类属性
```

super作为对象在子类的静态方法调用（` super.getName();`）这里的` super.getName() === People.getName() `。虽然调用的是`People.getName()`但是方法中的this指向的是子类(`MyPeople`).所以`this.age === MyPeople.age `  结果为：“子类属性”

## 使用super设置属性的话，那么super就是当前的this。

```javascript
class People {
  constructor() {
    // 父类实例的属性
    this.age = 38;
  }
}
// 父类的属性
People.sex = 30;
class MyPeople extends People {
  constructor() {
    super();
    this.age = 10;
    
    // 使用super设置属性，就是当前的this，所有给当前的实例的属性age设置值为20
    super.age = 20;
    // 普通方法中，super作为对象的话，指向父类的原型；
    console.log(super.age); 
    console.log(this.age); 
  }
}
const people = new MyPeople();

// undefined ---> 父类的原型没有age属性
// 20 --->使用super设置属性，就是当前的this，所有给当前的实例的属性age设置值为20

```






学习都是来自[[ECMAScript 6 入门--Class](https://es6.ruanyifeng.com/#docs/class)。