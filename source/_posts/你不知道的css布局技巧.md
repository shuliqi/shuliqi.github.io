---
title: 你不知道的css技巧
date: 2019-08-27 22:37:46
tags: CSS
---

#### **1.用 :\*-Of-Type 选择元素**

> 不兼容IE8

假如我们有这样的htm结构

```html
<p>11111</p>
<p>22222</p>
<p>33333</p>
<p>44444</p>
<p>55555</p>
<p>66666</p>
```

 <!--more-->

给第一个 p 段落文本变红：

```
p:first-of-type {
	
}
```

给最后一个 p 加边框:

```css
p:last-of-type {
  border: 1px solid green;
}
```

让偶数列的 p 段落背景设置黑色：

```css
p:nth-of-type(even) {
  background: #000;
}
```

此外， 还可以设置其他类型的参数

```css
/* only 第三个 */
:nth-of-type(3) {

}

/* 每第三个 */
:nth-of-type(3n) {

}

/* 每第四加三个，即 3, 7, 11, ... */
:nth-of-type(4n+3) {

}
```

完成的代码如下：

```html
<html>
  <style>
    /* 给第一个 p 段落文本变红： */
    p:first-of-type {
      color: red;
    }
    /* 给最后一个 p 加边框 */
    p:last-of-type {
      border: 1px solid green;
    }
    /* 让偶数列的 p 段落背景设置黑色： */
    p:nth-of-type(even) {
      background: #000;
    }
    /* only 第三个 */
    :nth-of-type(3) {

    }

    /* 每第三个 */
    :nth-of-type(3n) {

    }

    /* 每第四加三个，即 3, 7, 11, ... */
    :nth-of-type(4n+3) {

    }
  </style>
  <body>
      <p>11111</p>
      <p>22222</p>
      <p>33333</p>
      <p>44444</p>
      <p>55555</p>
      <p>66666</p>
  </body>
</html>
```



#### **2.用 unset 做 CSS Reset**

> 不兼容IE8

```html
<html>
  <style>
    button {
      color: red;
      border: 1px solid #ccc;
    }
    /* 取消 item 中 button 的 color 设置 */
    .item button {
      color: unset;
    }

  </style>
  <body>
    <button>按钮111</button>
    <div class="item">
        <button>按钮222</button>
    </div>
  </body>
</html>
```



####  3.使用 :empty 区分空元素

> 不兼容IE8



假如我们有以下的HTML结构：

```html
<div class="item">1</div>
<div class="item">2</div>
<div class="item"></div>
```

如果我们希望对空元素和非空元素区别处理。那么有这两种方案。

使用 **:empty** 选择空元素:

```css
.item:empty {
  display: none;
}
```

或者使用**:not(:empty)** 选择非空元素

```css
 :not(:empty) {
    color: red
  }
```

完成的代码如下：

```html
<html>
  <style>
    .item {
      width: 100%;
      height: 50px;
      margin: 10px 0;
      background: #ccc
    }
    /* 使用 :empty 选择空元素: */
    .item:empty {
      display: none;
    }
    /*使用:not(:empty) 选择非空元素 */
    :not(:empty) {
      color: red
    }
  </style>
  <body>
    <div class="item">1</div>
    <div class="item">2</div>
    <div class="item"></div>
  </body>
</html>

```



### 4.每个单词的首字母大写 text-transform

>  CSS2 中的属性，参数有 capitalize | uppercase | lowercase | none

参数介绍：

1. none：默认。定义带有小写字母和大写字母的标准的文本。
2. capitalize：文本中的每个单词以大写字母开头。
3. uppercase：定义仅有大写字母。
4. lowercase：定义无大写字母，仅有小写字母。

从这个属性我们可以知道全部大写（小写）的需求这个属性也能轻易实现。

例子：每个单词的首字母大写

```html
<div class="title">shuliqi is boy</div>
```

```css
.title {
   text-transform:capitalize;
 }
```

例子链接： http://jsrun.pro/IYWKp/edit



