---
title: canvas的学习及其使用
date: 2021-05-07 20:28:48
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

这三个方法的 `x`, `y` 是指在`canvas`画布上所绘制的矩形的左上角（相对于远点）的坐标，`width `和 `height `是设置矩形的尺寸.

举个例子:

<iframe height="610" style="width: 100%;" scrolling="no" title="canvas 绘制矩形" src="https://codepen.io/shuliqi/embed/MWpYMaJ?height=462&theme-id=dark&default-tab=js,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/MWpYMaJ'>canvas 绘制矩形</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

上面例子中`fillRect)`绘制了一个宽高都是150ox的正方形，`clearRect()` 从距离 x 轴`45px`， y 轴 `50px` 的地方开始擦除。

`strokeRect`绘制了一个距离 x 轴`20px`， y 轴 `30px` 的宽高都是`150px`的正方形。



## 绘制路径

图形的基本元素是路径。 路径是通过不同的颜色和宽度的线段或曲线相连形成的不同形状的点的集合。一个路径都是闭合的。使用路径绘制图形需要一些如下的步骤：

- 首先，需要创建路径的起始点。
- 然后使用 [画图命令](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D#paths)去画出路径。
- 之后把路径封闭。
- 路径生成之后。就能通过描边或者填充路径来渲染图形。



































