---
title: canvas的学习及其使用
date: 2020-05-07 20:28:48
tags: HTML
categories: HTML
---

# canvas的简介

`canvas`是`HTML5`新增加的标签，它是一个可以使用脚本（`JavaScript`）来绘制图形的`HTML`元素。它可以用来制作图片，动画，甚至可以进行实时视频处理和渲染。

`canvas`是由`HTML`代码高度，宽度属性而定义出来的可绘制的区域，`JavaScript`代码可以可以访问该区域，通过一套完整的绘制函数来动态生成图形。



# canvas 的基本用法

## canvas 元素

```html
<canvas id="canvas" height="300" width="300"></canvas>
```

`canvas` 标签有两个可选的属性`height`，`width`。如果没有给`canvas`设置`width`，`height`属性，则`height`默认150，`width`默认为 300。单位都是`px`。当然也可以使用`CSS`设置宽高，但是如果宽高和初始化的比例不一致，是会出现扭曲的。所以是不建议使用`css`来设置`canvas`的宽高的。



## 替换的内容

由于某些比较老的浏览器或者浏览器不支持`HTML`元素`canvas`，在这些浏览器上应该要显示替代的内容。

支持`canvas`标签的浏览器会只渲染`canvas`标签，而忽略其中的替代内容。不支持`canvas`标签的浏览器则会直接渲染替代的内容。

```html
<canvas  id="canvas">
  <P>你的浏览器不支持canvas标签</P>
</canvas>
```



**注意：**`canvas`标签的结束标签是不可省略的。如果省略的话，文档的其余部分会被认为是替代的内容，不会显示出来。



## 渲染上下文

`canvas`元素创造了一个固定大小的画布，它公开了一个或多个**渲染上下文（The rendering context）**，它可以用来绘制和处理要展示的内容。

`canvas`起初是空白的，为了展示，首先脚本需要找到渲染上下文，然后在它的上面绘制。`canvas`元素有一个叫做`getContext()`的方法，这个方法用来获得渲染上下文和它的绘画功能。`getContext()`只有一个参数：上下文的格式。

**绘制2D**

```javascript
  const canvas = document.getElementById("canvas");
  const context = canvas.getContext("2d");
```

**绘制3D**

`HTML5`有一个`WebGL`规范，允许在`canvas`绘制`3D`图形。

```javascript
  const canvas = document.getElementById("canvas");
  const context = canvas.getContext("webgl");
```

# canvas绘制图形

## 栅格

在开始绘图之前， 我们先了解一下画布栅格（`canvas grid`）以及坐标空间。默认的`canvas`元素会被网格所盖。如下图：

{% asset_img 1.png %}

网格中的一个单元相当于`canvas`元素中的一个像素。网格的左上角坐标为：（0,0）也称原点。 所有元素的位置都相对于原点定位。所以图中蓝色的方形左上角的坐标为距离左边（X轴） x 像素，距离上边（Y轴）y 像素；坐标为（x, y）。

## 绘制矩形

`canvas`只支持两种形式的图形绘制；**矩形**和**路径（由一系列点连成的线段）**。所有其他类型的图形都是通过一条或者多条路径组合而成。不过我们有很多路径生成的`API`。

- `fillRect(x, y, width, height)`: 用于绘制一个填充的矩形。
- `strokeRect(x, y, width,height)`: 用于绘制一个矩形的边框。
- `clearRect(x, y width, height)`: 用于清除指定矩形区域，让清除的部分完全透明。

这三个方法的 `x`, `y` 是指在`canvas`画布上所绘制的矩形的左上角（相对于原点）的坐标，`width `和 `height `是设置矩形的尺寸.

- `rect(x, y, width, height)`: 绘制一个左上角坐标为（x,y）,宽高为 `width`和`height`的矩形。

  这个方法执行的时候， `moveTo()`方法自动设置坐标参数（0,0）。也就是说当前笔触自动重置回默认坐标。

举个例子:

<iframe height="577" style="width: 100%;" scrolling="no" title="canvas 绘制矩形" src="https://codepen.io/shuliqi/embed/MWpYMaJ?height=577&theme-id=dark&default-tab=js,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/MWpYMaJ'>canvas 绘制矩形</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

上面例子中`fillRect)`绘制了一个宽为60px高为70ox的矩形，`clearRect()` 从距离 x 轴`15px`， y 轴 `50px` 的地方开始擦除。

`strokeRect`绘制了一个距离 x 轴`20px`， y 轴 `30px` 的宽为`90px`，高为`90px`高为`80px`的矩形。



## 绘制路径

图形的基本元素是路径。 路径是通过不同的颜色和宽度的线段或曲线相连形成的不同形状的点的集合。一个路径都是闭合的。使用路径绘制图形需要一些如下的步骤：

- 首先，需要创建路径的起始点。
- 然后使用 [画图命令](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D#paths)去画出路径。
- 之后把路径封闭。
- 路径生成之后。就能通过描边或者填充路径来渲染图形。

### 绘制路径所用到的函数

- `beginPath()`：新建一条路径，生成之后，图形绘制命令被指令到路径上生成路径。
- `closePath()`：闭合路径。闭合路径之后图形绘制命令又重新回到上下文中。
- `stroke()`：通过线条来绘制图形轮廓。
- `fill()`：通过填充路径的内容区域生成实心的图形。

生成路径的第一步叫做`beginPath()`。本质上路径是有很多的子路径构成，这些子路径都是在一个列表中，所有的子路径(线，弧形等等)构成图形。而每次调用这个方法之后，路径列表就会清空，我们就可以绘制新的图形。

> **注意：**当当前的路径为空(也就是调用`beginPath`之后或者`canvas`刚建的时候)。第一条路径构造命令通常是`moveTo(x,y)`、`moveTo(x,y)`设置路径的起始位置。无论出自于什么原因，总是要在设置路径之后专门指定路径的起始位置。

> `moveTo(x,y)`函数下面会讲到。

第二步就是调用函数指定绘制路径。文章之后会讲到。

第三步就是闭合路径`closePath()`。但是`closePath()`不是必须的。这个方法会通过绘制一条从当前点到开始点的直线来闭合图形。如果图形是是已经闭合了的(也就是说当前点为开始点)， 那么该函数什么也不做。

> 注意：当调用`fill()`函数时，所有没有闭的形状都会自动闭合。所以是不需要使用`closePath()`函数来闭合形状的。但是`stroke()`函数是不会自动闭合的。

举个例子：
<iframe height="378" style="width: 100%;" scrolling="no" title="绘制路径" src="https://codepen.io/shuliqi/embed/jOBPKYo?height=378&theme-id=dark&default-tab=js,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/jOBPKYo'>绘制路径</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>


### 移动笔触

**`moveTo(x,y):`** 将笔触移动到指定的坐标（`x`，`y`） 上。

`canvas`初始化或者`beginPath(x,y)`调用之后，通常使用`mnoveTo(x，y)`来设置起点。也可以使用`mnoveTo(x，y)`来设置一些不连续的路径。



### 线

**`lineTo(x,y):`** 绘制直线。绘制一条从当前位置到指定 `x`，`y`位置的直线

这个方法有两个参数： `x`,` y`；代表坐标系中直线结束的点。开始点和之前的绘制路径有关。开始点也可以通过`moveTo()`函数改变。

举个例子：
<iframe height="426" style="width: 100%;" scrolling="no" title="moveTo" src="https://codepen.io/shuliqi/embed/ExWjpjv?height=426&theme-id=dark&default-tab=js,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/ExWjpjv'>moveTo</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>


### 圆弧

绘制圆弧或者圆，我们使用`arc()`方法。当然也是可以使用`arcTo()`， 不过该方法并没有那么可靠。所以这里不做介绍。

**`arc(x,y,radius, startAngle, endAngle, anticlockwise)`：**

- `(x,y)`： 圆/圆弧的圆心
- `radius`： 圆的半径
- `startAngle`： 开始的弧度
- `endAngle`： 结束的弧度
- `anticlockwise`： 画圆的防线：`true`为逆时针，`false`为顺时针

画一个以`(x,y)`为圆心的以`radius`为半径的圆弧(圆)。从`startAngle`开始到`endAngle`结束。按照`anticlockwise`给定的方向(默认为顺时针)来生成。

> 注意：`arc()`函数中表示角的单位是弧度，而不是角度。角度与弧度的换算`js`表达式：弧度 =角度 \* Math.PI / 180； 1弧度等于半径的长度

举个例子：

<iframe height="563" style="width: 100%;" scrolling="no" title="圆弧" src="https://codepen.io/shuliqi/embed/RwpPOMR?height=563&theme-id=dark&default-tab=js,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/RwpPOMR'>圆弧</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### 贝赛尔曲线

`quadraticCurveTo(cplx, cply, x, y):`绘制二次贝塞尔曲线，`cplx`,` cply`是控制点，`x`， `y`为结束点。

`bezierCurveTo(cp1x, cp1y, cp2x, cp2y, x, y):`绘制三次贝塞尔曲线。`cp1x,cp1y`为控制点一，`cp2x,cp2y`为控制点二，`x,y`为结束点。

{% asset_img 2.png %}

二次贝塞尔曲线有一个开始点（蓝色）， 一个结束点（蓝色）， 一个控制点（红色）；

三次贝塞尔曲线有一个开始点（蓝色）， 一个结束点（蓝色），有两个控制点（红色）；

两个方法中的`x,y`都是结束坐标。`cp1x,cp1y`为坐标中的第一个控制点，`cp2x,cp2y`为坐标中的第二个控制点。



举个例子：

<iframe height="393" style="width: 100%;" scrolling="no" title="贝塞尔曲线" src="https://codepen.io/shuliqi/embed/jOBPjoB?height=393&theme-id=dark&default-tab=js,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/jOBPjoB'>贝塞尔曲线</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>
# 使用样式和颜色

## 色彩 

到目前为止， 我们只看到过绘制内容的方法，如果想要给图形上色，需要两个重要的属性：

- `fillStyle = color`：设置图形的填充颜色。
- `strokeStyle = color`：设置图形的轮廓颜色。

`color`值可以是表示`CSS`颜色字符串，渐变对象，图案对象;默认情况下，线条个填充颜色都是黑色(#0000).

> **注意：**一旦设置了`fillStyle`和`strokeStyle`的值，那么这个新值就会成为新绘制图形的默认值，如果需要给每个不同的图形上不同的颜色值，那么需要重新设置`fillStyle`，`strokeStyle`的值。

举个例子：
<iframe height="714" style="width: 100%;" scrolling="no" title="fillStyle/strokeStyle" src="https://codepen.io/shuliqi/embed/KKWdqeg?height=714&theme-id=dark&default-tab=js,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/KKWdqeg'>fillStyle/strokeStyle</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

## 透明度 

通过设置`globalAlpha`属性或者使用一个半透明颜色作为轮廓或填充的样式来绘制半透明的图形。

`globalAlpha = transparencyValue `：`transparencyValue`的优先范围为 0.0（完全透明） 到 1.0（完全不透明）

> **注意：**这个属性会影响到`canvas`里面所有图形的透明度。所以在绘制大量拥有相同透明度的图形的时候相当有用。不过使用设置` strokeStyle` 和` fillStyle`的值为有透明度的更有用。 如：
>
> ```js
> // 指定透明颜色，用于描边和填充样式
> ctx.strokeStyle = "rgba(255,0,0,0.5)";
> ctx.fillStyle = "rgba(255,0,0,0.5)";
> ```

 举个例子：

<iframe height="318" style="width: 100%;" scrolling="no" title="透明度" src="https://codepen.io/shuliqi/embed/OJpyQxv?height=318&theme-id=dark&default-tab=js,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/OJpyQxv'>透明度</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>
## 线形



可以通过一系列属性来设置线的样式。

### lineWidth

`lineWidth = value`: 设置线条的宽度。

举个例子：

<iframe height="235" style="width: 100%;" scrolling="no" title="lineWidth" src="https://codepen.io/shuliqi/embed/zYZvWOK?height=235&theme-id=dark&default-tab=js,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/zYZvWOK'>lineWidth</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### lineCap

`lineCap = type`: 设置末端的样式。

`type`的类型有：

- `butt`：末端以方形结束
- `round`： 末端以圆形结束
- `square`: 末端以方形结束，但是增加了一个宽度和线段相同，高度是线段厚度一半的矩形区域

举个例子：

<iframe height="564" style="width: 100%;" scrolling="no" title="lineCap" src="https://codepen.io/shuliqi/embed/BaWorob?height=564&theme-id=dark&default-tab=js,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/BaWorob'>lineCap</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>




# 绘制文本

 `canvas` 有两种方式可以绘制文本：

- `fillText(text, x, y [, maxWidth])`

  在指定的坐标（x，y）填充指定的文本，可以指定最大的宽度。

- `strokeText(text,x,y,[,maxWidth])`

  在指定的坐标上（x,y）绘制指定的文本边框，可以指定最大的宽度。

举个例子：

<iframe height="296" style="width: 100%;" scrolling="no" title="text" src="https://codepen.io/shuliqi/embed/xxqZgGg?height=296&theme-id=dark&default-tab=js,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/xxqZgGg'>text</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### 文本的样式

有很多的属性可以改变`canvas`的显示的文本的样式。

- `font = value`:

  当前用来绘制文本的样式，`font`的使用和`CSS` 的 [font](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial/Drawing_text) 语法是一样的。默认值："10px sans-serif"

- `textAlign = value`:

  当前绘制文本的对齐方式，默认值：`start`；值有：

  - `start`：文本对齐界面开始的地方
  - `end`：文本对齐界线结束的地方
  - `left：文本左对齐`
  - ,`right`： 文本右对齐
  -  `center`: 文本居中

- `textBaseline = value`:

  当前绘制文本的基线对齐方式，值有：`top`, `hanging`, `middle`, `alphabetic`, `ideographic`, `bottom`。默认值是 `alphabetic`

- `direction = value`:

  文本方向。默认值是 `inherit。值有：`

  - `ltr`： 文本从左到右
  - `rtl`：文本从右到左
  - `inherit`  根据情况继承



举个例子：

<iframe height="305" style="width: 100%;" scrolling="no" title="绘制文字的样式" src="https://codepen.io/shuliqi/embed/xxqZgYX?height=265&theme-id=dark&default-tab=js,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/xxqZgYX'>绘制文字的样式</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>


未完待续....学不动了





























































