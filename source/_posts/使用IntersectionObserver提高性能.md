---
title: 使用IntersectionObserver提升性能
date: 2020-03-26 19:08:51
tags: JavaScript
---

## 问题引出？

之前， 我们要做**懒加载**  或者 **无限加载**的时候。通常是这么做的：

1. **懒加载**

    <!--more-->

   图示：

   {% asset_img lazyLoad.jpg %}

   主要代码：

   ```javascript
   var imgs = document.querySelectorAll('img');
   window.onscroll = function(){
     // 浏览器滚动过的高度
     var scrollTop = document.body.scrollTop || document.documentElement.scrollTop;
     // 可视区域的高度
     var winTop = window.innerHeight; 
     for(var i=0;i < imgs.length;i++){
       // 当图片距离页面顶部的距离 < 浏览器滚动过的高度 +  可视区域的高度
       if(imgs[i].offsetTop < scrollTop + winTop ){
         imgs[i].src = imgs[i].getAttribute('data-src');
       } 
   }
   ```

2. **无限滚动**

   图例：

   {% asset_img wuxian.jpg %}

   主要代码：

   ```javascript
   var page=1; //当前页的页码
   var flagNoData = false; //false
   function showAjax(page){
     // ...请求数据啥啥的
   }
   function scrollFn(){
     //真实内容的高度
     var pageHeight = Math.max(document.body.scrollHeight,document.body.offsetHeight);
   
     //视窗的高度
     var viewportHeight = window.innerHeight 
     	                 || document.documentElement.clientHeight
     	                 || document.body.clientHeight
     	                 || 0;
   
     //隐藏的高度
     var scrollHeight = window.pageYOffset 
                       || document.documentElement.scrollTop 
                       || document.body.scrollTop 
                       || 0;
   
        if(falgNoData){ //数据全部加载完了
          return;
        }
   
       else if(pageHeight - viewportHeight - scrollHeight < 10){ //如果满足触发条件，执行    
         showAjax(page);
     }
   }
   $(window).bind("scroll",scrollFn);    //绑定滚动事件
   ```

   

传统的实现方法是，监听到`scroll`事件后，获取相关元素的坐标来进行判断。这种方法是有缺点的。由于`scroll`事件密集发生，计算量很大，容易造成性能。

那么在这样的背景下， 我们有没有更好的办法呢？

## 关于IntersectionObserver

`IntersectionObserver`的出现解决了这个问题。

MDN上给的官方概念：

> `IntersectionObserver`接口 (Intersection Observer API)为开发者提供了一种可以异步监听目标元素与其祖先或者视窗（viewport）交叉状态的手段。祖先元素与视窗(viewport)被称为根(root)。

这概念的重点就是：**监听目标元素与其祖先或视窗交叉状态发生改变的手段** 

图解如下图：

{% asset_img 2.jpg %}

**目标元素与root元素刚开始交叉**和**目标元素与root元素刚开始不交叉**都能检测到。

**看看小🌰：** 

<iframe height="598" style="width: 100%;" scrolling="no" title="wvKBBWb" src="https://codepen.io/shuliqi/embed/wvKBBWb?height=598&theme-id=dark&default-tab=js,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/wvKBBWb'>wvKBBWb</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>



##  IntersectionObserver 如何解决？

IntersectionObserver API 是异步的， 不随着目标元素的滚动同步触发。即只有在线程空闲下来才会执行观察器。这意味着这个观察器的优先级非常的低，只有在其他的任务执行完，浏览器空闲了才会执行。

## IntersectionObserver API

这个API的调用非常的简单：

```javascript
var io = new IntersectionObserver(callback, options)
```

`IntersectionObserver`支持两个参数：

1. `callback` 是当被监听元素的可见性变化时，触发的回调函数
2. `options`是一个配置参数对象，可选的， 有默认的属性值

构造函数的返回值是一个观察实例， 实例的`observe`方法可以指定观察哪个DOM节点。

```javascript
//  对元素target添加监听，当target元素变化时，就会触发回调
io.observe(document.getElementById('shuliqi'));

// 移除一个监听，移除之后，target元素的交叉状态变化，将不再触发回调函数
io.unobserve(element)

// 停止所有的监听
io.disconnect();
```

上面的`observe()`的参数是一个DOM节点对象，如果要观察多个节点，就要多次调用这个方法。

```javascript
io.observe(eleA);
io.observe(eleB);
```

## callback 参数

目标元素的交叉状态发生改变时，就会调用观察器的回调函数`callback`。

`callback`一般会调用两次。一次是目标元素刚刚进入root元素（开始交叉）, 另一次是完全离开root（开始不相交）。

```javascript
var io = new IntersectionObserver((entries) => {
	console.log(entries);
})
io.observe($0)
```

以上的代码， 在chrome控制台进行调试，这里的$0代表我审查元素选中的节点。

运行的结果如下：

{% asset_img 3.jpg %}

由图我们可知callback函数有个参数，它是`IntersectionObserverEntry`对象数组，举例来说，如果同时有两个被观察的对象的可见性发生变化， 那么entries数组就会有两个成员。

接下来我们重点讲`IntersectionObserverEntry`



## IntersectionObserverEntry对象

`IntersectionObserverEntry`对象提供目标元素的信息，

还是以上的例子：

{% asset_img 4.jpg %}

一共有8 个属性：

```json
{
  time: 78463997.025,
  rootBounds: null,
  boundingClientRect: DOMRectReadOnly {
   // ...
  },
  intersectionRect: DOMRectReadOnly{
   // ...
  },
  isIntersecting: true,
  intersectionRatio: 1,
  target: html,
  isVisible: false,
}
```

每个属性的含义如下：

1. **time：** 

   返回一个记录从`IntersectionObserver`开始实例化的时间到交叉状态发生改变的时间的时间戳对比时间：实例化的时间。例子：值为1000时，表示在IntersectionObserver实例化的1秒钟之后，目标元素的交叉状态发生改变了

2. **rootBounds：** 根元素的矩形区域的信息，`getBoundingClientRect()`方法的返回值，如果没有根元素（即直接相对于视口滚动），则返回`null`

3. **boundingClientRect：**  目标元素的矩形信息

4. **isIntersecting：**目标元素当前是否可见 Boolean值 可见为true

5. **intersectionRect：** 目标元素与视口（或root根元素）的交叉区域的信息

6. **intersectionRatio：** 目标元素的可见比例，即`intersectionRect`占`boundingClientRect`的比例，完全交叉时为`1`，完全不交叉时小于等于`0`

7. **target：** 被观察的目标元素，是一个 DOM 节点对象

    

**注意：**在Chrome 78版本中会返回`isVisible`属性，但是不知道是不是Bug，无论元素是否可见，都为`false`，但是`isTntersecting`的表现是正常的，所以判断是否可见，可以根据`intersectionRatio`或者`isTntersecting`来进行判断。



上面的矩形信息的关系如下：

{% asset_img 5.jpg %}



## options参数

`IntersectionObserver`构造函数的第二参数是一个配置对象， 他可以设置以下属性：

1. **threshold属性**

   `threshold`属性 决定了什么时候触发回调函数，它是一个数组， 每一个成员也是一个门槛值，当目标元素和根元素相交的面积占目标元素面积的百分比到达或跨过某些指定的临界值时就会触发回调函数

   `threshold`的默认值是`:[0]`，即只有在开始进入，或者是完全离开视图区域时，才会触发。

   ```javascript
   var io = new IntersectionObserver(callback, {
   	threshold: [0, 0.5, 1],
   })
   io.observe($0);
   ```

   用户可以自定义这个属性， [0, 0.5, 1]就表示 0%， 50%，75%， 100%交叉状态发生改变时， 就会触发回调函数。

      **看看小🌰：  **

   <iframe height="423" style="width: 100%;" scrolling="no" title="dyYoJJJ" src="https://codepen.io/shuliqi/embed/dyYoJJJ?height=423&theme-id=dark&default-tab=js,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
     See the Pen <a href='https://codepen.io/shuliqi/pen/dyYoJJJ'>dyYoJJJ</a> by shuliqi
     (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
   </iframe>

2. **rootMargin**属性

   用来扩大或者缩小视窗的大小， 使用css的定义方式，	`10px 10px 10px 20px` 表示top，right,bottom, left的值。

   ```javascript
   const options = {
       threshold: [0, 0.5, 1],
       rootMargin: '30px 20px 30px 20px'
   }
   ```

   如图：

   {% asset_img 6.jpg %}

   图上的绿色部分是定义好的root元素， 我们添加了`rootMargin`属性， 将视窗增大了。

   由此可见，root元素只有在`rootMargin`为空的时候才是绝对的视窗。

    **看看小🌰：**

   <iframe height="628" style="width: 100%;" scrolling="no" title="PoGWJVv" src="https://codepen.io/shuliqi/embed/PoGWJVv?height=628&theme-id=dark&default-tab=js,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
     See the Pen <a href='https://codepen.io/shuliqi/pen/PoGWJVv'>PoGWJVv</a> by shuliqi
     (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
   </iframe>

3. **root **属性

   `root`属性指定目标元素所在的容器节点（即根元素）。注意，容器元素必须是目标元素的祖先节点。

   

## 应用

1. **懒加载（lazy load）**

   我们希望某些静态资源（比如图片），只有用户向下滚动，它们进入视口时才加载，这样可以节省带宽，提高网页性能。这就叫做"惰性加载"。

   有了 IntersectionObserver API，实现起来就很容易了。

   ```javascript
   const io = new IntersectionObserver(callback);
       let imgs = document.querySelectorAll("[data-src]"); // 将图片的真实url设置为data-src src属性为占位图 元素可见时候替换src
       function callback(entries) {
         entries.forEach((item) => {
           // 遍历entries数组
           if (item.isIntersecting) {
             // // 当前元素可见
             item.target.src = item.target.dataset.src; // 替换src
             io.unobserve(item.target); // 停止观察当前元素 避免不可见时候再次调用callback函数
           }
         });
       }
   
       // io.observe接受一个DOM元素，添加多个监听 使用forEach
       imgs.forEach((item) => {
         io.observe(item);
       });
   ```

   上面代码中，只有目标区域可见时，才会将模板内容插入真实 DOM，从而引发静态资源的加载。

   **看看小🌰： [小小例子](https://codepen.io/shuliqi/pen/gOababR)**

   <iframe height="570" style="width: 100%;" scrolling="no" title="gOababR" src="https://codepen.io/shuliqi/embed/gOababR?height=570&theme-id=dark&default-tab=js,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
     See the Pen <a href='https://codepen.io/shuliqi/pen/gOababR'>gOababR</a> by shuliqi
     (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
   </iframe>

2. **无限加载**

   ```javascript
   document.addEventListener("DOMContentLoaded", function () {
     var sum = 1;
     var loadData = function () {
       var fragment = document.createDocumentFragment();
       for (var i = 0; i < 10; i++) {
         var div = document.createElement("div");
         div.className = "unit";
         div.innerText = `第 ${sum} 个数据`;
         fragment.appendChild(div);
         sum++;
       }
       document
         .getElementById("app")
      .insertBefore(fragment, document.getElementById("loading"));
     };
     var io = new IntersectionObserver(function (entries) {
       if (entries[0].isIntersecting) {
         // 如果loading元素不可见，就加载数据
         loadData();
       }
     });
     io.observe(document.getElementById("loading"));
   });
   ```
   
   无限滚动时，最好在页面底部有一个页尾栏（又称[sentinels](http://www.ruanyifeng.com/blog/2016/11/sentinels)）。一旦页尾栏可见，就表示用户到达了页面底部，从而加载新的条目放在页尾栏前面。这样做的好处是，不需要再一次调用`observe()`方法，现有的`IntersectionObserver`可以保持使用。
   
     **看看小🌰： [小小例子](https://codepen.io/shuliqi/pen/KKdwdNb)**
   
   <iframe height="608" style="width: 100%;" scrolling="no" title="KKdwdNb" src="https://codepen.io/shuliqi/embed/KKdwdNb?height=608&theme-id=dark&default-tab=js,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
     See the Pen <a href='https://codepen.io/shuliqi/pen/KKdwdNb'>KKdwdNb</a> by shuliqi
     (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
   </iframe>
   
   

## 疑问的点

1. **一次性到达或跨过的多个临界值中选一个最近的**

   **问题：**如果一个观察者实例设置了 11 个临界值：[0, 0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 1]，那么当目标元素和根元素从完全不相交状态滚动到相交率为 1 这一段时间里，回调函数会触发几次？

   **答案：** 不确定的。

   如果滚动速度足够慢，每次相交率到达下一个临界值的时间点都发生在了不同的帧里（浏览器至少绘制了 11 次），那么就会有 11 次相交被检测到，回调函数就会被执行 11 次

   如果滚动速度足够快，从不相交到完全相交是发生在同一个帧里的，浏览器只绘制了一次，浏览器虽然知道这一次滚动操作就满足了 11 个指定的临界值（从不相交到 0，从 0 到 0.1，从 0.1 到 0.2 ··· ），但它只会考虑最近的那个临界值，那就是 1，回调函数只触发一次.

    **看看小🌰： [例子](https://codepen.io/shuliqi/pen/zYvGaLL)**

   <p class="codepen" data-height="568" data-theme-id="dark" data-default-tab="js,result" data-user="shuliqi" data-slug-hash="zYvGaLL" style="height: 568px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="zYvGaLL">
     <span>See the Pen <a href="https://codepen.io/shuliqi/pen/zYvGaLL">
     zYvGaLL</a> by shuliqi (<a href="https://codepen.io/shuliqi">@shuliqi</a>)
     on <a href="https://codepen.io">CodePen</a>.</span>
   </p>
   <script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

2. **如何判断当前是否相交？**

   **问题：** 前面的几个例子， 都使用了isIntersecting 来判断目标元素是否在窗口里面，为什么？难道用entry.intersectionRatio > 0 判断不可以吗？

   如果你滚动页面速度很慢，当目标元素的顶部和视口底部刚好挨上时，浏览器检测到相交了，回调函数触发了，但这时 entry.intersectionRatio 等于 0，会进入 else 分支，继续向下滚，回调函数再不会触发了，提示文字一直停留在不可见状态；但如果你滚动速度很快，当浏览器检测到相交时，已经越过了 0 那个临界值，存在了实际的相交面积，entry.intersectionRatio > 0 也就为 true 了。所以这样写会导致代码执行不稳定，不可行。

   **看看小🌰： [例子](https://codepen.io/shuliqi/pen/gOapKZP)**

   <iframe height="444" style="width: 100%;" scrolling="no" title="gOapKZP" src="https://codepen.io/shuliqi/embed/gOapKZP?height=444&theme-id=dark&default-tab=js,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
     See the Pen <a href='https://codepen.io/shuliqi/pen/gOapKZP'>gOapKZP</a> by shuliqi
     (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
   </iframe>

3. **贴边的情况是特例**

   当目标元素从距离根元素很远到和根元素贴边，这时也会触发回调（假如 thresholds 里有 0），但这和工作原理相矛盾啊，离的很远相交率是 0，就算贴边，相交率还是 0，值并没有变，不应该触发回调啊。的确，这和基本工作原理矛盾，但这种情况是特例，目标元素从根元素外部很远的地方移动到和根元素贴边，也会当做是满足了临界值 0，即便 0 等于 0。

   还有一个反过来的特例，就是目标元素从根元素内部的某个地方（相交率已经是 1）移动到和根元素贴边（还是 1），也会触发回调（假如 thresholds 里有 1）



## 总结

在当前判断可视性的方法，基本就是监听`scroll`事件，但是由于其高频的计算频率，会导致浏览器性能的损失，尤其是，如果一个同一个页面中，有多个地方，需要这样的判断，那么就需要绑定多个`scroll`事件，或者有多个计时器在轮询的话，那么对性能的损失就更为客观了。

虽然现在的浏览器性能一直在增强，但是也有更多的消耗性能的比较炫的技术在产生，它们依然在占据着浏览器的大量的计算内存，所以，尽量在可以节省性能的时候，就节省一下性能吧。

而该方法给我们提供了一个更简单直接，性能更好的解决方案，希望以后的浏览器，可以越来越广泛的支持吧。

最后， 毕竟是一个新兴的`API`，所以浏览器的支持性并不好，这里可以看看当前浏览器对于`IntersectionObserver`的支持性：

{% asset_img caniuse.png %}



## 最后

这篇文章的ppt也有哦， [请点击这里](https://shuliqi.github.io/ppt/IntersectionObserver/%E5%85%B3%E4%BA%8EIntersectionObserver.html)

