---
title: 《javascript设计模式》读书笔记三：函数
date: 2019-06-15 19:53:18
tags: ES6
categories: ES6
---

### 一. 函数的特点

#### 函数是第一类对象

函数就是对象，其表现是：

* 函数可以在运行时动态创建，也可以在程序执行过程中创建

* 函数可以分配给变量，可以将它们的引用复制到其他变量，可以被扩展，此外，除了极少数特殊情况外，函数也可以被删除。

   <!--more-->

* 可以作为参数传递给他们函数，并且还可以被其他函数返回。

* 函数可以有自己的属性和方法。



#### 函数提供了作用域

在JavaScript中没有花括号{}语法以定义局变量，也就是说， 块并不创建作用域。JavaScript仅存在函数作用域。在函数内以var关键词定义的任何变量都是局部变量。对于函数外是不可见的。

### 二. 函数声明和函数表达式

#### 函数表达式

```javascript
// 命名函数表达式
var add = function add(a, b) {
  return a + b;
};
// 函数表达式
var add = function(a, b) {
  return a + b
};
```

命名函数表达式是函数表达式的一种特殊情况。

#### 函数声明

```javascript
function foo() {
  return a + b;
}
```

#### 函数声明与函数表示式的区别

* 函数表达式中需要分号结尾，并且总应该用分号。函数声明不需要分号。

* 使用函数声明时，不仅仅函数定定义被提升，函数体也被提升了。使用函数表达式时， 只有函数定义提升，函数体不提升。

  ```javascript
  function foo() {
    console.log('global foo')
  }
  function bar() {
    console.log('global bar')
  }
  function host() {
    console.log(typeof foo);  // "function"
    console.log(typeof bar); 	// "undefined"
    foo(); // local foo
    bar(); // bar is not a function
    // 函数声明，函数定义和函数体一起被提升到函数host顶部
    function foo() {
      console.log('local foo')
    }
    // 函数表达式，只有函数定义被提升到函数host顶部
    var bar = function() {
        console.log('local bar')
    }
  }
  ```


### 三. 函数有关的有用的模式

#### API模式

* 配置对象：有助于保持受到控制的函数的参数数量

  如果函数的参数列表很长（超过3个），一个更好的办法是仅使用一个参数对象来替代所有的参数，然后实际的参数在这个对象里面配置。

  ```javascript
  var addPerson = function(config) {
    var { age, name } = config;
    ...
  }
  var config = {
  	age: 12,
  	name: "shuliqi",
  }
  addPerson(config)
  ```

  该方法的优点：

  * 不需要记住总众多的参数和参数的顺序。
  * 可以安全的忽略可选参数。
  * 更加易于阅读和维护。
  * 更加易于添加参数和删除参数。

  配置对象的不足：

  * 需要记住参数名称
  * 属性名称无法被压缩

#### 初始化模式

* 即时函数：是一种支持在定义函数后立即执行该函数的语法

  **语法：**

  * 可以使用函数表达式定义一个函数（函数声明则无法达到这个效果）
  * 在末尾添加一组括号，这将导致该函数立即执行
  * 将整个函数包装在括号中

  ```javascript
  (function () {
  	console.log('111')
  }())
  ```

  **好处：**可以包装需要想要执行的任务，且不会在后台留下任何全局变量，定义的所有变量将会是用于自身调用函数的局部变量，并且不会担心全局空间呗临时的变量所污染。

* 即时对象初始化：跟即时函数类似，可以保护全局作用域不收污染。

  这种模式使用带有init()方法的对象。该方法在创建对象之后将会立即执行。init()函数需要负责所有的初始化任务。

  **语法：**

  * 使用对象字面量创建一个普通的对象。
  * 将字面量包括到括号中。
  * 在括号结束之后，可以立即调init()方法。

  ```javascript
  ({
  	width: 600,
  	height: 300,
  	getMax: function() {
  		return this.width + this.height;
  	},
  	init: function() {
  		console.log(this.getMax());
  	}
  }).init();
  ```

  **优点：**

  * 于即时函数的优点相同
  * 如果初始化任务更加复杂，它将会使得整个初始化过程更加有结构化。

* 初始化时分支：也称加载时分支，是一种优化模式。

  帮助分支代码在初始化代码执行过程中仅检测一次。

  ```javascript
  // 之前
  var utils = {
  	addListenner: function(el, type, fn) {
  		if (typeof window.addEventListener === 'function') {
  			el.addEventListener(type, fn, false);
  		} else if (typeof document.attachEvent === 'function') {
  			el.attachEvent('on' + type, fn)
  		} else {
  			el['on' + type] = fn;
  		}
  	},
    removeListenner: function(el, type, fn) {
      // 几乎一样的
    }
  }
  ```

  这种方式效率比较低下，每次调用utils.addListenner 和utils.addListenner时，都会重复的执行相同的检查。

  ```javascript
  var utils = {
   	addListenner: null,
   	removeListenner: null,
  }
  if (typeof window.addEventListener === 'function') {
  	utils.removeListenner =function(el, type, fn) {
  		el.addEventListener(type, fn, false);
  	}
  	utils.removeListenner =function(el, type, fn) {
  		el.removeListenner(type, fn, false);
  	}
  } else if(typeof document.attachEvent === 'function') {
  	utils.addListenner =function(el, type, fn) {
  		el.attachEvent('on' + type, fn)
  	}
  	utils.removeListenner =function(el, type, fn) {
  		el.detachEvent('on' + type, fn)
  	}
  } else {
  	utils.addListenner =function(el, type, fn) {
  		el['on' + type] = fn;
  	}
  	utils.removeListenner =function(el, type, fn) {
  		el['on' + type] = fn;
  	}
  }
  ```

  这种方式就是只会检测一次了。

#### 性能模式

* 备忘录模式：使用函数属性以便使的计算过的值无需在此计算

  以为函数是对象，所以有属性和方法。例如：无论以什么方式创建了对象，都会自动获得一个length属性，包含了该函数期望的参数的数量

  ```javascript
  function fun(a, b) {}
  console.log(fun.length)  // 2
  ```

  所以可以在任何时候将自定义属性添加到函数中，其中的一个作用就是缓存函数结果（返回值）。因此在下一次调用的时候就不用重复做潜在的繁重计算功能。缓存函数结果被称：备忘

  ```javascript
  var myFun = function(a, b) {
    var f = arguments.callee;
    var result;
  	if (!f.cache) {
  		result = "shuliqi";
  		// ...开销很大的操作
  		myFun.cache = result;
  	}
  	return f.cache;
  }
  // 缓存存储
  myFun.cache = {};
  ```

* 自定义模式（惰性函数定义）： 以新的主体重写本身。使的在第二次或者以后调用时仅需要执行更少的工作

  函数可以被动态定义，也可以分配给变量。如果创建了一个新的函数并且把它分配给了另外一个函数的的同一个变量。那么就是以新函数覆盖了就函数。

  ```javascript
  var myFun = function() {
  	console.log('111');
  	myFun = function() {
  		console.log('2222');
  	}
  }
  myFun(); // '111'
  myFun(); // '2222'
  console.log(myFun.name)
  
  ```

  如果函数有一些初始化的准备工作要做。并且只需要执行一次。那么这种模式就很有用。

  **缺点**

  * 如果改函数使用了不同的名称（比如分配给不同的变量或者以对象的方式来使用）那么重定义就不会发生，并且会执行原始函数体。

    ```javascript
    var myFun = function() {
    	console.log('111');
    	myFun = function() {
    		console.log('2222');
    	}
    }
    
    // 赋值给另一个不同名称的变量
    var newMyFun = myFun;
    
    // 作为一个方法被调用
    var obj = {
     	fun: newMyFun,
    }
    
    newMyFun() // '111'
    newMyFun() // '111'
    
    obj.fun() // '111'
    obj.fun() // '111'
    
    myFun() // '2222'
    myFun() // '2222'
    
    ```

    这些调用不断的重写全局myFun()指针。以至于它最终被调用时，它才第一次更新函数主体。

 



