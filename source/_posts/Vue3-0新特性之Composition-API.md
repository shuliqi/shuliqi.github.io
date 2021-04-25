---
title: Vue3.0新特性之Composition API
date: 2021-04-25 17:55:25
tags:
---

`Composition API`是`Vue3.0`版本中主要特色语法，这是一个全新的逻辑重用和代码组织的方法。`Vue2.0`（选项）所有数据都定义在`data`中，方法定义在`methods`中。所以为了个组件添加逻辑， 我们可能需要填充（选项）属性`data`, `methods`,`computed`等。但是`Vue3.0`我们可以不这么写了。具体怎么写， 我们先看看`Vue2.0`的写法有什么缺陷。

 <!--more-->

# option API

使用`Vue2.0`的`option API`实现如下功能：

<iframe height="645" style="width: 100%;" scrolling="no" title="rNjPvPg" src="https://codepen.io/shuliqi/embed/rNjPvPg?height=645&theme-id=dark&default-tab=js,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/rNjPvPg'>rNjPvPg</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

