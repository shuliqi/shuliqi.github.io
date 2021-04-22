---
title: TypeScript 入门教程
date: 2019-10-16 13:36:19
tags: JavaScript
---

最近一直在看阮一峰老师的 [TypeScript入门教程](https://ts.xcatliu.com/)。教程写的非常好，清晰明了。学习完了自己也试着写一些总结吧。

其次：TypeScript [官方文档](http://www.typescriptlang.org/docs/handbook/basic-types.html), [非官方中文文档](https://zhongsp.gitbooks.io/typescript-handbook/content/doc/handbook)

<!--more-->

## TypeScript 是什么

TypeScript官网是这么解释的：

>TypeScript is a typed superset of JavaScript that compiles to plain JavaScript. Any browser. Any host. Any OS. Open source.

翻译过来就是：

> TypeScript 是 JavaScript 的类型的一个超集，它可以编译成纯JavaScript。编译出来的JavaScript可以运行在任何的浏览器上。TypeScript编译的工具可以运行在任何服务器和任何系统上。TypeScript是开源 的。

而我自己的理解是：

TypeScript是 JavaScript的一个超集。主要提供了**类型系统**和对**es6**的支持。



## TypeScript的优点

[TypeScript 官网](http://www.typescriptlang.org/)列举了TypeScript的很多优点。 自己总结的话， 我觉得有以下优点：

##### TypeScript增加代码的可读性和可维护性

* 在编译的过程中发现大部分的bug的错误。
* 增强了编辑器和 IDE 的功能，包括代码补全、接口提示、跳转到定义、重构等。
* 类型系统非常好。大部分看类型的定义就知道函数该如何使用了。

##### TypeScript 非常包容

* TypeScript是JavaScript的一个超集。所以后缀`.js`可以直接改名为`.ts`。
* TypeScript编译报错， 也可以生成JavaScript文件。



## TypeScript的缺点

* 有一定的学习成本，需要理解接口（Interfaces）、泛型（Generics）、类（Classes）、枚举类型（Enums）等前端工程师可能不是很熟悉的概念。

* 短期可能会增加一些开发成本，毕竟要多写一些类型的定义，不过对于一个需要长期维护的项目，TypeScript 能够减少其维护成本。

* 集成到构建流程需要一些工作量。

* 可能和一些库结合的不是很完美。



## TypeScript的安装

TypeScript 的命令行工具安装方法如下：

```javascript
npm install -g typescript
```



## TypeScript 的例子

我们创建一个文件：hello.ts

在该文件中写入：

```javascript
function helloWorld(person: string) {
    return 'Hello, ' + person;
}

let user = 'shuliqi';
console.log(helloWorld(user));
```

然后我们进行编辑，可以执行以下命令：

```javascript
tsc hello.ts
```

执行完，会生成一个编译好的文件：hello.js

```javascript
function helloWorld(person) {
    return 'Hello, ' + person;
}
var user = 'shuliqi';
console.log(helloWorld(user));
```

注：TypeScript中使用 **：**指定变量的类型。 **：**前后有没有空格都无所谓。

上面的例子中，我们的变量 person 的类型是 string。 但是在编译之后的js 中， 并没有检查变量类型的代码插入进来。



**TypeScript只会静态检查，如果有错误。在编译的时候会抛出错误**

例如：我们在hello.ts中的user变量赋值一个数组

```javascript
function helloWorld(person: string) {
    return 'Hello, ' + person;
}

let user = [1,2,3];
console.log(helloWorld(user));
```

编译的时候就会报错：

```javascript
hello.ts:6:24 - error TS2345: Argument of type 'number[]' is not assignable to parameter of type 'string'.

6 console.log(helloWorld(user));
                         ~~~~


Found 1 error.
```

**TypeScript在编译的时候检查出错误，抛出错误，但还是会生成js文件**。

如上的例子，还是会生成**.js**文件

```javascript
function helloWorld(person) {
    return 'Hello, ' + person;
}
var user = [1, 2, 3];
console.log(helloWorld(user));
```



## TypeScript基础

#### 一.原始数据类型

>TypeScript支持与Javascript几乎相同的数据类型。
>
>Javascript分为原始数据类型（布尔值，数值，字符串，null, undefined）和对象类型。

##### 布尔值

在typeScript中使用`boolean`定义布尔值类型：

```javascript
let isOk:boolean = false;

// 编译通过
// 后面约定，未强调编译错误的代码片段，默认为编译通过
```



注意：只有构造函数 `Boolean`创建的对象不是布尔值：

```javascript
let isOk:boolean = new Boolean(1);
```

编译的时候报错：

```javascript
hello.ts:10:5 - error TS2322: Type 'Boolean' is not assignable to type 'boolean'.
  'boolean' is a primitive, but 'Boolean' is a wrapper object. Prefer using 'boolean' when possible.

10 let isOk : boolean = new Boolean(1);
```

原因：new Boolean()返回的是一个 `Boolean` 对象

```javascript
let isOk:Boolean = new Boolean(1);
```

编译不回报错



其次直接调用 `Boolean`也是返回布尔值：

```javascript
let isOk:boolean = Boolean(1);
```



##### 数值

在TypeScript中使用`number`定义数值类型。

TypeScript和Javascript一样，所有的数字都是浮点数。类型都`number`。TypeScript除了支持十进制和十六进制字面量外，还支持[ES6的八进制和二进制的字面量](http://es6.ruanyifeng.com/#docs/number)

```javascript
let decLiteral: number = 6;
let hexLiteral: number = 0xf00d;
// ES6 中的二进制表示法
let binaryLiteral: number = 0b1010;
// ES6 中的八进制表示法
let octalLiteral: number = 0o744;
let notANumber: number = NaN;
let infinityNumber: number = Infinity;
```

编辑的结果：

```javascript
var decLiteral = 6;
var hexLiteral = 0xf00d;
// ES6 中的二进制表示法
var binaryLiteral = 10;
// ES6 中的八进制表示法
var octalLiteral = 484;
var notANumber = NaN;
var infinityNumber = Infinity;
```

其中 `0b1010` 和 `0o744` 是 [ES6 中的二进制和八进制表示法](http://es6.ruanyifeng.com/#docs/number#二进制和八进制表示法)，它们会被编译为十进制数字。



##### 字符串

用`string`表示文本数据类型。 和JavaScript一样，可以使用双引号（`"`），单引号（`’`）或 `模版字符串` 表示字符串：

```javascript
// 双引号
let firstName:string = "shu"

// 单引号
let lastName:string = "liqi"

// 模版字符串
let name:string = `my name is ${firstName}${lastName}`

console.log(name) // my name is shu liqi
```



##### 空值

JavaScript中时没有空值的概念。但在TypeScript 中可以使用`void` 表示没有任何返回值的函数。

注意：定义一个空值的变量是没有用的。因为只能赋值 null 或者 undefined。

```javascript
function getName():void {
	console.log('my name is shuliqi')
}
```

```javascript
let name:void = null
```



 ##### Null 和 Undefined

可以使用 Null 和 Undefined 来定义这两个原始类型。

```javascript
let myName:null = null;
let age:undefined = undefined;
```



注意：null 和 undefined 是所有类型的子集。也就是说 null ，undefined 可以赋值给其他类型：

```javascript
let myName: string = undefined;
let age: number = null;

// 或者

let myName: undefined = undefined;
let age: null;
let myLastName: number = myName;
let myAge = age;

// 都不会报错
```

但是 `void` 类型的变量不能赋值给 其他 类型的变量：

```javascript
let myAge:void = null;
let myGender:void = undefined;

let age: number = myAge;
let gender: string = myGender;
```

编译报错：

```javascript
hello.ts:52:5 - error TS2322: Type 'void' is not assignable to type 'number'.
52 let age: number = myAge;
hello.ts:53:5 - error TS2322: Type 'void' is not assignable to type 'string'.
53 let gender: string = myGender;
```



#### 二. 任意值

任意值（Any）用来表示允许赋值为任何类型。

##### 什么是任意值类型

在TypeScript中普通的类型，如果在赋值的过程中改变类型是不允许的

```javascript
let age: string = '12';
age = 5;
```

编译时报错：

```javascript
hello.ts:62:1 - error TS2322: Type '5' is not assignable to type 'string'.

62 age = 5;
```

但是类型是任意值类型的话。就允许在赋值的过程中改变类型：

```
let age: any = '12';
age = 5
```

##### 任意值的属性和方法

访问任意值的属性和方法都是允许的：

```javascript
let anyName: any = "shuliqi";
console.log(anyName.firstName);
anyName.setName('shulina');
```

可以认为，声明一个变量为任意值之后，对它的任何操作，返回的内容的类型都是任意值。

##### 未声明的类型的变量

如果在声明变量的时候，没有制指定类型。那么会被识别为任意值类型。

```javascript
let myName;
myName = 'shuliqi';
myName = 12;
myName = Boolean(1);
myName = null; 
myName = undefined;
```



#### 三. 类型推论

如果在定义一个变量的时候。没有明确的指定类型。那么在TypeScript就会按照**类型推论**规则来推断出一个类型。

##### 什么是类型推论

如以下的代码虽然没有指定类型，但是在编译时会出错：

```javascript
let age = 34;
age = "shuliqi";

// 编译报错：
Type '"shuliqi"' is not assignable to type 'number'.
```

其实上面的代码等价于这段代码：

```javascript
let age: number = 34;
age = "shuliqi";

// 编译报错：
Type '"shuliqi"' is not assignable to type 'number'.
```

原因：TypeScript会在没有明确定义变量的类型的时候推测出一个类型。这就是**类型推论**



注意：在定义的时候没有赋值。不管之后有没有赋值，都会被推断为`any`类型。而不做类型检查。sss



#### 联合类型

联合类型表示取值可以为多种类型中的一种。

联合类型 用 `|`分隔每个类型：

```javascript
let haha: string | number;
haha = 'shuliqi';
haha = 12;
```

以上例子表示：haha这个变量类型可以是`string` , `number`	 但是不能是其他的类型。



#### 对象的类型---接口

在TypeScript中，我们使用接口来定义对象的类型。

例子：

```javascript
interface Person {
  name: string;
  age: number;
}
let shu: Person = {
  name: 'shuliqi',
  age: 12,
}
```

例子中我们定义了一个接口 Person， 然后我们定义了一个变量 shu，它的类型是 Person。这就约束了shu的形状必须和Person一致。

注意：接口的首字母一般大写。

如果我们定义的变量比接口少一些属性是不允许的：

```javascript
interface Person {
  name: string;
  age: number;
}
let shu: Person = {
  name: 'shuliqi',
}
```

编译时会报错的

```javascript
'age' is declared here.
```

如果我们定义的变量比接口多一些属性也是不允许的：

```javascript
interface Person {
  name: string;
  age: number;
}
let shu: Person = {
  name: 'shuliqi',
  age: 12,
  parent: 'shulina'
}
```

编译报错：

```javascript
error TS2322: Type '{ name: string; age: number; parent: string; }' is not assignable to type 'Person'.
  Object literal may only specify known properties, and 'parent' does not exist in type 'Person'.

8  parent: 'shulina'
```

##### 可选属性

有的时候我们不需要完全匹配一个形状。我们可以使用可选属性：在属性后端加 ？ 符号。

```javascript
interface Person {
  name: string;
  age?: number;
}
let shu: Person = {
  name: 'shuliqi',
}
```

可选属性的含义可以不存在。注意： 这时候仍然不可以添加未定义的属性。

##### 任意属性

有时候我们希望一个接口允许有任意的属性，可以使用如下的方式 ：

```javascript
interface Person {
  name: string;
  [propName: string]: any;s
}
let shu: Person = {
  name: 'shuliqi',
  age: 12
}
```

[propName: string ] 表示属性的类型是string

注意：**如果我们的定义了任意属性，那么确定属性和可选属性都必须是它值类型的子集：**

```
interface Person {
  name: string;
  age?: number
  [propName: string]: string
}
let shu: Person = {
  name: 'shuliqi',
  age: 12,
  parent: '123',
}
```

编译额时候会报错：

```javascript
Type '{ name: string; age: number; parent: string; }' is not assignable to type 'Person'.
  Property 'age' is incompatible with index signature.
    Type 'number' is not assignable to type 'string'.

6 let shu: Person = {
      ~~~


Found 2 errors.
```

**原因**：可选属性age的值的类型是number， 任意属性的值的类型 是string。number 不是stringl类型。所以会报错.



未整理完….待继续整理



