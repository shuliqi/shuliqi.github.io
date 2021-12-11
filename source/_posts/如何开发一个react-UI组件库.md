---
title: 如何开发一个react UI组件库
date: 2021-12-02 18:52:08
tags:
---

一直都想实现一个自己的 `UI` 组件`npm`库。这样可以沉淀自己的组件，也可以学习更多的知识点；但是有很多的概念不是很了解，所以一致觉得目标太过遥远。但是在了解一些细节和原理之后，构建自己的组件库其实也是一件很简单的事。

 <!--more-->

# 前期

通过一通查找阅读， 最终觉得使用  [dumi](https://d.umijs.org/zh-CN) 来构建是最好的。`dumi` 是一款为组件开发场景而生的文档工具。它提供的脚手架可以直接帮我们构建组件开发，文档编写， 打包文档，打包组件等功能。 能帮助我们省掉很多的工作。

> 当然了， 也是可以不用的， 只是你需要自己搭个架子， 自己引入一编写的工具如 [docz](https://www.docz.site/)。自己编写打包脚本等等。

那我们我们接下来一步一步的开始吧！

预览地址： https://shuliqi.github.io/shuliqi-design/

# 准备

首先我们的有 [node](https://nodejs.org/en/) 环境， 确保node的版本在 10.13 以上。可通过如下命令查看：

```bash
$ node -v
v14.17.6
```

# 项目创建

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
$ npm install
```

脚手架创建完成的目录为：

{% asset_img 1.png %}

我们可以看出来开发脚手架创建了包含  `dumi`和基础的文档之外， 还包含了一个简单的组件，单元测试，组件文档和 `father build`。

执行`npm run start`或 `npx dumi start` 即可开始调试组件和编写文档。 

`执行npm run build` 即可打包编译。

# 配置修改

我们执行`npm run start`之后，可得到这样的服务界面：

{% asset_img 3.png %}

上面有很多信息不是我们想要的，如`favicon`,`logo`图片等。 那么这些信息我们是可以 `.umirc.ts`文件中进行修改和添加配置。具体的配置项可参考 [dumi配置项](https://d.umijs.org/zh-CN/config)

# 组件调试

 本地完成组件的开发之后，在发布到`npm `之前，需要在本地调试，避免带着问题上传到`npm`。那怎么调试呢？ 那就需要用到`npm link / yarn link` 了。

>什么是`npm link`/ yarn link ？
>
>在本地开发模块的时候， 我们可以使用`npm link`/  `yarn link`命令，将模块链接到对应的运行项目中去， 方便地对模块进行调试和测试。

**如何使用？**

我们这里使用` yarn link `为例：

我们开发完组件之后， 执行`npm run build`。之后再项目的根目录执行`yarn link`;如下：

{% asset_img 2.png %}

其实我们创建一个测试的组件的`demo`项目。这里我们就直接使用`react` 脚手架创建一个项目结构：` create-react-app demo --typescript`可得到如下的项目结构：

{% asset_img 4.png %}

我们在跟目录执行：`yarn link shuliqi-ui`如下便成功引入我们的组件：

{% asset_img 5.png %}

这时候我们使用我们的组件： `app.txs`的代码改成：

```react
// demo/src/app.tsx
import React from 'react';
import  { Foo } from 'shuliqi-ui'

function App() {
  return (
    <div className="App">
     <Foo title="引用shuliqi-ui的 Foo 组件"/>
    </div>
  );
}

export default App;
```

最后执行`npm run start`

{% asset_img 6.png %}

可以看出来， 没任何问题。

# 单元测试

单元测试是书写组件库必备的。 在`React` 中常见的测试库有两个：	[Enzyme](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fenzymejs%2Fenzyme) 和 [react-testing-library](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Ftesting-library%2Freact-testing-library)。

我们使用`dumi`创建项目结构的时候， 默认给我们写了一个测试文档， 我们可以先它的命令`npm run  test` 执行试试看：

> 小提示：关于这些命令是如何知道-->具体是在`package.json`中的`script`字段知道的

{% asset_img 7.png %}

可看出单元测试是通过的。 我们这里主要看看是用的什么测试框架和测试库。 使用的测试框架是 [jest](https://testing-library.com/docs/react-testing-library/intro/)。测试库是

[react-testing-library](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Ftesting-library%2Freact-testing-library)

> 小提示：测试库是需要在测试框架上运行的。所以单元测试是需要测试库和测试框架的

## jest

如果不会使用`jest` 的话， 我们就简单的介绍一下。

###  describe 

`describe`用来包含一块测试的代码， 通常用它来对记个测试进行分组，它也可以自己嵌套多层的。

```js
describe('<Foo />组件', () => {
  // xxx
 	describe('文案检查', () => {
     // xxx
  })
});
```

###  test

`test` 则是每一个测试，内部包含需要测试的方法，它的别名函数是`it`， 它们是等效的。

```javascript
it('组件文案是否正确', () => {
  // xxx
});
```

它可以写在`describe`里面或者外面。

### expect

`expect`  是期望值，它需要和很多匹配器使用，如`toBe`， `toEqual`匹配器。

```javascript
expect(1).toBe(1)
```

这里的意思： 期望 1 的值是 1。`toBe`默认是直接比较，如果想要判断对象是否相等就需要使用`toEqual`。

### toBeCalled

`toBeCalled` 用来匹配函数使用被调用了。通常用来测试传入组件的事件。

```javascript
test("onClick", () => {
  const fn = jest.fn()
  const btn = <button onClick={fn}>button</button>
  // 点击btn
  expect(fn).toBeCalled()
});
```

传入的测试函数需要使用`jest.fn()`创建。

### toBeTruthy 和 toBeFalsy

和名字一样，用来判断值得真假

### not

如果我们想要的测试的值不为某个值得时候， 就可以使用`not`。

```javascript
expect(2).not.toBe(1);
```

表示想测试的是： 2 值不为 1

## React Testing Library 

虽然它的名字叫 `React Testing Library ` 但是包名叫：`@testing-library/react`。 为了测试`dom`。`@testing-library/jest-dom` 包添加了一些额外的配置。

关于`@testing-library/react` 的使用， 这里就不细说了，可以到官网 [testing-library.com](https://testing-library.com/docs/queries/bylabeltext)学习。

下面会简单举几个例子， 我们在我的项目目录`src`下创建`demo.test.tsx`。 用来写下面这几个例子的。

### 测试点击

通常我们无法判断判断按钮是否点击，所以都是通过模拟用户点击后，按钮的事件是否被调用来判断的。

用到的相关的知识点： [fireEvent](https://testing-library.com/docs/dom-testing-library/api-events)，[render](https://testing-library.com/docs/angular-testing-library/api/#render)，[Bylabeltext](https://testing-library.com/docs/queries/bylabeltext)

```javascript
import '@testing-library/jest-dom';
import React from 'react';
import { render, fireEvent } from '@testing-library/react';
describe('单元测试demo', () => {
  it('测试点击', () => {
    // 测试函数
    const fn = jest.fn();

    // render 用来渲染元素
    const { getByLabelText } =render(<button aria-label="Button" onClick={fn}></button>);

    // getByLabelText 可以通过aria-label的值来获取元素
    const btn = getByLabelText('Button');

    //  模拟点击事件
    fireEvent.click(btn);

    // 期望 fn 函数被调用
    expect(fn).toBeCalled();

    // 期望 fn 被调用一次
    expect(fn).toBeCalledTimes(1);
  });
});
```

我们`npn run test`测试一下：

{% asset_img 7.png %}

### 测试 input 的值和输入

```javascript

it('测试 input 的值和输入', () => {
  // 测试函数
  const fn = jest.fn();

  const { getByTestId } = render(<input data-testid="input" onChange={fn}/>);

  //  通过 data-testid 获取元素
  const inputDom = getByTestId('input');

  // 模拟 change 事件
  fireEvent.change(inputDom, { target: { value: "shuliqi"}});

  // 期望函数 fn 被调用
  expect(fn).toBeCalled();

  // 期望input的值是： shuliqi
  expect(inputDom).toHaveValue('shuliqi');
})
```

### 测试元素是否被 `disable`

我们可以使用`toBeDisabled`来匹配是否被`disable`。

```javascript
it("测试元素是否被disable", () => {
    const { getByText } = render(<button disabled>按钮</button>);

    // getByText 从 text 获取元素
    const btnDom = getByText('按钮');

    // 期望元素是禁的状态
    expect(btnDom).toBeDisabled();
  })
```

### 测试元素包含某类名

我们可以使用`toHaveClass`来匹配是否包含某个类名。

```javascript
  it("测试元素包含某类名", () => {
    const { getByText } = render(<button className="btn">按钮</button>);

    // getByText 从 text 获取元素
    const btnDom = getByText('按钮');

    // 期望元素有 btn 类名
    expect(btnDom).toHaveClass('btn');
  })
```

### props 改变元素是否生效

在`@testing-library/react` 中需要使用`rerender`方法来改变`props`, `toHaveTextContent`来匹配内容。

```javascript
 it('props 改变元素是否生效', ()=> {
    const Demo = ({ text } : any) =>  <div aria-label="shuliqi">{ text || '默认值'}</div>;
    const {getByLabelText, rerender} = render(<Demo/>);

    // 获取 aria-label 的元素
    const dom = getByLabelText('shuliqi');

    // 当没有传 props 参数时， 期望内容是： 默认值
    expect(dom).toHaveTextContent('默认值');

    // 使用 rerender来模拟 props 改变
    rerender(<Demo text="传入的值"/>);

    //  props 改变之后，期望值是传入的值
    expect(dom).toHaveTextContent('传入的值');
  })
```

### 子元素是否包含某一类名

有时候会通过受控值为子元素添加类名，可以用`getElementByClassName`通过类名获取子元素，应用的场景如：判断下拉框是否有开启某类名，或者是列表是否存在被选择元素的类名。

```javascript
  it('子元素是否包含某一类名', () => {
    // 组件
    const Demo = ({ className }: any) => {
      return  (
        <div>
          <span className={className}>子元素1</span>
          <span className={className}>子元素2</span>
        </div>
      )
    }

    const { baseElement} = render(<Demo className="shuliqi"/>);
    // 获取具有相同名字的子类元素
    const childrenEle = baseElement.getElementsByClassName("shuliqi");

    // 期望子类元素有两个
    expect(childrenEle.length).toBe(2)
  })
```

 当然测试单元测试不仅仅只是这些,还有很多如： 测试异步， 定时器等等。我们就不在这里一一举例了。有需要的时候可以搜索看看即可

# 组件发布npm

最后， 我们的组件开发调试测试都完成之后， 就需要发布，这样别人就能`npm install ` 来使用我们的组件了

## package.json 相关字段

打包发布这一部分跟`package.json `里面的字段是有一些关系， 我们来看主要的一些字段解释。

- `main`：

  ```javascript
  // 如：
  main:"lib/index.js"
  ```

  

  定义`npm` 包的入口文件，`browser` 和` node` 环境都是可用的

- `module`: 

  ```javascript
   // 如：
   main:"es/index.js"
  ```

  定义`npm`包的`esm`规范的入口文件，`browser` 和` node` 环境都是可用的

  > 关于打包的规范， 有哪规范，每种规范是如何引用的可看这篇文章：[Javascript模块化编程总结](https://shuliqi.github.io/2021/03/06/webpack%E5%AD%A6%E4%B9%A0-%E4%BC%98%E5%8C%96%EF%BC%88optimization%EF%BC%89/%E5%89%8D%E7%AB%AF%E6%A8%A1%E5%9D%97%E5%8C%96/)

- `typings`：

  ```javascript
  // 如：
  "typings": "es/index.d.ts",
  ```

  定义包的类型声明文件。

## 打包发布

`dumi ` 已经为我们弄好打包的脚本了, 采用的是 [father](https://github.com/umijs/father) 这个包来进行构建打包。只需要执`npm run build`即可

{% asset_img 9.png %}

之后我们就可以进行发布了:

```
$ npm version patch
$ npm publish
```

> 如果没有登录的话， 需要`npm login`登录一下

由于在这过程中出现了包名已被注册了， 所以改了个名字：

```
  "name": "shuliqi-design",
```

之后发布：

{% asset_img 10.png %}

只有登录 [npm官网](https://www.npmjs.com/)就可以看到自己发布的包

{% asset_img 11.png %}



## 测试

发布了包之后我们来测试一下， 还是继续使用的之前的 `demo`项目。

```javascript
npm i shuliqi-design --save
```

我们之前的`APP.tsx` 修改成：

```javascript
// demo/src/app.tsx
import React from 'react';
import   {Foo}  from "shuliqi-design"

function App() {
  return (
    <div className="App">
     <Foo title="引用shuliqi-ui的 Foo 组件"/>
    </div>
  );
}

export default App;

```

运行之后是可以的。

那么到现在我们关于组件的编写，本地调试， 单元测试， 发布 `npm` 已经完成了。



**上传组件代码到github:**

到目前我们把我们的组件代码上传到`github`。

我们在`github`上创建一个仓库， 然后在我们的项目跟目录执行：

```
$ git init    
$ git remote add origin <自己创建的仓库.git 地址>
```

# 文档编写

我们写完一个组件之后，需要写文档，别人才知道是怎么使用你的组件的， 所以文档的编写他也是重中之重的。

我们使用的  [dumi](https://d.umijs.org/zh-CN) 就是一款为组件开发场景而生的文档工具。我们构建的时候默认给我们做好了配置。使用`markdown`方式写的。

那么接下来我们就写个例子：

修改 `Foo` 组件：

```javascript
import React from 'react';

interface Props {
   /**
   * 可以这样写属性描述
   * @description      不是必填的
   * @default _
   */
    title: string; // 文案
}

const Foo: React.FC<Props> = ({title}) => {
  return <h1>{title}</h1>
}
export default Foo;

```



`Foo`组件目录中修改`index.md`

````markdown

# Foo 组件文档
----
## 例子:

```tsx
import React from 'react';
import Foo from './index.tsx';

export default () => <Foo title="我是一个例子" />;

```
## API 
<API hideTitle src="./index.tsx"></API>

````

然后执行`npm run start`。 即可看到文档的界面。

{% asset_img 12.png %}

这样呢， 一个组件的文档就写完了。 当然这是很简单的写法， 可以根据自己的需要编写。具体的编写配置需要看： [dumi 文档](https://d.umijs.org/zh-CN)  

# 文档打包部署

执行`npm run docs:build ` 可把文档全部打包到`docs-dist`。 我们可以将 `docs-dist` 目录部署在 `now.sh`、`GitHub Pages `等静态站点托管平台或者某台服务器即可访问。

这里我们以 `GitHub Pages` 为例,

由于我自己的博客也是用的`GitHub Pages`， 所以文档就是新加的一个页面。

执行`npm run deploy`将会打包文档并且上传到`github`。 分支为：gh-page。

我们之前在`github`上创建了仓库，并且上传了我们的代码。设置`page`:

{% asset_img 13.png %}

之后就可以访问到我们的组件文档说明：https://shuliqi.github.io/shuliqi-design/

{% asset_img 14.png %}



# 总结

这样呢！我们就有了属于我们自己的组件库。之后就可以考虑我们写哪些组件进去了。放到之后慢慢写吧

如果有兴趣的可以看  [组件库的代码](https://github.com/shuliqi/shuliqi-design/tree/master)



参考文章：

- https://d.umijs.org/zh-CN
- https://github.com/jokingzhang/blog/issues/2
- https://juejin.cn/post/6968821346088255525

- https://segmentfault.com/a/1190000021663334?utm_source=sf-similar-article
- https://juejin.cn/post/6862494892569051143#heading-2





































































