---
title: vue项目配置eslint+husky+lint-staged
date: 2021-04-03 14:39:04
tags: 工程配置
categories: 工程配置
---

最近使用了`vue-cli3`构建开发一个新的项目， 发现在开发的过程中有很多人会不经意把`console.log`等提交上去了。于是觉得有必要把`eslint`加上，因为在多人开发中，每个人的代码风格可能是不一样的。大团队协作时，良好的代码规范显得格外重要，因为这是保障一个团队代码风格相同、避免低级bug的途径之一。为了解决这样的问题！ 我们在每次提交的时候都可以进行一次`eslint`检查。不符合`eslint`规范的不给提交。这样就能解决问题了。

 <!--more-->

# 创建项目

下面讲解的项目范例会使用`vue-cli`来创建。

> 具体如何创建一个项目可以看 [官方的文档](https://cli.vuejs.org/zh/guide/)

## **安装全局的`vue-cli`**

```javascript
npm install -g @vue/cli
```

## **创建项目**

```javascript
vue create eslint-example
```

## 选择配置

弹出的配置中， 我们选择如下的配置：

{% asset_img 1.png %}

> 当然配置可以根据自己的需要要选择， 我这里只是举个例子



# 配置ESlint

`ESlint` 是什么？`ESlint` 是一个语法规则和代码风格的检查工具。关于一些配置项这里不做过多的讲解。

> 学习的话可以直接去官网学习：[ESlint英文官网](https://eslint.org/)，[ESlint 中文官网](https://cn.eslint.org/)

`Eslint`如何去使用呢？ 前提肯定是先安装（至于是本地安装还是全局安装，看自己项目的需要）：

```javascript
npm install eslint --save
```

> 本文在创建的时候选择的配置已经把`eslint`安装上，所以在该项目中就不需要再安装一次了。
>
> 小tips：但是这里创建项目时候选择的`eslint`版本是有问题， 这个问题下面会暴露， 这里可以先跳过

## 创建.eslintrc文件

手动在项目的根目录添加`.eslintrc`文件。

安装完之后，不要着急，我们不使用官方提供的`eslint --init`来生成配置文件而是手动添加。为什么呢？ 因为这样我们需要手动配置很多个的规则。

那如果不这样？那有没有更好的简便的方法呢？那肯定是有的。网上有一个叫 [eslint-config-standard](https://www.npmjs.com/package/eslint-config-standard)的插件，它是标准的`ESlint`规则， 我们在项目中继承这个标准就可以了。

## 安装eslint-config-standard

```javascript
npm install eslint-config-standard --save
```

如`eslint-config-standard`官网所说：如果我们是需要手动加一些配置的,需要手动安装如下的`npm`包

所以我们需要安装如下：

```javascript
npm install --save-dev eslint-config-standard eslint-plugin-promise eslint-plugin-import eslint-plugin-node --save
```

因为` eslint-config-standard `校验规则的时候需要依赖这些`plugins`去验证。

然后我们在`.eslintrc`文件添加

```javascript
{
  "extends": [
    "standard"
  ]
}
```

这时候校验肯定是校验不了， 因为少安装了一些插件。因为` eslint-config-standard `校验规则的时候需要依赖这些`plugins`去验证。

```javascript
npm install --save-dev eslint-config-standard eslint-plugin-promise eslint-plugin-import eslint-plugin-node --save
```

这时候是不是就可以验证了了么？我们执行运行项目看看：`npn run serve`: 发现抛出错误了。

{% asset_img 2.png %}

这是因为`eslint`版本过低。` eslint-config-standard `插件必须使用`7.11.0`版本以上的`eslint`。具体原因可看：https://github.com/standard/eslint-config-standard/issues/176

> 注意：这也就是我们上面说的问题；使用`eslint-config-standard` 只要需要`ESlint：7.11.0`版本以上，不然报错，无法校验。

重新安装`eslint`:

```javascript
 npm i eslint@7.11.0 --save
```

这时候运行我们的项目就不会报上面的错了。可以进行代码风格校验了。

## 安装eslint-plugin-html

因为`vue`的项目，`.vue`文件写的是类似像`html`这样的格式，它不是标准的`Javascript`文件。`ESlint`是无法识别`.vue`文件的`js`代码的。所以需要安装另外一个插件 [eslint-plugin-html](https://www.npmjs.com/package/eslint-plugin-html)。这个插件的作用是识别一个文件里面的`script`标签的`js`代码。

> `.vue`文件的`js`代码就是写在`script`标签中的。

**安装**

```javascript
npm i eslint-plugin-html --save
```

**配置**

在`.eslintrc`添加：

```javascript
{
  "extends": [
    "standard"
  ],
  "plugins": ["html"]
}
```



## package.json 添加命令

首先先去除`package.json`里面`eslint`的配置字段：`eslintConfig`

上面的步骤完成之后，在项目的`packge.json`的`script`字段修改`lint`命令

```javascript
"lint": "eslint --ext .js --ext .jsx --ext .vue src/",
"lint-fix": "eslint --fix --ext .js --ext .jsx --ext .vue src/"
```

>--ext后面加上我们需要检测文件的后缀，如`.js`、.`jsx`、 `.vue`等，紧接加上检测哪个目录下面的文件的目录，如： 我们要检查`src`目录下面的文件，就直接写`src`。

到目前为止，我们可以使用`npm run lint`来校验了。当然我们可以使用`npm run lint-fix`来自动修复。

## 安装 eslint-plugin-vue 

上面的步骤配置完成之后， 我们试着`npm run lint`。控制台是能够显示出来那些是有规则问题的，如：

{% asset_img 3.png %}

我们可以看到，那些代码是有规则问题都被标出来了。

但是我们进到相应的文件去看，相应有问题的地方却没有被标红（vscode编辑器）这是为什么呢？(之前的项目是可以被标识出来的)

这是因为：我们使用的是`vue-cli3`构建的项目，我们使用之前的方案去实现标识是无法识别的。

想要`vscode`编辑器对`.vue`文件的`eslint`检测时有错误标红出来， 则需要`eslint-plugin-vue `插件。

### 安装

```
 npm install --save-dev eslint-plugin-vue
```

### 添加`plugin`说明

`.eslintrc`文添加`plugin`说明

`.eslintrc`:

```javascript
{
	// ...其他配置项
  "plugins": [
  	// ...其他plugins
  	"vue"
  ]
  // ...其他配置项
}
```

### 添加`extend`说明

`.eslintrc`文添加`extend`说明

`.eslintrc`:

```javascript
{  
	// ...其他配置项
	extends: [
	  // ... 其他的规则
    'plugin:vue/base'
  ]
  // ...其他配置项
}
```

> 这里使用的是`base`里面的规则。更多的配置可看 [官方文档](https://eslint.vuejs.org/rules/)

### 解析器配置

`.eslintrc`配置解析器

`.eslintrc`:

```javascript
{
	// ...其他配置项
	parserOptions: {
    parser: 'babel-eslint',
    sourceType: 'module'
  }
 // ...其他配置项
}
```

完整的`.eslintrc`:

```javascript
{
  "parserOptions": {
    "parser": "babel-eslint",
    "sourceType": "module"
  },
  "extends": [
    "standard",
    "plugin:vue/base"
    
  ],
  "plugins": ["html", "vue"]
}
```

那么现在有问题的代码就会标红出来了，如：

{% asset_img 4.png %}

到目前为止， `eslint`相关的配置就完成了。

# husy

有这样的情况：我们试着提交一次有`eslint`错误的代码:

{% asset_img 5.png %}

我们发现是可以提交的。并且尝试`push`到远端的时候，也是可以的。但其实我们是希望在有`eslint`错误的时候，不能做提交和`push`这类操作的， 防止有问题的代码托送到远端。那么`husky`就可以帮助我们。

> [husky官方文档](https://typicode.github.io/husky/#/?id=automatic-recommended)

`husky` 是一个`npm`安装包，安装了之后可以很方便的配置`git hook`脚本， 以防止不规范代码被`commit`、`push`、`merge`等。

## 安装

由于目前`husky`已经升级到了 6 版本， 变化还是很大的，网上的很多文章讲解的都是4版本的，  不是很对了。所以我们直接看[husky官方文档](https://typicode.github.io/husky/#/?id=automatic-recommended)的安装方式及其使用方式

```javascript
npx husky-init && npm install 
```

我们会发现`init`生成了一个默认的`pre-commit`钩子：

{% asset_img 6.png %}

我们把该配置的`npm test`去掉。

## 配置钩子

修改`pre-commit`钩子：

```javascript
npx husky add .husky/pre-commit 'npm run lint'
```

配置`pre-push`钩子：

```javascript
npx husky add .husky/pre-push 'npm run lint'
```

配置`pre-merge`钩子：

```javascript
npx husky add .husky/pre-merge 'npm run lint'
```

现在我们试着测试，看是否生效:

{% asset_img 6.png %}

我们尝试提交了代码不符合规范的代码，确实被阻止了， 生效。

# lint-staged

我们希望每次`commit` ,`push`和`merge`等这样的操作的时候，只希望校验自己修改的文件代码规范，而不是全局的检验。 这就需要 [`lint-staged`](https://www.npmjs.com/package/lint-staged)了。

 [`lint-staged`](https://www.npmjs.com/package/lint-staged)的作用是只对`git add`缓存区的代码进行`eslint`代码规范检验。这样就能避免全局校验的问题。 符合了我们修改了什么文件，就校验什么文件。其他的的代码不做`eslint`校验。

## 安装

```javascript
npm install --save-dev lint-staged
```

## 配置

在 `package.json`中添加：

```javascript
"lint-staged": {
   "src/**/*.{js,vue}": [
     "npm run lint"
   ]
},
```

`src/**/*.{js,vue}` 是需要校验的目录， 可以根据目录去修改。



# 总结

到目前为止，配置就做完了。最后觉得在做一个多人开发的项目的时候， 前期一些代码规范的配置觉得是很有必要的。

本文中的例子github链接：[shuliqi](https://github.com/shuliqi)/**[eslint-husky-lint-staged-example](https://github.com/shuliqi/eslint-husky-lint-staged-example)**