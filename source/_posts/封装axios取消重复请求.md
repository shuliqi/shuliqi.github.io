---
title: 手把手封装axios取消重复请求
date: 2020-08-31 19:23:16
tags: JavaScript
---



在我们web开发过程中，很多地方需要我们取消重复的请求。但是哪种场合需要我们取消呢？ 我们如何取消呢？带着这些问题我们阅读本文。

阅读完本文，你将了解以下内容：

 <!--more-->

* 需要取消重复请求的场景
* 我们如何取消重复请求
* axios如何取消重复的请求
* 封装axios
* 如何给开源的项目提供源码
*  如何在本地调试npm包

# 提出问题

最近做的项目中，用的用户经常遇到这样的问题：

* 用户频繁切换筛选条件去请求数据，初次的筛选条件数据量大。用的时间比较多。 后面的筛选条件的数据量小。导致后面请求的数据先返回。内容先显示在页面上。但是等一段时间，初次(或者前面)的请求数据返回了， 会覆盖后面的请求的数据。这就导致了筛选条件和内容不一致的情况。
* 用户点击了一次提交按钮，接口没有很快响应，导致页面没办法做逻辑语句判断的提示。用户觉得可能没提交上，便会快速又点了按钮几次。如果后端没有去重的判断，就会导致数据中有很多条重复的数据。

这些问题给用户的体验是很不友好的。那么取消无用的请求是很有必要的。

# 解决思路

我们用的请求库是axios。那么我们可以在请求的时候拦截请求判断当前的请求是否重复，如果重复我们就取消当前的请求。大致的实现过程如下：

**我们把目前处于pending的请求存储（假如我们放在一个数组）起来。每个请求发送之前我们都要判断当前这个请求是否已经存在于这个数组。如果存在，说明请求重复了，我们就在数组中找到重复的请求并且取消。如果不存在，说明这个请求不是重复的，正常发送并且把这个请求api添加在数据中，等请求结束之后删除数组中的这个api。**

我们这个解决思路有了，但是axios如何取消请求的呢？ 我们先来了解下

# axios 如何取消请求

查看axios文档发现axios [提供了两种取消请求的方法](http://www.axios-js.com/zh-cn/docs/#%E5%8F%96%E6%B6%88)

* 第一种方法

  通过`axios.CancelToken.source`生成取消令牌`token`和取消方法`cancel`

  这是官方给的例子：

  ```javascript
  const CancelToken = axios.CancelToken;
  const source = CancelToken.source();
  
  axios.get('/user/12345', {
    cancelToken: source.token
  }).catch(function(thrown) {
    if (axios.isCancel(thrown)) {
      console.log('Request canceled', thrown.message);
    } else {
      // handle error
    }
  });
  
  axios.post('/user/12345', {
    name: 'new name'
  }, {
    cancelToken: source.token
  })
  
  // cancel the request (the message parameter is optional)
  source.cancel('Operation canceled by the user.');
  ```

  我们自己写个例子：[axios取消请求第一种方法](http://jsfiddle.net/shuliqi/pfj6nsy9/354/) 

  <iframe width="100%" height="300" src="//jsfiddle.net/shuliqi/pfj6nsy9/356/embedded/js,result/dark/" allowfullscreen="allowfullscreen" allowpaymentrequest frameborder="0"></iframe>

  在这个例子中可以通过注释：` source.cancel('手动把请求被取消了');`这段代码来看看

* 第二种方式

  通过传递一个 executor 函数到 `CancelToken` 的构造函数来创建 cancel token;

  官方的例子：

  ```javascript
  const CancelToken = axios.CancelToken;
  let cancel;
  
  axios.get('/user/12345', {
    cancelToken: new CancelToken(function executor(c) {
      // executor 函数接收一个 cancel 函数作为参数
      cancel = c;
    })
  });
  
  // cancel the request
  cancel();
  ```

  我们自己来写一个例子看看 [axios取消请求第二种方法]](http://jsfiddle.net/shuliqi/ucy6vde1/7/)

  <iframe width="100%" height="300" src="//jsfiddle.net/shuliqi/ucy6vde1/7/embedded/js,result/dark/?bodyColor=1c2128" allowfullscreen="allowfullscreen" allowpaymentrequest frameborder="0"></iframe>

# 封装axios

解决取消请求的思路有了，取消请求的办法也有了，那么剩下的就是封装了

由于同事之前已经封装了axios [very-axios](https://github.com/verymuch/very-axios)（基于 axios 进行二次封装，更简单、更统一地使用 axios。）。那么我们就这个基础上提一个pr吧。那么从现在开始我们就一步一步的来实现，这个过程包含了【如何给开源的项目贡献代码】【如何在本地调试npm】如果已经料机的同学可以直接略过。

## 准备工作

由于同事已经封装了axios并且已经开源了。那么我贡献代码的方式主要有两种：

* 代码仓库的管理者给我们添加这个仓库的写入权限，如果这样，我们就可以直接提push。
* 如果我们没有权限(大多数情况)。我们使用经典的fork & pull request 的方式来提交代码。

我们采用的第二种方式。 我们去 [very-axios](https://github.com/verymuch/very-axios)把代码fork到自己的仓库(如果你还没有自己的github,需要自己注册下哦)。

{% asset_img 1.png %}

那么你回到自己的github仓库下面就会看有一个一摸一样的项目

{% asset_img 2.png %}

那么我们现在就可以`git clone `这个仓库的代码到本地，新建branch进行开发,就比如我新建了一个这样的branch：

{% asset_img 3.png %}

现在已经有本地的代码了，但是我们调试呢？如何确保我们改的是对的呢？那么就需要我们本地调试本地的npm包了。 那就需要`npm link` 了 

首先在我们要修改的npm 包中`npm link`,,如：

{% asset_img 4.png %}

之后我们会得到

```
/Users/shuliqi/.nvm/versions/node/v12.17.0/lib/node_modules/very-axios -> /Users/shuliqi/study/axios/very-axios
```

这意思就是我们把`very-axios`链接到全局的`node_modules`

 然后我们进入我们my-project-of-axios 目录下面执行`npm link very-axios  ` 如图：

{% asset_img 5.png %}

这意思就是`very-axios`被安装在`my-project-of-axios` 下面了。`very-axios`的修改都会同步到`my-project-of-axios`。就实现本地测试了。

我们在`my-project-of-axios`中的`HelloWorld.vue`文件中做列子。

如果这里看的不是很懂的同学可以 看看这两篇文章 [如何在本地调试npm包](https://github.com/allenGKC/Blog/issues/13)  [如何使用 GitHub Flow 给开源项目贡献代码](https://juejin.im/post/6844903636863041550)

## 开始封装

准备工作完成了, 那我们开始封装的事情。根据我们之前的思路。我们采用axios 如何取消请求的第一种方式。

声明一个Map。用来存储每个请求的 标识 和 取消的函数

```javascript
// stores the identity and cancellation function for each request
this.pendingAjax = new Map();
```

自定一个字段来让用户自己决定是否需要取消重复的请求

```javascript
// whether to cancel a duplicated request
cancelDuplicated = false,
```

自定一个字段来让用户是否有全局的统一的设置重复标识的函数。如果没有设置全局的统一的函数，则默认是请求的`method` 和` url`作为重复的标识

```javascript
// how to generate the duplicated key
duplicatedKeyFn,
this.duplicatedKeyFn = isFunction(duplicatedKeyFn) ? duplicatedKeyFn : (config) => `${config.method}${config.url}`;
```

添加请求

```javascript
/**
 * add request to pendingAjax
 * @param {Object} config
 */
addPendingAjax(config) {
  // if need cancel duplicated request
  if (!this.cancelDuplicated) return 
  const veryConfig = config.veryConfig || {};
  const duplicatedKey = JSON.stringify({
    duplicatedKey:  veryConfig.duplicatedKey || this.duplicatedKeyFn(config), 
    type: REQUEST_TYPE.DUPLICATED_REQUEST,
  });
  config.cancelToken = config.cancelToken || new axios.CancelToken((cancel) => {
    // if the current request does not exist in pendingAjax, add it
    if (duplicatedKey && !this.pendingAjax.has(duplicatedKey)) {
      this.pendingAjax.set(duplicatedKey, cancel);
    }
  });
}
```

这里面我们可以使用`duplicatedKey`字段来让用户对单一请求自定义重复的标识。或者可以使用一个函数`duplicatedKeyFn`统一的让用户自定义重复的标识

删除请求

```javascript
  /**
   * remove the request in pendingAjax
   * @param {Object} config
   */
  removePendingAjax(config) {
    // if need cancel duplicated request
    if (!this.cancelDuplicated) return
    const veryConfig = config.veryConfig || {};
    const duplicatedKey = JSON.stringify({
      duplicatedKey:  veryConfig.duplicatedKey || this.duplicatedKeyFn(config), 
      type: REQUEST_TYPE.DUPLICATED_REQUEST,
    });
    // if the current request exists in pendingAjax, cancel the current request and remove it
    if (duplicatedKey && this.pendingAjax.has(duplicatedKey)) {
      const cancel = this.pendingAjax.get(duplicatedKey);
      cancel(duplicatedKey);
      this.pendingAjax.delete(duplicatedKey);
    }
  }
```



封装好了， 我们在在哪里使用呢？肯定是在请求开始之前和请求完成之后使用。

在请求之前

```javascript
// intercept response
this.axios.interceptors.request.use((config) => {
  // check the previous request for cancellation before the request starts
  this.removePendingAjax(config);
  // add the current request to pendingAjax
  this.addPendingAjax(config);
  // ...
});
```

在请求完成之后去掉该请求

```javascript
// intercept response
this.axios.interceptors.response.use(response => {
  removePending(response) 
  return response
}, error => {
  // ...
})
```

到现在已经完成了该有的功能， 但是取消请求的错误我们不该返回给用户。所以：

```javascript
(err) => {
// whether is the type of duplicated request
let isDuplicatedType;
try {
  const errorType = (JSON.parse(error.message) || {}).type
  isDuplicatedType = errorType === REQUEST_TYPE.DUPLICATED_REQUEST;
} catch (error) {
  isDuplicatedType = false
}
if (isDuplicatedType) return; 
}

```

我们在请求完成之后的err里面做一个判断，判断如果当前请求是取消的类型，我们就不返回给用户错误的提示信息。



# 总结

至此 完成了我们的封装。[完成的pr可以点击这里查看](https://github.com/verymuch/very-axios/pull/1)   [本文测试npm包的项目](https://github.com/shuliqi/my-project-of-axios)







有任何问题可以联系我联系，联系方式：shuliqi@outlook.com  ，shuliqi@360.cn 







