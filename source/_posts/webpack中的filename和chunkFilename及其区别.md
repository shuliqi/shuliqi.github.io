---
title: webpack中的filename和chunkFilename及其区别
date: 2021-03-14 19:40:07
tags: webpack学习笔记
---

`webpack`中`output`对象有很多的配置选项，其中`output.filename`和`output.chunkFilename`两者都是给输出 `bundle` 命名的。那么它们具体怎么使用呢？两者之间又有什么区别呢？

<!--more-->

# 前置知识

首先我们先来看打包输出的字段分别代表什么意思？ 好为我们下面讲的 `filename` 和 `chunkFilename`做基础。

我们先来做我们的项目结构：

```
$ mkdir webpack-demo
$ cd  webpack-demo
```

然后本地安装 `webpack`

```
npm install --save-dev webpack@4.9.0
```

因为安装的是 `webpack4`版本的， 所以需要安装`CLI`

```
npm install --save-dev webpack-cli
```

然后在`package.json`的`script`字段中添加：

```
"build": "webpack --config webpack.config.js"
```

这个配置的可以用来执行`webpack`。

具体的`webpack`安装方式，[点击查看](https://webpack.docschina.org/guides/installation/)

在当前的目录上添加一些文件夹和文件

```
$ touch webpack.config.js
$ mkdir src
$ cd src
$ touch index.js
```

那么我们目前的项目机构为:

```
|--package.json
|--node_modules
|--package-lock.json
|--src
  |--index.js
|--webpack.config.js
```

那么现在开始我们的正题。

假如我们有如下的配置 `webpack.config.js`:

```javascript
module.exports = {
  entry: {
    index: './src/index.js',
  },
  output: {
    filename: 'shuliqi.js',
  }
}
```

我们在`package.json` 的`script`字段添加：

```
  "build": "webpack --config webpack.config.js",
```

最后执行命令 `npm run build` 使用`webpack`打包， 输出字段如下：

{% asset_img 1.png %}

打包的时候我们可得到上面如图的输出字段：

* Hash： 表示本次打包的唯一`hash`值，每次打包的`hash`值是不一样的。
* Version: 当前项目使用的`webpack`版本
* Time：这次打包用的时间
* Asset：打包输出的文件的名称
* Size：打包的文件的大小
* Chunks：打包后的文件的`id`号，从0 开始， 往后递增
* Chunks Names：打包的文件的名字。

其中 `hash` 和 `chunkhash` 的长度是可指定的，`[hash:8]` 代表取8位 Hash 值，默认是20位。

# filename

在`output`对象中的`filename` 是一个比较常见的属性，它对应的是 `entry`配置项中的输入文件，经过`webpack`打包之后的文件名。

webpack 常用的输出文件名内置的模板变量：

| 模板        | 描述                                 |
| :---------- | :----------------------------------- |
| [hash]      | 模块标识符(module identifier)的 hash |
| [chunkhash] | chunk 内容的 hash                    |
| [name]      | 模块名称                             |
| [id]        | 模块标识符(module identifier)        |



## 静态名称

如果输出文件只有一个， 可以把`filename`属性写成静态不变的名称。

`webpack.config_single.js`:

```javascript
// 单个文件输出文件，可以把filename写成静态不可变的名称
module.exports = {
  entry: './src/index.js',
  output: {
    // 配置打包输出的文件名称为静态名称：index.js
    filename: 'index.js'
  }
}
```

这样的配置打包输出的文件就是固定的名字`index.js`了。

我们在`package.json` 的`script`字段添加：

```
"build_single": "webpack --config webpack.config_single.js"
```

最后执行命令 `npm run build_single` 使用`webpack`打包。

{% asset_img 2.png %}

可以看出打包输出的文件名称(`Asset`)为 `index.js`。

但是， 当有多个入口起点，代码拆分或者插件从而创建多个输出文件时， 应该使用模板来为每个输出赋予不用的名称。

## 使用入口名称

`webpack.config_name.js`

```javascript
// 打包输出的文件名称使用入口的文件名称
module.exports = {
  entry: {
    // 两个入口的名称分别为： main，common
    main: '/src/index.js',
    common: './src/index.js',
  },
  output: {
    // 打包输出的文件名称为： 入口名称（main，common） + '_name.js'
    filename: '[name]_name.js'
  }
}
```

上面配置中，`filename`值中的 `[name]`指入口起点键值对的键，即`main`和 `common`。 输出的文件名称应该分别是：`main_name.js`，`common_name.js`.



我们在`package.json` 的`script`字段添加：

```
 "build_name": "webpack --config webpack.config_name.js"
```

最后执行命令 `npm run build_name` 使用`webpack`打包。

{% asset_img 3.png %}

以看出打包输出的文件名称是 `main_name.js`，`common_name.js`.

## 使用内部的 chunk id

`webpack.config_id.js`

```javascript
// 使用内部的chunk  id 作为输出的文件的名称
module.exports = {
  entry: {
    index: './src/index.js',
    common: './src/common.js',
  }, 
  output: {
    filename: '[id].js'
  }
}
```

上面配置中 `filename` 中的`[id]`表示 `chunk`内部的`id`。即输出字段的 `Chunks`字段。

我们在`package.json` 的`script`字段添加：

```
  "build_id": "webpack --config webpack.config_id.js"
```

最后执行命令 `npm run build_id` 使用`webpack`打包。

{% asset_img 4.png %}

可以看出， 我们的输出的文件的名称是以内部`chunk id`来命名的

## 使用 hash 生成

`webpack.config_hash.js`

```javascript
module.exports = {
  entry: {
    index: './src/index.js',
    common: '/src/common.js'
  },
  output: {
    filename: '[hash]_[name].js'
  }
}
```

上面配置中的`[hash]`是指本次打包的唯一`hash`值，即输出字段`Hash`字段。因为`hash`是一样的， 再加上`[name]`是为了区分各个文件。

我们在`package.json` 的`script`字段添加：

```
  "build_hash": "webpack --config webpack.config_hash.js"
```

最后执行命令 `npm run build_hash` 使用`webpack`打包。

{% asset_img 5.png %}

可以看出来打包的输出文件的名称是由输出字段的`[hash]` + `[name]` + `.js`拼接而成的。

## 使用 chunk 内容 hash

`webpack.config_chunkhash.js`:

```javascript
module.exports = {
  entry: {
    index: '/src/index.js',
    common: '/src/common.js'
  },
  output: {
    filename: '[chunkhash]_[name].js'
  }
}
```

上面配置中的 `[chunk_hash]`是指：chunk 内容的 hash。加上`[name]`是为了区分各个文件。

我们在`package.json` 的`script`字段添加：

```
 "build_chunkhash": "webpack --config webpack.config_chunkhash.js"
```

最后执行命令 `npm run build_chunkhash` 使用`webpack`打包。

{% asset_img 6.png %}

# chunkFilename

`chunkFilename`指未被列在`entry`中，却又是需要被打包出来的`chunk`的文件名称。

什么样的场景会需要呢？在项目中按需加载（异步）模块的时候， 这样的文件就是没有被列在`entry`中的。但是有时需要打包的。 `chunkFilename`就可以对这些文件进行命名。

> `chunk`是指`webpack`在進行模块的依赖分析时候，代码分割出来的代码块。

`chunkFilename` 支持和 `filename` 一致的内置模板变量。

我们在`src`文件夹中添加`chunkFilename.js`:

```javascript
console.log('shuliqi');
// 引入一个外部的js 文件
import('./common.js');
```

如果我们对这个文件打包的时候， 因为我们引入了外部的`js`文件，这个文件没有别列在`entry`中，但是又需要被打包的。

## 静态名称

`chunkFilename_single.js`:

```javascript
module.exports = {
  entry: './src/chunkFilename.js',
  output: {
    filename: 'main.js',
    // 没有列在entry 中， 但是又不得不打包的文件的命名
    chunkFilename: 'chunkFilename.js'
  }
}
```

我们在`package.json` 的`script`字段添加：

```
  "build_chunkFilename": "webpack --config chunkFilename_single.js"
```

打包结果：

{% asset_img 7.png %}

## 使用入口名称

`chunkFilename`是不能使用`[name]`的，因为它就没有被列入`entry`中、所以无法知道入口名称是多少， 如果`chunkFilename`使用了了`[name]`。 那么会自动转换成是使用默认设置:`[id].js`。即：

> `output.chunkFilename` 默认使用 `[id].js` 或从 `output.filename` 中推断出的值（`[name]` 会被预先替换为 `[id]` 或 `[id].`）

`chunkFilename_name.js`

```javascript
module.exports= {
  entry: {
    chunkfilename: './src/chunkFilename.js',
  },
  output: {
    filename: 'main.js',
    chunkFilename: '[name].js'
  }
}
```

我们在`package.json` 的`script`字段添加：

```
 "build_chunkFilename_name": "webpack --config chunkFilename_name.js"
```

执行打包命令`npm run build_chunkFilename_name`，结果：

{% asset_img 8.png %}

可以看出来，没有列在`entry`中的文件，打包之后的文件名是是 1.js， 其中1 是该 `chunk`的`id`.

## 使用 hash 生成

`chunkFilename_hash.js`:

```javascript
module.exports = {
    entry: './src/chunkFilename.js',
    output: {
      filename:'main.js',
      chunkFilename: '[hash].js'
    }
}
```

我们在`package.json` 的`script`字段添加：

```javascript
 "build_chunkFilename_hash": "webpack --config chunkFilename_hash.js"
```

执行打包命令`npm run  build_chunkFilename_hash`，结果：

{% asset_img 9.png %}

可以看出来， 没有列入`entry`但是有需要打包的`common.js`的`chunk`的名称为输出字段的`hash` + `.js`的拼接

## 使用 chunk 内容 hash

`chunkFilename_chunkhash.js`

```javascript
module.exports = {
  entry: '/src/chunkFilename.js',
  output: {
    filename: 'main.js',
    chunkFilename: '[chunkhash].js'
  }
}
```

我们在`package.json` 的`script`字段添加：

```javascript
 "build_chunkFilename_hash": "webpack --config chunkFilename_hash.js"
```

执行打包命令`npm run  build_chunkFilename_hash，结果：

{% asset_img 10.png %}

可以看出来， 没有列入`entry`但是有需要打包的`common.js`的`chunk`的名称为输出字段的`chunkhash` + `.js`的拼接

# 最后

**总结`filename 和 chunkFilename`：**

* `filename` 指**列在** `entry` 中，打包后输出的文件的名称。

* `chunkFilename` 指**未列在** `entry` 中，却又需要被打包出来的文件的名称。

关于上面的例子可移步[webpack-filename-chunkFilename](https://github.com/shuliqi/webpack-filename-chunkFilename)

**输出文件名内置的模板变量:**

当然输出文件名内置的模板变量不仅是上列出来的这些，我觉得这些是常用的， 还有很多其他的。如：

* 可在编译层面进行替换的内容：

| 模板       | 描述                       |
| :--------- | :------------------------- |
| [fullhash] | compilation 完整的 hash 值 |
| [hash]     | 同上                       |

* 可在 chunk 层面进行替换的内容：

| 模板          | 描述                                                         |
| :------------ | :----------------------------------------------------------- |
| [id]          | 此 chunk 的 ID                                               |
| [name]        | 如果设置，则为此 chunk 的名称，否则使用 chunk 的 ID          |
| [chunkhash]   | 此 chunk 的 hash 值，包含该 chunk 的所有元素                 |
| [contenthash] | 此 chunk 的 hash 值，只包括该内容类型的元素（受 `optimization.realContentHash` 影响） |

* 可在模块层面替换的内容：

| 模板          | 描述               |
| :------------ | :----------------- |
| [id]          | 模块的 ID          |
| [moduleid]    | 同上，但已弃用     |
| [hash]        | 模块的 Hash 值     |
| [modulehash]  | 同上，但已弃用     |
| [contenthash] | 模块内容的 Hash 值 |

* 可在文件层面替换的内容：

| 模板       | 描述                                   |
| :--------- | :------------------------------------- |
| [file]     | filename和路径，不含 query 或 fragment |
| [query]    | 带前缀 `?` 的 query                    |
| [fragment] | 带前缀 `#` 的 fragment                 |
| [base]     | 只有 filename（包含扩展名），不含 path |
| [filebase] | 同上，但已弃用                         |
| [path]     | 只有 path，不含 filename               |
| [name]     | 只有 filename，不含扩展名或 path       |
| [ext]      | 带前缀 `.` 的扩展名                    |

* 可在 URL 层面替换的内容：

| 模块  | 描述 |
| :---- | :--- |
| [url] | URL  |

具体也移步[官网](https://webpack.docschina.org/configuration/output/#outputfilename)

