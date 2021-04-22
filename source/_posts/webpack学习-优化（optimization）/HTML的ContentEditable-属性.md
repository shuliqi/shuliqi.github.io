---
title: 有趣的HTML属性：contenteditable
date: 2021-04-12 09:24:54
tags:
---

最近在做一个需求, 需求中有个这样效果:**有一个输入框，输入框有`placeholder`提示音输入框的宽度随着内容的长度自适应，并且输入框不可以换行显示，键盘回车就相当于确认值**。刚开始我接到这样的效果，想这还不简单，`input`不就可以实现吗？但是到我真的进入到开发的时候， 才发现不是很对劲， `input` 好像实现不了。 那么具体哪里实现不了？ 那么解决的办法是什么呢？有没有什么可以替代的办法呢？我们来一一看一下具体过程。

<!--more-->

# input 实现

由于项目是`VUE`技术栈，那么久直接使用`VUE`的方式举例子了。

首先我们来看`input` 是否能实现：

<iframe height="411" style="width: 100%;" scrolling="no" title="input自适应宽" src="https://codepen.io/shuliqi/embed/bGgpmrN?height=411&theme-id=dark&default-tab=js,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/bGgpmrN'>input自适应宽</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>
我们页面正常的放入`input`标签，没有给它设置一个固定的宽度值，那么它就会有默认不变的宽度，想让它动态宽度自适应是不可能的。或者我们给它设置一个宽度，那么`input`的宽度也就固定了，也不能动态自适应内容的宽度了。

那要是非得要用`input `标签来实现的话， 能不能实现呢？ 答案是能的， 代码那么神奇，看你自己的思路。 我这里提供一种思路吧!

既然`input`标签是有默认宽度的，所以`css`的宽度就不能不写。我们就直接设置它的宽度为100%，让它跟这父元素的宽度来改变。父元素的宽度我们可以使用`span`标签撑起来。`span`标签的内容与`input`的内容是一样，然后`input`标签绝对定位，把`span`盖起来。 大体就是这么个思路。我们来看看例子：

<iframe height="525" style="width: 100%;" scrolling="no" title="input 标签自适应宽度" src="https://codepen.io/shuliqi/embed/wvgGYbG?height=525&theme-id=dark&default-tab=js,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/wvgGYbG'>input 标签自适应宽度</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

这样就完美实现了。当然也有很多其他的思路， 看自己有么有想出更好的实现方案吧。

# contenteditable  属性实现

当然除了上面的`input`方式实现。也还有其他的方式可以实现，这就是我们这篇文章的重点：`HTML`的`ContentEditable` 属性。



`contenteditable` 属性是`HTML5`的新属性。规定元素的内容是否可编辑。它的属性值有两种：

- **true**：规定元素的内容是可编辑的
- **false**：规定元素的内容是不可编辑的

看到这里，大家都会想到`textarea`。但是`contenteditable`属性与`textarea` 是不一样的。

- ``textarea``支持多行文本的输入，满足我们很多编辑的需求。但是`textarea`有一个致命的去诶单，那就是不能像`div`类似这样的标签宽度，高度自适应。
- ``textarea`` 只支持文本的输入。但是现在的很多需求需要在编辑区假如图片，链接，视频等。``textarea`` 是做不到的， 但是标签上设置`contenteditable`为`true`的话。是可以实现的。

那我们现在使用 `contenteditable`属性来实现我们上面说的需求：**有一个输入框，输入框有placeholder提示音，输入框的宽度随着内容的长度自适应，并且输入框不可以换行显示，键盘回车就相当于确认值**;

具体的实现方式：

<iframe height="676" style="width: 100%;" scrolling="no" title="qBRqbaz" src="https://codepen.io/shuliqi/embed/qBRqbaz?height=676&theme-id=dark&default-tab=js,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/qBRqbaz'>qBRqbaz</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

下面需要注意的点，我们一一来讲解

### placeholder提示语

`input` 和 `textarea`能很轻松的实现`placeholder`提示语的效果，但是`contentediable`的元素`placeholder`不起作用。但是可以通过`CSS`的`:empty`解决：

**html：**

```html
  <span class="input" placeholder="请输入标签"></span>
```

**css:**

```css
/* placeholder的设置 */
.input:empty::before {
  content: attr(placeholder);
  color: #ccc;
}
```

> `:empty` [CSS](https://developer.mozilla.org/en-US/docs/Web/CSS) [伪类](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Pseudo-classes) 代表没有子元素的元素。子元素只可以是元素节点或文本（包括空格）。注释或处理指令都不会产生影响。

### 自适应宽度

我们对`span`标签设置即可，`span`标签的宽度是内容的宽度撑开的。

但是这里一点需要注意：需要把 `span` 设置为`display: inline-block`。否则标签得到焦点将无法看见光标。

```css
  .input {
    border: 1px solid #ccc;
    font-size: 16px;
    padding: 5px;
    
    /* 重要： 不设置这个光标看不见 */
    display: inline-block;
  }
```

为啥设置成 `inline-block` 就可以看见了呢？

> **块级元素**：是指当它们显示在浏览器中时，会在自身前后各插入一个空行，而使自身在页面中占据一个相对独立的块状区域的元素 
>
> **行内元素：** 默认是没有宽度的
>
> **答案：**因为行内元素是没有宽度的，光标需要1px 的宽度，所以需要设置成`inline-block`或者 `block`就可以显示光标了



### 内容不可以换行显示

我们在标签的 `keydown`事件的时候，判断当前是否是换行 `enter`键，如果是，那么我们直接阻止浏览器的换行操作。

```javascript
  if (e.keyCode === 13) {
    // 阻止换行
    e.cancelBubble = true;
    e.preventDefault();
    e.stopPropagation();
  }
```

> keyCode:13 就是键盘的 enter 键

### 获取内容

可以通过`innerHTML`，`innerText`、`textContent` 获取输入框的内容，介绍一下区别：

- [innerHTML](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/innerHTML) 返回标签之间的内容，包括标签元素和文本信息，基本上所有浏览器都支持。
- `innerText/textContent`：标签之间的纯文本信息，会将标签过滤掉。

`innerText`和`textContent`虽然都是获取标签的内容，但是两者也是存在差异的。具体可看 [innerText和textContent的区别](https://juejin.cn/post/6844903657935208456)

 我们这个例子使用的`innerText`：

```javascript
// 取值
const newValue = e.target.innerText;
```

有些文章说光标在火狐浏览器会有异常，但是看了，其实没问题，应该是浏览器修复bug了吧

# contenteditable 兼容性问题

{% asset_img 1.png %}





参考文章

[contenteditable 踩坑记](https://wuxinhua.com/2018/07/05/Contenteditable-The-Good-Part-And-The-Ugly/)

[contenteditable属性](https://www.jianshu.com/p/622d9c6e3fdf)





















