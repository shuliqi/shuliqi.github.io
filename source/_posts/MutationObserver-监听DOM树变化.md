---
title: MutationObserver 追踪DOM的变化
date: 2020-04-21 18:47:51
tags: JavaScript
categories: JavaScript
---

`Mutation observer`（变动观察器）是监听DOM变动的接口。当DOM对象树发生任何变动时，Mutation observer会得到通知。

其实`Mutation observer`代替了`Mutation events`作为观察DOM树结构发生变化时，作出相应处理的API。为什么要使用`Mutation observer`去替代 `Mutation events`呢？ 我们可以先了解一下`Mutation events`

<!--more-->

# Mutation events

`Mutation events`是在 [DOM3中定义](https://link.jianshu.com/?t=https://www.w3.org/TR/DOM-Level-3-Events/#events-mutationevents)，用于监听DOM树结构变化的事件

`Mutation events`的一个简单的例子**[Demo](https://codepen.io/shuliqi/pen/dyYObbo)**

可知它的简单用法如下：

```javascript
document.getElementById("container").addEventListener("DOMSubtreeModified", function() {
  console.log('我检测到子元素被修改')
}, false)
```

**Mutation events** 支持的事件列表如下：

1. **DOMAttributeNameChanged**：

2. **DOMCharacterDataModified**：在文本节点的值发生变化时触发

3. **DOMElementNameChanged**

4. **DOMNodeInserted**：监听元素子项的增加

5. **DOMNodeRemoved**： 监听元素子项的删除

6. **DOMNodeInsertedIntoDocument**：在一个节点被直接插入文档或通过子树间接插入到文档之后触发。这个事件在DOMNodeInserted之后触发

7. **DOMSubtreeModified**： 监听元素子项的修改(包括删除和新增)

8. **DOMAttrModified**： 是监听元素属性的修改，并且能够提供具体的修改动作

   

# Mutation events 遇到的问题

1. **浏览器兼容问题**

   * IE9不支持`Mutation events`
   * webkit内核不支持DOMAttrModified特性
   * DOMElementNameChanged 和 DOMAttributeNameChanged 在Firefox上不被支持
   
2. **性能问题**

   * `Mutation events` 是同步执行的，他每次调用，都需要从队列中取出事件，执行，然后从队列中移除，期间需要移动队列元素。如果事件触发的较为频繁的话， 每一次都需要执行上面的步骤，那么浏览器就会被拖慢。
   * `Mutation events` 本身是事件， 所以捕获是采用事件冒泡的形式，如果冒泡期间又出发了其他的 `Mutation events`的话，很有可能会导致阻塞javascript线程，甚至导致浏览器崩溃。

   

  # Mutation Observer

`Mutation Observer` 是在DOM4中定义的， 用于替代`Mutation events`的新的API，它的不同events 的是，`Mutation Observer` 所有监听操作以及相应处理都是在其他脚本执行完毕之后异步执行的，并且是所有的变动触发之后，将变的记录在数组中，统一进行回调，也就是说，当你使用observer监听多个DOM 变化时， 并且这若干DOM发生变化，那么observer会将变化记录到变化中， 等到一起都结束了，然后一次性的冲变化数组中执行相应饿回调函数。



## 特性

1. 只有在全部DOM操作完成之后才会调用callback, 可以看下面的**验证的例子**的`callback的回调次数`,
2. DOM 变动纪录封装成一个数组举行处置惩罚，而不是一条条地一般处置 DOM 变动。
3. 能够视察发作在 DOM 节点的一切变动，也能够视察某一类变动

如今，Firefox(14+)、Chrome(26+)、Opera(15+)、IE(11+) 和 Safari(6.1+) 支撑这个 API。 Safari 6.0 和 Chrome 18-25 运用这个 API 的时刻，须要加上 WebKit 前缀（WebKitMutationObserver）。能够运用下面的表达式搜检浏览器是不是支撑这个 API。

```javascript
const MutationObserver =
  window.MutationObserver ||
  window.WebKitMutationObserver ||
  window.MozMutationObserver
// 监测浏览器是不是支撑
const observeMutationSupport = !!MutationObserver
```



## 如何使用

Mutation Observer 的API 调用非常简单， 是一个构造函数

```javascript
const mutationObserver = new MutationObserver(mutations => {
  mutations.forEach(mutation => {
    console.log(mutation)
  })
})
```

MutationObserver 构造函数实例化一个观察者对象，其中的一个参数是一个回调函数，它会在指定的DOM节点发生变化之后执行的函数，回调函数会被传入两个参数， 一个是变化记录数组, 另外一个是观察者对象本身。

实例对象具有三个方法：

1. `observe`
2. `disconnect`
3. `takeRecords`

## observe

在观察者对象上，注册需要 观察的DOM节点，以及相应的参数

```javascript
void observe(Node target, optional MutationObserverInit options)
```

其中可选参数 MutationObserverInit的属性如下：

1. **childLIst**： 观察目标节点的子节点的新增和删除 。
2. **attributes**： 观察目标节点的属性节点(新增或删除了某个属性,以及某个属性的属性值发生了变化)。
3. **characterData**： 如果目标节点为characterData节点(一种抽象接口,具体可以为文本节点,注释节点,以及处理指令节点)时,也要观察该节点的文本内容是否发生变化
4. **subtree**：观察目标节点的所有后代节点(观察目标节点所包含的整棵DOM树上的上述三种节点变化)
5. **attributeOldValue**： 已经设为true的前提下,将发生变化characterData节点之前的文本内容记录下来(记录到下面MutationRecord对象的oldValue属性中)
6. **attributeFilter**： 一个属性名数组(不需要指定命名空间),只有该数组中包含的属性名发生变化时才会被观察到,其他名称的属性发生变化后会被忽略想要设置那些筛选参数的话，

如果想要使用哪个参数的话， 就将其值设定为true。

## disconnect

暂停在观察者对象上设置的节点的变化的监听， 直到重新调用observe 方法。

## takeRecords

在观察者对象上调用takeRecords 会返回其观察节点上的变化的记录(MutationRecords)数组。

其中MutationRecords数组也会作为观察者初始化时的回调函数的第一个参数。其中的属性如下：

1. **type**： 如果是属性发生变化，则返回attribuutes。如果是一个CharacterData节点发生变化，则返回characterData, 如果目标节点的某个字节点发生了变化，则返回childList.
2. **target**：返回此次变化影响到的节点，具体返回哪种节点类型是根据type值的不同而不同的。如果type为attributes，则返回发生变化的属性节点所在的元素节点。如果type值是characterData，则返回发生变化的这个characterData节点。如果type值是childLiist,则返回发生变化的的子节点的父节点。
3. **addedNodes**:返回被添加的节点。
4. **removeNodes**：返回被删除的节点
5. **previousSibling**: 返回被添加或者删除的节点的前一个兄弟节点。
6. **nextSibling**: 返回被添加或者删除的节点的后一个兄弟节点。
7. **attributeName**： 返回变更属性的本地名称。
8. **oldValue**: 根据type 值的不同， 返回的值也会不同。如果type为attributes,则返回该属性变化之前的属性值.如果type为characterData,则返回该节点变化之前的文本数据.如果type为childList,则返回null



# 验证的的例子

## callback的回调次数

**html结构：**

   ```html
<p>打开控制台看看log</p>
<div id='target' class='block' name='target'>
    <p>我是target的第一个子节点</p>
    <p>
       <span>我是target的后代</span>
    </p>
</div>
   ```

**javascri 代码：**

```javascript
const target = document.getElementById('target');
const i = 0
const observe=new MutationObserver(function (mutations,observe) {
  i++;
  console.log("我在全部DOM操作完成之后才会调用， 不信你看i的值:", i) // 
});

const config = {
  //设置true，表示观察目标子节点的变化，比如添加或者删除目标子节点，不包括修改子节点以及子节点后代的变化
  childList: true, 
}
observe.observe(target, config);

// 添加几个TextNode节点
target.appendChild(document.createTextNode('我新加的textNode1'));
target.appendChild(document.createTextNode('我是新加的textNode2'));
target.appendChild(document.createTextNode('我是新加的textNode3'));
```

**控制台打印的结果：**

控制只会打印一条：“ 我在全部DOM操作完成之后才会调用， 不信你看i的值: 1"

**说明**

MutationObserver的callback回调是异步的，只有在所有的DOM操作完成之后才会调用callback。

**[可以移步看看Demo](https://codepen.io/shuliqi/pen/zYvobdB) **

## childList属性

**html结构**：

```html
<p>打开控制台看看log</p>
<div id='target' class='block' name='target'>
    <p>我是target的第一个子节点</p>
    <p>
       <span>我是target的后代</span>
    </p>
</div>
```

**javascript代码**：

```javascript
const target = document.getElementById('target');
const observe=new MutationObserver(function (mutations,observe) {
   console.log(mutations);
});

const config = {
  //设置true，表示观察目标子节点的变化，比如添加或者删除目标子节点，不包括修改子节点以及子节点后代的变化
  childList: true, 
}
observe.observe(target, config);

// 添加几个TextNode节点
target.appendChild(document.createTextNode('新增Text节点'));   //增加节点，观察到变化

//删除节点，可以观察到
target.childNodes[0].remove();    

// 由于只设置 childList： true, 所以修改字节点以及自子节点后代， 不会被观察到
target.childNodes[0].textContent='改变子节点的后代'; 
```

**控制台打印的结果：**

{% asset_img mo1.jpg %}

**说明**：

从打印的结果得出结论，设置{ **childList: true**}时,表示观察目标子节点的变化,，如添加或者删除目标子节点，不包括修改子节点以及子节点后代的变化

**[可以移步看看例子](https://codepen.io/shuliqi/pen/wvKoZWB)**

如果想要观察到子节点以及后台的变化需要设置：

```json
{childList: true, subtree: true}
```

**[修改后代也能观察到的具体例子可以看看](https://codepen.io/shuliqi/pen/gOaLyeb)**

## CharacterData属性

**html结构**：

````html
<p>打开控制台看看log</p>
<div id='target' class='block' name='target'>
    我是target的text节点
    <p>
       <span>我是target的后代</span>
    </p>
</div>
````

**javascript:**

```javascript
const target = document.getElementById('target');
const observe=new MutationObserver(function (mutations,observe) {
   console.log(mutations);
});

const config = {
  // 观察目标节点的子节点的新增和删除。
  subtree: true, 
  // 如果目标节点为characterData节点(一种抽象接口,具体可以为文本节点,注释节点,以及处理指令节点)时,也要观察该节点的文本内容是否发生变化
  characterData: true,
}
observe.observe(target, config);
// target.childNodes[0]：“我是target的第一个子节点”， 是characterData类型节点 所以可以观察到
target.childNodes[0].textContent='改变Text节点';      

// arget.childNodes[1]： <p><span>我是target的后代</span></p>  不是是characterData类型节点， 所以不会观察到
target.childNodes[1].textContent='改变p元素内容'; 

 //添加text节点不会观察到
target.appendChild(document.createTextNode('新增Text节点')); 

//删除TEXT节点也不会观察到
target.childNodes[0].remove();  


// 注意：我们只需要记住只有对CharacterData类型的节点的数据改变才会被characterData为true的选项所观察到。
```

**控制台结果**

{% asset_img mo2.jpg %}

**说明：**

characterData这个选项，它是用来观察CharacterData类型的节点的，只有在改变节点数据时才会观察到，如果你删除或者增加节点都不会进行观察，还有如果对不是CharacterData类型的节点的改变不会观察到。

**[可以移步看看Demo](https://codepen.io/shuliqi/pen/KKdNYGq?editors=1010)**

## attributeFilter属性

**html结构**：

```html
<p>打开控制台看看log</p>
<div id='target' class='block' name='target' name="name">
    我是target的text节点
    <p>
       <span>我是target的后代</span>
    </p>
</div>
```

**javascript代码：**

````javascript
const target = document.getElementById('target');
const observe=new MutationObserver(function (mutations,observe) {
   console.log(mutations); // 我们可以看到打印的mutations有两个改变的状态
});

const config = {
  // 观察目标节点的子节点的新增和删除。
  subtree: true, 
  // 一个属性名数组(不需要指定命名空间),只有该数组中包含的属性名发生变化时才会被观察到,其他名称的属性发生变化后会被忽略想要设置那些删选参数的话
  attributeFilter: ["style"],
}
observe.observe(target, config);

target.style='color:red'; //可以观察到

target.removeAttribute('name'); //删除name属性，无法观察到                          
````

**控制台打印的结果：**

{% asset_img mo3.jpg %}

**说明：**

attributeFilte属性我们只设置了r: ["style"],没有设置name， 说明观察者只会观察style 属性的变化。

attributeFilte 选项主要是用来筛选要观察的属性

**[可以移步看看例子](https://codepen.io/shuliqi/pen/yLYVWNK?editors=1010)**

