---
title: Vue 组件 data 为什么必须是函数
date: 2021-05-14 14:58:36
tags: vue
---

在使用`vue`开发的时候，有一点觉得非常奇怪；使用`new Vue()`的时候，`data`是可以传入一个对象的；但是在组件中`data`就必须为一个函数；`vue`的官方文档是这么写的：

> 通过`Vue`构造器传入的各种选项大多数是可以在组件里使用。`data`是一个例外，它必须是一个函数。如果定义成了一个对象，那么`Vue`就会停止。并且会在控制台发出警告，告诉你在组件中`data`必须是一个函数。

```js
// new Vue() ---- 实例
const app = new Vue({
  el: "#app",
  data: {
  	name: shuliqi,
  	age: 12
  }
})

// 组件
Vue.compoment("people", {
	data: function () {
		return {
        name: shuliqi,
        age: 12
		}
	}
})
```



这是为什么？为什么在组件中`data`就必须为一个函数。

其实组件的配置和实例是需要分开的。最根本的原因是