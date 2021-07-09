---
title: webpack学习--概念以及配置项
date: 2020-05-15 12:47:25
tags: webpack学习笔记
categories: webpack学习笔记
hidden: true
---

在平常的工作中，经常使用到`webpack`，大多数是脚手架直接帮我们配置好。一直想好好从零开始学习`webpack`。减少对脚手架的依赖。

这篇博客将先学习基础的概念， 然后看`webpack`的配置项，由于有些配置项很长， 所以会分开写，但是总体的配置项还是会在这里显示。

所以会导致这篇文章持续待续的状态

 <!--more-->

# 概念

`webpack` 是一个用于现在`JavaScript` 应用程序的静态模块打包工具，当`webpack`处理应用程序时，会在内部构建一个依赖图，该依赖图对应映射到项目所需的每个模块，最后生成一个或者多个 bundle。

webpack 有一些核心的概念需要理解：

* 入口（`entry`）
* 输出（`output`）
* `loader`
* 插件（`plugin`）
* 模式（`mode`）
* 浏览器兼容（`browser compatibility`）

- 环境（`environment`）

下面我们就分别来学习这些模块

## 入口（entry）

入口（`entry`）指示webpack应该使用哪个模块来作为构建其内部依赖图的开始。进入入口起点后，`webpack`会找出有哪些模块和库是入口起点依赖的。

默认值：	`src/index.js`. 但是可以在配置文件中配置entry的属性。

`webpack`配置中有多种方式定义 entry 属性。

### 单个入口

用法：

```javascript
entry: string | [string]
```

`webpack.config.js`

```javascript
module.exports = {
	entry: './path/myproject/src/index.js'
}
```

也可以简写如下：

`webpack.config.js`

```javascript
module.exports = {
	entry: {
		main: './path/myproject/src/index.js',
	}
}
```



也可以将一个文件路径数组传递给`entry`属性，这将会创建一个所谓的 `multi-main entry`。如果想一次性注入多个依赖的文件，并且想它们依赖关系绘制在一个`chunk`中时， 就可以使用这种方式

`webpack.config.js`

```javascript
module.exports = {
	entry: [
		'src/1.js',
		'src/2.js',
		'src/3.js',
	],
  output: {
    filename: 'bundle.js'
  }
}
```

如果你希望通过一个入口为应用程序快速设置`webpac`k配置时候，单一入口的方式就是一个很合适的选择，但是使用这种语法方式来扩展或调整配置的灵活性不大。



### 对象语法

用法：

```javascript
entry: { <entryChunkName> string | [string]} | {}
```

`webpack.config.js`

```javascript
module.exports = {
	entry: {
		main: 'src/main.js',
		admin: 'src/admin.js',
    index: 'src/index.js'
	}
}
```

对象语法会比较繁琐，但是，这是应用程序中定义入口最可扩展的方式。

> 这里的最可扩展是指这些配置可以重复使用，并且与其他配置组合使用。





## 输出（output）

`output `属性告诉wbpack 在哪里输出它创建的 bundle，以及如何命名这些文件。


主要输出文件的默认值是`./dist/main.js`, 其他生成的文件默认放置在`./dist`文件夹中。



**注意：** 即使存在多个` entry` 起点，但也只能指定一个 `output `配置。



用法：  在`webpack `配置中， output属性的最低要求是将它配置成一个对象，然后为输出的文件配置名字(**output.filnename**)

`webpack.config.js`

```javascript
module.exports = {
	output: {
		filename: 'bundle.js'
	}
}
```

这个配置将一个单独的` bundle.js` 文件输出到 dist 目录中。



如果是多个入口起点， 则应该使用 **占位符** 来确保每一个文件具有唯一的名称。

- `name` 如果设置，则为`chunk`的名称，否则使用`chunk`的`ID `

* [ id ]:   chunk的ID

* [ contenthash ]: 此chunk的hash值只包括该内容类型的元素（受optimization.realContentHash影响

* [  chunkhash  ]: 此chunk的hash值，包含该chunk的所有元素

`webpack.config.js`

```javascript
module.exports = {
	entry: {
		main: './src/main.js'，
		admin: './src/admin.js'，
	},
	output: {
		filename: [name].js,
	}
}
```

这配置最终dist目录：main.js, admin.js

其次` output` 属性可配置的字段常用的还有**output.path**字段

```javascript
const path = require('path');
module.exports = {
	output: {
		filename: [name].js
		path: path.resolve(__dirname, '/dist')
	}
}
```



## loader

`webpack`只能理解`javaScript `和 `JSON` 文件。这是`webpack` 开箱可用的能力。`loader让webpack` 能够去处理其他类型的文件，并将它们转换成有效的模块，已供应用使用， 以及被添加到依赖图中，即loader用于对模块的源代码进行转换。loader可以使你在import 或者l ”load加载“模块时预处理文件。



在`webpac`k配置中，`loader `有两个属性：

- `test`属性， 识别出哪些文件会被转换
- `use`属性，定义在进行转换时，应该使用哪个`loader`

`webpack.config.js`

```javascript
module.exportas = {
	output: {
		filename: [name].js
	},
	module: {
		rules: [
			{
				test: /\.css$/, 
				use: 'css-laoder'
			}
		]
	}
}
```

上面的配置中， 对一个单独的`module `对象定义了 rules 属性、里面的每个配置都包含了两个必须的属性：`test`。` use` 。

这个配置的意思就是：

> 当遇到`import/ require`语句中解析有`.css`的路径时，在对它进行打包之前，先使用` css-loader` 转换一下。

**注意1：**在`webpack`配置中定义`rules`时， 要定义在`module.rules` 中而不是` rules`中。 

**注意2：**在`webpack` 中使用正则表达式时， 不给它添加引号。也就是说：` /\.css$/ `和 ` /\.css$/` 或者 ` /\.css$/` 是不一样的。前者的意思是匹配任何以`.css` 结尾的文件。而后者指示`webpack`匹配具有绝对路径`.css`的对单个文件。



 ### 使用loader 

在应用程序中使用`loader` 有三种方式：

- 配置方式（推荐使用）: 在配置文件中指定`loader`
- 内联方式 ：在每个import 文语句显示的指定`loader`
- `CLI`方式：在`shell`命令中指定它们



### 配置方式

这种方式是在配置文件中的`module.rules`中配置。允许webpack配置中配置多个`loader`。`loader`从右到左的取值。

```javascript
module.exports = {
	module: {
		rules: [
			{ 
				test: /\.css$/,
				use: ['style-loader', 'css-loader', 'sass-loader']
			}
		]
	}
}
```

这个配置表示：匹配所有的`.css`结尾的文件，先后分别使用` sass-loader`, `css-loader`,` style-loader`进行转换。



## 插件（plugin）

`loader` 是用于转换某些类型的模块，而插件则可以用于执行范围更广的任务。包括：打包优化，资源管理，注入环境变量等。

使用一个插件，需要`require（）`它，然后添加在`webpac`k配置中的 `plugins` 数组中。多数的插件是可以通过选项来自定义的。

由于插件可以携带参数/选项，所以在`webpac`k配置中，向 `plugins` 属性传入一个 new 实例。

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin');
module.exports = {
	plugins: [
		new HtmlWebpackPlugin()
	]
}
```

这个配置我们使用了一个插件` html-webpack-plugin` 这个插件为应用程序生成一个`HTML` 文件，并且自动注入所有生成的`bundle`。

插件的用法有两种：

- 配置方式

  就如上的示例

- Node API 方式：

  ```javascript
  const webpack = require('webpack'); // 访问 webpack 运行时(runtime)
  const configuration = require('./webpack.config.js');
  
  let compiler = webpack(configuration);
  
  new webpack.ProgressPlugin().apply(compiler);
  
  compiler.run(function(err, stats) {
    // ...
  });
  ```

想要看看 webpack 插件有哪些，可以看看 [插件列表](https://webpack.docschina.org/plugins/)。当然插件列表也不是全的， 有很多插件都是用户自己上传的，可以的话， 也可以到npm 上面搜 webpack等字样。

## 模式（Mode）

通过选择 development, production，none 之中的一个来设置 mode 参数，来启用webpack 内置在相应环境下的优化， 默认值： production。

```
module.exports = {
	mode: 'production'
}
```

或者也可以在CLI 参数中传递：

```
webpack --mode='development'
```

- development : 会将DefinePlugin中的 process.env.NODE_ENV 的值设置为 development，为模块和 chunk启用有效的名
- production： 会将 `DefinePlugin` 中 `process.env.NODE_ENV` 的值设置为 `production`。为模块和 chunk 启用确定性的混淆名称等
- none: 不使用任何的默认优化选项





# 配置

一些需要了解的概念已经了解的差不多了， 那么接下来就是具体的配置项了。

webpack 是开箱即用的， 也就是说无需任何的配置文件，这种情况下webpack会假定项目的入口起点是 `src/index.js`，然后会在 `dist/main.js`中输出结果。并且在生产环境开启压缩和优化。

举个例子：

假如我们有这样的目录结构：

```javascript
|--src
	 |--index.js
```

我们在根目录命令行输入：

```
webpack
```

这时候我们发现跟目录多了一个文件夹`dist`，该文件夹里面有main.js。即目前的文件文件目录为：

```javascript
|--dist
	 |--main.js
|--src
	 |--index.js
```

所有可以看出不用做任何的配置就可以直接使用webpack。





虽然我们可以直接使用webpack来打包， 但是通常的情况下， 我们的项目都需要继续扩展能力，因此可以在根目录下创建 webpack.config.js 文件。 然后webpack 会自动使用它。

我们在根目录新建一个文件`webpack.config.js`。内容如下：

````javascript
module.exports = {
  // 入口起点
  entry: './src/main.js',
  // 输出
  output: {
    // 输出的文件的名字
    filename: 'shuliqi.js'
  },
}
````

这配置文件，我们修改了入口起点。输出的文件名我们也改成`shuliqi.js`。然后我们在根目录执行命令：`webpack`. 最后我们会看到我们的文件目录是这样的：

```javascript
|--dist
	 |--shuliqi.js
|--src
	 |--main.js
|--webpack.config.js
```

由此可知，在根目录有webpack.config.js配置文件的话，输入`webpack`命令的时候会使用这个配置文件。





当然在某些特定的额情况下，需要根据实际情况使用不用的配置文件。webpack也可以使用不用的配置文件， 只需要在命令行中使用`--congig flag`修改配置文件名称。

我们新建一个文件`webpack.dev.js`,内容如下：

```javascript
module.exports = {
  // webpack.dev.js配置的入口
  entry: './src/dev.js',
  // 输出
  output: {
    // webpack.dev.js配置的输出的文件的名字
    filename: 'output-dev.js'
  },
}
```

然后我们在命令行执行：`webpack --config webpack.dev.js`。然后我们可以到打包之后的结果：

```javascript
|--dist
	 |--output-dev.js
|--src
	 |--dev.js
|--webpack.dev.js
```

可以看到打包结果跟我们配置文件写的一样(打包之后输出的名字，打包的入口)。



# 配置选项

## 上下文（context）

**用法:** ` context: string`

**概念：** 基础路径，绝对路径，用于从配置文件中解析入口起点(entry)

入口对象是用于webpack查找看是构建bundle的地方。 上下文是入口文件所处的目录的绝对路径的字符串。

webpack 默认的上下文是当前的目录。 我们可以使用context修改上下文。

我们建一个文件夹，目录结构如下:

```
|--app
	 |--index.js
|--webpack.config.js
```

我们的配置文件webpack.config.js的内容如下：

````javascript
const path = require('path');

module.exports = {
  // 将上下文修改为 当前绝对路径的app目录下面,我们的额入口起点将是在这个目录下面
  // ps:可以先注释注释点这个配置，如果这样打包是会报错的
  context: path.resolve(__dirname, 'app'),
  entry: './index.js',
  output: {
    filename: 'shuliqi.js'
  },
}
````

如果我们注释掉：` context: path.resolve(__dirname, 'app')。然后根目录命令:`webpack`。 这时候会报错：

   {% asset_img 10.png %}

我们不注释 ` context: path.resolve(__dirname, 'app')的话， 是能正常打包的，正常打包的结果如下：

```javascript
|--dist
	 |-- shuliqi.js
|--src
	 |--index.js
|--webpack.config.js
```

从上面的例子可以看出 context 的作用了，如果没有设置 context  我们的 entry 应该这么写：

```javascript
 entry: './app/index.js',
```

不禁想问， context 的好处是什么呢？ 我们不用设置 context 也可以进行打包的呀。嗯哈？

假如我们要打包 app 下面的很多js 文件，我们不设置的 context 修改的当前入口目录的话，那么我们是不是每个都需要写：

```javascript
 entry:['./app/1.js', './app/2.js', './app/3.js', './app/4.js', ...],
```

这样每个文件都要写 './app/XXX'， 是不是有点繁琐了？ 那这时候使用 contetxt 就很方便了。



## 入口（entry）

配置文件中的 entry 接受三种形式的值：String（字符串），Array（数组）， Object（对象）。

我们先来介绍对象形式，因为这是最完整的 entry 配置，其他的形式只是它的简化形式而已。

### 对象 entry

对象的形式如下：

````javascript
entry: {
	<key>: <value>
}
````

#### key

**key 值可以使简单的字符串， 比如： main，index等等。并且对应着 output.filename 配置中的  [name]变量。**

假如我们有这样的文件路径：

```javascript
|--src
	|--index.js
|--webpack.config.js
```

webpack.config.js：

```javascript
module.exports = {
  entry: {
    app: './src/index.js'
  },
  output: {
    filename: '[name].js',
  }
}
```

根据上面的配置，我们在根目录执行 `webpack` 命令之后可以生成：

```javascript
|--dist
	|--app.js
|--src
	|--index.js
|--webpack.config.js
```



**key 还可以是路径字符串， webpack 会自动生成路径目录，并将路径的最后最为[name]**。这个特性在多页面配置下是很有用的。

假如我们还是一样的目录：

```javascript
|--src
	|--index.js
|--webpack.config.js
```

webpack.config.js：

```javascript
module.exports = {
  entry: {
    'public/static/app': './src/index.js'
  },
  output: {
    filename: '[name].js',
  }
}
```

根据上面的配置，我们在根目录执行 `webpack` 命令之后可以生成：

```javascript
|--dist
	|--public
		|--static
			|--app.js
|--src
	|--index.js
|--webpack.config.js
```

#### value

#####  如果value是字符串

如果value 是字符串，那么必须是合理的 node reqiur  函数参数字符串，比如文件径：'./app.js'（require('./app.js')）；比如是安装的npm 模块路径：'lodash'（require('lodash'))

假如有这样的路径：

```javascript
|--src
	|--index.js
|--webpack.config.js
|--package.json
```

package.json：

```javascript
{
  "name": "webpack-demo",
  "version": "1.0.0",
  "description": "",
  "main": "webpack.config.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "lodash": "^4.17.20"
  }
}

```

webpack.config.js:

```javascript
module.exports = {
  entry: {
    'my-lodash': 'lodash'
  },
  output: {
    filename: '[name].js',
  }
}
```

这样的配置打包生成：

```javascript
|--dist
	|--my-lodash.js
|--src
	|--index.js
|--webpack.config.js
|--package.json
```





##### 如果value是数组

则数组中的元素要符合上面描述的合理字符串值，数组中的文件一般是没有互相依赖关联关系的。但是又处于某种原因要将它们打包在一起的。

假如这样的目录：

```
|--src
	|--index.js
	|--main.js
|--webpack.config.js
```

webpack.config.js:

```javascript
module.exports = {
  entry: {
    'all': ['./src/index.js', './src/main.js']
  },
  output: {
    filename: '[name].js',
  }
}
```

以上的配置打生成：

```javascript
|--dist
	|-all.js
|--src
	|--index.js
	|--main.js
|--webpack.config.js
```


##### 如果value是对象

###### Import字段

Import字段表示引入入口 chunk

如这样的目录：

```javascript
|--src
	|--index.js
	|--main.js
|--webpack.config.js
```



webpack.config.js

````javascript
module.exports = {
 entry: {
   'app': {  import: './src/index.js'}
 }
}
````

这样的配置打包生成：

```javascript
|--dist
	|--app.js
|--src
	|--index.js
	|--main.js
|--webpack.config.js
```

当然imprt 也可以是数组:

webpack.config.js

```javascript
module.exports = {
 entry: {
   'app': {  import: ['./src/index.js', './src/main.js']}
 }
}
```

打包生成和上面的例子相同。

###### filename字段

filename字段为特定的入口指定一个自定义的输出文件名

同样的目录：

```javascript
|--src
	|--index.js
	|--main.js
|--webpack.config.js
```

webpack.config.js

```javascript
module.exports = {
 entry: {
   //  给当前的入口 chunk 自定义输出文件名 shuliqi.js
   'app': {  import: './src/index.js', filename:'shuliqi.js'}
 }
}
```

这样的配置打包生成：

```javascript
|--dist
	|--shuliqi.js
|--src
	|--index.js
	|--main.js
|--webpack.config.js
```

###### dependOn

dependOn字段可以设置与另一个入口chunk 共享哪些模块

假如我们有这样的目录：

```
|--src
	|--common.js
	|--index.js
	|--main.js
|--webpack.config.js
```

common.js

```javascript
console.log('common.js')
```

index.js

```javascript
import './common.js'
console.log('index.js')
```

main.js

```javascript
import './common.js'
console.log('main.js')
```

webpack.config.js

```javascript
module.exports = {
  entry: {
    'common': './src/ common.js',
    'index': { import: './src/index.js'},
    'main': { import: './src/main.js'}
  }
}
```

如果我们是这样配置生成的是：

```javascript
|--dist
	|--main.js
	|--index.js
	|--common.js
|--src
	|--index.js
	|--main.js
	|--common.js
|--webpack.config.js
```

其中生成的 index.js  main.js 是包含 common 模块的。

index.js

````javascript
(()=>{var e={62:()=>{console.log("common.js")}},o={};function r(t){if(o[t])return o[t].exports;var n=o[t]={exports:{}};return e[t](n,n.exports,r),n.exports}r.n=e=>{var o=e&&e.__esModule?()=>e.default:()=>e;return r.d(o,{a:o}),o},r.d=(e,o)=>{for(var t in o)r.o(o,t)&&!r.o(e,t)&&Object.defineProperty(e,t,{enumerable:!0,get:o[t]})},r.o=(e,o)=>Object.prototype.hasOwnProperty.call(e,o),(()=>{"use strict";r(62),console.log("index.js")})()})();
````

main.js

```java
(()=>{var e={62:()=>{console.log("common.js")}},o={};function r(t){if(o[t])return o[t].exports;var n=o[t]={exports:{}};return e[t](n,n.exports,r),n.exports}r.n=e=>{var o=e&&e.__esModule?()=>e.default:()=>e;return r.d(o,{a:o}),o},r.d=(e,o)=>{for(var t in o)r.o(o,t)&&!r.o(e,t)&&Object.defineProperty(e,t,{enumerable:!0,get:o[t]})},r.o=(e,o)=>Object.prototype.hasOwnProperty.call(e,o),(()=>{"use strict";r(62),console.log("main.js")})()})();
```



但是这样造成就是资源重复很多，这时候如果使用：dependOn就可以共享资源了

webpack.config.js

```javascript
module.exports = {
  entry: {
    'common': './src/ common.js',
    'index': { import: './src/index.js', dependOn: 'common'},
    'main': { import: './src/main.js',  dependOn: 'common'}
  }
}
```

其中生成的 main 和 index chunk 就不会包含 common 模块了。

最后 `dependOn` 选项的也可以为字符串数组：

> ps： 不是很懂这个设置有什么用，等我之后再来研究研究  尴尬



### 字符串entry：

```javascript
module.exports = {
  entry: './src/index.js',
}
```

等价于：

```javascript
module.exports = {
  entry: {
  	main: './src/index.js',
  }
}
```

### 数组 entry

```javascript
module.exports = {
  entry: ['./src/index.js', './src/main.js'],
}
```

等价于：

```javascript
module.exports = {
  entry: {
  	main: ['./src/index.js', './src/main.js']
  },
}
```

## 模式（mode）

webpack 配置文件中 mode 配置选项是一个 string， 可能的值有：**production**，**development**，**none**。默认值是：**production**。

这个配置选项的主要作用是告诉 webpack 使用相应的内置的优化配置。

### 用法

可以直接在配置文件中配置该选项

```javascript
module.exports = {
  mode: 'development',
}
```

也可以通过 cli 来传递

```javascript
webpack --mode=development
```

### 选项描述

| 选项          | 描述                                                         |
| :------------ | :----------------------------------------------------------- |
| `development` | 会将 `DefinePlugin` 中 `process.env.NODE_ENV` 的值设置为 `development`。启用 `NamedChunksPlugin` 和 `NamedModulesPlugin` |
| `production`  | 会将 `DefinePlugin` 中 `process.env.NODE_ENV` 的值设置为 `production`。启用 `FlagDependencyUsagePlugin`, `FlagIncludedChunksPlugin`, `ModuleConcatenationPlugin`, `NoEmitOnErrorsPlugin`, `OccurrenceOrderPlugin`, `SideEffectsFlagPlugin` 和 `TerserPlugin`。 |
| `none`        | 没有任何默认优化选项                                         |

如果没有设置， webpack 会将 mode 设置为  production

>注意：设置 mode 可以改变 process.env,NODE_ENV。但是反过来，设置 NODE_ENV 并不会自动设置webpack 的 mode 选项。这里的  process.env,NODE_ENV 不是 node 的环境变量，而是webpack.DefinePlugin中定义的全局变量，允许根据不同的环境执行不用的代码

#### development

设置 mode 为 development

```javascript
module.exports = {
  mode: 'development'
}
```

就相当于如下的配置

```javascript
module.exports = {
  devtool: 'eval',
  cache: true,
  performance: {
    hints: false
  },
  output: {
    pathinfo: true
  },
  optimization: {
    namedModules: true,
    namedChunks: true,
    nodeEnv: 'development',
    flagIncludedChunks: false,
    occurrenceOrder: false,
    sideEffects: false,
    usedExports: false,
    concatenateModules: false,
    splitChunks: {
      hidePathInfo: false,
      minSize: 10000,
      maxAsyncRequests: Infinity,
      maxInitialRequests: Infinity,
    },
    noEmitOnErrors: false,
    checkWasmTypes: false,
    minimize: false,
  },
  plugins: [
    new webpack.NamedModulesPlugin(),
    new webpack.NamedChunksPlugin(),
    new webpack.DefinePlugin({ "process.env.NODE_ENV": JSON.stringify("development") }),
  ]
}
```

#### production

设置 mode 为 production

````javascript
module.exports = {
  mode:"production"
}
````

其相当于设置如下配置

```javascript
module.exports = {
  performance: {
    hints: 'warning'
  },
  output: {
    pathinfo: false
  },
  optimization: {
    namedModules: false,
    namedChunks: false,
    nodeEnv: 'production',
    flagIncludedChunks: true,
    occurrenceOrder: true,
    sideEffects: true,
    usedExports: true,
    concatenateModules: true,
    splitChunks: {
      hidePathInfo: true,
      minSize: 30000,
      maxAsyncRequests: 5,
      maxInitialRequests: 3,
    },
    noEmitOnErrors: true,
    checkWasmTypes: true,
    minimize: true,
  },
  plugins: [
    new TerserPlugin(/* ... */),
    new webpack.DefinePlugin({ "process.env.NODE_ENV": JSON.stringify("production") }),
    new webpack.optimize.ModuleConcatenationPlugin(),
    new webpack.NoEmitOnErrorsPlugin()
  ]
}
```

#### none

设置 mode 为 none

```javascript
module.exports = {
  mode:"none"
}
```

其相当于设置如下配置

```javascript
module.exports = {
  performance: {
   hints: false
  },
  optimization: {
    flagIncludedChunks: false,
    occurrenceOrder: false,
    sideEffects: false,
    usedExports: false,
    concatenateModules: false,
    splitChunks: {
      hidePathInfo: false,
      minSize: 10000,
      maxAsyncRequests: Infinity,
      maxInitialRequests: Infinity,
    },
    noEmitOnErrors: false,
    checkWasmTypes: false,
    minimize: false,
  },
  plugins: []
}
```

### 输出（output）

output 是最顶级的键，包括了一组选项，目的是提示webpack 如何去输出，以及在哪里输出。它的配置项很多， 我们跟着教程一个一个的来看看这些配置到底是什么怎么使用的。

#### library

#### libraryTarget

这两个配置项具体可以直接看这边文章 [模块化与Webpack属性 library,libraryTarget的关联](https://shuliqi.github.io/shuliqi.github.io/2021/02/03/webpack%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0%EF%BC%9Alibrary%EF%BC%8ClibraryTarget%EF%BC%8CauxiliaryComment/)

#### auxiliaryComment

注意：这个配置项在webpack4版本以下生效， 以上版本不生效。

如果是webpack 3 的 可以具体看看 [output.auxiliaryComment](https://webpack.docschina.org/configuration/output/#outputauxiliarycomment)

#### filename 

#### chunkFilename

这两个配置项的使用 具体移步 [webpack中的filename和chunkFilename及其区别](https://shuliqi.github.io/shuliqi.github.io/2021/02/22/webpack%E4%B8%AD%E7%9A%84filename%E5%92%8CchunkFilename%E5%8F%8A%E5%85%B6%E5%8C%BA%E5%88%AB/)

#### path

`output.path`配置输出文件存放在本地的目录，必须是`string`类型的绝对路径。通常通过`Node.js`的`path`模块去获取绝对的路径。[Node.js  path](http://nodejs.cn/api/path.html#path_path_relative_from_to)

`webpack.config.js`

```javascript
// 引入Node的path模块
const path = require('path');

module.exports = {
  entry: {
    index: './src/index.js',
  },
  output: {
    path: path.resolve(__dirname, './dist')
  }
}
```

上面代码中的` path.resolve(path1, path2, ...)`方法的作用是：按照顺序依次拼接，获取绝对路径，路径末尾不会带有路径分隔符号， 若合并后的路径没有构成一绝对路径，则会默认使用当前工作目录的绝对路径。

```javascript
// 引入 Node 模块
const path = require('path'); 
// 拼接路径中没有带有绝对路径
var _path = path.resolve('path3', 'path4', 'a/b/cc/'); // 没有末尾的路径分隔符\
console.log(_path)
// 结果为：/Users/shuliqi/study/webpack-demo/path3/path4/a/b/cc


// 拼接路径中带有绝对路径
var _path2 = path.resolve('/Users/shuliqi/study/webpack-demo/', 'path3', 'path4', 'a/b/cc/');
console.log(_path2)
// 结果为：/Users/shuliqi/study/webpack-demo/path3/path4/a/b/cc
```

而 `__dirname`在 `Node.js`中总是指向被执行的`js`文件的绝对路径。如： 当你在`：/Users/shuliqi/study/webpack-demo/src/index.js`文件中写了`__dirname`，那么它的值就是`/Users/shuliqi/study/webpack-demo/src/`

#### publicPath

- 默认值： 空字符串

在复杂的项目中可能会有一些构建出的资源需要异步加载，加载这些异步资源需要对应的`url`地址。`output。publicPath`配置就是发布到线上资源的`url`前缀，是`string`类型，默认是空字符串，即相对路径。

举个例子🌰：

我们需要把构建出的资源上传到`CDN`服务上，以便于加快页面的打开速度，配置的代码假设如下：

```javascript
// 引入Node的path模块
const path = require('path');

module.exports = {
  entry: {
    index: './src/index.js',
  },
  output: {
    path: path.resolve(__dirname, './dist'),
    publicPath: 'https://shuliqi.github.io/public/js/'
  }
}

```

这时发布到线上的`HTML`在引入`Javascript`文件时就需要：

```html
<script src='https://shuliqi.github.io/public/js/jkashdjashd.js'></script>
```





































