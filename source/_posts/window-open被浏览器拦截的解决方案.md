---
title: window.open被浏览器拦截的解决方案
date: 2019-09-04 19:28:17
tags: 浏览器
categories: 浏览器
---

### 存在的现象

最近在做一个需求，遇到了使用window.open()跳转到一个新的页面会被浏览器拦截的情况。虽然在开发环境。我们会手动放行。但是对于客户来说，不能要求用户都来通过拦截。而且在出现拦截的时候。很多小白客户根本不知道发生了啥，以为系统出来安全性问题。

此外，还发现：**当window.open为用户触发事件内部或者加载时，不会被拦截，一旦将弹出代码移动到ajax或者一段异步代码内部，马上就出现被拦截的表现了**。

<!--more-->

### 原因分析

当浏览器检测到非用户操作产生的新弹出窗口，则会对其进行阻止。因为浏览器认为这不是一个用户希望看到的页面。

**例1：**直接打开一个页面

```javascript
window.open('https://app.mokahr.com', '_blank');
```

在各个浏览器的表现：

| 浏览器       | ie8    | **chrome 40** | **firefox 34** | **opera 27** | **safari 5.1.7** |
| ------------ | ------ | ------------- | -------------- | ------------ | ---------------- |
| **是否拦截** | **NO** | **NO**        | **YES**        | **YES**      | **YES**          |

**例2：**ajax中的使用window.open

```javascript
$.ajax({
    type:"post",
    url:"Webservices/WS_BBS_Login.asmx/GetUserInfo",
    data:"{}",
    dataType:"json",
    success:function(result){
      window.open(result.url, '_blank');
    }
 });
```

也会出现浏览器拦截的情况。

**例3：**但是对于这样的代码：

```javascript
document.body.addEventListener('click', function() {
   window.open('https://app.mokahr.com', '_blank');
});
```

所有的浏览器都不会被拦截

**结论：各浏览器对拦截时机的判断不一致，而对于放在ajax回调中的代码，也会被拦截。但是，被浏览器拦截我们代码中要弹出的窗口并不是程序员所希望的。**



### 解决办法

1. **用a标签代替，这样用户点击这个超链接，浏览器会认为它是打开一个新的链接，所以就不会拦截**

   ```
   <a href="url" target="_blank" />
   ```

   也可以自己创建一个`a`标签。

   给出如下函数，将此函数绑定到click的事件回调中，就可以避免大部分浏览器对窗口弹出的拦截。

   ```
   function newWin(url, id) {
     var a = document.createElement('a');
     a.setAttribute('href', url);
     a.setAttribute('target', '_blank');
     a.setAttribute('id', id);
     // 防止反复添加
     if(!document.getElementById(id)) document.body.appendChild(a);
     a.click();
   }
   ```

2. 使用form的submit方法打开一个页面

   ```
   <html>
       <header>
     </header>
       <body>
        <form id="shu" action="form_action.asp" method="get"></form> 
       </body>
       <script>
         $("#shu").attr('target', '_blank');
         $("#shu").submit();
       </script>
     </html>
   ```

1. 通过定时器

   ```
   var newOpenWindow=window.open();
   setTimeout(function(){
   　　newOpenWindow.location=locationurl;
   }, 1000);
   ```

1. 终极解决方案–先弹出窗口，然后重定向

   **注意：大家注意，以上方法不适合放在ajax的回调函数中，如果放在回调函数中，依然会被浏览器拦截。**

   这种方法的核心思想就是：先通过用户点击打开页面，然后再对页面进行重定向。代码如下：

   ```javascript
   xx.addEventListener('click', function () {
       // 打开页面，此处最好使用提示页面
       var newWin = window.open('loading page');
   
       ajax().done(function() {
           // 重定向到目标页面
           newWin.location.href = 'target url';
       });
    });
   ```

此篇文章是参考： [window.open被浏览器拦截的解决方案](http://zakwu.me/2015/03/03/dan-chu-chuang-kou-bei-liu-lan-qi-lan-jie-de-jie-jue-fang-an/)， [ajax回调中window.open弹出的窗口会被浏览器拦截的解决方法](https://www.cnblogs.com/hss-blog/p/10194830.html)，