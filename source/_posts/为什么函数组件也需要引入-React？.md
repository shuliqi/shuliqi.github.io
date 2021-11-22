---
title: 为什么函数组件也需要引入 React？
date: 2021-11-22 16:34:27
tags: React
categories: React
---

# 问题

最近一直使用 `React` 开发， 是遇到一些觉得比较疑惑的问题；如： 我写的都是纯粹的函数组件，我们明明没有使用`React`， 为什么仍然需要在头部引入 `import React from 'react';`呢？如下面的例子：

```js
import React from 'react';

const Hello = () => {
  return <div>Hello shuliqi</div>;
};
export default Hello;
```

<!--more-->

如这个例子， 我们在头部引入了 `import React from 'react';`但是我们在代码中并没有使用`React`，但是我们去掉的话， 程序执行的时候就会报错：

{% asset_img 1.png %}

这是为什么呢？

# 原因

深入了解`React` 的底层执行原理之后我们其实就能找到答案。

原因是： `JSX` 语法只是一种语法糖，它最终是会被转译成`javaScript`语法，因为在`babel`转译之后，我们的代码就变成了：

{% asset_img 2.png %}

在`babel`转译我们的函数组件的代码之后， 会把`JSX`语法糖转换成`React.createElement`。所以说这就是为什么我们在写纯粹的函数组件的时候， 也需要引入`import React from 'react';`



# 自动引入 React

当然，我们有时候频繁的手动引入`React`过于繁琐，我们能不能直接写纯函数组件，不需要在头部引入`React`语句：`import React from 'react';`呢？

当然是可以的，我们可以使用插件`babel-plugin-react-require`: [babel-plugin-react-require](https://github.com/vslinko/babel-plugin-react-require)来实现自动实现`React`的自动导入， 实际上这个插件的功能非常简单，在转移的时候，在文件头部插入`import React from 'react'`。

安装：

```
npm install babel-plugin-react-require --save-dev
```

`.babelrcreact-require`

```
{
  "plugins": [
    "react-require"
  ]
}
```





