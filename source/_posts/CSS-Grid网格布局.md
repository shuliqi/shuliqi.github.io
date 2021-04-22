---

title: CSS Grid网格布局
date: 2020-12-04 10:23:59
tags:
---

最近在参加一个小程序比赛，其中在做瀑布流布局的时候，有人拿Grid网格布局来做的，但是我觉得瀑布流并不是个用Grid网格布局来做，因为它每一个项目占多少网格，都需要人工去指定。虽然我觉得Grid网格布局不是个做瀑布流， 但是我觉得它还是非常强大的，甚至觉得比Flex强大。那么这篇文章就是来记录Grid网格布局的一个使用教程的。当翻阅可看。

<!--more-->

# 前言

在我们web页面开发过程中，我们可以使用css控制页面中元素的位置，主要的布局样式有以下几种：

- 正常的布局流
- display 属性
- 弹性盒子（[FlexBox](https://shuliqi.github.io/shuliqi.github.io/2018/03/31/Flex%E5%B8%83%E5%B1%80/)）
- 网格（display：table）
- 浮动（float）
- 定位（position）
- CSS Grid网格布局
- 多列布局（Multi-column layout）

# 什么是网格布局（Grid）？

网格布局（Grid）将网页分成一个个网格，可以任意组合不同的网格，设计出各种各样的的布局。

网格布局（Grid）将容器划分成“行” 和 “列”， 然后指定“项目所在”的单元格，可以看做是二维布局。Grid 布局远比 Flex 布局强大。



# 基本概念

学习 Grid 布局之前，需要了解一些基本概念。

## 容器和项目

使用网格布局的区域，称为“容器”（container），容器内采用网格定位的子元素，称为“项目”（item）

例如：

```html
<div class="container">
	<div class="item"><span></span></div>
	<div class="item"><span></span></div>
	<div class="item"><span></span></div>
</div>
```

如果我们给 class 为 `container`的元素设置它为 网格布局（Grid）。那么该元素就是容器， 里面的三个class为`tem`的元素就是项目。

**注意：**项目只能是容器最顶层子元素，不包含项目的子元素。例如上面的`span`标签就不是项目。Grid布局就对项目生效。

## 行，列，单元格，网格线

{% asset_img 3.png %}

**行和列：**

容器里面的水平区域称为”行“（row）；垂直区域称为”列“（column）。 上图中的水平绿色区域就是”行“。垂直的蓝色区域就是”列“。

**单元格：**

行和列的交叉区域称为”单元格“(cell)。如上图显示”单元格“的区域就是其中的一个单元格

**网格线：**

划分为网格的线，称为：”网格线“（grid line）。水平网格线划分出行，垂直网格线划分出列。日上图中就有5个水平网格线 和 10 个垂直网格线。



# 容器属性

Grid 布局的属性分为两类。一类是定义在容器上面，称为容器属性；另一类定义在项目上面，称为项目属性。这里我们先讲容器属性。

## display 属性

使用`display:grid`指定一个容器采用网格布局。

```css
.grid {
  display: grid;
}
```

如：

<iframe height="400" style="width: 100%;" scrolling="no" title="KKgVeKX" src="https://codepen.io/shuliqi/embed/KKgVeKX?height=265&theme-id=dark&default-tab=css,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/KKgVeKX'>KKgVeKX</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>
默认情况下，容器元素都是块级元素，单也可以设置为行内元素。

```css
.grid {
  display: inline-grid;
}
```

如：

<iframe height="400" style="width: 100%;" scrolling="no" title="网格布局inline-grid" src="https://codepen.io/shuliqi/embed/qBabKoB?height=265&theme-id=dark&default-tab=css,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/qBabKoB'>网格布局inline-grid</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

上面的这个例子指定了 class 为 `grid`的 div 为行内元素，该元素内部的项目使用网格布局。

**注意：** 设置网格布局之后，容器项目的 `float`，`display: inline-block`,`display: table-cell`、`vertical-align`和`column-*`等设置将会失效。

## grid-template-columns属性

## grid-template-rows 属性

容器指定了网格布局之后，就需要划分行和列。`grid-template-columns` 属性用来定义每一列的宽度，`grid-template-rows`属性用来定义每一行的行高。

例如我们上面两个例子：

```CSS
.grid {
  display: grid;
  grid-template-columns: 100px 100px;
  grid-template-rows: 100px 100px;
}
```

如上：指定了一个两行两列的网格，列宽和行高都是 `100px`。

当然`grid-template-columns` 和`grid-template-rows`属性除了可以使用绝对值外，也可以使用百分比的

```css
.grid {
  display: grid;
  grid-template-columns: 50% 50%;
  grid-template-rows: 50% 50%;
}
```

#### repeat()

有时候列宽可能是固定， 到那时我们在指定每一列的列宽时，都需要一一写上。就很麻烦。这时候就可以使用repeat()函数，简化重复的值。

**repeat()：**接受两个参数，第一个参数是重复的次数，第二个参数是所要重复的值。例如上面的代码就可以写成：

```css
.grid {
  display: grid;
  grid-template-columns: repeat(2, 50%);
  grid-template-rows: repeat(2, 50%);
}
```

**repeat()：** 也可以重复某种模式。如：

```CSS
.grid {
  display: grid;
  grid-template-columns: repeat(2, 100px 200px);
  grid-template-rows:  repeat(2,  200px 100px);
}
```
<iframe height="600" style="width: 100%;" scrolling="no" title="grid网格布局repeat()" src="https://codepen.io/shuliqi/embed/LYRGgPv?height=265&theme-id=dark&default-tab=css,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/LYRGgPv'>grid网格布局repeat()</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

如上这个例子的代码就表示：第一列和第三列的宽为80px，第二列和第四列的宽为100ox。

#### auto-fill 关键字

有时候单元格的大小是固定的，但是容器的大小是不固定的。如果我们是希望没一行或者每一列能容纳尽可能多的单元格。就可以使用`auto-fill`关键字了。这个关键字表示自动填充。

```css
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, 100px);
}
```

<iframe height="400" style="width: 100%;" scrolling="no" title="auto-fill" src="https://codepen.io/shuliqi/embed/MWjKPwQ?height=265&theme-id=dark&default-tab=css,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/MWjKPwQ'>auto-fill</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

例子代码表示：每列列宽为100px，然后自动填充，直到容器不能放置更多的列。

#### fr关键字

为了方便的表示比列关系，网格布局提供了`fr` 关键字。如果两列的了宽度分别是 `1fr` 和 `2fr` 表示 后者的列宽是前者的两倍。

如：

```CSS
.grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  grid-template-rows: 1fr 1fr;
}
```

这表示容器的高度被分为2等份，第一行和第二行的行高都各占了一份。容器的宽被分成2等份，第一列和第二列的列宽都个占了1份。

<iframe height="400" style="width: 100%;" scrolling="no" title="fr" src="https://codepen.io/shuliqi/embed/OJRMBWW?height=265&theme-id=dark&default-tab=css,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/OJRMBWW'>fr</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

 再如：

```css
.grid {
  display: grid;
  grid-template-columns: 2fr 1fr 3fr;
}
```

表示容器的宽被分成了5份等份，第一列的列宽占了2份。第二列的列宽占了一份，第三列的列宽占了3份。

<iframe height="400" style="width: 100%;" scrolling="no" title="xxEZydN" src="https://codepen.io/shuliqi/embed/xxEZydN?height=265&theme-id=dark&default-tab=css,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/xxEZydN'>xxEZydN</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

#### minmax()

`minmax()`函数产生一个长度范围。它有两个参数，最大值和最小值。

```css
.grid {
	display: grid;
	grid-template-columns: 100px minmax(100pox, 200px)
}
```

这就表示，该项目的宽度不小于100px，不大于200px



#### auto关键字

该关键字表示项目的宽度有浏览器自己决定。

```
.grid {  
  display: grid;  
  grid-template-columns: auto 100px;
}
```

表示第一个项目的宽度基本是等于该单元格的最大宽度。

#### 网格线的名称

使用`grid-template-columns`属性和 `grid-template-rows`属性时，可以使用方括号，指定每一根网线的名字。方便之后的引用。

```css
.grid {  
  display: grid;  
  grid-template-columns: [a1] auto [a2] 100px [a3];
  grid-template-rows: [b1] 100px [b2] 100px [b3]
}
```

如上的代码指定的网格布局为两列两行，因此有3根水平网格线和3根垂直网格线。方括号里面依次是这6根网格线的名字。

<iframe height="400" style="width: 100%;" scrolling="no" title="行间距和列间距" src="https://codepen.io/shuliqi/embed/OJRRaOm?height=265&theme-id=dark&default-tab=css,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/OJRRaOm'>行间距和列间距</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

**注意：**网格布局允许同一根线可以有多个名字。如：

```css
grid-template-columns: [a1 m1] auto [a2] 100px [a3];
```

## grid-row-gap 属性，

## grid-column-gap属性

## grid-gap 属性

`grid-row-gap`属性设置行与行之间的间隔行间距

`grid-column-gap` 属性设置列与列之间的间隔（列间距）

```css
.grid {
  display: grid;
  grid-template-columns: 100px 100px;
  grid-template-rows: 100px 100px;
  /*   行间距 */
  grid-row-gap: 20px;
  /*   列间距 */
  grid-column-gap: 20px;
}
```

`grid-gap` 属性是 `grid-column-gap`和 `grid-row-gap`的合并简写。语法:`grid-gap: <grid-row-gap> <grid-column-gap>`

**注意：**如果`grid-gap`省略了第二个值。那么浏览器会默认为第二个值等于第一个值

**注意**：根据最新的标准。`grid-row-gap`，`grid-column-gap`，`grid-gap` 这三个属性不用写前缀`grid`。

## grid-template-areas 属性

网格布局允许执行“区域”（areas），一个区域有多个单元格组成。那么`grid-template-areas`属性就是用来定义区域的.

```css
.grid {
  display: grid;
  grid-template-columns: 100px 100px;
  grid-template-rows: 100px 100px;
  grid-row-gap: 20px;
  grid-column-gap: 20px;
  /*   指定区域 */
  grid-template-areas: 'a b'
                       'c d';
}
```

<iframe height="400" style="width: 100%;" scrolling="no" title="指定区域" src="https://codepen.io/shuliqi/embed/oNzzQaj?height=265&theme-id=dark&default-tab=css,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/oNzzQaj'>指定区域</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

上面的这个例子我们审查元素来看看。可以看出区域被分成了4部分 a b c d;

{% asset_img 4.png %}

**多个单元格合并成一个单元格的写法：**

```css
grid-template-areas: 'a a'
                     'b b';
```

如图：

{% asset_img 5.png %}

这样就把把四个单元格分成了 a b  两个区域。

如果某个区域不需要的话，则使用  .  来表示。

```css
grid-template-areas: 'a .'
                     '. b';
```

如图：

{% asset_img 6.png %}

**注意：**区域的命名会影响到网格线，每个区域的起始网格线会自动命名为`区域名-start`,终止网格线会自动命名为`区域名-end`.



## grid-auto-flow 属性

`grid-auto-flow`属性用来设置容器项目的放置顺序。主要有四个值：

### `row`:

默认值。表示放置顺序为”先行后列“。 也就是先填满第一行。再放入第二行。

> 我们上面的所有的例子都是这样的（看元素中的数字可以看出），都是先放满第一行，再放如第二行。

### `column:` 

表示"先列后行"。

如：

<iframe height="462" style="width: 100%;" scrolling="no" title="先列后行" src="https://codepen.io/shuliqi/embed/MWjjNZw?height=462&theme-id=dark&default-tab=css,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/MWjjNZw'>先列后行</a> by shuliqi
    (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

从例子中可以看出来 1，2  在左边；3，  4 在右边。这是因为我们设置：

```css
 /*   先列后行 */
 grid-auto-flow: column;
```

### `row dense:` 

表示"先行后列"，每一行尽可能精密填满，尽量不要有空格；

这个设置的意义在哪里呢？ 我们先来看一个例子：

<iframe height="537" style="width: 100%;" scrolling="no" title="尽量不要有空格" src="https://codepen.io/shuliqi/embed/VwKKoNQ?height=537&theme-id=dark&default-tab=css,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/VwKKoNQ'>尽量不要有空格</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>
在这个例子中， 我们采用的是”先行后列“的放置顺序；设置第一个项目和第二个项目列宽占两个单元格:

```
 grid-column-start: 1;
 grid-column-end: 3;
```

然后就得到如上的结果，从结果可以看出，第一行空出了一个空白的地方。这是为什么呢？ 这是因为第三个项目默认跟着第二个项目，所以会排在第二个项目之后。

 但是我们把放置顺序改为：`row dense`。那么结果就是：

  <iframe height="467" style="width: 100%;" scrolling="no" title="QWKGLGb" src="https://codepen.io/shuliqi/embed/QWKGLGb?height=467&theme-id=dark&default-tab=css,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
    See the Pen <a href='https://codepen.io/shuliqi/pen/QWKGLGb'>QWKGLGb</a> by shuliqi
    (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
  </iframe>

这个例子就可以看出来： 先是填满第一行，再填满第二行，并且每一行尽量不要有空格。所以第三个元素会在第一个元素后面

### `column dense` 

表示 ”先列后行“ 并且每一列尽量不要有空格。

跟上一个的适用场景一样。 我们将放置顺序改为`column dense`；那么将得到这样的结果：

<iframe height="544" style="width: 100%;" scrolling="no" title="GRjNKQb" src="https://codepen.io/shuliqi/embed/GRjNKQb?height=544&theme-id=dark&default-tab=css,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/GRjNKQb'>GRjNKQb</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>



## justify-items 属性

## align-items 属性

## place-items 属性

`justify-items`属性用来设置**单元格内容**的水平位置。`align-items` 属性用来设置**单元格内容**的垂直位置。这两个都有如下的值：

* `start：` 对齐单元格的起始位置
* `end：`对齐单元格的结束边缘
* `center:`单元格内部居中
* `stretch `:项目大小没有指定时，拉伸占据整个网格容器。

这里需要注意的是**单元格内容**

例：

<iframe height="315" style="width: 100%;" scrolling="no" title="jOMBmZg" src="https://codepen.io/shuliqi/embed/jOMBmZg?height=315&theme-id=dark&default-tab=css,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/jOMBmZg'>jOMBmZg</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

如上的代码， 我们让每个单元格里面的内容水平位置居中，垂直位置居中。我们可以通过审查元素来看：

{% asset_img 7.png %}

可以看出来，class为 item的元素在第一个单元格的位置都是单元格内部居中（水平位置居中，垂直位置居中）。

其他值我就不一一举例了。可以直接使用上面的例子，修改 ` justify-items` 和 `align-items`的值看效果。



`place-items 属性`是 `align-items`属性和`justify-items`属性的合并简写形式。

```css
place-items: <align-items> <justify-items>;
```

例：

<iframe height="330" style="width: 100%;" scrolling="no" title="OJRpgJY" src="https://codepen.io/shuliqi/embed/OJRpgJY?height=330&theme-id=dark&default-tab=css,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/OJRpgJY'>OJRpgJY</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>
如上的例子： 我们使用`place-items`设置了 垂直方向是对齐单元格的结束方向。水平方向是居中。我们可以审查元素来看：

{% asset_img 8.png %}

可以看出 来class为 item的元素在第一个单元格的位置是垂直方向是对齐单元格的结束方向。水平方向是居中。

**注意：** `place-items`属性的第二个参数如果省略的话，则浏览器默认与第一个值相等。



## justify-content 属性

## align-content 属性

## palce-content 属性



`justify-content` 属性是设置整个内容区域在容器里面的水平位置，`align-content`属性是设置整个内容区域在容器里面的垂直位置。这两个属性都有如下的值：

- **start**： 对齐容器的起始边框；

  {% asset_img 9.png %}

- **end**：对齐容器的结束边框;

  {% asset_img 10.png %}

- **center**：容器内部居中；

  {% asset_img 11.png %}

- **stretch** - 项目大小没有指定时，拉伸占据整个网格容器。

  {% asset_img 12.png %}

- **space-around** - 每个项目两侧的间隔相等。所以，项目之间的间隔比项目与容器边框的间隔大一倍。

  {% asset_img 13.png %}

- **space-between** - 项目与项目的间隔相等，项目与容器边框之间没有间隔。

  {% asset_img 14.png %}

- **space-evenly** - 项目与项目的间隔相等，项目与容器边框之间也是同样长度的间隔。

  {% asset_img 15.png %}

`place-content`属性是 `justify-content`属性和`align-content`属性的合并缩写形式：

```css
place-content: <align-content> <justify-content>
```

如果`place-content`省略第二个值，那么浏览器就会默认第二个值等于第一个值。



## grid-auto-columns 属性

## grid-auto-rows 属性

有时候我们的项目的指定在网格的外面，那么浏览器就会自动根据单元格的大小生成多余的网格。

`grid-auto-cloumns`属性 和 `grid-auto-rows`属性就是用来设置当浏览器自动创就按多余的网格的列宽和行高。这两个属性的法与`grid-template-columns`,`grid-template-rows`是一样的;

如：
<iframe height="553" style="width: 100%;" scrolling="no" title="grid-auto-columns" src="https://codepen.io/shuliqi/embed/dypvRMq?height=553&theme-id=dark&default-tab=css,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/dypvRMq'>grid-auto-columns</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

这个例子：第7 个项目超出了当前的网格（使用了`grid-row-start` 和 `grid-column-start`下面会有介绍），那么浏览器会自动生成多余的网格，我们设置多余的网格列宽和行高都是50px；



# 项目的属性

定义在项目上的属性。

## grid-column-start 属性

## grid-column-end 属性

## grid-row-start 属性

## grid-row-end 属性

容器中的项目是可以指定的。使用这四个属性定义项目的边框。这四个属性是定义项目在哪根网格线；

* grid-column-start 属性：项目左边框所在的垂直网格线；
* grid-column-end属性：项目右边框所在的垂直网格线；
* grid-row-start 属性：项目上边框所在的水平网格线；
* grid-row-end 属性：项目下边框所在的水平线网格线；

我们讲解上个属性的时候也用到了这些属性 `grid-row-start` 和 `grid-column-start`。我们可以再举个例子：

<iframe height="595" style="width: 100%;" scrolling="no" title="grid-auto-columns" src="https://codepen.io/shuliqi/embed/jOMBZWd?height=595&theme-id=dark&default-tab=css,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/jOMBZWd'>grid-auto-columns</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

我们可以通过审查元素看出来第7个元素占的网格线情况：

{% asset_img 16.png %}

第七个项目的上边框在水平网格线的第四根，左边框在垂直网格线的第一根。下边框和右边框没有指定，所以会采用默认位置(下边框滴5根网格线。有边框第二根网格线)。

再看一个例子：

<iframe height="629" style="width: 100%;" scrolling="no" title="grid-auto-columns" src="https://codepen.io/shuliqi/embed/QWKpQpo?height=629&theme-id=dark&default-tab=css,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/QWKpQpo'>grid-auto-columns</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

我们可以通过审查元素看出来第7个元素占的网格线情况：

{% asset_img 17.png %}

我们可以看出第一个项目的左边框在垂直网格线的第二个，右边框在垂直网格线的第四个网格线。上下边框没有指定使用默认位置。除了第一个项目之外，其他项目没有指定位置，由浏览器自动布局。这时它们的位置由容器的`grid-auto-flow`属性决定，这个属性的默认值是`row`，因此会"先行后列"进行排列



**注意1：**这四个属性的值，也可以指定为网格线的名字。

<iframe height="634" style="width: 100%;" scrolling="no" title="grid-auto-columns" src="https://codepen.io/shuliqi/embed/xxEqYYg?height=634&theme-id=dark&default-tab=css,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/xxEqYYg'>grid-auto-columns</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

第一个项目的左边框指定为a2的网格线，右边框指定为a4的网格线，



**注意2：**这四个属性可以使用 `span`关键字。表示上下边框（左右边框）之间跨越多少网格。

<iframe height="634" style="width: 100%;" scrolling="no" title="grid-auto-columns" src="https://codepen.io/shuliqi/embed/xxEqYYg?height=634&theme-id=dark&default-tab=css,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/xxEqYYg'>grid-auto-columns</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

第一个项目的左右边框，上下边框都跨越2个网格（绿色部分）。