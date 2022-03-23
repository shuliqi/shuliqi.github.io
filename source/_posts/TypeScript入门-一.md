---
title: TypeScript 基础学习
date: 2021-11-24 13:36:19
tags: JavaScript
categories: JavaScript
---

最近做的项目用到了`TypeScript` ，之前也看过一些，但是没有看完。所以既然项目用到了，觉得还是有必要过一遍，趁着最近正好有点时间可以再看看，把之前没学习完也正好补上。本文就当是一个学习的记录，方便供后期翻阅。读完这篇文章将了解到：

- `TypeScript` 是什么？
- `TypeScript` 的优缺点
- `TypeScript` 有哪些基础类型？
- `TypeScript` 如何定义函数，类，接口?
- `TypeScript` 的泛型

> TypeScript [官方文档](http://www.typescriptlang.org/docs/handbook/basic-types.html), [非官方中文文档](https://zhongsp.gitbooks.io/typescript-handbook/content/doc/handbook)

<!--more-->

# TypeScript 是什么

TypeScript官网是这么解释的：

>TypeScript is a typed superset of JavaScript that compiles to plain JavaScript. Any browser. Any host. Any OS. Open source.

翻译过来就是：

> TypeScript 是 JavaScript 的类型的一个超集，它可以编译成纯`JavaScript`。编译出来的`JavaScrip`t可以运行在任何的浏览器上。`TypeScript`编译的工具可以运行在任何服务器和任何系统上。`TypeScript`是开源 的。

而我自己的理解是：

`TypeScript`是 `JavaScript`的一个超集。主要提供了**类型系统**和对**es6**的支持。

# TypeScript的优点

[TypeScript 官网](http://www.typescriptlang.org/)列举了TypeScript的很多优点。 自己总结的话， 我觉得有以下优点：

**增加代码的可读性和可维护性**

* 在编译的过程中发现大部分的bug的错误。
* 增强了编辑器和` IDE` 的功能，包括代码补全、接口提示、跳转到定义、重构等。
* 类型系统非常好。大部分看类型的定义就知道函数该如何使用了。

**非常包容**

* `TypeScript`是`JavaScript`的一个超集。所以后缀`.js`可以直接改名为`.ts`。
* `TypeScript`编译报错， 也可以生成JavaScript文件。

# TypeScript的缺点

* 有一定的学习成本，需要理解接口（`Interfaces`）、泛型（`Generics`）、类（Classes）、枚举类型（`Enums`）等前端工程师可能不是很熟悉的概念。

* 短期可能会增加一些开发成本，毕竟要多写一些类型的定义，不过对于一个需要长期维护的项目，`TypeScript `能够减少其维护成本。

* 集成到构建流程需要一些工作量。

* 可能和一些库结合的不是很完美。

# TypeScript的安装

`TypeScript` 的命令行工具安装方法如下：

```javascript
npm install -g typescript
```

# TypeScript 的例子

我们创建一个文件：`hello.ts`

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

注：`TypeScript`中使用 **：**指定变量的类型。 **：**前后有没有空格都无所谓。

上面的例子中，我们的变量 `person `的类型是 string。 但是在编译之后的js 中， 并没有检查变量类型的代码插入进来。



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

# TypeScript 类型



>TypeScript支持与Javascript几乎相同的数据类型。
>
>Javascript分为原始数据类型（布尔值，数值，字符串，null, undefined）和对象类型。

## 布尔值（boolean）

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

编译不会报错



其次直接调用 `Boolean`也是返回布尔值：

```javascript
let isOk:boolean = Boolean(1);
```

## 数值（number）

在TypeScript中使用`number`定义数值类型。

`TypeScript`和`Javascript`一样，所有的数字都是浮点数。类型都`number`。`TypeScript`除了支持十进制和十六进制字面量外，还支持[ES6的八进制和二进制的字面量](http://es6.ruanyifeng.com/#docs/number)

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

## 字符串（string）

用`string`表示文本数据类型。 和`JavaScript`一样，可以使用双引号（`"`），单引号（`’`）或 `模版字符串` 表示字符串：

```javascript
// 双引号
let firstName:string = "shu"

// 单引号
let lastName:string = "liqi"

// 模版字符串
let name:string = `my name is ${firstName}${lastName}`

console.log(name) // my name is shu liqi
```

## 空值	(void)

`JavaScript`中时没有空值的概念。但在`TypeScript` 中可以使用`void` 表示没有任何返回值的函数。

注意：定义一个空值的变量是没有用的。因为只能赋值 null 或者 undefined。

```javascript
function getName():void {
	console.log('my name is shuliqi')
}
```

```javascript
let name:void = null
```

## null 和 undefined

可以使用 `Null` 和 `Undefined` 来定义这两个原始类型。

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

## 任意值（any）

任意值（any）用来表示允许赋值为任何类型。

### 什么是任意值类型

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

### 任意值的属性和方法

访问任意值的属性和方法都是允许的：

```javascript
let anyName: any = "shuliqi";
console.log(anyName.firstName);
anyName.setName('shulina');
```

可以认为，声明一个变量为任意值之后，对它的任何操作，返回的内容的类型都是任意值。

### 未声明的类型的变量

如果在声明变量的时候，没有制指定类型。那么会被识别为任意值类型。

```javascript
let myName;
myName = 'shuliqi';
myName = 12;
myName = Boolean(1);
myName = null; 
myName = undefined;
```

## 枚举（Enum） 

在我们的实际应用中， 有的变量只有几种可能取值， 如人的性别就只有两种可能取值，星期就只有七种可能取值。所谓的枚举就是将变量的值一一列举出来，变量只限于列举出来的范围内取值。

```js
enum weekday {	sun,mon,tue,wed,thu,fri,sat}
```

如上， 我们定义了一个枚举类型 `weekday`。

```js
let day: weekday = weekday.mon;
```

如上， 定义了一个枚举类型变量`day`。`sun`, `mon`... 我们称为枚举元素或者枚举常量。

关于枚举常量，有以下几点说明：

- 枚举常量不是变量，而是常量，所以是不能对枚举元素进行赋值操作的。
- 枚举常量作为常量，它们是有值的，值为 1,2,3,....

所以说，`sun` 的值为 0，`mon` 的值为 1，…`sat `的值为 6。

## 数组类型（array）

```tsx
const arr1: string[] = ['1', '2', '3']; 
// 或
const arr2: Array<string> = ['1', '2', '3'];

const arr3: number[] = [1,2,3];
const arr4: any[] = ['shu', 3, {name: 'shuliqi'}]
```

## 元组类型（tuple）

在`TypeScript`中，元组类型表示一个已知数量和类型的数组， 其实可以理解为是一种也特殊的数组。

```javascript
const tupleArr:[number, string, boolean] = [1 ,'2', true]
```

## unknown 类型

`unknown` 指的是不可预先定义的类型，在很多的场景下可以替代`any`的功能并且同时保持静态检查的能力。

```js
const myName: unknown = 'shuliqi, shulina';
myName.join(','); //  Error: 静态检查不通过报错

const myAge: any = '12, 24';
myAge.join(''); // Pass: any类型相当于放弃了静态检查
```

## never 类型

`never`类型表示那些永远不存在的值的类型。 如`nerve`类型是那些总是会抛出异常或根本就不会有返回值的函数表达式或者箭头函数表达式的返回值。

```javascript

function throwFun(): never {
  // return true; // error
  throw new Error("我是一个抛出异常的函数");
}
function throwFun(): never {
  throw new Error("我是一个抛出异常的函数"); // PASS
}

const throwFuncton = ():never => {
  console.log("123"); // error
}
const throwFuncton = ():never => {
  throw new Error("我是一个抛出异常的箭头函数函数"); // pass
}
```

> `never` 与 `void`的区别：`void`是可以被赋值为`null `和` undefined` 类型。` never` 则是一个不包含值得类型。 拥有` void `返回值类型的函数能够正常运行。 拥有 `never `返回值的类型的函数无法正常运行，无法终止， 或会抛出异常

## bigInt

因为 `JavaScript`的所有数字成 64 位浮点数， 这就会给数字的表示带来了两个比较大的限制；第一个限制：数值的精度只能给到53个二进制位（相当与16 个十进制位）， 大于这个范围的数，`JavaScript` 是无法精确表示的，这使得`JavaScript`不适合科学和金融方面的精确计算，第二个限制：大于或者等于 2 的1024次方的数值，`JavaScript`是无法表示，会返回 `Infinity`。

```javascript
// 超过53个二进制的数值，无法保持精度
console.log(Math.pow(2,53) === Math.pow(2,53) + 1); // true

// 超过或等于 2 的 1.24次方的数值，无法表示
console.log(Math.pow(2,1024)); // Infinity
```

于是为了解决该问题，`ES2020`引入了一种新的数据类型 `BigInt `（大整数）来解决这个问题。`BigInt`只用来表示整数，没有位数的限制，任何位数的整数都可以精确表示。为了区别 与`Number` 类型的区别，`BigInt` 类型的数据必须添加后缀n

```javascript
// 普通整数
const number =  123; 
console.log(typeof number); // number

// 大整数
const big = 1231243141n 
const big2 =  BigInt(12312); 
console.log(typeof big, typeof big2); // bigint bigint
```

在 `TypeScript` 中也是有 `BigInt`类型的。我们可以如下表示：

```javascript
const bigNum: bigint = 123123n;
```

## object,Object 和 {} 类型

`object` 类型用来表示非o原始数据类型

```javascript
let objCase: object;
 objCase = 'string' //  error
 objCase = 1123 //  error
 objCase = true //  error
 objCase = null //  error
 objCase = undefined //  error
 objCase = {} //  ok
```

`Object`表示所有拥有 `toString` 和 `hasOwnProperty`方法的类型。所有的原始类型，非原始类型都可以赋给 `Object`（严格模式下 `nul`l 和 `undefined `不行 ）。

```javascript
let objCase: Object;
 objCase = 'string' //  ok
 objCase = 1123 //  ok
 objCase = true //  ok
 // 非严格模式
 objCase = null //  ok
 objCase = undefined //  ok

 objCase = {} //  ok
```

 `{}`对象和 `Object`类型一样， 都是表示原始类型和非原始类型。

```javascript
let objCase: {};
 objCase = 'string' //  ok
 objCase = 1123 //  ok
 objCase = true //  ok
 objCase = null //  ok
 objCase = undefined //  ok
 objCase = {} //  ok
```

## 类型推论

如果在定义一个变量的时候。没有明确的指定类型。那么在TypeScript就会按照**类型推论**规则来推断出一个类型。

**什么是类型推论**

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

原因：`TypeScript`会在没有明确定义变量的类型的时候推测出一个类型。这就是**类型推论**

注意：在定义的时候没有赋值。不管之后有没有赋值，都会被推断为`any`类型。而不做类型检查。sss

## 联合类型

联合类型表示取值可以为多种类型中的一种。

联合类型 用 `|`分隔每个类型：

```javascript
let haha: string | number;
haha = 'shuliqi';
haha = 12;
```

以上例子表示：haha这个变量类型可以是`string` , `number`	 但是不能是其他的类型。

## 类型断言

有时候我们会遇到这种情况，我们会比`TypeScript`更了解某个值的详细信息，通常这种情况会发生在我们清楚地知道一个实体具有比它现在类型更加确切的类型。其实这时候我们需要手动告诉`TypeScript`就按照我们自己断言的类型通过编译（这很关键， 有时间会帮助我们解决很多的编译报错）

类型断言都两种形式：

```javascript
 const someValue: any  = "我是字符串";
 // as 语法
 //  当前我们更加确切的知道 someValue 的类型是 string
 const len: number = (someValue as string).length;

 const someNum: any = [1,2,3];
 // 尖括号 语法
 //  当前我们更加确切的知道 someNum 的类型是 数组
 const myLen: number = (<number[]>someNum).length;
```

> 注意： 以上这两种做法没有任何的区别，但是尖括号会与 `React`语法的`JSX`语法产生语法冲突，所以更加推荐使用 `as`语法。

## 字面量类型

在`TypeScript`中，字面量不仅可以表示类型， 也可以表示类型， 即所谓的字面量类型，目前 `TypeScript` 包含了三种字面量类型：字符串字面量类型，布尔值字面量类型，数字字面量类型， 对应的字符串字面量，布尔值字面量，数字字面量分别拥有与其值一样的字面量类型。

```javascript
//  字符串字面量类型
const str: 'string' = 'string';

// 数字字面量类型
const num: 12 = 12;

//  布尔值字面量类型
const isOk: true = true
```

## 类型别名

类型别名就是给类型起个新名字, `TypeScript`中使用`type`来给类型起新名字

```javascript
//  给类型起新的名字
type  myType = string |  number;
const name:myType = "shuliqi"
```

## 交叉类型

交叉类型就是将多个类型合并成一个类型， 通过 & 运算符将现有的多种类型叠加到一起成为一种新的类型，该新类型包含了所有类型的特性。

```javascript
type  myType1 = {
  name: string
}
type  myType2 = {
  age: number
}

const user: myType1 & myType2 = {
  name: "shuliqi",
  age: 12
}
```

## 类型保护

`TypeScript` 熟知 `JavaScript` 中 `instanceof` 和 `typeof`运算符的用法。如果在一个条件块中上使用这些，`TypeScript`将会推导出在条件块中的变量类型。`TypeScript` 甚至能够理解 `else`。当你使用 `if` 来缩小类型时，TypeScript 知道在其他块中的类型并不是 `if` 中的类型。

**typeof**

```javascript
function doSome(value: string |  number[]) {
  if (typeof value === 'string') {
    // 在这个语句块中， ts 知道 value 的类型必须是 string
    console.log(value.join(',')); // err: 类型“string”上不存在属性“join”。
    console.log(value.split(",")); //ok
  } else {
    // 在这个语句块中， ts 知道 value 的类型必须是 number[]
    console.log(value.join(',')); //  ok
    console.log(value.split(",")); // err: 类型“number[]”上不存在属性“split”。
  }
}
```

**instanceof**

```javascript
class Cat {
  catName = "haha";
  catAge = 12;
}
class Dog {
  dogName = "nani";
  dogAge = 122;
}
function doSomthing(arg: Cat | Dog) {
  if (arg instanceof Cat) {
    // 在这个语句块中，ts 知道 arg 的类型必须是 Cat
    console.log(arg.catName); // ok
    console.log(arg.dogName); // error: 类型“Cat”上不存在属性“dogName”
  } else {
     // 在这个语句块中，ts 知道 arg 的类型必须是 Dog
     console.log(arg.catName); // error: 类型“Dog”上不存在属性“catName”
     console.log(arg.dogName); // ok
  }
}
```

**in**

`in`操作符可以安全的检查一个对象上是否存在一个属性，也通常被作为类型保护来使用：

```javascript
  interface A {
    x: string
  }

  interface B {
    y: number[]
  }

  function toDo(arg: A | B) {
    if ('x' in arg) {
      // arg: A
    } else {
      // arg: B
    }
  }
```

# TypeScript 函数

## 函数的定义

可以指定参数的类型和返回值的类型

```javascript
// viod 表示该函数没有返回值
function getUser(name: string, age: number): void {
  console.log("name:", name, "age:",age);
}

//  函数有返回值， 返回值是一个对象
function getUser(name: string, age: number): {} {
  return {
    name,
    age,
    sex: '女'
  }
}
```

## 函数表达式

```javascript
//  函数无返回值
const getUser: (name: string, age: number) => void = (name, age) => {
  console.log("name:", name, "age:",age);
}
// 或者
type funType =  (name: string, age: number) => void;
const getUser: funType = (name, age) => {
  console.log("name:", name, "age:",age);
}


// 函数有返回值
type funType =  (name: string, age: number) => Object;
const getUser: funType = (name, age) => {
  return {
    name,
    age,
    sex: '女'
  }
}
```

## 可选参数

使用`?`来表示参数是可选的，需要注意的是： 可选参数必须在必须参数之后

```javascript
// age 参数是可选的
type funType =  (name: string, age?: number) => Object;
const getUser: funType = (name, age) => {
  return {
    name,
    age: age || 12,
    sex: '女'
  }
}
```

## 默认参数

```javascript
// age 默认的值为 12
function getUser(name: string, age: number = 12): void {
  console.log("name:", name, "age:", age); // name: shuliqi age: 12
}
getUser("shuliqi")
```

## 剩余参数

```javascript
function getUser(name: string, ...arg: number[]): void {
}
getUser("shuliqi", 1,2,3)
```

## 函数重载

函数重载/方法重载就是使用相同名称和不同参数或类型创建多个方法的一种功能， 在`TypeScript`中表现为给同一个函数提供多个函数类型定义。

函数重载真正执行的是同名函数最后定义的函数体，在最后一个函数体定义之前全部数据函数类型定义，不能写具体的函数实现方法，只能定义类型

```javascript
// 函数重载：给同一个函数提供多个函数类型定义
function toDo(value: string ): void ;
function toDo(value: number ): void ;
function toDo(value: any ): void {
  // ...to
}

//  直接写成这样不香吗？
function toDo(value: string | number | any ): void {
  // ...to
}
```

# TypeScript 类

## 类的类型

在 `TypeScript` 中，我们也是可以通过 `Class` 关键字来定义一个类。 类的类型和接口类型类似。

```javascript
class People {
  name: string
  constructor(_name: string) {
    this.name = _name
  }
  getName() {
    return this.name
  }
}
const people = new People("shuliqi");
console.log(people.getName()); // shuliqi
```

## 存取器

在`TypeScript`中， 我们可以通过存取器来来改变一个类中属性的读取和赋值的行为：

```javascript
class People {
  myName: string
  constructor(_name: string) {
    this.myName = _name
  }
  get name() {
    return this.myName
  }
  set name(value) {
    this.myName = value;
  }
}
const people = new People("shuliqi");
people.name = "世界那么大";
console.log(people.name); //  世界那么大
```

## readonly 

只读属性关键字，只允许出现在属性声明或者索引签名或构造函数中。

```javascript
class People {
  readonly myName: string
  constructor(_name: string) {
    this.myName = _name
  }
}
const people = new People("shuliqi");
people.myName = "世界那么大"; // error:  无法分配到 "myName" ，因为它是只读属性。
```

## 类修饰符

在`TypeScript` 中可以使用三种修饰符：`public`,`private`,`protected`:

- `public` 修饰的属性或方法是公有的，可以在任何地方被访问到，默认的所有属性和方法都是`public`的
- `private`修饰的属性或方法是私有的，不可以在声明它的类的外部访问
- `protected`修饰的属性或方法都是搜保护的，它和`private`类似，区别是它在子类中也是允许被访问的

```javascript
  class People {
    public myName: string
    private age: string
    protected sex: string
    constructor(_name: string) {
      this.myName = _name
    }
    getAge() {
      console.log(this.age)
    }
  }
  class MyPeople  extends People {
    constructor(name) {
      super(name);
      console.log(this.sex); // 可以被访问
      console.log(this.age); // err：属性“age”为私有属性，只能在类“People”中访问
    }
  }
  const people = new MyPeople("shuliqi");
  console.log(people.age); // err: 属性“age”为私有属性，只能在类“People”中访问
```

## 抽象类

使用`abstract`来定义一个抽象类；所谓的抽象类就是指不能被实例化的类：

```javascript
// 定义一个抽象类
abstract class People {
  public myName: string
  private age: string
  protected sex: string
  constructor(_name: string) {
    this.myName = _name
  }
  getAge() {
    console.log(this.age)
  }
}
// 抽象类是不允许被实例化的
const people = new People("shuliqi"); // ERR: 无法创建抽象类的实例。
```



# TypeScript 接口

接口既可以在面向对象编程中表示为行为的抽象，也可以用来描述对象的形状。

## 接口定义

在`TypeScript`中，使用`interface`关键字来定义接口，在接口中可以使用分号或者逗号分割每一项，也可以什么也不加。

```javascript
// 可以什么分割号都不加
interface Props {
  name: string
  age: 12
}
// 可以使用 逗号分割
interface Props {
  name: string,
  age: 12,
}
// 可以使用分号分割
interface Props {
  name: string;
  age: 12;
}
```

## 对象形状

```javascript
// 描述对象的形状
interface Person {
  name: string;
  age: number;
}
let shu: Person = {
  name: 'shuliqi',
  age: 12,
}
```

例子中我们定义了一个接口 `Person`， 然后我们定义了一个变量 `shu`，它的类型是 `Person`。这就约束了`shu`的形状必须和`Person`一致。

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

## 可选属性

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

## 任意属性

如果我们在定义接口的时候无法预先知道哪些属性的时候，可以使用`[propName: string]:any`。`propName` 属性名是任意的。

```javascript
interface Person {
  name: string;
  [propName: string]: any;
}
let shu: Person = {
  name: 'shuliqi',
  age: 12
}
```

注意：**如果我们的定义了任意属性，那么确定属性和可选属性都必须是它值类型的子集：**

```javascript
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

## 只读属性

一些对象的属性只能在对象刚刚创建的时候能修改其值。 可以在属性，可以在属性前面加`readonly` 来指定可读属性：

```javascript
interface User {
  // name 属性的值是只读的
  readonly name: string
  age: number
}
let people: User = { name: "shuliqi", age: 12 };
// 尝试修改 name 属性的值， 会 err
people.name = "haha"; // err

```

## 实现(implements)接口

实现是面向对象中的一个重要概念，一般来讲，一个类只能继承自另外一个类， 但是有时候不同类之间可以有一些共有的特性，这时候就可以把特性提取成接口`interface`。

举个例子：门是一个类，防盗门是门的子类。 如果防盗门有一个报警器的功能，我们就可以简单的给防盗门添加一个报警方法。这时候如果有另外一个类 车，也有该报警器的功能，我们就可以考虑把报警器提取出来，作为一个接口，让防盗门和车都去实现它：

```javascript
// 报警器
interface Alarm {
  // 报警方法
  alert(): void;
}
// 类 门
class Door {}

// 子类：SecurityDoor继承Door; 并且要实现（implements）报警器接口(Alarm)
class SecurityDoor extends Door implements Alarm {
  alert() {
    console.log("我是防盗门的报警方法")
  }
}

// 类：Car 实现 Alarm 接口
class Car implements Alarm {
  alert() {
    console.log("我是车的报警方法")
  }
}
```

一个也可以实现多个接口：

```javascript
// Alarm接口
interface Alarm {
  // 报警方法
  alert(): void;
}
// Light 接口
interface Light {
  lightOn(): void;
  lightOff(): void;
}
// 类 门
class Door {}

// 子类：SecurityDoor继承Door; 并且要实现（implements）报警器接口(Alarm)
class SecurityDoor extends Door implements Alarm, Light{
  alert() {
    console.log("我是防盗门的报警方法")
  }
  lightOn() {
    console.log("防盗门灯打开")
  }
  lightOff() {
    console.log("防盗门灯关闭")
  }
}
```

`implements`的意义是什么？



## 接口(extends)继承

除了类可以继承，接口同样也是可以继承。同样的使用 `extends`关键字。

```javascript
interface User {
  name: string
  age: number
}
// 接口 DetailUser 继承了 User
interface  DetailUser extends User {
  sex: string
  getName: () => string
  other: Object
}
class Person implements DetailUser {
  name = "shuliqi"
  age =  12
  sex = '女'
  getName() {
    return this.name
  }
  other = {}
}
```

## 函数类型接口

在`javaScript`中，有两种常见定义函数的方法--- 函数声明 和 函数表达式。

```javascript
// 函数声明
function getDetail(name, age) {
  console.log("name:", name, "age:", age)
} 

// 函数表达式
const getDetail = (name, age) => {
  console.log("name:", name, "age:", age)
}
```

如果用接口来定义函数的形状：

**函数声明:**

函数声明的类型定义比较简单

```javascript
// 函数声明 类型定义
function getDetail(name: string, age: number): void {
  console.log("name:", name, "age:", age)
} 
```

**函数表达式:**

如果我们现在要对一个函数表达式的ts 约定， 可能会写成这样：

```javascript
// 函数表达式 类型定义
const getDetail = (name: string, age: number) : void  => {
  console.log("name:", name, "age:", age)
}
```

当然这种方式是可以通过编译的。但是事实上，上面的代码只是对等号右边的匿名函数进行了类型定义。而等号右边的`getDetail`是通过赋值操作进行类型推论而推断出来的。 如果需要我们手动给`getDetail`添加类型的话，则应该是这样：

```javascript
// 给等号右边的 getDetail 进行类型定义
const getDetail:(name: string, age: number)=> void= (name: string, age: number) : void  => {
  console.log("name:", name, "age:", age)
}
```

> 注意： 不要混淆 `TypeScript`中的 `=>`  和 `ES6`中的`=>`
>
> 在`TypeScript`的类型定义中，`=>`是用来表示函数的定义，左边是输入的类型。需要用括号括起来，右边是输出的类型

 上面函数表达式的类型定义我们可以使用接口的方式来定义一个函数需要符合的形状：

```javascript
  interface Detail {
    (name: string, age: number): void;
  }
  const getDetail:Detail = (name: string, age: number) : void  => {
    console.log("name:", name, "age:", age)
  }
```

采用接口这种方式，对函数表达式等号左边的进行类型限制，就可以保证以后对该函数赋值时保证参数个数，参数类型，返回值类型不变。

## 构造函数的类型接口

当我们需要把一个类作为参数的时候， 需要对传入的构造函数类型进行约束就需要使用 `new`  关键字来代表类的构造函数类型，用以与普通函数进行区别。

```javascript
class People {
  constructor (name: string) { }
}
interface NewPeople {
  // 使用 new  说明是构造函数
  new (name: string): People
}
function createPeople( createClass:NewPeople ) {
    return new createClass("shuliqi")
}
```

# TypeScript 泛型

泛型（Generics）是指在定义函数，接口或者类的时候，不预先指定具体的类型，而是在使用的时候再指定类型的一种特性。

## 简单例子：

为了更加了解泛型的作用， 我们先来看一个简单的例子：实现可以创建指定的长度的数组，每一项都是填充一个默认值。

```javascript
function createArray(length: number, value: any): Array<any> {
    let result = [];
    for (let i = 0; i < length; i++) {
        result[i] = value;
    }
    return result;
}

createArray(3, 'x'); // ['x', 'x', 'x']
```

上面的代码编译是不会出错的， 但是有一个缺陷，它没有准确的定义返回值的类型： 无论我们传入什么类型的`value`。返回的数组用以是`any`类型。如果我们想要的效果是： 我们预先不知道会传入什么类型的值，但是我们希望不管我们传入什么类型，我们返回的数组里面的类型应该和参数一致。那么这时候就需要用到泛型了。

```javascript
function createArray<T>(length: number, value: T): Array<T> {
  let result = [];
  for (let i = 0; i < length; i++) {
      result[i] = value;
  }
  return result;
}
createArray<string>(3, 'x'); // ['x', 'x', 'x']
```

我们在函数名后端添加`<T>`,其中`T`用来指代任意输入的类型，在后面输入 `value: T` 和`Array<T>`即可。

## 多个类型参数

定义泛型的时候，可以一次性定义多个类型参数：

```javascript
function swap<T,U>(value: [T,U]): [U,T] {
  return [value[1], value[0]]
}
swap([1,'str'])
```

## 泛型约束

在函数内部使用泛型变量的时候，由于事先不知道它是哪种类型，所以不能随意操作它的属性或方法：

```javascript
function logLen<T>(arr: T): void {
  console.log(arr.length); // err: 类型“T”上不存在属性“length”
}
```

例子中 泛型 `T` 不一定包含属性`length`， 所以编译的时候出错了。 

这时候我们可以对泛型进行约束，只允许这个函数传入那些包含`length` 属性的变量。这就是泛型约束：

```javascript
interface Length{
  length: number
}
function logLen<T extends Length>(arr: T): void {
  console.log(arr.length);  // OK
}
```

例子使用`extends`约束了泛型`T`必须符合接口`Length`。 

## 泛型接口

上面有说道可以使用接口的方式来定义一个函数需要符合的形状：

```javascript
interface Detail {
  (name: string, age: number): void;
}
const getDetail:Detail = (name: string, age: number) : void  => {
   console.log("name:", name, "age:", age)
}
```

当然也是可以使用泛型的接口来定义函数的形状

```javascript
interface Detail {
  <T>(name: string, age: T): void;
}
const getDetail:Detail = <T>(name: string, age: T) : void  => {
   console.log("name:", name, "age:", age)
}
```

我们可以把泛型提到接口名上, 但是这时候需要注意使用泛型接口的时候，需要定义泛型的类型：

```javascript
// 将泛型提到接口名上
interface Detail<T> {
  (name: string, age: T): void;
}
// 使用泛型的时候，需要定义泛型的类型
const getDetail:Detail<any> = <T>(name: string, age: T) : void  => {
   console.log("name:", name, "age:", age)
}
```

## 泛型类

与泛型接口类似，泛型也可以用于类的类型定义中：

```javascript
class People<T> {
  name: string
  age: T
}
const user = new People<number>();
user.name = "shuliqi";
user.age =  12;
```

## 泛型参数的默认类型

我们可以为泛型中的类型参数指定默认类型。当使用泛型时没有在代码中直接指定类型采纳数， 从实际值参数中也无法推测出时，这个默认值类型就会起到作用:

```javascript
class People<T = string> {
  name: string
  age: T
}
```



学习参考文章：

-  https://ts.xcatliu.com/advanced/class-and-interfaces.html
- https://juejin.cn/post/7031787942691471396#heading-35



