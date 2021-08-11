---
title: 如何理解DSL
date: 2021-08-07 13:18:17
tags: 
---

从一个简单的问题开始：

如何使用程序来描述下面的问题？

在一组候选人中，找到满足以下条件的的数据：

- 所有的候选人性别为女
- 数据按照年龄来排序
- 输出姓名以及年龄的信息

 <!--more-->

问题跑出来，那么让我们使用`JavaScript`来实现这个需求：

> 假设数据集合叫`data`

```javascript
const result = data.filter((people) => people.sex === '女')
                    .sort((cur, pre) => cur.age - pre.age)
                    .map((item) => {
                      return { name: item.name, age: item.age }
                    })
```

上面的代码看起来，在对于`JavaScript`语言十分熟悉的人来看， 可以很快的看明白代码的意思，但是如果是外行，那就可能有点难猜了。

那怎么办呢？有没有什么更好的办法/更简单易懂的方式来描述这个问题呢？

```mysql
select 'name', 'age' from "data" where sex = "女" sortBy "age";
```

上面这个语句是一个`SQL`语句, 相信不管是不是程序员，都能快速的理解这段语句的含义。

显然`SQL`在查询这个领域比`JavaScript`更加简洁，表达力更强；我们管这种针对特殊领域的语言就叫`DSL`(特殊领域语言)。

# 什么是DSL

`DSL`是(Domain Specific Language)特定领域语言， 是为了处理某一特定领域的问题而设计出来的语言，这类语言比较简单，在专属的领域内，有着非常强的表达力。

与之相对应的是的`GPL`(General Purpose Language) 通用编程语言。这类语言是能够表达科倍计算的逻辑，必须是图灵完备，他们的侧重点是灵活，全面。 `Java`,`python`,`JavaScript`,`go`,`C 这类都属于`GPL`。

# 典型的DSL 有哪些

- `HTML`
- `CSS`
- `JSX`
- `PUG`
- `REGEX`
- `SQL`
- `XML`
- `YMAL`

上面的这些语言，几乎都不是图灵完备的计算机语言，他们的语法很简单，主要以声明式的表达方式来进行编写的。

因为是使用声明式的表达方式, 所以在在阅读上会更加流畅。

> 关于什么是图灵机，什么是图灵完备等，可阅读这篇文章  [什么是图灵完备？](https://www.zhihu.com/question/20115374)

# 为什么要用 DSL



`DSL`语法往往简单，并且非常易于阅读，在其擅长的领域内编码是非常高效的。

我们再拿 `HTML` 和 `JavaScript`对象来最一个对比：

```javascript
{ name: "div", class: "container", children: "shuliqi" }
```

使用`JavaScript`来描述一个`HTML`节点，只能把所有的的属性平铺到一个对象中，让人在阅读中无法找到重点，无法有效的做区分。

```html
<div class="container" > shuliqi </div>
```

使用了 `XML`标签，就可以快速的辨别标签的类型，内容，整体语义化很强，缺点就是多余的符号和为了表达嵌套关系的的闭合标签。

```html
div(class='container') hello world
```

使用`pug`独特的语法，在不损失`XML`的表达力的前提下，省略了无用的符号，并通过缩进来表达嵌套逻辑，降低了很多的语法噪音。



可以看出来, `DSL`的引入确实能够有效的提高特定场景的表达效率, 在熟悉了相关语法后, 能够是代码更易理解, 也更加简洁。



# 要不要引入DSL



在日常的开发中， 为了解决某一场景的需求而引入一个新的`DSL`, 在我看来是需要慎重考虑的。因为这是引入了一种新的语言，这可能会让整个项目的维护难度提升。

所以否引入`DSL`需要考虑几个点：

- 产能的提升是否能够抹平新语言引入带来的学习成本
- 是否可以使用第三方工具替代`DSL`

# 内部DSL? 外部DSL?

既然引入`DSL`是有成本的，那么有没有什么成本比较低的引入方法呢？

也许有的，上面我们谈到的`DSL`在广义上作区分的话可以叫做**外部DSL**，既然有**外部DSL**， 那么肯定有**内部DSL**，我们举个例子：

```mysql
select 'name' from 'applications' where id = 234
```

```js
select('name')
.from('applications')
.where(id, '=', 234)
```

上面的`SQL` 和下面的`JavaScript`代码实际上是等价的， 上面的`SQL`代码我们叫它`外部的DSL`， 下面的`JavaScript`我们叫做`内部DSL`----> [knexjs](https://knexjs.org/)



可以再来举个例子:

**外部DSL**:

```css
.container {
  .content {
      color: #fff;
      margin: 10px
  }
}
```

**内部的DSL**

```js
$('.container .content')
.css('color', '#fff')
.css('margin', '10px')

```

显然这个内部的`DSL`看着很眼熟，感觉就是我们普调的`js`方法调用，就是普调的第三方接口调用。

类似这样的内部`DSL`我们见到还很多：

**Jquery：**

`JQuery`可以被称作是对`DOM`的操作的**内部DSL**:

````js
$('.mydiv')
.addClass('flash')
.draggable()
.css('color', 'blue');
````

**Moment：**

`Moment`可以被称作是对日期操作的**内部DSL**

```javascript
moment()
.subtract(10, 'days')
.calendar();
```

**Chai**

`Chai`就可以被称作是对断言的**内部DSL**

```js
tea.should.have.property('flavors').with.lengthOf(3);
```

# 如何区分内部或外部DSL

通常来讲, 如果实现的功能,无法被宿主语言直接支持, 需要自己额外实现代码的编译和解析, 都属于外部DSL

如果实现的功能, 可以呗宿主语言直接支持, 就想上面的例子一样, 都是简单的函数形式, 那么这就属于内部的DSL



那么现在这里有一个问题, 就是我们写`React`常用的`JSX`算是什么? 是内部的`DSL`么? 下面放一点示例代码

```HTML
<Container>
  <Menu list={this.state.list}/>
  <Footer/>
</Container>
```

会被转成：

```react
React.createElement(
  Container,
  null,
  React.createElement(Menu, {
    list: this.state.list
  }),
  React.createElement(Footer, null)
);
```

虽然转义后的代码是合法的`javaScript`代码，但是源代码并不是一个合法的`javaScript`代码，正常的浏览器是无法正常识别这些代码的。

因为这些代码再被浏览器执行前, 会被`Babel`进行转移, 转移成可被浏览器识别的`javaScript`代码，而`Babel`在转义这些`JSX`的时候,
实际上内部已经实现了对这种特殊语法的编译和解析, 所以, 虽然`jsx`通常也写在`.js`文件中, 但它还是一个外部的`DSL`

# 如何看待内部DSL

`DSL`实际上就是为了解决特定问题而出现的.
使用内部`DSL`来对某个领域进行扩展,在我看来是一个很好的解决方案, 因为没有引入新语言带来的新问题,

社区对,内部`DSL`的划分界定实际上很模糊, 希望大家不要过于纠结, 无论是`API` 还是` DSL`, 能够有效的抽象并解决问题的, 都是好的解决之道。



















































