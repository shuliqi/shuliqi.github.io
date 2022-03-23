---
title: Koa 源码阅读学习笔记
date: 2022-01-04 15:54:43
tags: Koa
---

最近看了 [Koa](https://koajs.com/#) 源码， 觉得`Koa`源码设计得特别巧妙而且也很简单，易读懂。基础的代码只有不到 2000 行。我们知道 `Koa` 是一个很轻量级的`web`框架， 里面除了`middleware` 和 `ctx` 之外就什么没有了。虽然很简单， 但是功能还是很强大的，它仅仅是靠中间件就是搭建完整的`web`应用。这篇文章主要记录一下学习的笔记， 供之后翻阅。

读完这篇文章将会了解到：

- 著名的洋葱模型
- 上下文如何构建

> [Koa官方](https://koajs.com/#)  [Koa  中文文档](https://koa.bootcss.com/)。`Koa`的源码可直接从 [github](https://github.com/koajs/koa) 上获取

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

## 使用Demo

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

## 主要原理

我们来回忆一下上面这个`Demo`的过程： `new` 一个 `app` 之后，使用`use(fn)` 将函数`push` 到 ` this.middleware`。然后`lisen`创建了`http.createServer(requestListener)`参数`requestListener`是请求处理函数，用来响应`request`事件，而这里的`requestListener` 是我们的`this.callback`, 而`this.callback`的第一行代码就是我们的`koa-compose`。

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
      // 为什么会有这个？ 是因为可能开发者在中间件函数中对此调用next函数， 是为了解决这个问题的， 具体看下面的问题2
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

## 问题1

以上的例子是符合的我们的预期的，但是会有人觉得奇怪，造成这样的预期是因为我们在中间件函数中使用了`async`函数， 但是真的是`async`函数起的作用吗？显然是不是的：，**async并不是koa洋葱模型的必要条件**

{% asset_img 2.png %}

{% asset_img 3.png %}

## 问题2

如果在中间件函数多次调用了`next` 函数会怎么样? `Koa` 如何处理呢？

`Koa`做了一个标记 `i`， 默认为 -1。 每一次调用`dispatch` `i`都会加1，在每个`dispatch`内部判断这个`index`是否小于等于现在的i（也就是在middleware的index），然后将更新这个`index`为i。 这时如果多次调用`next`，`i`就会>=现在的`index`，抛出错误。

也就是这段代码：

```javascript
  // ...
  let index = -1
    return dispatch(0)
    function dispatch (i) {
      if (i <= index) return Promise.reject(new Error('next() called multiple times'))
      index = i
      // ...
```

当前的`Koa`例子代码： [洋葱模型](https://github.com/shuliqi/koa-study/tree/example/two)



# createContext

在 `callback` 函数中， 洋葱模型之后， 返回了一个处理`request` 的函数（`handleRequest`）:

```javascript
  const handleRequest = (req, res) => { 
      // 将req, res包装成一个ctx返回
      const ctx = this.createContext(req, res); 
      return this.handleRequest(ctx, fn); 
  }
```



 这函数里面 使用 ` this.createContext(req, res)` 创建 `context`。这里传入了`http`模块提供的两个参数`req`、`res`，然后声明`ctx`为`this.createContext(req, res)`的返回值。看下这个`this.createContext`。

```javascript
createContext(req, res) {
  const context = Object.create(this.context);
  const request = context.request = Object.create(this.request);
  const response = context.response = Object.create(this.response);
  context.app = request.app = response.app = this;
  context.req = request.req = response.req = req;
  context.res = request.res = response.res = res;
  request.ctx = response.ctx = context;
  request.response = response;
  response.request = request;
  context.originalUrl = request.originalUrl = req.url;
  context.state = {};
  return context;
}
```

这个函数的目的就是包装出一个全局唯一的 `context`

上面的`constructor` 里面声明了 

```javascript
this.context = Object.create(context);
this.request = Object.create(request);
this.response = Object.create(response);
```

而在这里使用`Object.create` 又包装了一层

```javascript
const context = Object.create(this.context);
const request = context.request = Object.create(this.request);
const response = context.response = Object.create(this.response);
```

目的是让每一次的`http`请求都生成全局唯一，相互之间隔离的`context`,`request`, `response`。

最后总结一下 `callback` 函数: 首先`koa-compose`生成统一的中间件，然后`handleRequest`被调用。`handleRequest` 里面生成全局唯一的且包装好`context`。然后调用`this.handleRequest`  传入包装好`context`和中间件函数。

# handleRequest

接下来就是`this.handleRequest` 了。 这个函数主要是用于真正的进行业务逻辑处理了。

```javascript

  handleRequest(ctx, fnMiddleware) {
    const res = ctx.res;
    res.statusCode = 404;
    const onerror = err => ctx.onerror(err);
    const handleResponse = () => respond(ctx);
    onFinished(res, onerror);
    return fnMiddleware(ctx).then(handleResponse).catch(onerror);
  }
```

`fnMiddleware`是`koa-compose`包装后的函数， 函数签名是`(context, next) => Promise`， 内部会依次调用每个中间件。

`onFinished` 用于在请求`close`，`finish`，`error`时执行传入的错误回。

`respond` 函数用于将中间件处理后的结果通过`res.end`返回给客户端：

```javascript
// Response helper.
function respond(ctx) {
  // allow bypassing koa
  if (false === ctx.respond) return; // ctx.respond = false用于设置自定义的Response策略

  if (!ctx.writable) return;

  const res = ctx.res;
  let body = ctx.body;
  const code = ctx.status;

  // ignore body
  if (statuses.empty[code]) {
    // strip headers
    ctx.body = null;
    return res.end();
  }

  // HEAD请求不返回body
  if ('HEAD' == ctx.method) {
    // headersSent表示是否发送过header
    if (!res.headersSent && isJSON(body)) {
      ctx.length = Buffer.byteLength(JSON.stringify(body));
    }
    return res.end();
  }

  // status body
  if (null == body) {
    if (ctx.req.httpVersionMajor >= 2) {
      body = String(code);
    } else {
      body = ctx.message || String(code);
    }
    if (!res.headersSent) {
      ctx.type = 'text';
      ctx.length = Buffer.byteLength(body);
    }
    return res.end(body);
  }

  // responses
  if (Buffer.isBuffer(body)) return res.end(body);
  if ('string' == typeof body) return res.end(body);
  if (body instanceof Stream) return body.pipe(res); // 流式响应使用pipe，更好的利用缓存

  // body: json
  body = JSON.stringify(body);
  if (!res.headersSent) {
    ctx.length = Buffer.byteLength(body);
  }
  res.end(body);
}
```

它做的是**ctx返回不同情况的处理**，如`method`为`head`时加上`content-length`字段、`body`为空时去除`content-length`等字段，返回相应状态码、`body`为`Stream`时使用`pipe`等
