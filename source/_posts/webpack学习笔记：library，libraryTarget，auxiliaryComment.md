---
title: 模块化与Webpack属性 library,libraryTarget的关联
date: 2021-02-03 14:53:43
tags: webpack学习笔记
categories: webpack学习笔记
hidden: true
---

`webpack` 是前端目前很热门的打包工具，相信大家应该在项目中都使用过。它有很复杂的配置项。其中 `library`，`libraryTarget` 属性大家可能就更陌生了，因为在一般的项目中使用 `wabpacK` 是不需要关注 这两个属性的，但是如果我们是开发类库，如果使用webpack 来打包的话，那么`library`，`libraryTarget` 这两个属性是一定要了解的

> ps： 也不一定非要使用 `webpack` 来打包， 打包的工具很多，` rollup` 之类的都是可以打包的。当然也是可以不打包的，只要你写的模块(类库)符合规范就可以。

这篇文章将从下面这几个问题来进行解答的

* 模块一般是怎么引入（有多少种方式）;
* 为什么有那么多种引入的方式呢？
* `webpack` 的` library` 和 `libraryTarget` 属性具体是用来干什么的？
* `webpack` 的 `library` 和` libraryTarget `属性 能将我们写的模块暴露成什么样的？

<!--more-->

# 模块的引入方式

当我们引用别人开发的模块时有几种引入放入的方式？，假如我们有一个开发好的模块`demo`。那么如何使用它呢？

## 传统方式：script 标签

```javascript
<script src="demo.js"></script>
<script>
  demo();
</script>
```

## AMD 方式

```javascript
require(['demo'],function(demo) {  
  demo();
})  
```

## commonJs 方式

```javascript
const demo = require('demo');
  
demo();
```

## ES6 module 方式

```javascript
import demo from 'demo';

demo();
```



为什么这些模块能支持不用的方式引用呢？ 是怎么实现的呢？

---



# 引入方式取决于模块如何导出(暴露)

上面如何引入也提出了问题，为什么模块支持不同的引入？其实这取决于模块之前的是如何导出的。

我们举个列子：

我们开发一个 模块 demo。这个模块抛出一个 `getName` 函数，函数里面打印 ”shuliqi“字符串。

基本的项目结构：

```javascript
|--src
  |--index.js
  |--index.html
|--webpack.config.js
```

执行:

```
npm init
```

本地安装 `webpack`：

```
npm install --save-dev  webpack@5.19.0 
```

安装 `webpack-cli`(目的是为了在命令行中是用 `webpack` 命令)

```
npm install -D webpack-cli
```

package.json 文件的` scripts`字段改成：

```javascript
"scripts": {
 "build": "webpack --config webpack.config.js"
},
```

index.js

```javascript
export function getName() {
  console.log('shuliqi')
}
```

webpack.config.js

```javascript
module.exports = {
  entry: './src/index.js',
  output: {
    // 打包文件名字
    filename: 'default.js'
  },
}
```

index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>demo</title>
    <script src='../dist/default.js'></script>
  </head>
  <body>
    demo 示例
  </body>
  <script>
   getName();
  </script>
</html>
```

到目前为止我们的结构目录变成：

```
|--src
  |--index.js
  |--index.html
|--webpack.config.js
|--package.json
|--package-lock.json
|-node_modules
```

然后我们在根目录执行 `webpack` 打包命令(package.json 文件 script 字段我们定义好的)

```
npm run build
```

打包完之后会出现 `dist` 文件夹，里面有 `default.js`

```
|--dist
  |--default.js
|--src
  |--index.js
  |--index.html
|--webpack.config.js
|--package.json
|--package-lock.json
|-node_modules
```

我们在浏览器打 `index.html`

{% asset_img 1.png %}

发现报错：index.html:12 Uncaught ReferenceError: getName is not defined

为什么会报错呢？

显然页面在调用 `getName`的时候，在 window 上寻找 `getName`方法，但是模块化的开发呢，是拒绝一切的全局变量的，所以在全局找不到该对象。所以使用 `script` 标签引入该模块是访问不到 `getName`方法的。

那如何根据不用的引入需要来决定如何导出模块呢？ 如果是使用` webpack` 对类库进行打包的话，那么` library`，`libraryTarget` 这两个属性是可以决定如何导出类库的。

> PS: 当然不是只有webpack 可以打包，想 rollup 之类的都是可以打包的



---



# webpack 的 library, libraryTarget

`webpac`k的`output `的配置项` library` 和` libraryTarget `可以决定如何导出模块。

## output.library

这个字段支持 `string` 和 `object`类型的值

> 注意：webpack 3.1.0 才支持 object 类型的，并且这个仅限与 libraryTarget: “umd” 时使用

`output.library` 的值被如何使用是根据 `output.libraryTarget` 的取值不同而不用.。

## output.libraryTarget

这个字段支持` string` 类型的值，。默认值: `var`

这个配置的作用是控制 `webpack` 打包的内容是如何暴露的（即上面说的如何导出的）

---



#  模块的导出方式

根据以上我们知道 `webpack` 的 `library`,` libraryTarget` 可以决定如何导出模块，那么可以导出（暴露）什么样的类库呢？ 

## 暴露为一个变量

如果将` libraryTarget` 设置为 `var`/ `assign`, webpack 会把模块返回的值（无论暴露的是什么）绑定到一个由 `output.library`指定的变量上。这两种导出方式使用**传统方式：script 标签方式** 来引入的。

### libraryTarget: "var"

这是` libraryTarget` 的默认值。当模块加载完成时候，模块的的返回值（入口起点的任何导出值）将分配到一个使用 var 声明的变量上。

```javascript
// _entry_return_: 模块的返回值
// 变量 demo 使用 var 声明 
var demo = _entry_return_;
```

**举个例子🌰**

还是基于以上例子，目录中添加一个新的配置文件  `webpack-config-var.js`

```javascript
module.exports = {
  entry: './src/index.js',
  output: {
    // 打包的文件名
    filename: 'webpack-var.js',

    // 模块的返回值分配到一个使用var 声明的变量上， 该变量由  library 决定
    library: 'demo',
    libraryTarget: 'var',
  }
}
```

package.json 的 `script` 添加一条心的打包命令

```javascript
"build-var":  "webpack --config webpack-config-var.js"
```

在根目录执行新的打包命令:

```
npm run  build-var
```

打包之后的文件 `webpack-var.js`

```javascript
var demo;
demo = (()=>{
    "use strict";
    var e = {
        138: (e,r,o)=>{
            function t() {
                console.log("shuliqi")
            }
            o.r(r),
            o.d(r, {
                getName: ()=>t
            })
        }
    }
      , r = {};
    function o(t) {
        if (r[t])
            return r[t].exports;
        var n = r[t] = {
            exports: {}
        };
        return e[t](n, n.exports, o),
        n.exports
    }
    return o.d = (e,r)=>{
        for (var t in r)
            o.o(r, t) && !o.o(e, t) && Object.defineProperty(e, t, {
                enumerable: !0,
                get: r[t]
            })
    }
    ,
    o.o = (e,r)=>Object.prototype.hasOwnProperty.call(e, r),
    o.r = e=>{
        "undefined" != typeof Symbol && Symbol.toStringTag && Object.defineProperty(e, Symbol.toStringTag, {
            value: "Module"
        }),
        Object.defineProperty(e, "__esModule", {
            value: !0
        })
    }
    ,
    o(138)
}
)();

```

打包结果：模块的返回值分配到了 使用 `var` 声明的变量 `demo` 上。

**引用模块**

可以通过 传统的引入方式（script标签）来加载使用

在原来的目录结构上` src` 目录下添加 `index-var.html`

index-var.html

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>demo</title>
     <!-- 传统引入方式：`script` 标签 -->
    <script src='../dist/webpack-var.js'></script>
  </head>
  <body>
    demo 示例
  </body>
  <script>
    console.log(demo);
    demo.getName();
  </script>
</html>
```

然后我们打开 `index.html`:

{% asset_img 2.png %}

从截图中可以看出来， 我们是能访问到暴露出的` demo` 模块 及其它 `getName`方法的。



注意： 如果没有设置 library，那么打包将会直接报错



---



### libraryTarget: "assign"

使用这个设置， `webpack`把模块返回值分配给一个没有使用 var 声明的变量，如果这个变量在没有引入作用域提前申请过，那么将会挂载在全局作用域中，**这个设置可能会重新分配全局中已经存在的值** 。

```javascript
// _entry_return_: 模块的返回值
// 变量 demo 没有使用 var 声明 
demo = _entry_return_;
```

**举个例子🌰**

还是基于以上例子，目录中添加一个新的配置文件  `webpack-config-assign.js`

```javascript
module.exports = {
  entry: './src/index.js',
  output: {
    // 打包文件名字
    filename: 'webpack-assign.js',

    // 模块的返回值将分配到一个没有使用var声明的变量上，变量由 library 控制
    library: 'demo',
    libraryTarget: 'assign',
  }
}
```

package.json 的 `script` 字段添加一条心的打包命令

```java
"build-assgin": "webpack --config webpack-config-assigin.js"
```

在根目录执行新的打包命令:

```
npm run build-assign
```

打包之后的文件 `webpack-assign.js`文件

```javascript
demo = (()=>{
    "use strict";
    var e = {
        138: (e,r,o)=>{
            function t() {
                console.log("shuliqi")
            }
            o.r(r),
            o.d(r, {
                getName: ()=>t
            })
        }
    }
      , r = {};
    function o(t) {
        if (r[t])
            return r[t].exports;
        var n = r[t] = {
            exports: {}
        };
        return e[t](n, n.exports, o),
        n.exports
    }
    return o.d = (e,r)=>{
        for (var t in r)
            o.o(r, t) && !o.o(e, t) && Object.defineProperty(e, t, {
                enumerable: !0,
                get: r[t]
            })
    }
    ,
    o.o = (e,r)=>Object.prototype.hasOwnProperty.call(e, r),
    o.r = e=>{
        "undefined" != typeof Symbol && Symbol.toStringTag && Object.defineProperty(e, Symbol.toStringTag, {
            value: "Module"
        }),
        Object.defineProperty(e, "__esModule", {
            value: !0
        })
    }
    ,
    o(138)
}
)();

```

打包结果：模块的返回值分配到了没有使用`var` 声明的变量 `demo` 上。

**引用**

如果使用的话， 可以通过 传统的引入方式（script标签）来引用。

在原来的目录结构上` src` 目录下添加 `index-assign.html`

index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>demo</title>
     <!-- 传统引入方式：`script` 标签 -->
    <script src='../dist/webpack-assign.js'></script>
  </head>
  <body>
    assign 配置
  </body>
  <script>
    console.log(demo);
    demo.getName();
  </script>
</html>
```

打开我们的 html 文件，结果如下：

{% asset_img 3.png %}

我们能打印出模块`demo`，及可以使用它的`getName`方法



**注意：**但是如果在加载模块之前已经存在变量， 那么加载模块之后，  会覆盖已存在的变量

在 `src` 目录中添加 `index.cover-assign.html`

````html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>demo</title>
    <script>
      // 全局已经存在的变量 demo
      demo = "我是demo";
      console.log(demo);
    </script>
     <!-- 传统引入方式：`script` 标签 -->
    <script src='../dist/webpack-assign.js'></script>
  </head>
  <body>
    assign 配置:
    <br>
    但是如果在加载模块之前已经存在变量， 那么加载模块之后，  会覆盖已存在的变量
  </body>
  <script>
    // 加载完我们的模块之后， 会覆盖全局中已经存在的变量 demo
    console.log(demo)
    demo.getName();
  </script>
</html>
````

打开HTML文件结果如下：

{% asset_img 4.png %}

结果说明：我们加载模块之后的 重新赋值了之前的变量 `demo`

---



## 通过对象上赋值暴露

以下 的四个设置选项 `webpack` 会将模块的返回值（入口起点的任何导出值）赋值给一个特定对象的属性上; 这个对象`output.libraryTarget` 指定，这个属性由 `output.library`指定。如果`output.library`没有指定，那么默认行为是将模块返回值的所有属性都分配到对象中。

> 注意： `output.library`没有指定的时候， webpack并不会检查对象中是否已经存在某些属性设置，即会发生覆盖行为

### libraryTarget: "this"

这个配置项表示 模块的返回值将分配给`this` 对象 的一个属性，这个属性是由`output.library`指定；如果没有指定，则模块返回值的所有属性将直接分配到 ``this`` 对象上（并且`webpack`并不会检查对象中是否已经存在某些属性设置，即会发生覆盖行为）

使用这个配置，当模块被加载时，那么模块的返回值会被分配到 this 对象的 属性（library决定属性）上 。

```javascript
// _entry_return_:模块的返回值
this['demo'] = _entry_return_;
```

**举个例子🌰**

在上面例子的基础上， 我们在根目录添加一个新的配置文件 `webpack-config-this.js`

```javascript
module.exports = {
  entry: './src/index.js',
  output: {
    // 打包后的文件名字
    filename: 'webpack-this.js',

    // 模块的返回值分配在 this 对象的 demo 属性上
    library: 'demo',
    libraryTarget: 'this',
    
  }
}
```

package.json 文件的`script`字段添加新一条新的打包命令：

```javascript
 "build-this": "webpack --config webpack-config-this.js"
```

在根目录执行 新的打包命令：

````javascript
npm run build-this
````

打包的文件 `webpack-this.js`

```javascript
this.demo = (()=>{
    "use strict";
    var e = {
        138: (e,r,t)=>{
            function o() {
                console.log("shuliqi")
            }
            t.r(r),
            t.d(r, {
                getName: ()=>o
            })
        }
    }
      , r = {};
    function t(o) {
        if (r[o])
            return r[o].exports;
        var n = r[o] = {
            exports: {}
        };
        return e[o](n, n.exports, t),
        n.exports
    }
    return t.d = (e,r)=>{
        for (var o in r)
            t.o(r, o) && !t.o(e, o) && Object.defineProperty(e, o, {
                enumerable: !0,
                get: r[o]
            })
    }
    ,
    t.o = (e,r)=>Object.prototype.hasOwnProperty.call(e, r),
    t.r = e=>{
        "undefined" != typeof Symbol && Symbol.toStringTag && Object.defineProperty(e, Symbol.toStringTag, {
            value: "Module"
        }),
        Object.defineProperty(e, "__esModule", {
            value: !0
        })
    }
    ,
    t(138)
}
)();

```

打包结果：模块的返回值分配给了`this`对象的 `demo`属性



**引用模块：**

可以通过传统的引入方式（script标签）来引用我们的模块。

`src`目录下添加 `index-this.html`

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>demo</title>
    <script src='../dist/webpack-this.js'></script>
  </head>
  <body>
   this 配置
  </body>
  <script>
    const demo = this.demo;
    console.log(demo)
    demo.getName();
  </script>
</html>
```

 在浏览器打开HTML， 结果如下：

{% asset_img 5.png %}

从截图可以看出 我们在`this`对象能找到 `demo` 模块



**举个例子🌰**

如果没有设置 library 的值，那么将模块返回值的所有属性都分配到 `this`对象中。

添加新的配置文件 `webpack-config-this-cover.js`

```javascript
const { ModuleFilenameHelpers } = require("webpack");

module.exports = {
  entry: './src/index.js',
  output: {
    filename:'webpack-this-cover.js',

    // 如果没有设置 library 的值，那么将模块返回值的所有属性都分配到 `this`对象中。
    libraryTarget: 'this',
  }
}
```

packge.json 文件的 `script` 字段添加新的打包命令：

```javascript
 "build-this-cover": "webpack --config webpack-config-this-cover.js"
```

执行我们新的打包命令：

```javascript
npm run build-this-cover
```

打包之后的文件 `webpack-this-cover.js`:

```javascript
!function(e, r) {
    for (var o in r)
        e[o] = r[o];
    r.__esModule && Object.defineProperty(e, "__esModule", {
        value: !0
    })
}(this, (()=>{
    "use strict";
    var e = {
        138: (e,r,o)=>{
            function t() {
                console.log("shuliqi")
            }
            o.r(r),
            o.d(r, {
                getName: ()=>t
            })
        }
    }
      , r = {};
    function o(t) {
        if (r[t])
            return r[t].exports;
        var n = r[t] = {
            exports: {}
        };
        return e[t](n, n.exports, o),
        n.exports
    }
    return o.d = (e,r)=>{
        for (var t in r)
            o.o(r, t) && !o.o(e, t) && Object.defineProperty(e, t, {
                enumerable: !0,
                get: r[t]
            })
    }
    ,
    o.o = (e,r)=>Object.prototype.hasOwnProperty.call(e, r),
    o.r = e=>{
        "undefined" != typeof Symbol && Symbol.toStringTag && Object.defineProperty(e, Symbol.toStringTag, {
            value: "Module"
        }),
        Object.defineProperty(e, "__esModule", {
            value: !0
        })
    }
    ,
    o(138)
}
)());

```

**引用模块**

添加新的文件 `index-this-cover.html`

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>demo</title>
    <script src='../dist/webpack-this-cover.js'></script>
  </head>
  <body>
   this 配置:
   <br>
   如果没有设置 library 的值，那么将模块返回值的所有属性都分配到 `this`对象中
  </body>
  <script>
    getName();
  </script>
</html>
```

打开 HTML 结果如下：

{% asset_img 6.png %}

结果： 没有设置 library 的值，那么将模块返回值的所有属性都分配到 `this`对象中。所有可以直接执行`getName`方法

---



### libraryTarget: "window"

这个配置项表示模块的返回值将分配给 `window` 对象的一个属性，这个属性是由`output.library`指定；如果没有指定，则模块返回值的所有属性将直接分配到 `window` 对象上（并且`webpack`并不会检查对象中是否已经存在某些属性设置，即会发生覆盖行为）

使用这个配置，当模块被加载时，那么模块的返回值会被分配到 `window` 对象的 属性（library决定属性）上 。

```javascript
// _entry_return_:模块的返回值
window['demo'] = _entry_return_;
```

**举个例子🌰**

添加新的配置文件 `webpack-config-var.js`

````javascript
module.exports = {
  entry: '/src/index.js',
  output: {
    filename:'webpack-window.js',

    // 模块的返回值分配到 window 对象的 demo 属性上
    library: 'demo',
    libraryTarget: 'window',
  }
}
````

package.json 文件的`script`字段添加新的打包命令：

```javascript
 "build-window": "webpack --config webpack-config-window.js"
```

执行新的打包命令：

```
 npm run build-window
```

打包之后的文件  `webpack-window.js`

```javascript
window.demo = (()=>{
    "use strict";
    var e = {
        138: (e,o,r)=>{
            function t() {
                console.log("shuliqi")
            }
            r.r(o),
            r.d(o, {
                getName: ()=>t
            })
        }
    }
      , o = {};
    function r(t) {
        if (o[t])
            return o[t].exports;
        var n = o[t] = {
            exports: {}
        };
        return e[t](n, n.exports, r),
        n.exports
    }
    return r.d = (e,o)=>{
        for (var t in o)
            r.o(o, t) && !r.o(e, t) && Object.defineProperty(e, t, {
                enumerable: !0,
                get: o[t]
            })
    }
    ,
    r.o = (e,o)=>Object.prototype.hasOwnProperty.call(e, o),
    r.r = e=>{
        "undefined" != typeof Symbol && Symbol.toStringTag && Object.defineProperty(e, Symbol.toStringTag, {
            value: "Module"
        }),
        Object.defineProperty(e, "__esModule", {
            value: !0
        })
    }
    ,
    r(138)
}
)();

```

打包结果：模块的返回值 分配到了` window` 的 `demo` 属性上。

**引用模块：**

可以通过传统的引入方式（script标签）来引用我们的类库。

index-window.html

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>demo</title>
    <!-- 传统引入方式：`script` 标签 -->
    <script src='./dist/main.js'></script>
  </head>
  <body>
    demo 示例
  </body>
  <script>
    demo.getName();
  </script>
</html>
```

打开index.html

{% asset_img 7.png %}

结果：我们可以直接在`window` 上获取`demo`; 说明模块模块的返回值将分配给 window 的 demo 属性上。

---



**举个例子🌰**

如果没有设置 library 的值，那么将模块返回值的所有属性都分配到 `window` 对象中。

添加一个新的配置文件：webpack-config-window-cover.js

```javascript
module.exports = {
  entry: './src/index.js',
  output: {
    filename:'webpack-window-cover.js',

    // 如果没有设置 library 的值，那么将模块返回值的所有属性都分配到 `window`对象中。
    libraryTarget: 'window',
  }
}
```

package.json 文件`script`添加新的打包命令:

```javascript
 "build-window-cover": "webpack --config webpack-config-window-cover.js"
```

执行我们新的打包命令：

```javascript
npm run build-window-cover
```

打包之后的文件

```javascript
!function(e, o) {
    for (var r in o)
        e[r] = o[r];
    o.__esModule && Object.defineProperty(e, "__esModule", {
        value: !0
    })
}(window, (()=>{
    "use strict";
    var e = {
        138: (e,o,r)=>{
            function t() {
                console.log("shuliqi")
            }
            r.r(o),
            r.d(o, {
                getName: ()=>t
            })
        }
    }
      , o = {};
    function r(t) {
        if (o[t])
            return o[t].exports;
        var n = o[t] = {
            exports: {}
        };
        return e[t](n, n.exports, r),
        n.exports
    }
    return r.d = (e,o)=>{
        for (var t in o)
            r.o(o, t) && !r.o(e, t) && Object.defineProperty(e, t, {
                enumerable: !0,
                get: o[t]
            })
    }
    ,
    r.o = (e,o)=>Object.prototype.hasOwnProperty.call(e, o),
    r.r = e=>{
        "undefined" != typeof Symbol && Symbol.toStringTag && Object.defineProperty(e, Symbol.toStringTag, {
            value: "Module"
        }),
        Object.defineProperty(e, "__esModule", {
            value: !0
        })
    }
    ,
    r(138)
}
)());

```

**引用模块**

可以通过传统的引入方式（script标签）来引用我们的类库。

index-window-cover.html

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>demo</title>
    <script src='../dist/webpack-window-cover.js'></script>
  </head>
  <body>
   window 配置
   <br>
   如果没有设置 library 的值，那么将模块返回值的所有属性都分配到 window 对象中。
  </body>
  <script>
    getName();
  </script>
</html>
```

打开HTML 文件，结果如下：

{% asset_img 8.png %}

结果说明：没有设置 library。demo 模块返回值的所有属性（例如: `getName`）都会被分配到 `window`对象中（即 我们可以直接调用 `getName`方法的原因）


### libraryTarget: "global"

这个配置项表示模块的的返回值将分配给·`global` 对象 的一个属性，这个属性是由`output.library`指定；如果没有指定，则模块返回值的所有属性将直接分配到 `global` 对象上（并且`webpack`并不会检查对象中是否已经存在某些属性设置，即会发生覆盖行为）

使用这个配置，当模块被加载时，那么模块的返回值会被分配到 `global` 对象的 属性（library决定属性）上 。

```javascript
// _entry_return_:模块的返回值
global["demo"] = _entry_return_;
```

**举个例子🌰**

添加新的配置文件 `webpack-config-global.js`

```javascript
module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'webpack-global.js',

    // 模块的的返回值将分配给 global对象 demo 属性上
    library: 'demo',
    libraryTarget: 'global',
  }
}
```

package.json 文件 `script` 添加新的打包命令：

```javascript
 "build-global": "webpack --config webpack-config-global.js"
```

执行新的打包命令：

```javascript
npm run build-global
```

打包之后的文件 `webpack-global.js`:

```javascript
self.demo = (()=>{
    "use strict";
    var e = {
        138: (e,r,o)=>{
            function t() {
                console.log("shuliqi")
            }
            o.r(r),
            o.d(r, {
                getName: ()=>t
            })
        }
    }
      , r = {};
    function o(t) {
        if (r[t])
            return r[t].exports;
        var n = r[t] = {
            exports: {}
        };
        return e[t](n, n.exports, o),
        n.exports
    }
    return o.d = (e,r)=>{
        for (var t in r)
            o.o(r, t) && !o.o(e, t) && Object.defineProperty(e, t, {
                enumerable: !0,
                get: r[t]
            })
    }
    ,
    o.o = (e,r)=>Object.prototype.hasOwnProperty.call(e, r),
    o.r = e=>{
        "undefined" != typeof Symbol && Symbol.toStringTag && Object.defineProperty(e, Symbol.toStringTag, {
            value: "Module"
        }),
        Object.defineProperty(e, "__esModule", {
            value: !0
        })
    }
    ,
    o(138)
}
)();

```

打包结果：我们的模块绑定在了 `self` 上。 有的同学可能会问， 不是该绑定在 `global` 上吗？

其实：

>通常讨论全局对象时，很少会区分全局对象是 global 对象，还是 global 对象上的属性。由于作用域链的原因，global 对象上的属性都可以直接在全局访问。不同的 JavaScript 环境存在不同的全局对象，例如浏览器中的 `window`、`self`、`location`、`navigator`、`event`，Node.js 中的 `global`、`module`、`exports`、`process` 等。访问宿主环境中不存在的全局变量会产生 `ReferenceError` 错误（即访问未声明变量）。



**引用模块**

可以通过传统的引入方式（script标签）来引用我们的类库。

index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>demo</title>
    <!-- 传统引入方式：`script` 标签 -->
    <script src='./dist/main.js'></script>
  </head>
  <body>
    demo 示例
  </body>
  <script>
    demo.getName();
  </script>
</html>
```

打开HTML 结果如下：

{% asset_img 9.png %}

结果：我们可以直接访问全局对象的 `demo` 属性。

> 通常讨论全局对象时，很少会区分全局对象是 global 对象，还是 global 对象上的属性。由于作用域链的原因，global 对象上的属性都可以直接在全局访问。不同的 JavaScript 环境存在不同的全局对象，例如浏览器中的 `window`、`self`、`location`、`navigator`、`event`，Node.js 中的 `global`、`module`、`exports`、`process` 等。访问宿主环境中不存在的全局变量会产生 `ReferenceError` 错误（即访问未声明变量）。



**举例子🌰**

如果没有设置 library 的值，那么将模块返回值的所有属性都分配到 `global` 对象中。

webpack-config-global-cover.js

```javascript
module.exports = {
  entry: './src/index.js',

  output: {
    filename: 'webpack-global-cover.js',

    // 如果没有设置 library 的值，那么将模块返回值的所有属性都分配到 `global` 对象中
    libraryTarget: 'global',
  }
}
```

packge.js 文件 `script` 字段添加 新的打包命令：

```javascript
"build-global-cover": "webpack --config webpack-config-global-cover.js"
```

执行打包命令：

```javascript
npm run build-global-cover
```

打包后的文件 `webpack-global-cover.js`

```javascript
!function(e, r) {
    for (var o in r)
        e[o] = r[o];
    r.__esModule && Object.defineProperty(e, "__esModule", {
        value: !0
    })
}(self, (()=>{
    "use strict";
    var e = {
        138: (e,r,o)=>{
            function t() {
                console.log("shuliqi")
            }
            o.r(r),
            o.d(r, {
                getName: ()=>t
            })
        }
    }
      , r = {};
    function o(t) {
        if (r[t])
            return r[t].exports;
        var n = r[t] = {
            exports: {}
        };
        return e[t](n, n.exports, o),
        n.exports
    }
    return o.d = (e,r)=>{
        for (var t in r)
            o.o(r, t) && !o.o(e, t) && Object.defineProperty(e, t, {
                enumerable: !0,
                get: r[t]
            })
    }
    ,
    o.o = (e,r)=>Object.prototype.hasOwnProperty.call(e, r),
    o.r = e=>{
        "undefined" != typeof Symbol && Symbol.toStringTag && Object.defineProperty(e, Symbol.toStringTag, {
            value: "Module"
        }),
        Object.defineProperty(e, "__esModule", {
            value: !0
        })
    }
    ,
    o(138)
}
)());

```

**引用模块**

index-global-cover.html

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>demo</title>
    <script src='../dist/webpack-global-cover.js'></script>
  </head>
  <body>
    global 配置
    <br>
    如果没有设置 library 的值，那么将模块返回值的所有属性都分配到 `global` 对象中
  </body>
  <script>
    getName();
  </script>
</html>
```

浏览器打开HTML, 结果如下：

{% asset_img 9.png %}

由截图克制： 模块返回值的 `getName`属性直接分配到了全局对象上，所有我们才能直接执行 `getName()`

### libraryTarget: ”commonjs"

将模块的返回值分配给 `exports` 对象的属性上， 这个属性由 `output.library` 指定；如果没有指定，则模块返回值的所有属性将直接分配到 `exports` 对象上（并且`webpack`并不会检查对象中是否已经存在某些属性设置，即会发生覆盖行为）

使用这个配置，当模块被加载时，那么模块的返回值会被分配到 `window` 对象的 属性（library决定属性）上 。

```javascript
// _entry_return_:模块的返回值
exports["demo"] = _entry_return_;
```

**举个例子🌰**

添加新的配置文件 `webpack-config-global.js`

```javascript
module.exports = {
  entry: './src/index.js',
  output: {
    filename:'webpack-commonjs.js',

    // 模块的返回值分配给 `exports` 对象的属性上， 这个属性由 `output.library` 指定
    library: 'demo',
    libraryTarget: 'commonjs',
  }
}
```

packge.js 文件 `script` 字段添加 新的打包命令：

```javascript
"build-commonjs": "webpack --config webpack-config-commonjs.js"
```

执行新的打包命令：

```javascript
npm run build-commonjs
```

打包之后的文件 `webpack-commonjs.js`；

```javascript
exports.demo = (()=>{
    "use strict";
    var e = {
        138: (e,r,o)=>{
            function t() {
                console.log("shuliqi")
            }
            o.r(r),
            o.d(r, {
                getName: ()=>t
            })
        }
    }
      , r = {};
    function o(t) {
        if (r[t])
            return r[t].exports;
        var n = r[t] = {
            exports: {}
        };
        return e[t](n, n.exports, o),
        n.exports
    }
    return o.d = (e,r)=>{
        for (var t in r)
            o.o(r, t) && !o.o(e, t) && Object.defineProperty(e, t, {
                enumerable: !0,
                get: r[t]
            })
    }
    ,
    o.o = (e,r)=>Object.prototype.hasOwnProperty.call(e, r),
    o.r = e=>{
        "undefined" != typeof Symbol && Symbol.toStringTag && Object.defineProperty(e, Symbol.toStringTag, {
            value: "Module"
        }),
        Object.defineProperty(e, "__esModule", {
            value: !0
        })
    }
    ,
    o(138)
}
)();

```

从打包结果可以看出，我们的 模块 `demo` 挂载在了 `exports` 对象上

**引用模块**

引用的话，需要在 CommonJS 环境中

在 `src` 目录中添加 `test-commonjs.js`

```javascript
const my_export = require('./dist/main.js');
const demo = my_export.demo;
console.log(demo)
demo.getName();
```

 然后我们在 CommonJS 环境下执行这个代码（可以在 vscode 编辑器中执行，安装run-code 插件就可以执行); 

执行的结果为：

{% asset_img 11.png %}



**举个例子🌰**

如果没有设置 library 的值，那么将模块返回值的所有属性都分配到 `exports` 对象中。

webpack-config-commonjs-cover.js

````javascript
module.exports = {
  entry: './src/index.js',

  output: {
    filename: 'webpack-commonjs-cover.js',

    // 如果没有设置 library 的值，那么将模块返回值的所有属性都分配到 `exports` 对象中。
    libraryTarget: 'commonjs'
  }
}
````

packge.js 文件 `script` 字段添加 新的打包命令：

```javascript
  "build-commonjs-cover": "webpack --config webpack-config-commonjs-cover.js"
```

执行新的打包命令：

```javascript
npm run build-commonjs-cover
```

打包之后的文件（dist/webpack-commonjs-cover.js）

```javascript
!function(e, r) {
    for (var o in r)
        e[o] = r[o];
    r.__esModule && Object.defineProperty(e, "__esModule", {
        value: !0
    })
}(exports, (()=>{
    "use strict";
    var e = {
        138: (e,r,o)=>{
            function t() {
                console.log("shuliqi")
            }
            o.r(r),
            o.d(r, {
                getName: ()=>t
            })
        }
    }
      , r = {};
    function o(t) {
        if (r[t])
            return r[t].exports;
        var n = r[t] = {
            exports: {}
        };
        return e[t](n, n.exports, o),
        n.exports
    }
    return o.d = (e,r)=>{
        for (var t in r)
            o.o(r, t) && !o.o(e, t) && Object.defineProperty(e, t, {
                enumerable: !0,
                get: r[t]
            })
    }
    ,
    o.o = (e,r)=>Object.prototype.hasOwnProperty.call(e, r),
    o.r = e=>{
        "undefined" != typeof Symbol && Symbol.toStringTag && Object.defineProperty(e, Symbol.toStringTag, {
            value: "Module"
        }),
        Object.defineProperty(e, "__esModule", {
            value: !0
        })
    }
    ,
    o(138)
}
)());

```

结果说明：没有设置 library 的值，那么将模块返回值的所有属性都分配到对象 `export` 中

**引用模块**

src/test-commonjs-cover.js

```javascript
const demo = require('../dist/webpack-commonjs-cover.js');
console.log(demo)
demo.getName()
```

结果如下：

{% asset_img 12.png %}

可以看出来:`require` 的结果就是 `demo`模块本身，可知没有设置 library 的值，那么将模块返回值的所有属性都分配到对象 `export` 中

---



## 模块定义系统

这些选项将产生一个包含更完整兼容代码的包，以确保各个模块系统的兼容性。这时候 `output.library` 选项在不同的 `output.libraryTarget`选项中具有不同的含义。

### libraryTarget: "commonjs2"

这个选项配置是指将模块的返回值（入口的返回值）分配给 `module.exports`对象上的属性， 这个属性由`library`指定; 如果没有指定，模块的返回值直接分配给 `module.exports `对象。

正如名字所指可以在使用在 `CommonJS` 环境。

> 有没有发现commonjs 和 commonjs2 长的很像？ 其实他们确实很相似， 但是也有一些很微妙的区别的，可以点击这个 [issue](https://github.com/webpack/webpack/issues/1114) 了解了解

使用这个配置，当模块被加载时，那么模块的返回值会被分配给 `module.exports`对象，

```javascript
// _entry_return_: 模块的返回值
// 打包方式结果
module.exports = _entry_return_;

// 使用方式: 使用加载器来使用
// 如：在commonJS 环境中使用 require 来加载模块
const demo = require("demo");
demo.getName();
```

注意，这种情况下`output.library` 不能与 `output.libraryTarget` 一起使用。即使使用也会被忽略掉。具体原因请参照[此 issue](https://github.com/webpack/webpack/issues/11800)。

**举个例子🌰**

- 模块的返回值（入口的返回值）分配给 `module.exports`对象上的属性, 这个属性由`library`指定

  

新加配置文件 `webpack-config-commonjs2.js`

```javascript
module.exports = {
  entry: './src/index.js',
  output: {
    library:'demo',
    libraryTarget: 'commonjs2',
  },
}
```

package.json 添加新的打包命令：

```javascript
 "build-commonjs2": "webpack --config webpack-config-commonjs2.js"
```

执行我们新打包的命令：

```javascript
npm run build-commonjs2
```

打包之后的文件 `dist/webpack-commonjs2.js`

```javascript
module.exports.demo = (()=>{
    "use strict";
    var e = {
        138: (e,o,r)=>{
            function t() {
                console.log("shuliqi")
            }
            r.r(o),
            r.d(o, {
                getName: ()=>t
            })
        }
    }
      , o = {};
    function r(t) {
        if (o[t])
            return o[t].exports;
        var n = o[t] = {
            exports: {}
        };
        return e[t](n, n.exports, r),
        n.exports
    }
    return r.d = (e,o)=>{
        for (var t in o)
            r.o(o, t) && !r.o(e, t) && Object.defineProperty(e, t, {
                enumerable: !0,
                get: o[t]
            })
    }
    ,
    r.o = (e,o)=>Object.prototype.hasOwnProperty.call(e, o),
    r.r = e=>{
        "undefined" != typeof Symbol && Symbol.toStringTag && Object.defineProperty(e, Symbol.toStringTag, {
            value: "Module"
        }),
        Object.defineProperty(e, "__esModule", {
            value: !0
        })
    }
    ,
    r(138)
}
)();
```

打包结果可以看出来：我们的模块 `demo`挂载在我们的`nodule.exports` 对象上。

**引用模块**

可以使用加载器来加载模块使用；如 requireJS等

`src/test-commonjs2.js`

```javascript
const my_module_exports = require('../dist/webpack-commonjs2.js');
const demo = my_module_exports.demo;
console.log(my_module_exports);
console.log(demo);
demo.getName()
```

执行结果:

{% asset_img 13.png %}

结果说明：我们的 `demo`模块 挂载在了 `nodule.exports`对象上

**举个例子🌰**

如果没有指定 `library`， 那么模块的返回值将直接分配到 `module.exports`上。

`webpack-config-commonjs2-cover.js`

```javascript
module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'webpack-commonjs2-cover.js',
    
    // 如果没有指定 `library`， 那么模块的返回值将直接分配到 `module.exports`上。
    libraryTarget: 'commonjs2',
  }
}
```

package.json 添加新的打包命令：

```javascript
"build-commonjs2-cover": "webpack --config webpack-config-commonjs2-cover.js"
```

执行我们新打包的命令：

```javascript
npm run build-commonjs2-cover
```

打包之后的文件 `dist/webpack-commonjs2-cover.js`

```javascript
module.exports = (()=>{
    "use strict";
    var e = {
        138: (e,r,o)=>{
            function t() {
                console.log("shuliqi")
            }
            o.r(r),
            o.d(r, {
                getName: ()=>t
            })
        }
    }
      , r = {};
    function o(t) {
        if (r[t])
            return r[t].exports;
        var n = r[t] = {
            exports: {}
        };
        return e[t](n, n.exports, o),
        n.exports
    }
    return o.d = (e,r)=>{
        for (var t in r)
            o.o(r, t) && !o.o(e, t) && Object.defineProperty(e, t, {
                enumerable: !0,
                get: r[t]
            })
    }
    ,
    o.o = (e,r)=>Object.prototype.hasOwnProperty.call(e, r),
    o.r = e=>{
        "undefined" != typeof Symbol && Symbol.toStringTag && Object.defineProperty(e, Symbol.toStringTag, {
            value: "Module"
        }),
        Object.defineProperty(e, "__esModule", {
            value: !0
        })
    }
    ,
    o(138)
}
)();

```

打包结果： 可以看出来， 我们模块的返回值直接分配给了 `module.exports`。

**使用模块**

`src/test-commonjs2-cover.js`

```javascript
const demo = require('../dist/webpack-commonjs2-cover.js');
console.log(demo);
demo.getName()
```

在 commonJS环境中执行，结果如下：

{% asset_img 14.png %}

结果说明： 我们`require`d的结果就是 `demo`模块本身。

---



### libraryTarget: "amd"

这个选项将模块的返回值作为 `AMD`模块导出。`AMD`模块要求输入脚本被定义为具有特定属性，通常通过 RequireJS 或者任何的加载器（例如almond）提供的 `reuqire` 和 `define`属性。否则直接加载生成的AMD 模块捆绑包将会导致 `define is not defined`的错误。

**注意：**这一小节的内容和网上的文章会有所不同，那是因为 `webpack` 版本的原因。 网上的教程的 webpack4 以下版本， 我们这个是 webpack5.19.1 版本的， 是有一些不一样的。

如果定义了`output.library`，打包定义成的代码将会是如下：

```javascript
// _entry_return_： 模块的返回值
// 打包方式的结果
define('demo', [], function() =>{
	return _entry_return_;
})
```

如果`output.library`没有定义有效值，那么生成的代码将如下：

```javascript
define([], function() {
  return _entry_return_;
});
```

> 建议是使用不定义 `output.library`值

使用可以通过 模块加载器来加载使用（具体使用方式看下面的例子）：

```javascript
require(['demo'], function(myDemo) {
 //... dosomething
});
```

**举个例子🌰**

* `output.library`没有定义有效值

`webpack-config-amd-cover.js`

```javascript
module.exports = {
  entry: './src/index.js',

  output: {
    filename: 'webpack-amd-cover.js',
    // 没有定义 library 
    libraryTarget: 'amd',
  }
}
```

package.json 添加新的打包命令：

```javascript
"build-amd-cover": "webpack --config webpack-config-amd-cover.js"
```

执行新的打包命令：

```javascript
npm run build-amd-cover
```

打包之后的文件 `dist/webpack-amd-cover.js`

````javascript
define((()=>(()=>{
    "use strict";
    var e = {
        138: (e,r,t)=>{
            function o() {
                console.log("shuliqi")
            }
            t.r(r),
            t.d(r, {
                getName: ()=>o
            })
        }
    }
      , r = {};
    function t(o) {
        if (r[o])
            return r[o].exports;
        var n = r[o] = {
            exports: {}
        };
        return e[o](n, n.exports, t),
        n.exports
    }
    return t.d = (e,r)=>{
        for (var o in r)
            t.o(r, o) && !t.o(e, o) && Object.defineProperty(e, o, {
                enumerable: !0,
                get: r[o]
            })
    }
    ,
    t.o = (e,r)=>Object.prototype.hasOwnProperty.call(e, r),
    t.r = e=>{
        "undefined" != typeof Symbol && Symbol.toStringTag && Object.defineProperty(e, Symbol.toStringTag, {
            value: "Module"
        }),
        Object.defineProperty(e, "__esModule", {
            value: !0
        })
    }
    ,
    t(138)
}
)()));

````

**引用模块**

我们知道 AMD 模块 是为了能在客户端使用的，所以可以通过 加载器来加载使用（requireJS等加载器）

`src/index-amd-cover.html`

```html
<!DOCTYPE html>
<html>
  <head>
    <title>AMD 模块的引用</title>
     <!-- 引入require.js并指定js主文件的入口 -->
     <script src="https://requirejs.org/docs/release/2.3.6/minified/require.js"></script>
     <script>
      require(['../dist/webpack-amd-cover.js'], function(demo) {
        console.log(demo,'加载完成了')
      })
    </script>
  </head>
  <body>
    amd 配置:
    <br>
    没有定义了 output.library 的值
  </body>
</html>
```

打开HTML文件， 结果如下：

{% asset_img 15.png %}

> 注意： 没有定义 library 的情况下，我们加载完 模块之后， 是可以得到模块（html 的 demo 参数是有值的），所以我们是可以使用模块的， 但是定义了 library 就取不到了，

**举个例子🌰**

- `output.library`定义有效值

`webpack-config-amd.js`

````javascript
module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'webpack-amd.js',

    // library 定义有效值
    library: 'demo',
    libraryTarget: 'amd',
  }
}
````

package.json 添加新的打包命令:

```javascript
"build-amd": "webpack --config webpack-config-amd.js"
```

执行我们新的打包命令：

```
npm run build-amd
```



打包之后的文件`dist/webpack-amd.js`

```javascript
define("demo", [], (()=>(()=>{
    "use strict";
    var e = {
        138: (e,r,o)=>{
            function t() {
                console.log("shuliqi")
            }
            o.r(r),
            o.d(r, {
                getName: ()=>t
            })
        }
    }
      , r = {};
    function o(t) {
        if (r[t])
            return r[t].exports;
        var n = r[t] = {
            exports: {}
        };
        return e[t](n, n.exports, o),
        n.exports
    }
    return o.d = (e,r)=>{
        for (var t in r)
            o.o(r, t) && !o.o(e, t) && Object.defineProperty(e, t, {
                enumerable: !0,
                get: r[t]
            })
    }
    ,
    o.o = (e,r)=>Object.prototype.hasOwnProperty.call(e, r),
    o.r = e=>{
        "undefined" != typeof Symbol && Symbol.toStringTag && Object.defineProperty(e, Symbol.toStringTag, {
            value: "Module"
        }),
        Object.defineProperty(e, "__esModule", {
            value: !0
        })
    }
    ,
    o(138)
}
)()));

```

打包结果说明：模块的返回值作为 AMD 模块导出。

**引用模块**

我们知道 AMD 模块 是为了能在客户端使用的，所以可以通过 加载器来加载使用（requireJS等加载器）

`src/index-amd.html`

```html
<!DOCTYPE html>
<html>
  <head>
    <title>AMD 模块的引用</title>
     <!-- 引入require.js并指定js主文件的入口 -->
     <script src="https://requirejs.org/docs/release/2.3.6/minified/require.js"></script>
     <script>
      require(['../dist/webpack-amd.js'], function(demo) {
        console.log(demo,'加载完成了')
      })
    </script>
  </head>
  <body>
    amd 配置:
    <br>
    定义了 output.library 的值
  </body>
</html>
```

运行我们的 HTML 文件， 结果是打印出： undefined "加载完成了"

> 注意： 使用定义了 library 值的模块， 我们在模块加载之后是回调参数是 undefined。没法使用模块。使用这种方式，看自己的场景来使用吧。

**举个例子🌰**

### libraryTarget: "umd"

这个选项表示将模块的返回值导出为` AMD模式` 和 导出到`global 对象`上，所以它能在 AMD 环境中运行、在浏览器充当全局对象的属性。

注意： 这时候的library 是必须的。

**举个例子🌰**

`webpack-config-umd.js`

```javascript
module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'webpack-umd.js',
    // 模块的返回值导出为` AMD模式` 和 导出到`global 对象`上，所以它能在 AMD 环境中运行、在浏览器充当全局对象的属性
    library: 'demo',
    libraryTarget: 'umd',
  }
}
```

package.json 添加新的打包命令：

```javascript
"build-umd": "webpack --config webpack-config-umd.js"
```

执行新的打包方式：

```
npon run build-umd
```

打包之后的文件 `dist/webpack-umd.js`

```javascript
!function(e, o) {
    "object" == typeof exports && "object" == typeof module ? module.exports = o() : "function" == typeof define && define.amd ? define([], o) : "object" == typeof exports ? exports.demo = o() : e.demo = o()
}(self, (function() {
    return (()=>{
        "use strict";
        var e = {
            138: (e,o,t)=>{
                function r() {
                    console.log("shuliqi")
                }
                t.r(o),
                t.d(o, {
                    getName: ()=>r
                })
            }
        }
          , o = {};
        function t(r) {
            if (o[r])
                return o[r].exports;
            var n = o[r] = {
                exports: {}
            };
            return e[r](n, n.exports, t),
            n.exports
        }
        return t.d = (e,o)=>{
            for (var r in o)
                t.o(o, r) && !t.o(e, r) && Object.defineProperty(e, r, {
                    enumerable: !0,
                    get: o[r]
                })
        }
        ,
        t.o = (e,o)=>Object.prototype.hasOwnProperty.call(e, o),
        t.r = e=>{
            "undefined" != typeof Symbol && Symbol.toStringTag && Object.defineProperty(e, Symbol.toStringTag, {
                value: "Module"
            }),
            Object.defineProperty(e, "__esModule", {
                value: !0
            })
        }
        ,
        t(138)
    }
    )()
}
));

```

打包结果：封装了两种模式（amd 和 global）这两种方式

**引用模块**

* 加载器加载使用(requireJS)

`src/index-umd.html`

```html
<!DOCTYPE html>
<html>
  <head>
    <title>umd 使用 requireJs 加载器加载</title>
     <!-- 引入require.js并指定js主文件的入口 -->
     <script src="https://requirejs.org/docs/release/2.3.6/minified/require.js"></script>
     <script>
      require(['../dist/webpack-umd.js'], function(demo) {
        console.log(demo);
        demo.getName()
      })
    </script>
  </head>
  <body>
    umd 配置:
    <br>
    使用 requireJs 加载器加载
  </body>
</html>
```

浏览器打开 html 文件， 结果如下：

{% asset_img 16.png %}

结果说明：加载我们的模块之后， 可以使用我们的模块（`demo.getName()`）

- script标签 方式引入

`index-umd-cover.html`

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>umd配置: script 标签引入方式</title>
    <!-- 传统银土方式：script 标签引入 -->
    <script src='../dist/webpack-umd.js'></script>
  </head>
  <body>
    umd配置: 
    <br>
    script 标签引入方式
  </body>
  <script>
  demo.getName();
  </script>
</html>
```

打开html，结果如下：

{% asset_img 17.png %}

说明：`demo` 模块是挂载在全局变量上面了， 我们才可以直接使用 `demo.getName()`

---



剩下还有两中暴露方式（暴露`jsonp`，暴露为` system`）， 我自己没怎么看懂如何使用，等我懂的时候再补充上来吧！！

上面的例子 的所有代码都是 上传到 `github` 上[webpack-libraryTarget](https://github.com/shuliqi/webpack-libraryTarget)，如果有需要， 可以下载。

## 参考文章：

[详解webpack的out.libraryTarget属性](https://www.xlaoyu.info/2018/01/05/webpack-output-librarytarget/)

[output.libraryTarget ](https://webpack.docschina.org/configuration/output/#outputlibrarytarget)

[【深入理解webpack】library,libraryTarget,externals的区别及作用](https://juejin.cn/post/6844903618081095688)

