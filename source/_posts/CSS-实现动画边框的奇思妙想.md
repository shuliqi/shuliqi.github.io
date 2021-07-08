---
title: CSS 实现动画边框的奇思妙想
date: 2021-06-26 17:34:39
tags: CSS
categories: CSS
---

最近在找一些周刊文章的时候读到一篇挺有意思的文章 [CSS 奇思妙想边框动画](https://juejin.cn/post/6918921604160290830)，经过学习之后，每实现一个效果的背后都有一些值得去复习或者是学习的知识点，觉得收获挺大。记录一下学习或者是复习到的一些知识点。

<!--more-->

# 思考？

如果我们希望实现一个有样式的边框，比如说：给边框加上一些动画，我们改如何实现呢？

既然说到边框， 我们肯定会想到 `border`。那么`border`都有哪些属性呢？那我们了可以先复习一下`border`

# border 属性

`border` 主要有三个属性：

- `border-width`: 设置盒子模型的边框宽度
- `border-style`：用来设定元素所有边框的样式
- ` border-color`:  用于设置元素四个边框颜色的快捷属性

而我们一般可以简写成：

```css
div {
  border: 2px solid #000
}
```

[MDN-border 详情](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border)

而关于`border-style`我们常用到的可能就是`solid`，`dashed`，`dotted`。但是还有很多其他的设置， 我们来看看：

<iframe height="300" style="width: 100%;" scrolling="no" title=" border-style" src="https://codepen.io/shuliqi/embed/wvJLdqE?defaultTab=html%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/shuliqi/pen/wvJLdqE">
   border-style</a> by shuliqi (<a href="https://codepen.io/shuliqi">@shuliqi</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

----



# 效果1：hover边框长度变化

{% asset_img 1.gif %}

这种效果的实现，我们的`border`属性好像实现不了， 那有没有其他的方式来实现呢？

**实现思路：**我们有一个元素，然后生成这个元素的两个伪元素，目的是用来设置上、左边框 和 下、右边框， 鼠标hover改变伪元素长度。

{% asset_img 2.png %}

蓝色的是我们的元素，绿色和红色是两个伪元素，这两个元素的是有宽和高的。

最后我们只需要 `hover`上去的时候，改变这两个伪元素的宽和高即可。

<iframe height="300" style="width: 100%;" scrolling="no" title="hover边框长度变化" src="https://codepen.io/shuliqi/embed/OJpegdV?defaultTab=css%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/shuliqi/pen/OJpegdV">
  hover边框长度变化</a> by shuliqi (<a href="https://codepen.io/shuliqi">@shuliqi</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>


----



# 渐变

因为下面的效果会用到渐变相关的内容， 所以我们先来复习一下渐变。

浏览器支持的渐变类型三种：

- [线性渐变 (linear-gradient)](https://developer.mozilla.org/zh-CN/docs/orphaned/Web/CSS/linear-gradient())
- [径向渐变 (radial-gradient)](https://developer.mozilla.org/zh-CN/docs/orphaned/Web/CSS/radial-gradient())
- [圆锥渐变(conic-gradient)](https://developer.mozilla.org/en-US/docs/Web/CSS/gradient/conic-gradient())

渐变在 CSS 中属于一种 Image 类型，可以结合 background-image 属性使用。

由于篇幅有限， 我们主要讲下面特效用到的线性渐变相关的一些小知识点。

## 线性渐变

**使用方式：**

```css
linear-gradient( angle, start-color, end-color )
```

- `angle`：是渐变的角度，可以是具体的角度（45deg），也可以是 to + 方向（to bottom right）
- ` start-color`：是渐变的初始颜色。
- `end-color`：是结束的颜色。

其中颜色的 ` start-color` 和` end-color` 我们可以称为**色标**, 色标可以是多个的,只是至少两个。

例子1:

```css
.item {
  width: 200px;
  height: 100px;
  background: linear-gradient(70deg, #077373, #1b6708);
}
```

{% asset_img 4.png %}

例子2：色标可以是多个的

```css
.item2 {
    width: 200px;
    height: 100px;
    background: linear-gradient(70deg, #077373, yellow, red, #1b6708);
}
```

{% asset_img 5.png %}

例子3: 如果需要自定义间距，可以在色标后面接具体的位置

```css
.item3 {
    margin-top: 10px;
    width: 200px;
    height: 100px;
    background: linear-gradient(90deg, #077373 30px, red 80%);
}
```

{% asset_img 6.png %}



**特点：**如果多个色标具有相同的位置，他们会产生一个无限小的过渡区域，过渡的起止色分别是第一个和最后一个指定值。从效果上看，颜色会在那个位置突然变化，而不是一个平滑的渐变过程。

<iframe height="300" style="width: 100%;" scrolling="no" title="线性渐变特点" src="https://codepen.io/shuliqi/embed/mdWZZej?defaultTab=css%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/shuliqi/pen/mdWZZej">
  线性渐变特点</a> by shuliqi (<a href="https://codepen.io/shuliqi">@shuliqi</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

----



# 效果2：虚线边框动画

{% asset_img 3.gif %}

提到虚线，我们是不是就想到了 `border-style`的`dashed` 属性？

<iframe height="300" style="width: 100%;" scrolling="no" title="border-style" src="https://codepen.io/shuliqi/embed/YzZoboY?defaultTab=css%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/shuliqi/pen/YzZoboY">
  border-style</a> by shuliqi (<a href="https://codepen.io/shuliqi">@shuliqi</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>


但是存在一个问题：`dashed` 不能让边框动起来呀。所以这个方法 `pass`掉。

那有没有其他的方法呢？那当然是有的， 实现虚线的方式在`CSS`中有很多种，比如我们上面复习到的渐变。



**实现思路:**

1. 我们首先来实现一条横着的虚线，这个虚线其实就是“—”的循环，所以我们只需要实现一个“—”， 然后使用 `background-repea`t进行循环即可。

<iframe height="300" style="width: 100%;" scrolling="no" title="一条虚线" src="https://codepen.io/shuliqi/embed/OJpevXG?defaultTab=css%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/shuliqi/pen/OJpevXG">
  一条虚线</a> by shuliqi (<a href="https://codepen.io/shuliqi">@shuliqi</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

我们使用` linear-gradient`来生成一个背景图，或利用` linear-gradient`的特点

> 如果多个色标具有相同的位置，他们会产生一个无限小的过渡区域，过渡的起止色分别是第一个和最后一个指定值。从效果上看，颜色会在那个位置突然变化，而不是一个平滑的渐变过程

来生成元素的50%曲线是黑色，50%是透明的背景图。并且渐变方向是`90deg`即使左右渐变的。使用[ background-size](https://developer.mozilla.org/zh-CN/docs/Web/CSS/background-size)来设置背景图片的大小，背景图片宽`10px`，高`4px`。背景图的循环方式是 x 轴循环。使用 [background-position](https://developer.mozilla.org/zh-CN/docs/Web/CSS/background-position)来设置初始位置。

1. 有了第一条， 同理， 就可以有第二，三，四条, 然后，利用 animation 变换每条虚线边框位置，然后虚线动起来。

<iframe height="300" style="width: 100%;" scrolling="no" title="虚线动画" src="https://codepen.io/shuliqi/embed/qBrzXyW?defaultTab=css%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/shuliqi/pen/qBrzXyW">
  虚线动画</a> by shuliqi (<a href="https://codepen.io/shuliqi">@shuliqi</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

画其他虚线的时候， 需要注意的 背景图片的大小，和位置。

---



# 效果3：动画彩条边框

{% asset_img 7.gif %}

这个效果也是可以使用渐变来实现的， 我们来一步一步的实现它

首先使用伪元素和渐变实现一个如下效果：

<iframe height="300" style="width: 100%;" scrolling="no" title="彩条框" src="https://codepen.io/shuliqi/embed/MWpMGbQ?defaultTab=css%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/shuliqi/pen/MWpMGbQ">
  彩条框</a> by shuliqi (<a href="https://codepen.io/shuliqi">@shuliqi</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

上面这个效果的重要的一点为：伪元素实现这个元素， 每一个色块都是父元素的2倍大， 再通过父级的`overflow: hidden`将超出部分隐藏



然后给色块加上动画效果，让色块动起来:

<iframe height="300" style="width: 100%;" scrolling="no" title="动画彩条框" src="https://codepen.io/shuliqi/embed/NWpZMvg?defaultTab=css%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/shuliqi/pen/NWpZMvg">
  动画彩条框</a> by shuliqi (<a href="https://codepen.io/shuliqi">@shuliqi</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

上面代码中的 1turn：一圈，一个圆共一圈。 90deg = 0.25turn。



最后再添加一个伪元素， 将div内部部分遮住， 即可实现想要的效果，

<iframe height="285" style="width: 100%;" scrolling="no" title="动画边框" src="https://codepen.io/shuliqi/embed/PoprRaM?defaultTab=css%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/shuliqi/pen/PoprRaM">
  动画边框</a> by shuliqi (<a href="https://codepen.io/shuliqi">@shuliqi</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

---



# clip-path

下面的效果会用到这个`CSS`属性， 我们先来了解下：

CSS 的 [clip-path](https://developer.mozilla.org/en-US/docs/Web/CSS/clip-path) 属性是 [clip](https://developer.mozilla.org/en-US/docs/Web/CSS/clip) 属性的升级版，它们的作用都是对元素进行 “剪裁”，不同的是 clip 只能作用于 position 为 absolute 和 fixed 的元素且剪裁区域只能是正方形，而 clip-path 更加强大，可以以任意形状去裁剪元素，且对元素的定位方式没有要求。基于这样的特性，clip-path 常用于实现一些炫酷的动画效果。



**clip-path 的其中两大类，分别为：**

1. `basic-shape`：基本图形，包括` inset()`、`circle()`、`ellipse()`、`polygon()`
2. `clip-source`：通过 url() 方法引用一段 SVG 的 <clipPath> 来作为剪裁路径

 我们接下来重点学习 `basic-shape`

## inset

用于定义一个插进的矩形，即剪裁元素内部的一块矩形区域。

参数类型：`inset( <shape-arg>{1,4} [round <border-radius>]? )`

- `shape-arg`：分别为矩形的上右下左边框到被剪裁元素边缘的距离

{% asset_img 10.PNG %}

- `border-radius`：为可选参数，用于定义 `border` 的圆角。

我们举个例子：

<iframe height="298" style="width: 100%;" scrolling="no" title="clip-path:inset" src="https://codepen.io/shuliqi/embed/ExWBGBB?defaultTab=css%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/shuliqi/pen/ExWBGBB">
  clip-path:inset</a> by shuliqi (<a href="https://codepen.io/shuliqi">@shuliqi</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

这里例子中，我们初始是没有做任何的裁剪的（clip-path: inset(0);）。当hover 上去的时候我们使用` inset `来进行一个矩形的裁剪，并且裁剪出来的是矩形是有`30px`的圆角的。

##  ellipse

 用于定义一个椭圆。

参数类型：`ellipse( [<shape-radius>{2}]? [at <position>]? )`

- `shape-radius`: 为椭圆x、y轴的半径
- `position`: 为椭圆中心的位置。

举个例子：

<iframe height="300" style="width: 100%;" scrolling="no" title="clip-path:ellipes" src="https://codepen.io/shuliqi/embed/poeXGvx?defaultTab=css%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/shuliqi/pen/poeXGvx">
  clip-path:ellipes</a> by shuliqi (<a href="https://codepen.io/shuliqi">@shuliqi</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

## polygon

 用于定义一个多边形。

参数类型：`polygon( [<fill-rule>,]? [<shape-arg> <shape-arg>]# )`

- fill-rule： 可选，表示填充规则用来确定该多边形的内部。可能的值有nonzero和evenodd,默认值是nonzero
- 后面的每对参数表示多边形的顶点坐标（X,Y），也就是连接点

举个例子:

<iframe height="300" style="width: 100%;" scrolling="no" title="clip-path:polygon" src="https://codepen.io/shuliqi/embed/YzZoByp?defaultTab=css%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/shuliqi/pen/YzZoByp">
  clip-path:polygon</a> by shuliqi (<a href="https://codepen.io/shuliqi">@shuliqi</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>



---



# 效果4： 追逐的边框

{% asset_img 8.gif %}

## 方式1

根据我们上一个例子的效果，我们实现这个效果其实就很简单了， 我们只需要把：我们将 彩条边框 4个背景图变成 1 个背景图就可实现了。

<iframe height="300" style="width: 100%;" scrolling="no" title="追逐边框" src="https://codepen.io/shuliqi/embed/wvJLjje?defaultTab=css%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/shuliqi/pen/wvJLjje">
  追逐边框</a> by shuliqi (<a href="https://codepen.io/shuliqi">@shuliqi</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

**缺陷：**如果是单线条，边框的末尾是一个小三角而不是垂直的，可能些      场景不适用或者 PM 接受不了

{% asset_img 9.gif %}



## 方式2

有没有什么办法可以消除这些小三角形呢? 肯定是有的。那就是我们上面学到的 `clip`。

**实现思路：**借用伪元素作为背景进行裁剪并动画即可

<iframe height="350" style="width: 100%;" scrolling="no" title="clip-path: 追逐边框" src="https://codepen.io/shuliqi/embed/QWpXPBO?defaultTab=css%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/shuliqi/pen/QWpXPBO">
  clip-path: 追逐边框</a> by shuliqi (<a href="https://codepen.io/shuliqi">@shuliqi</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

这种方式的优点就是：切割出来的边框不会产生小三角

