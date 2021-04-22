---
title: Debounce和Throttle的原理及实现
date: 2018-04-16 18:55:30
tags: JavaScript
categories: 学习笔记
---

# 问题引出

在处理诸如`resize`,`scroll`,`mousemove` 和 `keydown`, `keyup`,`keypress`等事件的时候， 我们通常不希望这些事件太过频繁的触发。尤其是监听程序中涉及到大量的计算或者是非常耗资源的操作。

 <!--more-->

有多频繁呢？ 我们以`mousemove`为例，根据 [DOM Level 3](https://www.w3.org/TR/DOM-Level-3-Events/#event-type-mousemove) 的规定,。

> A [user agent](https://www.w3.org/TR/DOM-Level-3-Events/#user-agent) MUST dispatch this event when a pointing device is moved while it is over an element. The frequency rate of events while the pointing device is moved is implementation-, device-, and platform-specific, but multiple consecutive [`mousemove`](https://www.w3.org/TR/DOM-Level-3-Events/#mousemove) events SHOULD be fired for sustained pointer-device movement, rather than a single event for each instance of mouse movement. Implementations are encouraged to determine the optimal frequency rate to balance responsiveness with performance.

大概的意思就是：**如果鼠标连续的移动，那么浏览器就应该触发多个连续的`mousemove`事件。**

这就说明了浏览器在其内部计时器允许的情况下，根据用户鼠标的速度来触发`mousemove`事件。(当然了， 如果移动鼠标足够快，比如“刷”的一下扫过去，浏览器是不会触发的这个事件的），`resize`,`scroll` 和`key*`等事件于此类似。

具体的可以看例子体会下 [鼠标滑动 ](https://codepen.io/shuliqi/pen/NWGNXWv?editors=1010)

# Debounce

 DOM事件里的`debounce`概念其实是从机械开关和继电器的“去弹跳(debounce)”衍生出来的，基本的思路就是把多个信号的合并为一个信号。

在Javascript中， `debounce`函数所做的事情就是：强制某一个函数在某个连续的时间段内只执行一次，哪怕它本来会被调用很多次。即我们希望用户在停止某个操作一段时间之后才执行相应的舰艇函数，而不是在用户操作的过程中，浏览器触发多少次事件，就执行多少次舰艇函数。

比如，在某个5s的时间段内连续移动了鼠标，浏览器就可能会触发几十个（甚至几百个）`mousemove` 事件， 不使用`debounce`的话，监听函数就要执行这么多次；如果对监听函数使用1000ms的“去弹跳”。 那么浏览器就只会**执行一次**这个监听函数， 而且是在第6s的时候执行的。

那如何实现一个`ddebunce` 函数呢？

### 实现

我们`debunce`函数接受三个参数， 第一个参数是要“去弹跳”的回调函数**func**, 第二个参数是延迟的时间**wait**，第三个参数**immediate**表示在wait 这个时间区间内做开始执行(**immediate = true**)还是最后执行(**immediate = false**)。

```javascript
/**
* debounce函数， 返回函数连续调用时， 空闲时间必须大于或者等于wait, func 才会执行
* @param {function} func 回调函数
* @param {number} wait 回调函数延迟调用的时间
* param {booleam} 是否为立即调用
* @return {function} 返回客户调用函数
*/
function debounce(func, wait, immediate = true) {
  let timer, args;
  
  // 延迟执行函数
  const later = () => setTimeout(() => {
    // 延迟执行函数执行完毕，  需要清除定时器序号
    timer = null;
    if (!immediate) {
      func.apply(context, args);
      context = args = null;
    }
  }, wait);
  return function() {
    // 首次进入， 没有延迟函数，创建一个
    if (!timer) {
      timer = later();
      if (immediate) {
        // 如果立即执行， 则调用函数
        func.apply(this, arguments);
      } else {
        // 缓存调用函数时的上下文和参数
        context = this;
        args = arguments;
      }
    } else {
      // 如果已经存在延迟函数，清除，再重新设定
      clearTimeout(timer);
      timer = later()
    }
  }
}
```

### 原理

其实原理很简单，`debounce`返回了一个闭包， 这个闭包依然会被连续的频繁的调用。但是在闭包的内部， 却限制了原始函数**func**的执行， 强制**func**只能在连续操作停止后只执行一次。

### 调用及例子

调用方式如下:

```javascript
wwindow.addEventListener('resize', debounce(handle, 1000, false))
```

**[一个小小的keydown例子](https://codepen.io/shuliqi/pen/vYNGeZR)**

# Throttle 

`throttle`理解起来更容易，就是`固定函数执行的速率` 即所谓的`节流`。举个例子，正常的情况下，`mousemove`的监听函数可能会20ms(假设)执行一， 如果设置200ms的”节流“。那么它就会**每200ms**执行一次。 比如在1s的时间段内，正常的监听函数可能会执行50（1000/20）次。”节流"的就会执行5（1000/200）次。

我们来看个例子：**[例子](https://codepen.io/shuliqi/pen/WNQGQbV)**无论我鼠标移动的有多快， 我的count 都是匀速（每隔1s）增加。

### 实现

与`debounce`类似， 我们`throttle`	也接收`func`（一个实际要执行的函数），`wait`一个执行时间间隔值。

```javascript
/**
*
* @param func {Function}   实际要执行的函数
* @param wait {Number}  执行间隔，单位是毫秒（ms）
*
* @return {Function}     返回一个“节流”函数
*/
function throttle(func, wait) {
  let context, args;
  // 设置前一个函数调用的时间戳
  let previous = 0;
  return function() {
    let now = new Date().getTime();
    if (!previous) {
      // 首次进入
      previous = now;
     
    } else {
      context = this;
      args = arguments;
      let remaining = wait - (now - previous);
      if (remaining <= 0) {
        func.apply(context, args);
        previous = now;
        context = args = null;
      }
    }
  }
}
```

### 调用及例子

```javascript
window.addEventListener('resize', throttle(handle, 1000));
```

`throttle`常用的场景是限制`resize`和`scroll`的触发频率。我们以scroll 为例子**[scroll例子](https://codepen.io/shuliqi/pen/NWGRGQr) [resize例子](https://codepen.io/shuliqi/pen/ExVKwEW) ** 

 

# 可视化解释

如果还是不能完全体会`debounce`和`throttle`的差异， 可以看这个例子 **[可视化例子](http://demo.nimius.net/debounce_throttle/)**

# 总结

`debounce`强制函数在某段时间内只执行一次，`throttle`强制函数以固定的速率执行，在处理一些高频率触发的DOM 事件的时候， 它们都能极大提高用户体验。