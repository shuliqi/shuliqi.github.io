---
title: 前端发送http请求的常用方式
date: 2021-07-30 16:10:55
tags: 计算机网络
---



对于前端开发来说， 请求是日常工作必备的；前端主要通过请求与后端进行交互，特别在前后端分离的模式开发下，请求就更重要了。那么掌握前端发送请求的方式很重要的。那么前端请求常用的方式有哪些呢？

 <!--more-->







# from表单

这属于最原始的`http`请求方式

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>from表单方式</title>
</head>
<body>
  <div>
    <form action="https://www.fastmock.site/mock/d867c364f89208a7672e9e9d0a822417/qixiao/getFileDetail" method="get">
      <div>
        <label for="username">账号：</label>
        <input id="username" type="text" name="username" placeholder="输入你的名字">
      </div>
      <div >
        <label for="password">密码：</label>
        <input id="password" type="password" name="password" placeholder="输入密码">
      </div>
      <div>
        <label>性别：</label>
        <input type="radio" name="sex" value="男"> 男
        <input type="radio" name="sex" value="女"> 女
      </div>
      <div>
        <input type="submit" value="提交" />
      </div>
    </form>
</body>
</html>
```

当我们点击提交的时候：

{% asset_img 1.png %}

这种方式的请求的整个过程是什么呢？



> 这是最原始的一直请求方式， 现在几乎都不会采用这种方式了， 所以这里就不细讲了



# XMLHttpRequest对象



`XMLHttpRequest`一开始只是微软浏览器提供的一个接口，但是后来各大浏览器也效仿提供了这个接口， 再后来`W3C`对它进行了标准化，按照标准的前后可以分为两个版本；

## XMLHttpRequest 老版本

```javascript
  // 新建一个 XMLHttpRequest 对象
  const xhr = new XMLHttpRequest();

  // 进行请求
  xhr.open("GET", "https://www.fastmock.site/mock/d867c364f89208a7672e9e9d0a822417/qixiao/getFileDetail", true);
  xhr.send({ name: "shuliqi", age: 18 });

  // 等待服务器的响应
  xhr.onreadystatechange = function () {
    // 该函数会被调用四次， 因此需要判断状态是否是4
    if (xhr.readyState === 4 ) {
      if ( xhr.status === 200 ) {
        console.log("请求数据成功：", JSON.parse(xhr.responseText) )
      } else {
        console.log("请求数据失败：", xhr.statusText );
      }
    }
  }
```

具体的属性说明如下：

- `readyState`: 返回当前 `XMLHttpRequest`对象当前所处的状态， 有四个状态
  - 0：表示代理已被创建，但是尚未调用 `open`方法
  - 1：表示`open`已经被调用，在这个状态中可以通过 `setRequestHeader()`方法来设置请求的头部。
  - 2：表示`send()`方法已经被调用，响应头也已经被接收
  - 3：表示响应了部分结果，但是没有全部响应完
  - 4：表示请求操作已经完成，以为着传输已经彻底的完成和失败
- `status`：表示服务器返回的状态码， 等于200表示成功响应
- `responesText`： 表示服务器返回的文本数据
- `responseXML`：表示服务器返回的XML格式的数据
- `statusText`：表示服务器返回的状态文本

> 由于老版本不是统一的标准，各大浏览器在实现上有一定的差异性，所以存在一些缺陷
>
> - 只支持文本数据的传送，无法用来读取和上传二进制文件
> - 传送和和接收数据时，没有进度信息，只能提示没有完成或者已完成
> - 受到”同域限制“只能向同一域名的服务器请求数据

例子：

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>XMLHttpRequest老版本</title>
</head>
<body>
  <script>
    // 新建一个 XMLHttpRequest 对象
    const xhr = new XMLHttpRequest();
    
    // 进行请求
    xhr.open("GET", "https://www.fastmock.site/mock/d867c364f89208a7672e9e9d0a822417/qixiao/getFileDetail", true);
    xhr.send({ name: "shuliqi", age: 18 });

    // 等待服务器的响应
    xhr.onreadystatechange = function () {
      // 该函数会被调用四次， 因此需要判断状态是否是4
      if (xhr.readyState === 4 ) {
        if ( xhr.status === 200 ) {
          console.log("请求数据成功：", JSON.parse(xhr.responseText) )
        } else {
          console.log("请求数据失败：", xhr.statusText );
        }
      }
    }
  </script>
</body>
</html>
```

打开页面之后，可得到结果：

{% asset_img 2.png %}

## XMLHttpRequest 新版本

为了更好的使用`XMLHttpRequest`，`w3school`发布了标准版本，该版本弥补了老版本的一些缺陷，也是被各大浏览器厂商接受和实现，具体的功能如下：

- 可以设置`http`请求的时限
- 可以上传文件
- 可以跨域请求
- 可以获取服务端的二进制数据
- 可以获取数据传输的进度信息

一般为了更加友好的进行兼容各个浏览器，会对浏览器进行判断并进行兼容性模式来获取`XMLHttpRequest`的对象

```javascript
    // 新建一个 XMLHttpRequest 对象
    let xhr;
    if (window.XMLHttpRequest) { // Mozilla, Safari...
      xhr = new XMLHttpRequest();
    } else if (window.ActiveXObject) { // IE
      try {
          xhr = new ActiveXObject('Msxml2.XMLHTTP');
      } catch (e) {
        try {
              xhr = new ActiveXObject('Microsoft.XMLHTTP'); //IE5,6
        } catch (e) {}
      }
    }

    // 请求成功回调函数
    xhr.onload = () => {
        console.log('请求成功', xhr.response);
    };
    // 请求结束
    xhr.onloadend = () => {
        console.log('请求失败', xhr.response);
    };
    // 请求出错
    xhr.onerror = e => {
        console.log('请求出错', e);
    };
    // 请求超时
    xhr.ontimeout = e => {
        console.log('请求超时', e);
    };

    // 请求取消触发
    xhr.onabort = (e) => {
      console.log('请求被取消了', e);
    }
        
    // 设置超时时间，0 表示永不超时
    xhr.timeout = 0;

    // 初始化请求
    xhr.open("GET", "https://www.fastmock.site/mock/d867c364f89208a7672e9e9da822417/qixiao/getFileDetail", true);

    // 设置期望的返回数据类型 'json' 'text' 'document' ...
    xhr.responseType = 'json';

    // 设置请求头
    // xhr.setRequestHeader('', '');

    // 发送请求
    xhr.send({ name: "shuliqi", age: 18 });

    // 取消请求
    xhr.abort();
```

例子：

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>XMLHttpRequest老版本</title>
</head>
<body>
  <script>
    // 新建一个 XMLHttpRequest 对象
    let xhr;
    if (window.XMLHttpRequest) { // Mozilla, Safari...
      xhr = new XMLHttpRequest();
    } else if (window.ActiveXObject) { // IE
      try {
          xhr = new ActiveXObject('Msxml2.XMLHTTP');
      } catch (e) {
        try {
              xhr = new ActiveXObject('Microsoft.XMLHTTP'); //IE5,6
        } catch (e) {}
      }
    }

    // 请求成功回调函数
    xhr.onload = () => {
        console.log('请求成功', xhr.response);
    };
        
    // 设置超时时间，0 表示永不超时
    xhr.timeout = 0;

    // 初始化请求
    xhr.open("GET", "https://www.fastmock.site/mock/d867c364f89208a7672e9e9d0a822417/qixiao/getFileDetail", true);

    // 设置期望的返回数据类型 'json' 'text' 'document' ...
    xhr.responseType = 'json';

    // 发送请求
    xhr.send({ name: "shuliqi", age: 18 });
  </script>
</body>
</html>
```

结果：

{% asset_img 2.png %}















参考文档:

[XMLHttpRequest](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest)







