---
title: Koa 源码阅读学习笔记
date: 2022-01-04 15:54:43
tags: Koa
---

最近看了 [Koa](https://koajs.com/#) 源码， 觉得`Koa`源码设计得特别巧妙而且也很简单，易读懂。基础的代码只有不到 2000 行。我们知道 `Koa` 是一个很轻量级的`web`框架， 里面除了`middleware` 和 `ctx` 之外就什么没有了。虽然很简单， 但是功能还是很强大的，它仅仅是靠中间件就是搭建完整的`web`应用。我们谈到`Koa`总能想它的： **洋葱模型**， **context的委托模式**，**错误处理**等优点。学习的时候也是重点围绕这三点来学习的。

这篇文章主要记录一下学习的笔记， 供之后翻阅。

> [Koa官方](https://koajs.com/#)  [Koa  中文文档](https://koa.bootcss.com/)。
>
> `Koa`的源码可直接从 [github](https://github.com/koajs/koa) 上获取



<!--more-->

# 使用Demo

我们在读源码之前， 我们先看一下如何使用`Koa`。 通过阅读官方，我们来写一个例子：

1. 创建`js` 文件并且初始化，然后安装`koa`

```bash
 touch index.js && npm init && npm i koa
```

`index.js` 文件直接使用官方提供的例子：

```javascript
const Koa = require('koa');
const app = new Koa();

app.use(async ctx => {
  ctx.body = 'Hello World';
});

app.listen(3000);
```



2. 启动应用

在项目的根目录下执行： 

   ```js
    node index.js
   ```

   


3. 查看结果


这时候我们 可以打开浏览器输入`localhost:3000` 查看结构。或者直接使用如下命令在终端看结果：

   ```
    curl http://localhost:3000
    Hello World
   ```

   结果返回了了"Hello World"文案

   


我们可以看到我们的项目中`index.js` 引用了`koa`。然后用`new`来实例化一个`app`。然后使用`app.use`传入一个`async`函数（又称：`koa`中间件）。 最后调用`app.listen`方法，这样一个`web`应用就跑起来了。

# 代码结构

`Koa`的代码结构很简单，总的只有四个文件：`application.js`,  `context.js`   ,`request.js`,  `response.js `，这四个文件在`lib`目录下:

{% asset_img 1.png %}

其实从文件名上，我们就能猜到每个文件是做什么的了， 接下来我们根据流程来看每个文件的代码逻辑。



# 启动流程

启动的入口在哪里呢？ 我们找到`package.json`文件，可知道`Koa` 的入口是 `lib`下的`application.js`。 那我们就先来看看该文件。

# application.js

在该文件中 ， 我们可以看到是暴露了一个类`Application`, 该类继承`Emitter`。

> 至于 `Emitter` 是什么， 我们下面会讲到

```javascript
module.exports = class Application extends Emitter {

  constructor (options) {
    super()
    options = options || {}
    this.proxy = options.proxy || false
    this.subdomainOffset = options.subdomainOffset || 2
    this.proxyIpHeader = options.proxyIpHeader || 'X-Forwarded-For'
    this.maxIpsCount = options.maxIpsCount || 0
    this.env = options.env || process.env.NODE_ENV || 'development'
    if (options.keys) this.keys = options.keys
    this.middleware = [] 
    this.context = Object.create(context)
    this.request = Object.create(request)
    this.response = Object.create(response)
    // util.inspect.custom support for node 6+
    /* istanbul ignore else */
    if (util.inspect.custom) {
      this[util.inspect.custom] = this.inspect
    }
  }
  ...
}
```

这里的`constructor`  主要是做了一些配置的处理, 主要的是： 

```javascript
this.middleware = [] // 配置了中间件数组
this.context = Object.create(context)
this.request = Object.create(request)
this.response = Object.create(response)
```

定义了中间件数组`middleware`; `Koa` 最大的特点就是可以有无数的中间件（`app.use(fn)`），所以得定义一个数组把这些中间件放到一起。其次使用`Object.create`来拷贝`context`,` request`,`response`。这三个都是`lib`目录下对应的文件。

**思考**： 为什么要拷贝（`Object.create`）呢？

**原因**：这是因为在一个应用中， 可能会有多个 `Koa` 实例（new Koa）。为了防止多个实例之间的污染，所以使用拷贝的方式来让其引用不指向同一个地址。

# use 方法

当我们的实例`new` 好之后， 我们就可以使用`app.use` 来使用不同的中间件。我们来看看`use`方法：

```javascript
use (fn) {
  if (typeof fn !== 'function') throw new TypeError('middleware must be a function!')
  debug('use %s', fn._name || fn.name || '-')
  this.middleware.push(fn)
  return this
}
```

首先判断传进来的参数是不是函数，如果不是则抛出错误。如果是函数， 则把该中间件（函数）添加到中间件数组中（`middleware`）

# listen 方法

看我们的简单实用案例，我们知道在 `new Koa` 的时候是并没有启动 `Server`的，是在`listen` 启动的。我们来看看逻辑：

```javascript
listen (...args) {
  debug('listen')
  // 调用Node原生的http.createServer([requestListener])
  // 参数requestListener是请求处理函数，用来响应request事件；
	//此函数有两个参数req,res。当有请求进入的时候就会执行this.callback函数
  const server = http.createServer(this.callback())
  return server.listen(...args)
}
```

`listen` 主要是封装了 `node`的 `http` 模块提供的`createServer` 方法方法来创建一个`http`服务对象和使用 `listen`方法来监听。

我们看 [http.createServer([options][, requestListener])](http://nodejs.cn/api/http.html#httpcreateserveroptions-requestlistener)的函数签名不难猜出 `this.callback`返回的是`options` 或 `requestListener`； 那我们来看 `callback` 方法。

```javascript
  callback () {
    // koa-compose 是Koa中间件洋葱模型的核心，具体讲解在下面
    const fn = compose(this.middleware)

    // listenerCount和on均是父类Emitter中的成员
    // 判断我们自己代码是否有自己写监听，如果没有就直接用 Koa 的 this.onerror 方法
    if (!this.listenerCount('error')) this.on('error', this.onerror)

   	// Koa 的委托模式主要在这个函数体现
    const handleRequest = (req, res) => {
      // 每个请求过来时，都创建一个context
      const ctx = this.createContext(req, res)
      
      // 注意： 这里的 this.handleRequest 是父类上的 handleRequest。不是本 handleRequest
      return this.handleRequest(ctx, fn)
    }
    return handleRequest
  }
```

`callback` 中我们遇到了 Koa 的核心库：`koa-compose` 洋葱模型。 它用于精心组合所有`middleware`，并按照期望的顺序调用。

# koa-compose（洋葱模型）

我们找到包`koa-compose`打开它， 只有一个函数：

```javascript
// middleware 就是我们使用app.use(fn) 传传入的函数数组(中间件)
function compose (middleware) {
  
  // 判断 middleware 是否是数组
  if (!Array.isArray(middleware)) throw new TypeError('Middleware stack must be an array!')
 
  // 判断 middleware 数组的每一个函数是否是函数
  for (const fn of middleware) {
    if (typeof fn !== 'function') throw new TypeError('Middleware must be composed of functions!')
  }

  // compose 函数返回一个函数
  return function (context, next) {
    let index = -1
    return dispatch(0)
    function dispatch (i) {
      if (i <= index) return Promise.reject(new Error('next() called multiple times'))
      index = i
      let fn = middleware[i]
      if (i === middleware.length) fn = next
      if (!fn) return Promise.resolve()
      try {
        return Promise.resolve(fn(context, dispatch.bind(null, i + 1)));
      } catch (err) {
        return Promise.reject(err)
      }
    }
  }
}
```

待续.... 还没看懂
