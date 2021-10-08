---
title: Service Worker 初探
date: 2021-09-08 13:34:21
tags:
hidden: true
---

`Service Worker`是浏览器再后台独立于网页运行的脚本，打开了通向不需要网页或者用户交互功能的大门， 现在它包含**推送通知**和**后台同步**等功能。 将来，`Service Worker`将会支持如定期同步或地理围栏等其他功能，

本篇主要是讲解它的核心功能拦截和处理网络请求，包括通过程序来管理缓存中的响应。

这个 `API`之所以令人兴奋，是因为它可以支持离线体验，让开发者能够全面控制着一体验。

在`Service Worker`出现前，存在能够再网络上为用户提供离线体验的另外一个  `API` 称为  [AppCache](https://www.html5rocks.com/en/tutorials/appcache/beginner/)。AppCache 存在的许多相关z，在设计 `Service Worker`的时候已经给予避免。

`Service Worker`相关注意事项：

- 它是一种 `JavaScript worker`, 无法直接访问`DOM`。`Service Worker`通过响应` postMessage`接口发送消息来与其控制的页面通信，页面可在必要的时对 `DOM`进行操作。

- `Service Worker`是一种可编程的网络代码，让你能够控制页面所发送网络请求的处理方式。

- `Service Worker`在不用时会被中止，并且在下次有需要的时候重启，所以你不能依赖 `Service Worker` 处理程序中的全局状态。

- `Service Worker`可以访问  [IndexedDB API](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)。

- `Service Worker`广泛的利用了 `promise`

  

`Service Worker`的生命周期

```
  path: '/veew/security',
    title: '安全组',
    component: Security,
  },
  {
    path: '/veew/security/create',
    title: '创建安全组',
    component: CreateSecurity,
```

