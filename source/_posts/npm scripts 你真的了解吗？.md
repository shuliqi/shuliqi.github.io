---
title: 你真的了解 npm 脚本吗？
date: 2022-02-23 18:42:10
tags:
---

最近在写一个`npm`包。写的过程中遇到的最大问题是：如何在 `npm install <package>` 之前能够先执行一个脚本呢？。那么本篇文章将带着这个问题去了解 `  npm` 脚本。读完本篇文章你可以了解：

- 什么是 `npm` 脚本？
- 如何使用 `npm` 脚本
- `npm` 脚本相关的生命周期（重点）

<!--more-->

# 什么是 npm 脚本 ？

`npm  `  脚本是 `package.json `文件 `scripts`  字段存储的脚本命令。存储的脚本命令可以是：

- 预设的生命周期脚本

- 自定义的任何脚本

`npm  `  脚本的目的是提供一种简单的方法来执行重复的任务。比如：

- 启动项目
- 打包项目
- 执行单元测试
- ...

当我们要定义一个 `npm` 脚本时， 我们需要做的就是设置它的名称，并且在 `package.json `文件 `scripts` 添加该名称和脚本。如：

```javascript
  "scripts": {
    "build": "node index.js",
  }
```

以上是 `package.json`文件的一个片段，可看出 `scripts` 字段是一个对象， 它每一个属性名称对应着一个脚本。如 `build` 对应的脚本是 `node index.js`。



可以使用 `npm run <event>  `  或者 `npm run-script  <event> `来执行相应的命令。 

```bash
$ npm run  build
# 或
$ npm run-script build

# 等同于执行
$ node index.js
```



想要查看当前的所有的 `npm` 脚本命令。可以使用不带任何参数的 `npm run `来查看。

```bash
$ npm run 
```

# 执行原理

`npm` 脚本的原理是每当执行 `npm run ` 时，就会自动新建一个`shell`，然后在这个  `shell`  里面执行指定的脚本命令。因此，只要是 `shell`可以运行的命令，就可以写在 `npm` 脚本里面。

> 上面所说的 `shell` 一般是指 `Bash`

但是有一点需要注意， `npm run ` 新建 `shell`的时候,  会把当前的目录的 `node_modules/.bin`  子目录加入`PATH `变量。 执行完之后再将 `PATH` 变量恢复原样。

> `PATH `变量即`PATH `环境变量，一般是指在操作系统中用来指定系统运行环境的一些参数，如： 临时文件夹和系统文件夹位置等。`windows`和`iOS`操作系统中的`PATH`环境变量，当要求系统运行一个程序而没有告诉告诉它程序所在的完整路径时，系统除了在当前目录下面寻找此程序外，如果找不到， 可以到 `PATH` 中执行的路径去找。

由于这一特点，所以当前目录的`node_modules/.bin`子目录里面的所有脚本，都可以直接用脚本名调用 ，而不必加上路径。如：

```bash
  "scripts": {
    "lint": "./node_modules/.bin/eslint .",
  }
```

而不用写成这样:

```bash
  "scripts": {
    "lint": "eslint .",
  }
```

`npm `脚本唯一要求就是可以在 `shell`执行，因此它不一定是 `NODE` 脚本，任何可执行的文件都可以可以写在里面。

`npm  `脚本的退出码，也遵守 `Shell` 脚本规则，如果退出码不是 0， `npm ` 就认为这个脚本执行失败。

# 通配符

因为 `npm` 脚本就是 `shell` 脚本，所以是可以使用`shell` 通配符。

```javascript
  "scripts": {
    "lint": "eslint *.js",
    "lint": "eslint **/*.js"
  }
```

上面代码表示 `*` 表示任意文件名，`**`表示任意一层子目录。



# 生命周期脚本

文章一开始我们就有一个问题： 如何在 `npm install <packge>` 之前先执行一个脚本呢？ 其实这就用到了 `npm `的生命周期脚本了。即官方自己的命令属性。

```js
  "scripts": {
    "build": "npm run clean && rollup -c",
    "postinstall": "node index.js"
  }
```

如上这个`package.json`字段，其中`build` 是用户自己自定义的命令属性，而`prepublish`则是`npm`自带的生命周期钩子，表示在`npm publish` 之前执行该脚本。



那么下面的内容我们就主要来介绍生命周期脚本。在这之前， 我们先初始化一个例子以供我们之后演示。



# 例子准备

我们先做个前期的例子准备， 之后用例子一一介绍相关的钩子

1. 创建文件夹

```bash
$ mkdir shuliqi-npm-demo & cd shuliqi-npm-demo
```

2.  初始化

```bash
$ npm init
```

3. 创建 `index.js`文件

```bash
$ touch index.js
```

```javascript
// index.js
console.log("shuliqi-npm-demo")
```

那么我们现在已经有了一个基础的例子。   



# pre[event 和 post[event] 

当我们执行任意的 `npm run <event>` 脚本的时候， 回依次自动触发`pre<event>`, `post<event>`的生命周期。

假如有一个命令叫 `build`。那么当执行`npm run build`时：

1. `prebuild`
2. `build`
3. `postbuild`

例子：

`packge.json:`

```json
{
  "name": "shuliqi-npm-demo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",

  "scripts": {
    "prebuild":  "echo \"--------prebuild------------\" ",
    "build":  "echo \"--------build------------\" ",
    "postbuild":  "echo \"--------prepare------------\" "
  },
  "author": "shuliqi",
  "license": "ISC"
}

```

如上代码我们执行 `npm run build` 的时候，结果如下：

{% asset_img 1.png %}

> `pre<event>`或则`post<event>`中的 `event` 可以是我们自定义的，也可以是系统自带的， 如何是我们自定义的， 我们需要把 `<event>`脚本也写上：如上是系统的， 是不需要写的` <event>` 脚本。具体可以看下面的内容。 如上的例子是自定义的`<event>`

本例子如需代码可点击： [npm-demo](https://github.com/shuliqi/npm-demo/tree/auto_pre_post)

# npm publish 

该命令是对包进行发布。当执行 `npm publish`  会执行如下的顺序脚本：

1. `prepublishOnly`
2. `prepack`
3. `postpack`
4. `publish`
5. `postpublish`

例子：

`packge.json:`

```json
{
  "name": "shuliqi-npm-demo",
  "version": "1.0.17",
  "description": "",
  "main": "index.js",
  "scripts": {
    "prepublishOnly":  "echo \"--------prepublishOnly------------\" ",
    "prepack":  "echo \"--------prepack------------\" ",
    "prepare":  "echo \"--------prepare------------\" ",
    "postpack":  "echo \"--------postpack------------\" ",
    "postpublish":  "echo \"--------postpublish------------\" "
  },
  "author": "shuliqi",
  "license": "ISC"
}
```

结果：

{% asset_img 2.png %}

本例子如需代码可点击： [npm publish](https://github.com/shuliqi/npm-demo/tree/npm_publish)

# npm pack 

该命令是对当前目录下任何可安装的内容（软件包，`tarball`，`tarball url`，`name@tag`，`name@version`，名称或者作用域名称）提取到缓存中。然后将 `tarball `复制到当前工作环目录，名为`<name-version>.tgz`。

当执行 `npm pack`的时候会执行如下的脚本顺序：

-  `prepack `
-  `postpack`

例子：

`packge.json:`

```json
{
  "name": "shuliqi-npm-demo",
  "version": "1.0.17",
  "description": "",
  "main": "index.js",
  "scripts": {
    "prepack":  "echo \"--------prepack------------\" ",
    "postpack":  "echo \"--------postpack------------\" "
  },
  "author": "shuliqi",
  "license": "ISC"
}
```

结果：

{% asset_img 3.png %}

本例子如需代码可点击： [npm pack](https://github.com/shuliqi/npm-demo/tree/npm_pack)

# npm install  

该命令是用来安装依赖的。具体操作是当发出`npm install <packge>`的命令时会发生如下的情况：

1. `npm` 向 `registry` 查询模块压缩包的网址。
2. 下载压缩包，存放在 `~/.npm` 目录。
3. 解压压缩包到当前目录的 `node_modules` 目录。

当执行 `npm install <packge>`的时候也会有生命周期，具体执行的脚本顺序如下：

1. `preinstall`

2. `install`

3. `postinstall`

例子：

`packge.json：`

```json
{
  "name": "shuliqi-npm-demo",
  "version": "1.0.18",
  "description": "",
  "main": "index.js",
  "scripts": {
    "preinstall":  "echo \"--------preinstall------------\" ",
    "postinstall":  "echo \"--------postinstall------------\" "
  },
  "author": "shuliqi",
  "license": "ISC"
}
```

表示在安装依赖之前先执行`preinstall`脚本。之后执行`postinstall`脚本。

到这我们的代码已经发成了， 现在我们需要把这个包发布，最后来 `npm install`该包看看结果。发布只需要在项目根目录执行`npm publish` 即可。

结果：

我们执行 `npm install shuliqi-npm-demo@1.0.18 --loglevel info `来下载我们这个 `npm` 包。 其中 `--loglevel info` 表示我们需要在控制台显示 `info`级别的信息。

> 关于 [--logleve 详情](https://docs.npmjs.com/cli/v8/using-npm/logging#foreground-scripts)

{% asset_img 4.png %}

从图上可看出： `preinstall` 脚本在 `postinstall`脚本之前。

> 这个生命周期也是我们文章开头想要实现的功能。解决！！！！

本例子如需代码可点击： [npm install](https://github.com/shuliqi/npm-demo/tree/npm_install)

# npm ci 

该命令跟 `npm install`类似。但是它主要是用于自动化环境中，如：测试平台，持续集成和部署，或者任何你希望确保对依赖项进行全新安装的情况。

`npm ci` 在以下这种情况会明显更快：

- 有 `package-lock.json`或者`npm-shrinkwrap.json`文件。
- `node_modules`文件夹丢失或为空。



和 `npm install`  的主要区别 `npm ci`是：

- 项目必须有 `package-lock.json`或者`npm-shrinkwrap.json`文件。

- 如果 `package-lock.json`或`npm-shrinkwrap.json` 中的依赖项 `package.json`不匹配，`npm ci`将退出并出现报错，而不是更新  `package-lock.json`或`npm-shrinkwrap.json`。

- `npm ci` 只能一次安装整个项目，不能使用该命令添加某个依赖项。

- 如果`node_modules`已经存在，它将在`npm ci`开始安装之前自动删除。

- 永远不会写入 `packge.json`或者任何包锁，安装基本上被冻结。



当执行 `npm ci` 的时候也会有生命周期，具体执行的脚本顺序如下:

- `preinstall`
- `install`
- `postinstall`

例子：

`package.json：`

```json
{
  "name": "shuliqi-npm-demo",
  "version": "1.0.21",
  "description": "",
  "main": "index.js",

  "scripts": {
    "preinstall":  "echo \"--------preinstall------------\" ",
    "postinstall":  "echo \"--------postinstall------------\" "
  },
  "author": "shuliqi",
  "license": "ISC"
}
```

然后进行该包的发布：`npm publish` 。

在当前目录下创建一个`npm_ci_demo` 用于测试：

```bash
$ mkdir npm_ci_demo
$ cd npm_ci_demo
$ npm init
$ npm install shuliqi-npm-demo@1.0.21 --save
```



在目录 `npm_ci_demo`  执行：` npm ci --loglevel info`  结果如下：

{% asset_img 5.png %}



本例子如需代码可点击： [npm ci](https://github.com/shuliqi/npm-demo/tree/npm_ci)

# npm rebuild 

该命令表示重建构建软件包。使用的场景如：在一个项目使用了`npm install`之后，当升级，降级了`Node `版本或者复制该项目到其他电脑（其他电脑`Node`可能跟当前的不一致）可使用该命令重新构建，如果重新构建，那么将使用新的`Node`（二进制文件）重新编译所有的 `C++`插件。



当执行该命令的时候，具体执行的生命周期脚本顺序如下：

- `preinstall`
- `install`
- `postinstall`



例子：

`package.json:`

```javascript
{
  "name": "shuliqi-npm-demo",
  "version": "1.0.23",
  "description": "",
  "main": "index.js",

  "scripts": {
    "preinstall":  "echo \"--------preinstall------------\" ",
    "postinstall":  "echo \"--------postinstall------------\" "
  },
  "author": "shuliqi",
  "license": "ISC"
}

```

之后我们在该目录新建一个`npm_rebuild`目录用于测试，然后在`npm_rebuild` 安装依赖：`npm i shuliqi-npm-demo@1.0.23 --save`。



使用命令：`npm rebuild --loglevel info `来重新构建。其中 `--loglevel info` 表示我们需要在控制台显示 `info`级别的信息。

{% asset_img 6.png %}



本例子如需代码可点击： [npm rebuild](https://github.com/shuliqi/npm-demo/tree/npm_rebuild)

# npm restart 

该命令表示将重启一个新的项目。相当于执行了`npm run-script restart`。



如果在`package.json`定义了`restart`脚本， 那么执行 `npm restar` 之后的生命周期顺序如下：

- `prerestart`

- `restart`

- `postrestart`

如果在 `package.json` 没有定义了 `restart `脚本。但是有`stop` 或 `start` 脚本。 那么执行 `npm restar` 之后的生命周期顺序如下：

- `prerestart`

- `prestop`

- `stop`

- `poststop`

- `prestart`

- `start`

- `poststart`

- `postrestart`

  

如果以上的场景都不符合， 那么执行 `npm restart`将会直接报错。



例子1：

`package.json:`

```json
{
  "name": "shuliqi-npm-demo",
  "version": "1.0.24",
  "description": "",
  "main": "index.js",
  "scripts": {
    "prerestart": "echo \"--------prerestart------------\" ",
    "restart": "echo \"--------restart------------\" ",
    "postrestart": "echo \"--------postrestart------------\" "
  },
  "author": "shuliqi",
  "license": "ISC"
}
```

执行`npm restart`结果如下：

{% asset_img 7.png %}



本例子如需代码可点击： [npm restart 第一种场景 ](https://github.com/shuliqi/npm-demo/tree/npm_restart)



例子2：

`package.json:`

```json
{
  "name": "shuliqi-npm-demo",
  "version": "1.0.24",
  "description": "",
  "main": "index.js",
  "scripts": {
    "prerestart": "echo \"--------prerestart------------\" ",
    "prestop": "echo \"--------prestop------------\" ",
    "stop": "echo \"--------stop------------\" ",
    "poststop": "echo \"--------poststop------------\" ",
    "prestart": "echo \"--------prestart------------\" ",
    "start": "echo \"--------start------------\" ",
    "poststart": "echo \"--------poststop------------\" ",
    "postrestart": "echo \"--------prestart------------\" "
  },
  "author": "shuliqi",
  "license": "ISC"
}

```

执行`nnpm restart`结果如下：

{% asset_img 8.png %}



本例子如需代码可点击： [npm restart 第二种场景 ](https://github.com/shuliqi/npm-demo/tree/npm_restart_2)

# npm start

该命令将尝试执行 `package.json` 定义的 `start  `脚本。分为以下这两种情况：



如果 `package.json` 没有`start `脚本。 那么将会执行 `node server.js `：

{% asset_img 9.png %}



如果 `package.json` 有`start `脚本，那么执行生命周期如下：

- `prestart`
- `start`
- `poststart`

例子

`package.json:`

```json
{
  "name": "shuliqi-npm-demo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "prestart": "echo \"--------prerestart------------\" ",
    "start":  "echo \"--------start------------\" ",
    "poststart":  "echo \"--------poststart------------\" "
  },
  "author": "shuliqi",
  "license": "ISC"
}
```

结果：

{% asset_img 10.png %}

# npm stop 

该命令将尝试执行 `package.json` 定义的 `stop  `脚本。分为以下情况：

如果 `package.json` 有定义`stop `脚本，那么执行生命周期如下：

- `prestop`
- `stop`
- `poststop`

{% asset_img 11.png %}

如果 `package.json` 没有定义`stop `脚本，它则不会像 `npm start` 一样去执行默认的脚本。而是直接报错

# npm test 

该命令将尝试执行 `package.json` 定义的 `test  `脚本。如果没有定义将会直接报错。 如果`package.json` 定义有 `test  ` 脚本。 那么执行的生命周期顺序如下：

- `pretest`
- `test`
- `posttest`

# 缺少 npm unstall  说明

最后来说明关于缺少`npm unstall`的原因。在`npm v6`是有 `uninstall` 生命周期脚本。但 `npm v7` 没有。去掉的原因是：没有明确的方法可以为脚本提供足够的上下文以使其有用。

因为删除包的方式很多：

- 用户直接卸载了这个包
- 用户卸载了依赖包，因此正在卸载此依赖项
- 用户卸载了一个依赖包，但另一个包也依赖于这个版本
- 此版本已作为副本与另一个版本合并
- 等等

由于缺乏必要的上下文，`uninstall`生命周期脚本没有实现并且无法运行。

