---
title: React hook 学习
date: 2021-09-14 21:27:32
tags:
---

又要开始抗起`React` 搬砖了，感觉棒棒哒😭。多了很多的 `hook`， 嗯😔，先好好学学吧！

# 什么是 Hook？

`Hook` 是`React 16.8`的新特性。它的主要作用是让我们在不写 `class`的情况下可以使用`state`和`React`本身的一些特性。

`Hook`本质上就是一个函数， 它有自己的状态管理，生命周期管理，状态共享等；如下面的`Hook`：

- `useState`
- `useEffect`
- `useContext`
- `useReducer`

<!--more-->

# Hook 解决什么问题

我们来看看`Hook`到底解决了什么问题。

我们首先来看 这两种组件类型有什么区别：

<iframe height="654" style="width: 100%;" scrolling="no" title="class 组件" src="https://codepen.io/shuliqi/embed/QWgJBQQ?default-tab=js%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/shuliqi/pen/QWgJBQQ">
  class 组件</a> by shuliqi (<a href="https://codepen.io/shuliqi">@shuliqi</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>



<iframe height="379" style="width: 100%;" scrolling="no" title="函数 组件" src="https://codepen.io/shuliqi/embed/eYRQPmB?default-tab=js%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/shuliqi/pen/eYRQPmB">
  函数 组件</a> by shuliqi (<a href="https://codepen.io/shuliqi">@shuliqi</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>



由上面的例子我们可以看出：

**class 组件特点：**

- 有组件实例
- 有生命周期
- 有`state` 和 `setState`

**函数组件的特点：**

- 没有组件实例
- 没有有生命周期
- 没有`state` 和 `setState`；只能接收 `props`
-  函数组件只是一个纯函数， 执行完就会被销毁，无法存储 `state`

**`class`组件存在的问题是什么呢？**

-   大型的组件很难拆分和重构

- 相同的业务逻辑分散到各个方法中，变得混乱

- 复用的逻辑变得很复杂

  所以 `React`  更提倡函数式编程，因为函数更加灵活，更容易拆分。但是呢！！！！ 函数组件又太简单， 所以才出现了` hook`；`hook` 就是用来增强函数组件的功能的

# Hook 的规则

 `Hook`  有两条比较重要的规则；

- **只能在最最顶层使用`Hook`**

- **只有在React 组件中才能调用**

1. 为什么需要在最顶层使用???????

其实是为了保证多个 `hook` 的调用顺序是一致的

也就是说不要**循环，条件或者嵌套的函数中调用`hook`**。这样可以做到各个`hook` 每一次渲染中， 调用的顺序是一致的。

  ```js
  function Name () { 
    useState("1111"); 
    if (name === 'xxx') { 
      // 错误，没有在最顶层调用 
      useState("22222"); 
    } 
    // 会出现未知问题，有时候是第二次调用，有时候是第三次 
    useState("3333"); 
  }
  ```

2. 为什么我们需要保证多个 `hook` 的调用顺序一致呢？

   这个就跟`React`  实现的 `hook` 的原理有关了。因为每次在渲染的时候， `React`会把所有调用的`hook` 存储起来。

   关于这一块原理感兴趣， 可以看看这篇文章 [React Hooks 原理](https://github.com/bricksp)， 写得很好。

> `React`  也发布了 `ESlint` 插件  [eslint-plugin-react-hooks](https://link.zhihu.com/?target=https%3A//www.npmjs.com/package/eslint-plugin-react-hooks)帮助我们强制执行这两条规则。
>
> ```json
> { 
>   "extends": [
>     // ...
>     "plugin:react-hooks/recommended" 
>   ] 
> } 
> ```
>
> 

  # React 内置的 Hook

   我们来看看`React` 内置的`hook`  有哪些：

-  `useState`  状态管理
- `useEffect` 生命周期管理
- `useContext` 共享状态数据
- `useMemo` 缓存值
- `useRef` 获取`Dom` 操作
- `useCallback` 缓存函数
- `useReducer` 与 `redux` 一样
- `useImperativeHandle` 子组件暴露值/方法
- `useLayoutEffect` 完成副作用操作，会阻塞浏览器绘制

  下面我们来分别学习一下这些内置的`Hook`

# useState 状态管理

 在` class` 组件中，我们获取 ` state` 是从`this.state` 中获取的。但是在函数组件中是没有 ` this` 的。

所以在函数组件中就可以使用`hook` 提供的` useState`来管理和维护` state`。

## 使用

```js
const [name,setName] = useState(initName)
```

- `useState`: 定义 state 变量的函数`hook`
- `name`: 定义出来的变量
- `setName`：为更新 `satate` 方法
- `initName`: `name`变量的初始值

 使用`useState` 可以定义一个变量，如上我们这个变量叫`name`。`useState`有一个参数（如上`initName`），为变量的初始值。初始值可以根据我们自己的需要使用不同类型（即可以是字符串，数字， 布尔值等， 不一定是非是对象）`useState`的返回值是返回当前的`state`和更新`state`的函数。

## 举个🌰

<iframe height="426" style="width: 100%;" scrolling="no" title="React Hook useEffect()" src="https://codepen.io/shuliqi/embed/ZEJzrby?default-tab=js%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/shuliqi/pen/ZEJzrby">
  React Hook useEffect()</a> by shuliqi (<a href="https://codepen.io/shuliqi">@shuliqi</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

# useEffect 生命周期管理

`useEffect` 可以看作是函数式组件的生命周期管理，因为在函数式组件中无法使用生命周期。这就可以使用`useEffect`来进行管理了。

`useEffect`主要管理三个生命周期函数：

- `componentDidmount`

  组件第一次渲染完成，此时dom节点已经生成，可以在这里调用ajax请求，返回数据`setState`后组件会重新渲染

- `componentDidUpdate`

  组件更新完毕后，react只会在第一次初始化成功会进入`componentDidmount`,之后每次重新渲染后都会进入这个生命周期，`componentDidUpdate(prevProps,prevState`这里可以拿到`prevProps`和`prevState`，即更新前的props和state。

- `componentWillUnmount`

  组件销毁之后触发的生命周期。一般用来：

  > - 清除在组件中的定时器（`setTimeout`,`setInterval`）
  > - 移除组件中的监听（`removeEventListener`）
  > - 取消还没有请求结果的`ajax`请求

  

## 无需清除的 effect

有时候我们只希望在`React`更新`DOM`之后运行一些额外的代码，那么只需要在`class` 组件生命周期`componentDidmount`和`componentDidUpdate`中执行即可。

那么在`useEffect`中如何写呢？我们可以这么写：

```js
useEffect( ()=> {
	// 默认会执行这部分，相当于 class 组件的生命周期（componentDidmount， componentDidUpdate）
}, [])
```

## 需要清除 effect

当我们希望在一个组件销毁的时候执行一些逻辑处理。那么就需要在 class 组件的 `componentWillUnmount` 执行即可。

那么在`useEffect`中如何写呢？我们可以这么写：

```js
useEffect( ()=> {
   return () => {
			// 组件销毁时执行的函数
	 }
}, [])
```

## 监听 state 的变化

当我们需要监听的`state`的变化然后做一些处理的时候,我们可以这么写：

```js
    useEffect(() => {
        // 监听num，count  状态的变化， 变化了则执行里面的代码
        // 不监听时为空 [] , 或者不写
    }, [num, count])
```

## 举个🌰

<iframe height="732" style="width: 100%;" scrolling="no" title="React Hook useEffect()" src="https://codepen.io/shuliqi/embed/RwZbQRm?default-tab=js%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/shuliqi/pen/RwZbQRm">
  React Hook useEffect()</a> by shuliqi (<a href="https://codepen.io/shuliqi">@shuliqi</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>





未完待续....最近太忙了， 救救孩纸吧！













