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

安装 `nodemon`

```bash
npm install --save-dev nodemon
```

> [nodemon](https://www.npmjs.com/package/nodemon) 是一个当我们的代码变动之后， 能够及时的帮助我们重启服务，而不需要我们手动的启动了

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
    nodemon index.js
   ```

​              


3. 查看结果


这时候我们 可以打开浏览器输入`localhost:3000` 查看结构。或者直接使用如下命令在终端看结果：

   ```
    curl http://localhost:3000
    Hello World
   ```

   结果返回了了"Hello World"文案   

我们可以看到我们的项目中`index.js` 引用了`Koa`。然后用`new`来实例化一个`app`。然后使用`app.use`传入一个`async`函数（又称：`koa`中间件）。 最后调用`app.listen`方法，这样一个`web`应用就跑起来了。

如果有需要看例子的可到： [官方的例子](https://github.com/shuliqi/koa-study/tree/example/one)

# 代码结构

`Koa`的代码结构很简单，总的只有四个文件：`application.js`,  `context.js`   ,`request.js`,  `response.js `，这四个文件在`lib`目录下:

{% asset_img 1.png %}

其实从文件名上，我们就能猜到每个文件是做什么的了， 接下来我们根据流程来看每个文件的代码逻辑。

# 启动流程

启动的入口在哪里呢？ 我们找到`package.json`文件的`nain`字段，可知道`Koa` 的入口是 `lib`下的`application.js`。 那我们就先来看看该文件。

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
this.middlewar-e = [] // 配置了中间件数组
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
    // 核心：中间件的管理和next的实现
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

在看 `koa-compose` 之前，我们先看一下它的具体体现，看一个例子：

`index.js：`

```javascript
const Koa = require('koa');

const app = new Koa();

app.use(async(ctx, next) => {
 console.log("----start1----");
  await next();
  console.log("----end1----");
});

app.use(async(ctx, next) => {
  console.log("----start2----");
  await next();
  console.log("----end2----");
});

app.use(async(ctx, next) => {
  console.log("----start3----");
  await next();
  console.log("----end3----");
 });

app.use(async(ctx, next)=> {
  console.log("----start4----");
  await next();
  console.log("----end4----");
});

app.listen(3000);

```

执行 ` nodemon index.js`  之后在命令行书输入： `curl http://localhost:3000` 可到结果为：

```
----start1----
----start2----
----start3----
----start4----
----end4----
----end3----
----end2----
----end1----
```

我们再来回忆一下这个过程： `new` 一个 `app` 之后，使用`use(fn)` 将函数`push` 到 ` this.middleware`。然后`lisen`创建了`http.createServer(requestListener)`参数`requestListener`是请求处理函数，用来响应`request`事件，而这里的`requestListener` 是我们的`this.callback`, 而`this.callback`的第一行代码就是我们的`koa-compose`。

> 注意：这个过程就是上面那些源码的解释

而从得出的结果来看我们知道 ：当程序运行到`await next()`的时候就会暂停当前程序，进入下一个中间件，**处理完之后才会回过头来继续处理**。而`koa-compose`就是用来做这个处理的--->`next`实现的。

那接下来我们就来看看`koa-compose` 的源码。我们找到包`koa-compose`打开它， 只有一个函数：

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
      // 当循环完中间件数组的函数，直接 return Promise.resolve(
      if (!fn) return Promise.resolve()
      try {
        // 核心代码：返回Promise
        // next时，交给下一个dispatch（下一个中间件方法）
        // 同时，当前同步代码挂起，直到中间件全部完成后继续
        return Promise.resolve(fn(context, dispatch.bind(null, i + 1)));
      } catch (err) {
        return Promise.reject(err)
      }
    }
  }
}
```

`dispatch`函数，它将遍历整个`middleware`，然后将`context`和`dispatch(i + 1)`传给`middleware`中的方法。其中的`dispatch(i + 1)`就是`middleware`中的方法的`next`函数。

主要是实现:

1. `context`传给中间件
2. 将`middleware`中的下一个中间件`fn`作为未来`next`的返回值。

**剖析：**

**`next()`返回的是`promise`，需要使用`await`去等待`promise`的`resolve`值。**`promise`的嵌套就像是洋葱模型的形状就是一层包裹着一层，直到`await`到最里面一层的`promise`的`resolve`值返回。

代码解析如下：

```javascript
const middleware1 = async (context, next) => {
  console.log("----start1----");
  await next();
  console.log("----end1----");
}
const middleware2 = async (context, next) => {
  console.log("----start2----");
  await next();
  console.log("----end2----");
}
const middleware3 = async (context, next) => {
  console.log("----start3----");
  await next();
  console.log("----end3----");
}
const middleware4 = async (context, next) => {
  console.log("----start4----");
  await next();
  console.log("----end4----");
}
const MyCompose = () => {
  return Promise.resolve(middleware1(this, ()=>{
    return Promise.resolve(middleware2(this, () => {
      return Promise.resolve(middleware3(this, () => {
        return Promise.resolve(middleware4(this, () => {
          return Promise.resolve();
        }));
      }))
    }))
  } ))
}
MyCompose();
```

结果：

```
----start1----
----start2----
----start3----
----start4----
----end4----
----end3----
----end2----
----end1----
```

当前的`Koa`例子代码： [洋葱模型](https://github.com/shuliqi/koa-study/tree/example/two)





未完待续....
