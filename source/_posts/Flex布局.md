---
title: Flex布局教程
date: 2018-03-31 17:31:33
tags: CSS
categories: CSS
---

## Flex 布局的概念

Flex 是 Flexible Box 的缩写，意思就是“弹性布局”。它的作用就是为盒状模型提供最大的灵活性。

任何容器都可以指定为Flex 布局.

<!--more-->

```css
.box {
	display: flex;
}
```

行内元素也可以使用flex 布局

```css
.container {
	dispaly: inline-block;
}
```

**注意：** 设置为Flex布局之后， 子元素的·`float`，`claer`，`vertical-align` 将会失效。

## Flex的基本概念

使用了Flex布局的元素， 称为：Flex容器， 简称：“容器”。 容器内的所有元素自动成为容器成员。称为：Flex项目， 简称“项目“。

{% asset_img 1.png %}

容器默认存在两根轴，水平的主轴（main axis）和垂直的交叉轴（cross axis），主轴的开始位置（与边框的交叉点）叫`main start`, 结束位置叫`main end`交叉轴的开始位置叫`cross start`, 结束位置叫`cross start`。

项目默认沿主轴排列。单个项目占据的主轴空间叫`main size`, 占据的交叉轴空间叫`cross size`。



## 容器的属性

一共有6 个属性是可以设置在容器上。

1. **flex-direction**
2. **flex-wrap**
3. **flex-flow**
4. **justify-content**
5. **align-items**
6. **align-content**

### flex-direction 属性

flex-direction 决定主轴的方向（项目的排列方向）。

```
.container{
	flex-direction: row | row-reverse | column | column-reverse
}
```

flex-direction的4个值的含义：

1. row：默认值，主轴为水平方向，起点在左端。

   ```css
   .container {
     width: 100%;
     height: 400px;
     background: #54b4c557;
     display: flex;
     flex-direction: row; // 默认值，主轴为水平方向，起点在左端 
     
   }
   .item {
     background: #f19f9e;
     width: 30px;
     height: 30px;
     text-align: center;
     line-height: 30px;
     margin: 2px;
   }
   ```

   {% asset_img direction1.jpg %}

   完整 [Demo](https://jsbin.com/cuvokizabu/1/edit?html,css,output)

2. row-reverse：主轴是水平方向，起点在右端

   ```css
   .container {
     width: 100%;
     height: 400px;
     background: #54b4c557;
     display: flex;
     flex-direction: row-reverse; // 主轴是水平方向，起点在右端
     
   }
   .item {
     background: #f19f9e;
     width: 30px;
     height: 30px;
     text-align: center;
     line-height: 30px;
     margin: 2px;
   }
   ```

   {% asset_img direction2.jpg %}

   完整 [Demo](https://jsbin.com/rihinemulu/edit?html,css,output)

3. column：主轴为垂直方向， 起点在上沿

   ```css
   .container {
     width: 100%;
     height: 400px;
     background: #54b4c557;
     display: flex;
     flex-direction: column; // 主轴为垂直方向， 起点在上沿
     
   }
   .item {
     background: #f19f9e;
     width: 30px;
     height: 30px;
     text-align: center;
     line-height: 30px;
     margin: 2px;
   }
   ```

   {% asset_img direction3.jpg %}

   完整 [Demo](https://jsbin.com/tasofavuro/edit?html,css,js,output)

4. column-reverse：主轴是垂直方向，起点在下沿

   ```css
   .container {
     width: 100%;
     height: 400px;
     background: #54b4c557;
     display: flex;
     flex-direction: column-reverse; //主轴是垂直方向，起点在下沿
     
   }
   .item {
     background: #f19f9e;
     width: 30px;
     height: 30px;
     text-align: center;
     line-height: 30px;
     margin: 2px;
   }
   ```

   {% asset_img direction4.jpg %}

   完整 [Demo](https://jsbin.com/kerinetanu/1/edit?html,css,js,output)

### flex-wrap 属性

默认情况下，所有的项目都排列在一条线上（又称为“轴线“）. flex-wrap属性定义， 如果在一条轴线上排不下， 如何换行。

```
.container {
	flex-wrap: nowrap | wrap | wrap-column
}
```

flex-wrap的3个值的含义：

1. **nowrap**： 默认值，不换行

   ```css
   .container {
     width: 100%;
     height: 400px;
     background: #54b4c557;
     display: flex;
     flex-wrap: nowrap: // 默认值，不换行
     
   }
   .item {
     background: #f19f9e;
     width: 100px;
     height: 30px;
     text-align: center;
     line-height: 30px;
     margin: 2px;
   }
   ```

   {% asset_img wrap1.jpg %}

   完整 [Demo](https://jsbin.com/wutepopode/edit?html,css,js,output)

2. wrap：换行， 第一行在上方

   ```css
   .container {
     width: 100%;
     height: 400px;
     background: #54b4c557;
     display: flex;
     flex-wrap: wrap; // 换行， 第一行在上方
     
   }
   .item {
     background: #f19f9e;
     width: 100px;
     height: 30px;
     text-align: center;
     line-height: 30px;
     margin: 2px;
   }
   ```

   {% asset_img wrap2.jpg %}

   完整 [Demo](https://jsbin.com/tuzihaqaza/1/edit?html,css,js,output)

3. wrap-reverse：换行， 第一行在下方

   ```css
   .container {
     width: 100%;
     height: 400px;
     background: #54b4c557;
     display: flex;
     flex-wrap: wrap-reverse; // 换行， 第一行在下方
     
   }
   .item {
     background: #f19f9e;
     width: 100px;
     height: 30px;
     text-align: center;
     line-height: 30px;
     margin: 2px;
   }
   ```

   {% asset_img wrap3.jpg %}

   [Demo](https://jsbin.com/pugusasura/edit?html,css,output)

### flex-flow 属性

`flex-flow` 属性是` flex-direction`属性  和 `flex-wrap`属性的简写形式。默认值：`row nowrap`

```
.container {
	flex-flow: <flex-direction> | <flex-wrap>
}
```

### justify-content 属性

`justify-content`属性定义了项目在主轴上的对齐方式。

```
.container {
	justify-content: flex-start | flex-end | center| space-between | space-around
}
```

justify-content属性有5个值：

1. **flex start:**      默认值，主轴的起点对齐
2. **flex-end：** 主轴的终点对齐
3. **center：**居中
4. **space-between：**两端对齐，项目之间的间隔是相等的
5. **space-around：** 每个项目两侧的距离相等，所以项目之间的间距与项目与边框的间距要大

```css
.container {
  width: 100%;
  height: 40px;
  background: #54b4c557;
  display: flex;
  border-bottom: 1px dashed #000;
  padding: 5px 0;
}
.container1 {
  /* 默认值，主轴的起点对齐 */
  justify-content: flex-start; 
}
.container2 {
 /*  主轴的终点对齐 */
  justify-content: flex-end;
}
.container3 {
  /* 居中 */
  justify-content: center;
}
.container4 {
  /* 两端对齐，项目之间的间隔是相等的 */
  justify-content: space-between;
}
.container5 {
  /* 每个项目两侧的距离相等，所以项目之间的间距与项目与边框的间距要大 */
  justify-content: space-around;
}

.item {
  background: #f19f9e;
  width: 30px;
  height: 30px;
  text-align: center;
  line-height: 30px;
  margin: 2px;
}
```

{% asset_img 7.jpg %}

完整[Demo](https://jsbin.com/xoqikozalu/1/edit?html,css,output)

### align-items 属性

`align items` 属性定义项目在交叉轴的对齐方式，

```css
.box {
   align-items: flex-start |flex-end | center | baseline| stretch
}
```

一共有5个值：

1. **flex start:**: 交叉轴的起点对齐

   ```css
   .container {
     width: 100%;
     height: 400px;
     background: #54b4c557;
     display: flex;
     align-items: flex-start; // 交叉轴的起点对齐
     
   }
   .item {
     background: #f19f9e;
     width: 100px;
     height: 50px;
     text-align: center;
     line-height: 50px;
     margin: 2px;
   }
   ```

   {% asset_img flex1.jpg %}

   完整 [Demo](https://jsbin.com/nihavapeve/1/edit?html,css,js,output)

2. **flex end:** 交叉轴的终对齐

   ```css
   .container {
     width: 500;
     height: 400px;
     background: #54b4c557;
     display: flex;
     align-items: flex-end; // 交叉轴的终点对齐 
     
   }
   .item {
     background: #f19f9e;
     width: 30px;
     height: 30px;
     text-align: center;
     line-height: 30px;
     margin: 2px;
   }
   ```

   {% asset_img flex2.jpg %}

   完整 [Demo](https://jsbin.com/nuxeqobude/1/edit?html,css,output)

3. **center：**居中

   ```css
   .container {
     width: 100%;
     height: 400px;
     background: #54b4c557;
     display: flex;
     align-items: center; // 居中 
     
   }
   .item {
     background: #f19f9e;
     width: 30px;
     height: 30px;
     text-align: center;
     line-height: 30px;
     margin: 2px;
   }
   ```

   {% asset_img flex3.jpg %}

   完整 [Demo](https://jsbin.com/sikuxofozu/edit?html,css,output)

4. **baseline**： 项目的第一行文字基线对齐

   ```css
   .container {
     width: 100%;
     height: 100px;
     background: #54b4c557;
     display: flex;
     align-items: baseline 
     
   }
   .item1 {
     background: #f19f9e;
     width: 30px;
     text-align: center;
     line-height: 50px;
      margin: 0 20px;
   }
   .item2 {
     width: 30px;
     font-size: 50px;
     background: #f19f9e;
     text-align: center;
     line-height: 50px;
     margin: 0 20px;
   }
   .item3 {
     width: 30px;
     background: #f19f9e;
     text-align: center;
     line-height: 40px;
      margin: 0 20px;
   }
   ```

   {% asset_img flex4.jpg %}

   完整  [Demo](https://jsbin.com/xinosalodo/1/edit?html,css,output)

5. **stretch：** 默认值， 如果项目未设置高度或者设置高度为auto，将铺满整个容器的高度

   ```
   .container {
     width: 100%;
     height: 100px;
     background: #54b4c557;
     display: flex;
     align-items: strecth; //  默认值， 如果项目未设置高度或者设置高度为auto，将铺满整个容器的高度
     
   }
   .item1 {
     background: #f19f9e;
     width: 30px;
     text-align: center;
     line-height: 50px;
      margin: 0 20px;
   }
   .item2 {
     width: 30px;
     font-size: 50px;
     background: #f19f9e;
     text-align: center;
     line-height: 50px;
     margin: 0 20px;
   }
   .item3 {
     width: 30px;
     background: #f19f9e;
     text-align: center;
     line-height: 40px;
     margin: 0 20px;
   }
   ```

   {% asset_img flex5.jpg %}

​       完整  [Demo](https://jsbin.com/vuqetirola/1/edit?html,css)

### align-content 属性

`align-content`属性设置了多根轴线的对齐方式，如果项目只有一根轴线，那么该属性是不生效的。

```
.box {
	align-content: flex-start | flex-end | center | space-between | space-around | stretch 
}
```

该属性一共有6 个属性值：

1. **flex start:**: 交叉轴的起点对齐
2. **flex end:** 交叉轴的终对齐
3. **center：**居中
4. **stretch：** 默认值， 如果项目未设置高度或者设置高度为auto，将铺满整个容器的高度
5. **space-between**:与交叉轴两端对齐，轴线之间的间隔平均分布
6. **space-around**：每根轴线两侧的间隔都相等。所以，轴线之间的间隔比轴线与边框的间隔大一倍
7. **stretch：**默认值，占满整个容器

**注意：** `align-items` 属性和`align-content`属性的区别：`align-items`属性是设置多根轴线和单根轴线的， `align-content`属性是设置多根轴线的。这里的多根轴线和单根轴线是指`flex-wrap：nowrap`是单根轴线，align-items 和align-content同时存在的话， align-items 生效，align-content 不生效。`flex-wrap：wrap| wrap-reverse` 且项目换行了，align-items 和align-content同时存在的话， align-items 不生效，align-content 生效。

例子如下：

```css
.container {
  width: 100%;
  height: 400px;
  background: yellow;
  display: flex;
  flex-wrap: wrap;
  align-items: flex-end; 
  align-content: flex-start;
  // align-content单根轴线是不生效的，当 flex-wrap: nowrap, 生效的是align-items
  // 多轴情况下， align-items 和 align-content 同时存在， align-content会覆盖align-items 
}
.item {
  background: white;
  width: 100px;
  height: 50px;
  text-align: center;
  line-height: 50px;
  margin: 2px;
}
```

完整的 [Demo](https://jsbin.com/sixesiquve/1/edit?html,css,output)

## 项目的属性

以下6个属性设置在项目上。

1. **`order`**

2. **`flex-grow`**

3. **`flex-shrink`**

4. **`flex-basis`**

5. **`flex`**

6. **`align-self`**

   

### order 属性

`order` 属性定义项目的排列顺序，数值越小，排列越靠前， 默认值为0。

**使用语法：**

```css
.item {
  order: <integer>; /* default 0 */
}
```

**例子：**

```css
.container {
  width: 100%;
  height: 40px;
  background: #54b4c557;
  display: flex;
  padding: 5px 0;
}
.item {
  background: #f19f9e;
  width: 30px;
  height: 30px;
  text-align: center;
  line-height: 30px;
  margin: 2px;
}
.item3 {
  /* order 属性定义项目的排列顺序，数值越小，排列越靠前 */
  order: -1; 
}
```

{% asset_img order.jpg %}

 [例子完整代码](https://jsbin.com/sazuwamure/1/edit?html,css,output)

### flex-grow 属性

`flex-grow`属性定义项目的放大比列。默认为0，即 即使存在剩余空间， 也不放大。 

**使用语法：**

```css
.item {
  flex-grow: <number>; /* default 0 */
}
```

**例子：**

```css
.container {
  width: 100%;
  height: 40px;
  background: #54b4c557;
  display: flex;
  border-bottom: 1px dashed #000;
  padding: 5px 0;
}


.item {
  background: #f19f9e;
  width: 30px;
  height: 30px;
  text-align: center;
  line-height: 30px;
  margin: 2px;
  flex-grow: 1;
}
.item3 {
  flex-grow: 4;
}
```

{% asset_img grow.jpg %}

 [例子完整代码](https://jsbin.com/zafuhamuwo/1/edit?html,css,output)

如果所有项目的`flex-grow`属性都为1，则它们将等分剩余空间（如果有的话）。如果一个项目的`flex-grow`属性为4，其他项目都为1，则前者占据的剩余空间将比其他多4倍。

### flex-shrink 属性

`flex-shrink`属性定义项目的缩小比例，默认值为1， 如果剩余空间不足，项目将缩小。

**使用语法：**

```css
.item { 
  flex-shrink: <number>; /* default 1 */
}
```

**例：子**

```css
.container {
  width: 100%;
  height: 40px;
  background: #54b4c557;
  display: flex;
  padding: 5px 0;
}


.item {
  background: #f19f9e;
  width: 300px;
  height: 30px;
  text-align: center;
  line-height: 30px;
  margin: 2px;
  flex-shrink: 1;
}
.item3 {
  flex-shrink: 10;
}
```

{% asset_img shrink.jpg %}

如果所有项目的`flex-shrink`属性都为1，当空间不足时，都将等比例缩小。

如果所有项目的`flex-shrink`属性都为1，其他的`flex-shrink` 为10，那么后者的缩小比例是前者的10倍

如果一个项目的`flex-shrink`属性为0，其他项目都为1，则空间不足时，前者不缩小。

负值对该属性无效。

 [例子完整代码](https://jsbin.com/gihubasolo/2/edit?html,css,output)

### flex-basis 属性

`flex-basis`属性定义在分配剩余空间之前， 项目占主轴的空间。浏览器根据这个属性，计算主轴是否有多余空间。它的默认值为`auto`，即项目的本来大小

**使用语法：**

```css
.item {
  flex-basis: <length> | auto; /* default auto */
}
```

**例子：**

```css
.container {
  width: 800px;
  height: 40px;
  background: #54b4c557;
  display: flex;
  padding: 5px 0;
}


.item {
  background: #f19f9e;
  width: 300px;
  height: 30px;
  text-align: center;
  line-height: 30px;
  margin: 2px;
}
.item3 {
  flex-basis: 700px;
}
```

{% asset_img basis.jpg %}

 [例子完整代码](https://jsbin.com/bepehogude/1/edit?html,css,output)

### flex属性

`flex`属性是`flex-grow` `flex-shrink` `flex-basis	`的简写，默认值为`0 1 auto`。后两个属性可选。

**使用语法：**

```css
.item {
  flex: none | [ <'flex-grow'> <'flex-shrink'>? || <'flex-basis'> ]
}
```

该属性有两个快捷值：`auto` (`1 1 auto`) 和 none (`0 0 auto`)。

建议优先使用这个属性，而不是单独写三个分离的属性，因为浏览器会推算相关值。

### align-self 属性

`align-self`属性允许单个项目与其他项目不一样的对齐方式，可覆盖`align-items`，默认值为auto，表示继承父级的`align-items`。如果没有父元素，则等同于`stretch`。

该属性可能取6个值，除了auto，其他都与align-items属性完全一致。

**使用语法：**

```css
.item {
  align-self: auto | flex-start | flex-end | center | baseline | stretch;
}
```

**例子:**

```css
.container {
  width: 750px;
  height: 100px;
  background: #54b4c557;
  display: flex;
  padding: 5px 0;
  align-items: flex-end;
}

.item {
  background: #f19f9e;
  width: 300px;
  height: 30px;
  text-align: center;
  line-height: 30px;
  margin: 2px;
}
.item3 {
  align-self: flex-start
}
```

{% asset_img self.jpg %}

 [例子完整代码](https://jsbin.com/luzusoceci/edit?html,css,output)

