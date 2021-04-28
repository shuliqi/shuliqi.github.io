---
title: webpack学习--优化（optimization）
date: 2021-03-01 17:50:57
tags: webpack学习笔记
---

`webpack`目前最新的版本是5， 但是早在`webpack4`的时候，可以通过选择不用的`mode`模式来优化打包的结果。不过所有的优化（optimization）还是可以通过手动配置和重写的。，在构建的时候我们可以针对各个环境配置最佳的输出方式；为此`webpack`提供了 `optimization`属性供使用者针对环境做配置。

 <!--more-->

# optimization 设置方式

`optimization`拥有很多的属性去设置各个关于`bundle`产出最佳化的配置。

```javascript
module.exports = {
  // 优化选项
  optimization: {
     ...
  }
}
```

下面讲解一些比较重要的属性。讲解的例子会配置`none`模式，避免模式设定影响结果，并且以各个数值作为`bundle`的档名方便观察。



>**ps**：很多人不知道`webpack`中的`module`,`chunk`,`bundle`的区别是什么？
>
>**解释**：`webpack`中的`module`,`chunk`,`bundle`其实就是同一份逻辑代码在不同的转换场景下取的三个名字： 我们直接写出来的是`module`,`webpack`处理的时候是`chunk`,最后生成浏览器可以直接运行的是`bundle`，
>
>具体可以看看这篇文章  [webpack 中，module，chunk 和 bundle 的区别是什么？](https://www.cnblogs.com/skychx/p/webpack-module-chunk-bundle.html) 讲的特别好！

# minimize

`minimize`的值是一个布尔值，默认值是`true`。主要是告知`webpack`使用 [TerserWebpackPlugin](https://webpack.docschina.org/plugins/terser-webpack-plugin/)或者是在`optimization.minimizer`中的插件压缩`bundle`。

```javascript
module.exports = {
  // 配置`none`模式，避免模式设定影响结果
  mode: 'none',
  entry: {
    index: './src/index.js',
  },
  // 优化选项
  optimization: {
      minimize: true,
      // minimize: false,
  }
}
```

我们对`minimize`字段分别设置`true` 和`false`。打包之后的结果如下：

   {% asset_img 1.png %}

可以看出，`minimize: true`输出的`bundle`小了很多。

# minimizer

`minimizer`属性存放一个数组，数组里面可以存放用于代码压缩的插件。可以存放一个或者多个，覆盖默认压缩工具。

即：允许我们通过提供一个或者多个定制过的  [TerserPlugin](https://webpack.docschina.org/plugins/terser-webpack-plugin/) 的实例，覆盖默认压缩工具。

使用语法：`[TerserPlugin]或者[function(compiler)]`

`webpack.config_minimizer.js`

```javascript
const terserPlugin = require('terser-webpack-plugin');
module.exports = {
  mode: 'none',
  entry: './src/index.js',
  optimization: {
    minimize: true,
    minimizer: {
      // 1：直接使用的方式
      new terserPlugin({
         // 你的配置
      }),
    }
  }
}
```

具体的 [TerserWebpackPlugin](https://webpack.docschina.org/plugins/terser-webpack-plugin/)配置。当然不同的压缩插件的配置方式，直接看他们的使用教程即可。

也是可以使用函数的形式的；如：

````javascript
module.exports = {
  mode: 'none',
  entry: './src/index.js',
  optimization: {
    minimize: true,
    minimizer: {
 			// 2：使用函数的形式
      function (compiler) {
        const webpackParallelUglifyPlugin = require('webpack-parallel-uglify-plugin');
        new webpackParallelUglifyPlugin({
          // 你的配置
        }).apply(compiler)
      }
    }
  }
}
````

具体的 [webpack-parallel-uglify-plugin](https://www.npmjs.com/package/webpack-parallel-uglify-plugin)。

也可以使用`...`来访问默认值



