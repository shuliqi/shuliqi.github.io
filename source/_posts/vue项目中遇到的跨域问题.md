---
title: vue项目中遇到的跨域问题
date: 2021-04-15 10:19:03
tags: Vue
categories:  Vue
---

最近在做一个全新的`Vue`前端项目搭建及其开发工作，后端和前端都是分离的，所以避免不了开发环境和生产环境的跨域问题。 开发环境或者是生产环境，前端和后端都是在同一个机器下面部署或者是使用不同的端口号。 当我们的前端资源访问后端服务时得不到数据或没有达到预期的效果。以前也是知道跨域问题的， 但是没有好好总结。那么这篇文章就主要来讲讲遇到的跨域问题。以及如何解决，在vue项目中如何解决等。

 <!--more-->

# 跨域的问题

上面我们也说了， 现在开发项目大部分都是前后端分离的，那么无论是什么环境就肯定会遇到跨域问题。我们举一个例子来说明：

## 后端部分

我们使用 `express`来弄一个后端的服务：

```javascript
mkdir serve
```

```
cd serve
```

```javascript
touch index.js
```

```
npm i express --save
```

添加 `.gitignore`文件

```
.DS_Store
node_modules
```

**index.js**文件的代码：

```javascript
const express = require('express')
const app = express()
const port = 3000

app.get('/api/getName', (req, res) => {
  res.send('舒丽琦')
})

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`)
})
```

这段代码说明：有一个在端口为 8000 的接口 '/getName`。 接口返回 `舒丽琦`。

现在我们启动这个服务：`node index.js`。 打开浏览器访问：`http://localhost:3000/api/getName`。可得到：

{% asset_img 1.png %}

## 前端部分

我们使用`Vue`的脚手架`vue-cli`来构建一个前端的框架：

```javascript
vue create web 
```

```javascript
cd web
```

```javascript
 npm install axios --save
```

我们把`App.vue`代码改成：

```vue
<template>
</template>
<script>
import axios from "axios";
export default {
  name: 'App',
  created() {
    // 去请求我们刚才启动的后端得服务的 getName 接口
    axios.get('http://localhost:8000/getName')
        .then(function (response) {
          console.log(response);
        })
        .catch(function (error) {
          console.log(error);
        });
  }
}
</script>

```

最后启动一下前端服务`npm run dev`。

## 结果及其原因

这时候我们后端和前端是准备完毕了， 我们来看看结果吧！打开` http://localhost:8080/`。然后打开控制台，得到的结果如下：

{% asset_img 2.png %}

我们发现是报错了。一看报错信息就知道你产生了跨域的问题。那跨域问题请求到底是什么返回什么呢？

我们继续 `debugger`

{% asset_img 3.png %}

发现`reponse:`undefined`，提示消息:`Network Error`。 那是不是说明请求没有到后端呢？我们来试试。

我们在后端的接口函数里面打印个`log`。然后看请求的时候， 看请求是否到了后端：

```javascript
const express = require('express')
const app = express()
const port = 8000

app.get('/getName', (req, res) => {
  // 加个log
  console.log("请求到后端了");
  res.send('舒丽琦')
})

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`)
})
```

最后我们前端页面请求下，结果是这样的：

{% asset_img 4.png %}

那就说明，**有跨域的时候， 请求是到达了后端的， 并且后端还返回了 数据。只是在浏览器被拦截了。**

[跨域体现的例子](https://github.com/shuliqi/crossDomain/tree/master)

# 跨域产生的原因

经过上面的验证，我们知道跨域是浏览器做了拦截，并且报了错。那为什么浏览器会做拦截呢？

那当然是为了**安全问题**。比如著名的CSRF[攻击](https://translate.google.com.hk/translate?hl=zh-CN&sl=zh-TW&u=https://zh.wikipedia.org/zh/%25E8%25B7%25A8%25E7%25AB%2599%25E8%25AF%25B7%25E6%25B1%2582%25E4%25BC%25AA%25E9%2580%25A0&prev=search&pto=aue)。XSS攻击。

所以浏览器因为安全问题而引入了**同源策略**：

{% asset_img 5.png %}

只有当 协议，域名，端口 三者都相等时，才不会产生跨域问题。就是说是同源，才能读取服务器的响应。

| 当前的url                      | 请求的url                      | 是否跨域                                                |
| ------------------------------ | ------------------------------ | ------------------------------------------------------- |
| https://shuliqi.github.io      | http://shuliqi.github.io       | 是，协议不同（https/http）                              |
| https://xiaoxiaoshu.github.io  | https://shuliqi.github.io      | 是，域名不同（xiaoxiaoshu.github.io/shuliqi.github.io） |
| https://shuliqi.github.io:3000 | https://shuliqi.github.io:8080 | 是，端口不同（3000/8080)                                |



但是 html中的 `img`,`script`, `link`, `iframe`等是允许跨域加载资源的

# 解决跨域问题

上面例子中，我们由于同源策略的原因（其中域名，端口不相同）产生跨域，导致浏览器拦截并报错。那我们如何解决呢?

## 前端proxy

浏览器是禁止跨域的，但是服务器不禁止，所以我们前端可以使用`webpack`给我们的本地起一个服务，作为请求的代理对象。

由于我们的项目是`vue-cli3`脚手架搭建的。所以`webpack`基础配置全部内嵌了，所以我么初始化项目之后`Webpack`的`config`初始化的配置不见了。但是`Vue-cli3`给我们留了一个`vue.config.js`文件供们对`webpack`进行自定义配置。

我们在我们的`vue`项目的根目录中添加`vue.config.js`内容如下：

```javascript
module.exports = {
  // 开发环境
  devServer: {
    proxy: {
      // 代理的标识， 告诉 node， url 前面是 api的就是需要代理的
      "/api": { 

        // 目标地址，一般指后端服务器地址
        target: "http://localhost:3000",

        // 是否允许跨域
        changeOrigin: true, 

        // 重写实 际Request Url中的'/api'用""代替， 因为我们后端接口没有api
        pathRewrite: {  

          // 我们请求url为：'/api/getNmae'话的，经过http-proxy-middleware的代理服务器时候改成'/getName',然后代理到 target 目标地址
          '^/api': "",  
        }
      }
    }
  }
}
```

上面的每一步解释已经很清楚了。由于我们做道代理的时候重写了请求url。所以我们代理服务器最终向目标服务请求的链接是：` http://localhost:8080/ /getName`。 所以我们后端写的接口也去掉`api`。

/serve/index.js:

```javascript
const express = require('express')
const app = express()
const port = 3000
// 去掉'api/getName'中的api，这样前端代理服务器请求过来的才能匹配到
app.get('/getName', (req, res) => {
  console.log("请求到后端了")
  res.send('舒丽琦')
})

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`)
})
```

最后我们在请求的时候，需要注意将`axios`的`baseUrl`改成 `api`

App.vue文件修改如下：

```javascript
  axios.get('http://localhost:3000/api/getName') --->   axios.get('/api/getName')
```



最后前端和后端服务都重启。打开浏览器，结果如下：

{% asset_img 6.png %}

我们可以看到，我们再开发环境成功请求到数据。

[本次例子代码](https://github.com/shuliqi/crossDomain/tree/webProxy)





## cors方式

上面解决我们在开发环境遇到的跨域问题，但是我们打包上线的话， 我们做的配置是不生效的。自然而然也就产生了跨域。那么这种情况就可以使用`cors`方式来解决跨域。

`cors`称: 跨域资源共享（Cross-origin resource sharing）， 是一中ajax 跨域请求资源的方式。但是这种方式是有兼容性的问题的

- cors 必须 浏览器和服务端同时支持，才能实现跨域
- 这种方式几乎所有的浏览器都支持， 但是IE 需要IE10以上才能支持
- IE8，IE9需要通过`XDomainRequest`来实现。

我们知道请求会分为**简单的请求**和**复杂的请求**：

**简单的请求：**

- 请求方式是 GET, POST, HEAD之一;

- Content-Type的值是 `text/plain`, ` multipart/form-data`, `application/x-www-form-urlencoded`之一；

那么这次的请求就是简单的请求。

**复杂的请求：**

- 请求方式是下面方式之一：PUT，

  ```javascript
  PUT
  DELETE
  CONNECT
  OPTIONS
  TRACE
  PATCH
  ```

- Content-Type的值不属于下列之一：

  ```javascript
  application/x-www-form-urlencoded
  multipart/form-data
  text/plain
  ```

 对于简单的请求，对于简单的请求，浏览器会直接发送cors请求，具体来说就是在header中加入origin请求头字段。在响应头回服务器设置相关的cors请求,响应头字段为允许跨域请求的源。

而对于复杂的请求，浏览器会先自动发送一个 `options`请求浏览器是否支持该请求， 如果不支持，则控制台直接报错， 如果支持， 那么就会发送真正的请求到后端。

我们继续使用我们上面的产生跨域的例子：

serve/index.js:

````javascript
const express = require('express')
const app = express()
const port = 3000

//设置跨域访问
app.all('*', function(req, res, next) {
  // 设置哪个源可以访问我
  res.header("Access-Control-Allow-Origin", "*");

  // 允许携带哪个头访问我
  res.header("Access-Control-Allow-Headers", "X-Requested-With");

   // 允许哪个方法访问我
  res.header("Access-Control-Allow-Methods","PUT,POST,GET,DELETE,OPTIONS");

  next();
});

app.get('/api/getName', (req, res) => {
  console.log("请求到后端了")
  res.send('舒丽琦')
})

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`)
})
````

然后起我们的前端服务`npm run serve` 和后端服务`node index.js`, 打开我们的`http://localhost:8080/`

可得到结果如下：

{% asset_img 7.png %}

[本例子的代码](https://github.com/shuliqi/crossDomain/tree/cors))

## 后端代理

第二种方式简单，但是还是会在一定的程度上有风险的，或者某些浏览器不支持的话。那也是没作用的。那第一种方式我们是前端实现代理， 那后端其实也是可以实现代理的。

这里我们以 `express` 为例子. 首先后端安装 ：

```javascript
npm install http-proxy-middleware --save
```

新建serve.js

```javascript
const express = require('express')
const app = express()
const { createProxyMiddleware } = require('http-proxy-middleware');
const port = 8000;

app.use('/api', createProxyMiddleware({
  // 接受到前端的请求，然后转到3001
  target: "http://localhost:3001",
  changeOrigin: true,
}))

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`)
})
```

这里我们监听 8000 端口， 当接收到请求前缀是`/api`， 我们就代理到 3001 端口。



我们的`index.js` 改成 `api.js`， 内容不变

```javascript
const express = require('express')
const app = express()
const port = 3001

app.get('/api/getName', (req, res) => {
  console.log("请求到后端了")
  res.send('舒丽琦')
})

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`)
})
```

这里我们监听的是 3001 端口。这里才是真正的接口响应的部分。

我们新建 index.html. 内容如下：

```html
<!DOCTYPE html>
<html>
<head>
    <title>首页</title>
    <meta charset="utf-8">
    <script type="text/javascript" src="//cdn.bootcss.com/jquery/3.1.1/jquery.min.js"></script>
</head>
<body>
 	<script type="text/javascript">
        $(function(){
            var contextPath = 'http://localhost:4000/api/getName';
             $.ajax({
                    type:'get',
                    data:'click',
                    url:contextPath,
                    success:function(data){
                        console.log(data);
                    },
                    error:function(data){
                        console.log(data);
                    }
			    })
        })
    </script>
</body>
</html>
```

我们把我们的html静态文件放在服务的8000 端口下面。当请求的时候， 整个过程是这样：

前端页面发起请求 ---> 后端的8000 服务接收请求，并代理到 3001 端口。----> 3001 端口处理响应





















