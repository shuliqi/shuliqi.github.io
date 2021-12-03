---
title: 如何开发一个react UI组件库
date: 2021-12-02 18:52:08
tags:
---

一直都想尝试 从零实现一个UI 组件库的整个过程， 也看了好多文章，但是那些文章写着写着总是会漏个啥东西，或者有些知识点没解释清楚，最后一通下来， 啥也没会，整个过程糊里糊涂的。所以觉得我还是得自己实践， 一步一步写下整个过程吧！就当成是自己的一个记录。

 <!--more-->

# 前期

通过一通查找阅读， 最终觉得使用  [dumi](https://d.umijs.org/zh-CN) 来构建是最好的。`dumi` 是一款为组件开发场景而生的文档工具。它提供的脚手架可以直接帮我们构建组件开发，文档编写， 打包文档，打包组件等功能。 能帮助我们省掉很多的工作。

> 当然了， 也是可以不用的， 只是你需要自己搭个架子， 自己引入一些 文档编写的工具如 [docz](https://www.docz.site/). 自己编写打包脚本等等。

那我们我们接下来一步一步的开始吧！



# 准备

首先我们的有 [node](https://nodejs.org/en/) 环境， 确保node的版本在 10.13 以上。可通过如下命令查看：

```bash
$ node -v
v14.17.6
```

项目结构

`dumi `提供两种不同的脚手架：

```bash
$ npx @umijs/create-dumi-lib        # 初始化一个文档模式的组件库开发脚手架
# or
$ yarn create @umijs/dumi-lib

$ npx @umijs/create-dumi-lib --site # 初始化一个站点模式的组件库开发脚手架
# or
$ yarn create @umijs/dumi-lib --site
```

我们先创建个空目录, 之后再使用脚手架构建：

```bash
$ mkdir shuliqi-ui
$ cd shuliqi-ui
$ npx @umijs/create-dumi-lib        # 初始化一个文档模式的组件库开发脚手架
$ yarn  				# or npm install
```

我们`yarn start` 即可开始调试组件。

# 第一个组件





