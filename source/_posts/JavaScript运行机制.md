---
title: JavaScript运行机制
date: 2019-08-10 20:40:11
tags: JavaScript
---

### JavaScript运行机制

我们知道JavaScript执行的时候是单线程的。但是Javascript为什么是单线程的呢， JavaScript为什么需要异步呢，JavaScript又如何靠单线程实现异步的呢？我们又为什么要掌握JavaScript的单线程。

<!--more-->

为了回答上面的问题， 我们先看一个例子：

```javascript
setTimeout(() => {
   console.log('我执行了')
}, 1000);
```

这段代码是真的会在1秒后执行吗？答案是不一定的。这就需要我们了解JavaScript的运行机制了。

#### Javascript为什么是单线程？

JavaScript是用在web浏览中。如果JavaScript是多线程的话。那么有两个线程是必然的，如果process1和procees2都操作在同一个DOM。process1要删除这个DOM， process2要编辑这个DOM。这样就会产生矛盾。所以JavaScript还是设计成单线程的。

####  JavaScript为什么需要异步？

如果JavaScript不存在异步， 代码都是自上而下执行，如果上一行代码解析时间过长等原因，那么下面的代码就会被阻塞。这种现象对于用户来说就是“卡死”。用户体验很不好。

####  JavaScript单线程如何实现异步呢？

JavaScript是通过事件循环（event loop）来实现的，事件循环机制也就是今天要说的JavaScript运行机制。

#### 同步和异步任务

例子：

```javascript
console.log("1");
setTimeout(function() {
   console.log('2')
});
console.log('3');
```

输出结果：1 3  2

setTimeout需要延迟一段时间爱能去执行。这类代码就是一异步代码。

根据这个结果。通常我们都这么理解JS的执行原理：

1. 判断JavaScript是同步的还是异步的。同步的直接进入主线程。异步的则进入Eevent table
2. 异步任务在Eevent table注册函数，当满足出发条件后，进入event queue（事件队列）。
3. 同步任务进入主线程后一直执行，直到主线程空闲。才会去event queue中查看是否有可执行的异步任务，如果有就推入主线程。

####  宏任务和微任务

但是。按照同步和异步任务来理解JavaScript运行机制并不准确。

例如看这个例子：

```javascript
setTimeout(function(){
    console.log("a");
},0)
new Promise(function(resolve){
    console.log("b");
    resolve()
}).then(function(){
    console.log("c");
})
console.log("d");
```

上述的代码按照异步和同步的理解，结果应该是：b  d a c 。按顺序在主线程自上而下的执行。而 a c 是异步。按顺序在主线程有空后自先而后执行

可是结果并不是这样的， 结果是这样的：b  d  c  a。

这是为什么呢？ 这就需要了解宏任务和微任务。

1. **宏任务：** script全部代码，setTimeout，setInterval， I/O， UI Rendering等。
2. **微任务：**Process.nextTick（Node独有），Promise等。

![](/Users/shuliqi/mage/20180924120852420.png)

根据这个例子我们理解的Javascript是这样的。

1. 执行宏任务(主线程的同步script代码),如果遇到微任务。就把任务发到微任务的时间队列中。
2. 当前的宏任务执行完（调用栈是是空的），会去查找微任务的任务队列。将全部的微任务依次执行完毕之后。再去依次执行宏任务事件队列。

以上的例子promise的then是一微任务，因此它的执行在setTimeout之前。

#### JavaScript运行机制

通过以上的结论证明， JavaScript运行机制应该是这样的

**同步宏任务→微任务promise→微任务process.nextTick→异步宏任务**

注意的点： 在node环境下，process.nextTick的优先级高于promise。也就是可以简单理解为，在宏任务结束后会先执行微任务队列中的nextTickQueue部分，然后才会执行微任务中的promise部分。

#### 总结

最后总结开头的例子，我们可以理解为：

1秒后，setTimeout里的函数会被推入event queue，而event queue（事件队列）里的任务，只有在主线程空闲时才会执行。也就是需要同时满足两个条件（1）1秒后。（2）主线程必须空闲，这样1秒后才会执行该函数。

下面有几个例子可以看看

```javascript
console.log('script start');

setTimeout(function() {
  console.log('setTimeout');
}, 0);

Promise.resolve().then(function() {
  console.log('promise1');
}).then(function() {
  console.log('promise2');
});
console.log('script end');
```

同步宏任务：script start， script end

异步宏任务：setTimeout

微任务：promise1， promise2

根据JavaScript运行机制我们可以得出结果是：script start， script end， promise1， promise2，setTimeout



 有兴趣的话， 可以再研究研究这个例子：

```javascript
console.log('script start')

async function async1() {
  await async2()
  console.log('async1 end')
}
async function async2() {
  console.log('async2 end') 
}
async1()

setTimeout(function() {
  console.log('setTimeout')
}, 0)

new Promise(resolve => {
  console.log('Promise')
  resolve()
})
  .then(function() {
    console.log('promise1')
  })
  .then(function() {
    console.log('promise2')
  })

console.log('script end')
```











文章参考出处：https://blog.csdn.net/qq_41635167/article/details/82827970， https://juejin.im/post/5c3d8956e51d4511dc72c200#heading-12