---
title: 如何使用 Javascript 编写 shell 脚本---Google/zx
date: 2022-03-13 11:32:15
tags:
---

[Google-zx](https://github.com/google/zx)  是谷歌推出的一个开源的项目， 一个可以使用前端熟悉的` JavaScript `语法来编写 `shell` 的工具。如：

```javascript
await $`cat package.json | grep name`

let branch = await $`git branch --show-current`
await $`dep deploy --branch=${branch}`

await Promise.all([
  $`sleep 1; echo 1`,
  $`sleep 2; echo 2`,
  $`sleep 3; echo 3`,
])

let name = 'foo bar'
await $`mkdir /tmp/${name}`

```

上面代码中可以看出在` JavaScript` 中 插入了 `shell`。那么这种写法行不行呢？ 答案是可以的，如果我们使用了[Google-zx](https://github.com/google/zx)  的话。

 <!--more-->

#  问题背景

创建 `shell` 脚本的主要目的是 `shell` 脚本能够帮忙我们自动化实现一些重复的任务的。在编写脚本的时候通常会选择一些更加方便的编程预言。 对于前端开发工程师来说 `Node.js` 无疑是最好的选择了，它提供了很多核心的模块，并且还可以导入前端其他的脚本库。可以降低很多的学习成本。



但是我们尝试编写一个在`Node.js`下运行的 `shell` 脚本，发现并不是很流畅。 需要为子进程编写特殊处理；注意转义命令行参数；然后使用使用标准输出   `stdout` 和错误 `stderr`  ，显得不是很直观； 并且在使用之前需要做很多额外的操作，如：引入库等。

```javascript
// 引入 execSync 命令 from child_process 模块
const { execSync } = require("child_process");

// 同步创建了一个 shuliqi 的文件夹
execSync("mkdir shuliqi");
```

```javascript
// 引入 exec 命令 from child_process 模块
const { exec } = require("child_process");

// 异步发布一个 npm 包
exec('npm publish', (err, data) => {
  if (err) {
    console.log("发布失败:", err);
    process.exit(1);
  } else {
    console.log("发布成功", data);
  }
});
```

`Bash shell` 脚本语言试试编写`shell`脚本的最佳选择，不需要编写代码来处理子进程，并且它有用于处理

`stdout` 和 `stderr` 的内置语言特性。 但是`Bash` 编写 `shell` 脚本也不是那么容易。语法比较混乱，使得实现逻辑或处理用户提示是输入之类的事情变得不是那么方便。



那么`Google`的`zx.js`库就很有助于使用`Node.js`高效快速的编写`shell`脚本。



# 安装

可根据如下命令进行全局安装：

```bash
npm i -g zx
#  or
yarn add -g zx
```

> 不一定非得全局安装， 可以局部安装的。下面的例子，都是使用的全局安装

目前需要的环境

```javascript
Node.js >= 16.0.0
```

> 具体的`Node` 环境最新可到 [Google/zx](https://github.com/google/zx#-zx)查看。

# 使用

安装好 `zx` 之后，有两种方式可以编写脚本：

1. 将 `shell` 脚本编写在后缀名为 `.mjs`  的文件中， 不需要额外的封装了
2. 将 `shell` 脚本编写在后缀名为 `.js`  的文件中，这种方式需要使用如下方式对脚本进行封装

```js
void async function() {
  //  shell 脚本内容
}()
```



其次需要在文件头添加如下脚本：

```bash
#!/usr/bin/env zx
```



运行的时候如果是全局安装的情况下直接在命令行使用`zx <文件>`即可

```bash
$ zx ./src/index.mjs
```



举个例子：

```bash
#!/usr/bin/env zx

const fileName = "zx-test"
await $`mkdir ${fileName}`;
```

{% asset_img 1.png %}





# 内置函数



## $\`command\`

该命令主要是使用了 `child_process` 的 `spawn` ( [child_process.spawn](http://nodejs.cn/api/child_process.html#child_processspawncommand-args-options) ) 函数来执行指定的字符串，并返回一个`ProcessPromise`。如果执行的程序返回非零退出代码。 将抛出`ProcessOutput`。

举个例子：

```javascript
// 命令
const commandPromise = $`ls -1 | wc -l`;

// 同步执行命令
const result = await commandPromise;

let count = parseInt(result)
console.log(`文件的个数: ${count}`)
```



### ProcessPromise

返回的 `ProcessPromise` 的 `typescript` 接口定义为：

```javascript
class ProcessPromise<T> extends Promise<T> {
  readonly stdin: Writable
  readonly stdout: Readable
  readonly stderr: Readable
  readonly exitCode: Promise<number>
  pipe(dest): ProcessPromise<T>
  kill(signal = 'SIGTERM'): Promise<void>
}
```

其中`pipe()`方法可用于重定向标准输出：

```javascript
await $`echo "你好世界"`
  .pipe(fs.createWriteStream('file.txt'))

await $`cat file.txt`
```

结果如下：

{% asset_img 2.gif %}

关于更多的[管道例子](https://github.com/google/zx/blob/main/docs/pipelines.md)

### ProcessOutput

如果执行的程序返回非零退出代码，`ProcessOutput` 将被抛出。`ProcessPromise`的`typescript`定义为：

```javascript
class ProcessPromise<T> extends Promise<T> {
  readonly stdin: Writable
  readonly stdout: Readable
  readonly stderr: Readable
  readonly exitCode: Promise<number>
  pipe(dest): ProcessPromise<T>
}
```

举个例子：

```javascript
try {
  // 执行的程序返回非零退出码
  await $`exit 2`;
} catch (error) {
  // ProcessOutput 被抛出
  console.log(error.exitCode); // 2
}
```

结果：

{% asset_img 3.gif %}

## cd() 

`cd（）` 可用于更改当前工作目录：

```javascript
// 更改当前的工作目录为： src 目录
cd("./src");

// 输出目前所在的工作目录的绝对路径名称
await $`pwd`;
```

结果：

{% asset_img 4.gif %}

## fetch()

相当于` node` 的  [node-fetch](https://www.npmjs.com/package/node-fetch) 包。 用于网络请求。

```javascript
let resp = await fetch('https://www.fastmock.site/mock/31e59cb8bdba6b63482fd4e6914b76f2/borrow/getName')
if (resp.ok) {
  console.log(await resp.text()); // {"data":"你好世界"}
}
```

结果：

{% asset_img 5.gif %}

## question()

是对 `Node Js` 的  [readline](https://nodejs.org/api/readline.html)  包的包装。它的作用是可以提示用户输入。

```javascript
let name = await question('请输入你的名字: ')
console.log("你的名字是：", name);
```

{% asset_img 6.gif %}

> 说是  question 函数的第二个参数是选项。但是没有试出来， 还没找到原因。



## sleep()

使用 setTimeout 实现的一个等待函数。

```javascript
let name = await question('请输入你的名字: ')

// 等待 2000 毫秒之后 才会输出结果
await sleep(2000)

console.log("2000 毫秒之后输出，你的名字", name);
```



## nothrow()

捕捉 `$` 执行命令时遇到的非 0 返回值，使其不抛异常。一般来说，`Shell` 编程中` Exit Code `不为0代表有异常

```javascript
try {
  // 执行的程序返回非零退出码
  await nothrow( $`exit 2`);
} catch (error) {
  // 但是错误没有被抛出
  console.log("该log不会打印，因为error没有被抛出", error.exitCode);
}
```

举个例子：

该例子中，开始没有使用 `nothrow` 函数，执行命令行遇到非0返回值是会抛出错误的。 之后使用 `nothrow` 函数， 执行，执行命令行遇到非0返回值则不会抛出错误。

{% asset_img 7.gif %}



# 内置的全局变量

## chalk

 即 `chalk` 包，用于输出彩色的内容

```javascript
console.log(chalk.green('绿色'));
console.log(chalk.red('红色'));
console.log(chalk.yellow("黄色"));
```

{% asset_img 8.png %}

更多的使用可看: [chalk](https://www.npmjs.com/package/chalk)



## fs

 引用的 [fs-extra](https://www.npmjs.com/package/fs-extra)  包，用于完成常见的文件操作。

```javascript
fs.copy('./copy.txt', 'copy1.txt')
  .then(() => console.log('复制成功!'))
  .catch(err => console.error(`复制失败：${err}`))
```

结果:

{% asset_img 9.gif %}

例子中复制了名为 `copy.txt` 文件， 复制后的文件名为：`copy1.txt`。

## globby

引用的 [globby](https://github.com/sindresorhus/globby) 包，用于模糊搜索文件名

```javascript
const paths = await globby(['*', '!cake']);

console.log(paths);
```

## os

引用的 [os](https://nodejs.org/api/os.html) 包，用于获取系统信息

```javascript
const type = os.type();
const arch = os.arch();
console.log(`你的系统是${type}, ${arch}位`)
```



## path

引用的 [path](https://nodejs.org/api/path.html) 包，用于对路径做处理。

```javascript
const file = path.basename("/foo/bar/baz/asdf/shuliqi.txt")
console.log(file); // shuliqi.txt
```



## minimist

引用的 [minimist](https://www.npmjs.com/package/minimist) 包，用于处理命令行参数。







关于更多可直接访问 [Github](https://github.com/google/zx) 或 [NPM](https://www.npmjs.com/package/zx) 查看 。
