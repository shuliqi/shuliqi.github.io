---
title: vue项目首屏加载优化
date: 2021-06-21 17:27:24
tags: Vue
categories:  Vue
---

最近这几个月一直在开发一个使用 `vue-cli`搭建起来的项目。最近打包上线之后。发现首屏打开特别慢，在网络好的情况下大约需要4s 至 5 s。 在网络不好的情况下，还需要7s 8 s。加载的期间一直显示白屏，导致用户的体验非常不好。所以针对这个问题来做一些优化；期望的结果是首屏加载得快一点。白屏缩短。

 <!--more-->

# 存在的问题

首先我们来看没有优化之前的耗时时间：

{% asset_img 2.png %}

在网络比较好的情况下， 耗时：6.86s。 平均是 7s 左右。

# 解决的办法

我们知道 `vue`， `react` 等框架都是`js` 渲染的`html`。是典型的单页应用，首次加载耗时多，因此优化`Vue`项目首屏加载对于提升用户体验非常重要。所以必须要等到这个` js `文件加载完成后界面才会显示。

## 路由懒加载

[[Vue Router路由懒加载]](https://router.vuejs.org/zh/guide/advanced/lazy-loading.html)

`vue-router`实现路由懒加载的方法有哪些？首先我们先来看`vue-router`的一些官方解释

### 什么是路由懒加载？

> 也叫延迟加载，即在需要的时候进行加载，随用随载。

### 为什么需要懒加载呢？

- 首先，我们知道路由中通常会定义很多不同的页面。
- 这个页面这项目build打包后，一般情况下，会放在一个单独的js文件中
- 但是，如果很多的页面都放在同一个js文件中，必然会造成这个页面非常大
- 如果我们一次性的从服务器中请求下来这个页面，可能会花费一定时间，用户体验不好

在我们的vue 项目中：

- 像vue这种单页面应用，如果没有应用懒加载，运用`webpack`打包后的文件将会异常的大。

- 造成进入首页时，需要加载的内容过多，时间过长，会出啊先长时间的白屏，即使做了`loading`也是不利于用户体验。

- 而运用懒加载则可以将页面进行划分，需要的时候加载页面，可以有效的分担首页所承担的加载压力，减少首页加载用时

总的来说：进入页面不用也不需要一次性加载过多资源造成加载时间过程！

### 路由懒加载的原理

- 主要作用是将路由对应的组件打包成一个个的js代码块

- 只有在这个路由被访问到的时候，才加载对应的组件，否则不加载！

总结的来说：只有在这个路由被访问到的时候，才加载对应的组件，否则不加载！

路由懒加载，在访问到当前页面才会加载相关的资源，异步方式分模块加载文件

### 如何实现路由懒加载？

- `Vue`异步组件
- `ES6`标准语法`import（）`---------推荐使用
- `webpac`k的`require`，`ensure()`

我们使用`ES6`标准语法`import（）`来实现懒加载



没有用到路由加载懒加载之前是这么写的

```javascript
import page404 from "@/views/error/page404.vue";

const routes = [
  {
    path: "/page404",
    name: "page404",
    meta: {
      title: "404"
    },
    components: {
      slideMenu: slideMenu,
      topBar: topBar,
      content: page404
    }
  },
];
export default routes;
```

使用路由懒加载：

```javascript
const page404 = () => import("@/views/error/page404.vue");
```



## 压缩图片

我们先看打包之后下面的img 的图片的大小：

{% asset_img 7.png %}

总的5.6M。 有点大

我们使用 `vue inspect > output.js`导出` vue-cli` 做的的默认`webpoack`配置：

{% asset_img 8.png %}

发现对于图片， 只用到了 url-loader 。 相对一些比较大的图片。是可以进行压缩的。可以使用 [image-webpack-loader](https://www.npmjs.com/package/image-webpack-loader)

我们在 `vue-config.js`配置：

```javascript
const chainWebpack = function chainWebpacks (config) {
  config.module
    .rule("images")
    .use("image-webpack-loader")
    .loader("image-webpack-loader")
    .options({
      bypassOnDebug: true
    })
    .end();
};

```

然后再进行`npm run build`打包在看看`img`:

{% asset_img 9.png %}

图片的大小从 5.8M 变到了 1.8M。 感觉效果还是明显的。



经过上面这两步优化，我们再看来首屏加载的时间：

{% asset_img 18.png %}

首页加载的数据的耗时明显减少了大概 1/2 时间， 棒棒哒

## gzip 压缩

{% asset_img 11.png %}

从上面我们可以看出:`vendor-chunks.js` 很大。当我们的项部署了之后， 我们的资源文件请求会保持原来的大小。如果文件过大，并且很多的情况下，会导致网络请求耗时。严重点可能会阻塞后面的进程。针对这样的情况， 我们有没有什么比较好的解决方法呢?  有的， 那就进行 `gzip`压缩。

`gzip`压缩有两种方式：

- 服务器压缩文件
- 前端 webpack 打包生成 gz 文件

那我们先来看看这两种方式：

### 服务器压缩文件

这种方式是浏览器请求文件时，服务器对该文件进行压缩后传输给浏览器。前端不用做任何的配置，不需要 `webpack`生成 `.gz`文件。而是服务器自己处理。就拿 [Nginx](https://zh.wikipedia.org/wiki/Nginx) 来举例，我么打开 nginx.conf 文件， 会有默认配置，默认的   `#gzip  on;`即不打开。

**nginx 文件结构**

```nginx
...              # 全局块

events {         # events块
   ...
}

http      # http块
{
    ...   # http全局块
    server        # server块
    { 
        ...       # server全局块
        location [PATTERN]   # location块
        {
            ...
        }
        location [PATTERN] 
        {
            ...
        }
    }
    server
    {
      ...
    }
    ...     # http全局块
}
```

在 http 块这里开启 `gzip`和相关的配置：

```nginx
http {
		 # ... 已省略
	   # 开启gzip
    gzip  on;
    
    # 设置缓冲区大小
    gzip_buffers 4 16k;
    
    #压缩级别官网建议是6
    gzip_comp_level 6;
 
    #压缩的类型
    gzip_types text/plain application/javascript text/css application/xml text/javascript application/x-httpd-php;
    
    # ... 已省略
}

```

这种方案的特点：使用`nginx`在线`gzip`，缺点就是耗性能，需要实时压缩，但是`vue`打包后的文件体积小。

### 前端 webpack 打包生成 gz 文件

这次优化主要是采用这种方式。

这种方式是打包的时候通过 `webpack`配置生成对应的`.gz`文件，浏览器请求文件时，服务器返回相应的的文件的 `.gz `文件。

安装 `compression-webpack-plugin`

```javascript
npm i compression-webpack-plugin -D
```

然后再`vue.config.js`中设置

```javascript
const CompressionPlugin = require("compression-webpack-plugin");

const productionGzipExtensions = /\.(js|css|json|txt|html|ico|svg)(\?.*)?$/i;
// ... 已省略
plugins: [
     // ... 已省略
      new CompressionPlugin({
        test: productionGzipExtensions, // 所有匹配此{RegExp}的资产都会被处理
        threshold: 512, // 只处理大于此大小的资产。以字节为单位
        minRatio: 0.8, // 只有压缩好这个比率的资产才能被处理
        deleteOriginalAssets: false // 是否删除未压缩的源文件，谨慎设置，如果希望提供非gzip的资源，可不设置或者设置为false（比如删除打包后的gz后还可以加载到原始资源文件）
      })
    ]
```

 启用gzip压缩打包之后，会变成下面这样，自动生成`gz`包。目前大部分主流浏览器客户端都是支持gzip的，就算小部分非主流浏览器不支持也不用担心，不支持gzip格式文件的会默认访问源文件的，所以不要配置清除源文件。所以这时候打包的总体积会变大， 是因为我们没有删除源文件。是为了防止有些浏览器不支持的时候能返回源文件。

{% asset_img 10.png %}

上面`test`匹配的压缩文件类型， 并没有对图片进行压缩，因为图片压缩并不能实际减少文件大小，反而会导致打包后生成很多同大小的gz文件，得不偿失。

这种方式是浏览器在请求资源时，服务器返回相应的 `.gz` 文件。 所以需要在服务器配置一个属性， 期望它能够正常返回我们需要的`.gz`文件

`ginx`举例（`nginx.conf`文件）:

```nginx
http {
  # ...已省略
  # 静态加载本地的gz文件。
  gzip_static on;
}
```

其中`gzip_static on`这个属性是静态加载本地的gz文件

我们先来看采用这种方法前的请求的`chunk-vendors.js`的大小：

{% asset_img 11.png %}

我们可以看到请求的这个文件大小有 5.4 MB。

我们采用`gzip`压缩之后，请求该文件的大小：

{% asset_img 12.png %}

可以看出来，请求文件的大小从 5.4MB 变成了 854KB。 而首页的加载时间较少的幅度不是很大， 但也是减少了。

nginx配置了静态gz加载后，浏览器也返回的是gz文件，这样就会请求小文件而不会导致请求卡线程，并且，因为保留了源文件，所以当我们删除gz后，浏览器会自动去请求原始文件，而不会导致界面出现任何问题

 静态加载gz文件主要是依托于下面的请求头：

{% asset_img 13.png %}

这种优化的主要特点：` webpack`打包，然后直接使用静态的`gz`，缺点就是打包后文件体积太大，但是不耗服务器性能。

## Webpack打包体积优化

{% asset_img 14.png %}

从图上我们发现首屏加载过程中总公发起了149次请求。这显然是不友好的。那我们可以考虑考虑减少首屏加载的请求次数。

减少首屏加载请求次数可以从下面这个方面入手：

- 使用`splitChunks`分离代码并实现相关模块共享，最终的目的就是减少请求资源的大小和请求次数

### splitChunks

SplitChunks插件是什么呢，简单的来说就是Webpack中一个提取或分离代码的插件，主要作用是提取公共代码，防止代码被重复打包，拆分过大的js文件，合并零散的js文件。  在Webpack出现前，提取公共代码是人为去处理，而SplitChunks插件的作用就是通过配置让Webpack去帮你提取公共代码

用SplitChunks插件来控制Webpack打包生成的js文件的内容的精髓就在于，防止模块被重复打包，拆分过大的js文件，合并零散的js文件。 最终的目的就是减少请求资源的大小和请求次数

我们先来看`splitChunks`的一下字段说明吧

```javascript
const path = require("path");
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
module.exports = {
	// ...
  optimization: {
    splitChunks: {
      // 表示选择哪些 chunks 进行分割，可选值有：async，initial和all, 默认：async
      chunks: "all",

      // 表示新分离出的chunk必须大于等于minSize，默认为30000，约30kb。
      minSize: 3000,

      // 表示一个模块至少应被minChunks个chunk所包含才能分割。默认为1。
      minChunks: 1,

      // 表示按需加载文件时，并行请求的最大数目。默认为5。
      maxAsyncRequests: 1,

      // 表示加载入口文件时，并行请求的最大数目。默认为3。
      maxInitialRequests: 3,

      // // 表示拆分出的chunk的名称连接符。默认为~。如chunk~vendors.js
      automaticNameDelimiter: "~",

      // // 设置chunk的文件名。默认为true。当为true时，splitChunks基于chunk和cacheGroups的key自动命名。
      // name: true,

      // 缓存组： cacheGroups 的配置项跟 splitChunks是一样的， 但是它自己有几个自己的配置项
      cacheGroups: {
        vender: {
          // 优先级：数字越大优先级越高，因为默认值为0，所以自定义的一般是负数形式
          priority: -10,
          // test:可以是一个函数也可以是一个正则，函数的返回值是：boolean RegExp string，通过返回值或者正则来进行匹配。
          test: /[\\/]node_modules[\\/]/,
        },
        default: {
          // type: "json",
          minChunks: 2,
          priority: -20,
           // 这个的作用是当前的chunk如果包含了从main里面分离出来的模块，则重用这个模块，这样的问题是会影响chunk的名称。
          reuseExistingChunk: true
        }
      }
    }
  }
}
```



### splitChunks优化  

使用[webpack-bundle-analyzer](https://www.npmjs.com/package/webpack-bundle-analyzer) 进行体积分析。该插件可生成依赖包形成可视化分析图谱，帮组开发者分析项目结构

安装：

```javascript
npm install --save-dev webpack-bundle-analyzer
```

将插件添加到`webpack`中，因为使用的是vue-cli，所以应在`vue.config.js`中添加配置：

```javascript
const BundleAnalyzerPlugin = require("webpack-bundle-analyzer").BundleAnalyzerPlugin;

const DEV = process.env.NODE_ENV !== "production";

const chainWebpack = function chainWebpacks (config) {
	// ...
  if (DEV) {
    // 开发环境使用
    config.plugin("compressionPlugin")
      .use(new BundleAnalyzerPlugin())
      .end();
  };
};
```

运行`npm run build`生成分析页面

{% asset_img 15.png %}

从分析页面可以看出 `chunk-vendor`很大。因为这个`chunk`是项目所有的依赖库，从它是打包了`node_modules`可以看出，所以很影响性能。从图中发现`monaco-editor` 在线编辑器占了很大体积。 我们可以把它抽离出来。因为用的地方太多了， 导致很难做成按需加载了的了。

在`vue.config.js`配置：

```javascript
  config.optimization.splitChunks({
    cacheGroups: {
      common: {
        name: "chunk-common",
        chunks: "all",
        minChunks: 2,
        maxInitialRequests: 5,
        minSize: 0,
        priority: 1,
        reuseExistingChunk: true
      },
      vendors: {
        name: "chunk-vendors",
        test: /[\\/]node_modules[\\/]/,
        chunks: "all",
        priority: 2,
        reuseExistingChunk: true,
        enforce: true
      },
      // 增加一个echarts cacheGroup
      echarts: {
        name: "chunk-echarts",
        test: /[\\/]node_modules[\\/]echarts[\\/]/,
        priority: 3,
        chunks: "all",
        reuseExistingChunk: true,
        enforce: true
      },
      // 增加一个monacoEditor cacheGroup
      monacoEditor: {
        name: "chunk-monaco-editor",
        test: /[\\/]node_modules[\\/]monaco-editor[\\/]/,
        chunks: "all",
        priority: 4,
        reuseExistingChunk: true,
        enforce: true
      }
    }
  });
```

我们再来看下分析图：

{% asset_img 16.png %}

可以看出`monaco-editor`和`echarts`被单独打包了。

现在我们来看最终的结果：

{% asset_img 17.png %}

请求次数从 149 减到了  115 次。可观的是首页加载耗时减到了 1.74s。 平均值大概 1.5 s。 表示还是可观的。

注意： `SplitChunks`插件对代码作更细致的拆分 需要注意的减少请求数必然使得单个文件体积变大,二者是矛盾的,最佳实践是取得一个中庸的值平衡优劣.



# 时间轴回放

| 优化方法            | 请求数量 | 耗时时间 |
| ------------------- | -------- | -------- |
| 未做任何优化        | 149次    | 6.86s    |
| 路由懒加载+压缩图片 | 149次    | 4.26s    |
| splitChunks分离代码 | 115次    | 1.74s    |

