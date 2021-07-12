---
title: ES6学习笔记-Promise
date: 2018-3-20 13:20:04
tags: ES6
categories: ES6
---



> **我们知道在Javascript中，所有的代码都是单线程的。也就是说是同步执行的。而Promise就为异步编程提供了一种解决方案**

<!--more-->

## 基本概念

`Promise` 是一个对象，从它那里可以取得异步操作的消息。Promise 提供统一的API，各种异步操作都可以用同样的方法进行处理。

* 状态：

  一共有三种状态：**pending（进行中**)， **fulfilled（已成功)**， **rejected（已失败)**

* 特点：

  1. **对象的状态不受外界的影响。**

     只有异步的结果可以决定当前是哪一种状态（pending， fulfilled, rejected）。 其他的手段都是无法改变的。

  2. **一旦状态改变，就不会再变，任何时候都可以拿到这个状态。**

     Promise的状态只有两种可能：从pending 到 fulfilled， 从peding 到rejected。只要这两种涨停发生了，状态也就凝固了，不会再变了。

     如果在对Promise对象添加回调函数，也会立刻拿到这个结果。

* 缺点：

  1. **无法取消Promise,一旦新建就会立即执行，中途无法取消**
  2. **如果没有设置回掉函数，Promise内部抛出错误，不会反应到外部**
  3. **当处于pending状态的时候，无法知道目前进展到哪一个阶段（刚刚开始还是即将完成）**

## 基本用法

**Promise 对象是有关键字new 和Promise 构造函数创建的**

```javascript

const promise = new Promise((resolve, reject) => {
  // do something
  if ( /** 异步操作成功 */ ) {
    resolve()
  } else {
    reject()
  }
})
```

Promise构造函数的参数是一个函数， 这个函数有两个参数`resolve`, `reject`，这两个参数也是函数，由Javacript引擎提供， 不用自己部署。

**resolve函数：**将Promise对象的状态“未完成” 变成”完成“ （pending ----> fulfilled）. 在异步操作成功的时候调用， 并且会异步操作的结果作参数传递出去；

**reject函数: **将Promise对象的状态“未完成” 变成”失败“ （pending ----> rejected）. 在异步操作失败的时候调用， 并且会异步操作的结果作参数传递出去；



**Promise实例生成之后， 可以使用then方法分别指定 resolve 和 rejected 状态的回调。**

```javascript
promise.then((value) => {
	// 异步操作成功的回调
}, (error) => {
  // 异步操作失败的回调
})
```

then方法可以有两个回调函数作为参数，。 第一个回调函数是Promise对象变为resolve时调用， 第二个回调函数是Promise对象状态变为rejected时调用。其中第二个参数是可选的，不一定要提供。这两个函数都接受promise对象传出的值作为参数。

再看一个例子：

```javascript
const timeout = (time) => {
  return new Promise((resolve, reject) => {
    setTimeout(resolve, time);
  });
};
timeout(1000).then(
  (value) => {
    console.log("异步操作成功");
  },
  (err) => {
    console.log("异步操作失败了");
  }
);
```



**Promise 新建之后就会立即执行。**

```javascript

const promise = new Promise((resolve, reject) => {
  console.log("11111");
  resolve();
});
promise.then(() => {
  console.log("2222");
});
console.log("33333");
// 11111
// 33333
// 2222
```

Promise执行之后是立即执行的， 先输出了 `11111` 然后then方法指定回调函数，将当前的同步操作的任务执行完毕之后(输出 `33333`)。 最后才输出 `222`



一个Promise封装的ajax

````javascript
const ajax = (method, url, async, data) => {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    xhr.onreadychange = () => {
      if (xhr.resayState === 4) {
        if (xhr.status === 200) {
          resolve(xhr.responseText);
        } else {
          reject(new Error(xhr.status));
        }
      }
    };
    xhr.open(method, url, async);
    xhr.send(data || null);
  });
};
ajax("get", "api/get/shu", true, { name: "sgykiuqu" }).then(
  (data) => {
    console.log("请求成功", data);
  },
  (error) => {
    console.log("请求抛出错误：" + error);
  }
);
````

如果调用resolve函数和rejecte函数时都带有参数，那么它们的参数就会被传递给回调函数。reject函数的参数通常是Error实例， 表示抛出错误，。 resolve函数的参数除了正常的值以外，还可能是另外一个Promise实例。

```
const p1 = new Promise((resolve, rejecte) => {});
const p2 = new Promise((resolve, rejecte) => {
	resolve(p1);
})
```

这个例子， p1和p2都是Pro mi se实例， 但是p2的resolve方法将p1作为参数，也就是说以后个异步操作返回了另外一个异步操作。

**注意：**这里的p1的状态会传递给p2，也就是说， p1 的状态决定了p2 的状态。如果p1的状态是pendin·g，那么p2的回调函数就会等待p1状态的改变；如果p1的状态是fulfilled 或者 rejected， 那么回调函数就会立即执行；

```javascript
const p1 = new Promise((resolve, reject) => {
  setTimeout(resolve, 1000);
});
const p2 = new Promise((resolve, reject) => {
  resolve(p1);
});
p2.then(
  (data) => {
    console.log("成功");
  },
  (error) => {
    console.log("失败");
  }
);
```

这个例子p2就会等p1 的状态改变回调函数才会调用（等了1 秒才调用）



**调用resolve或者rejecte并不会终止Promise的执行**

```javascript
const promise = new Promise((resolve, reject) => {
  resolve("111111");
  console.log("22222");
});
promise.then((data) => {
  console.log(data);
});
// 22222
// 111111
```

这个例子： 调用了 resolve("111111"); 之后， 后面的 console.log("22222"); 还是会执行， 并且首先打印出来。这是因为resolve 是在本轮时间循环的末尾执行， 总是晚于本轮的同步任务。



一般来说，调用`resolve`或`reject`以后，Promise 的使命就完成了，后继操作应该放到`then`方法里面，而不应该直接写在`resolve`或`reject`的后面。所以，最好在它们前面加上`return`语句，这样就不会有意外。

```javascript
const promise = new Promise((resolve, reject) => {
  return resolve("111111");  //加上return 
  console.log("22222");
});
promise.then((data) => {
  console.log(data);
});

// 111111
```

## 方法

### Promise.prototype.then()

**作用：**Promise 状态改变时的回调函数。

有两个函数参数：第一个是fulfilled状态的回调函数，一个是rejected状态的回调函数。 的哥参数是可选的。

then方法返回的是一个新的Pro mi se实例（不是原来的Promise实例）。 因此可以采用链式写法：then方法后面再调用另一个then方法。

```javascript

const timeout = () =>
  new Promise((resolve, reject) => {
    setTimeout(resolve, 1000);
  });
const promise = new Promise((resolve, reject) => {
  resolve();
});
promise
  .then((data) => {
    console.log("1111");
  })
  .then(() => {
    console.log("22222");
    return timeout(1000);
  })
  .then(() => {
    console.log("44444");
  });

```

这个例子可以看出，前一个回调函数，有可能返回一个promise对象(也就是有异步操作)， 这时后一个回调函数，就会等待该Promise对象状态发生改变(等待timeout)才会被调用。

###  Promise.prototype.catch()

**作用：**用于指定发生错误的回调函数。

```javascript
const ajax = (method, url, async, data) => {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    xhr.onreadychange = () => {
      if (xhr.resayState === 4) {
        if (xhr.status === 200) {
          resolve(xhr.responseText);
        } else {
          reject(new Error(xhr.status));
        }
      }
    };
    xhr.open(method, url, async);
    xhr.send(data || null);
  });
};
ajax("get", "/api/getname", true).then(() -> {
  console.log("成功")
}).catch((error) => {
  console.log("发生错误", error);
});
// 发生错误 ReferenceError: XMLHttpRequest is not defined
```

这个例子： 如果对象的promise异步操作成功就会调用t hen方法， 如果一步操作抛出错误，状态变味rejected,就会调用catch()方法指定的回调函数处理这个错误，另外，then()方法指定的回调函数运行中抛出错误，也会被catch()方法捕获。

```javascript
const promise = new Promise((resolve, reject) => {
  resolve();
});
promise
  .then(() => {
    console.log(shu);
  })
  .catch((error) => {
    console.log("发生错误", error);
  });
  // 发生错误 ReferenceError: shu is not defined
```

在then方法中没有 shu 这个变量， catch捕获到了。



```javascript
  const promise = new Promise(function(resolve, reject) {
    reject(new Error('test'));
  });
  promise.catch(function(error) {
    console.log(error);
  });
```

可以发现`reject()`方法的作用，等同于抛出错误;



再看一个例子：

```javascript
const promise = new Promise((resolve, reject) => {
  throw new Error("asd");
});
promise
  .then(() => {
    console.log(shu);
  })
  .catch((error) => {
    console.log("发生错误", error);
  });

```



如果Promise的状态已经变成fulfilled。 再抛出错误是没用的。

```javascript
const promise = new Promise((resolve, reject) => {
  resolve();
  throw new Error("asd");
});
promise
  .then(() => {
    console.log("成功");
  })
  .catch((error) => {
    console.log("发生错误", error);
  });
// 成功
```

上面代码中，Promise 在`resolve`语句后面，再抛出错误，不会被捕获，等于没有抛出。因为 Promise 的状态一旦改变，就永久保持该状态，不会再变了。



一般来说， 不要再then方法里面定义rejecte状态的回调函数（即then的第二个参数）。总是使用catch方法。

```javascript
// bad
promise
  .then(function(data) {
    // success
  }, function(err) {
    // error
  });

// good
promise
  .then(function(data) { //cb
    // success
  })
  .catch(function(err) {
    // error
  });
```

### Promise.prototype.finally()

**作用：**不管promise的最后状态如何，都会执行的操作。

该方法不接受任何参数，意思就是Promise状态到底是什么，是没有办法知道的。这表明finally方法里面的操作时跟状态无关的。

```javascript
const promise = new Promise((resolve, reject) => {
  throw new Error("asd");
});
promise
  .then(() => {
    console.log("成功");
  })
  .catch((error) => {
    console.log("发生错误", error);
  })
  .finally(() => {
    console.log("不管promise状态怎么样， 我都会执行");
  });
```

### Promise.all()

**作用:** 将多个Promise实例， 包装成一个新的Promise实例

```javascript
const p = Promise.all([p1,p2,p3])
```

Promise.all()方法接受一个数组作为参数， 也可以不是数组， 但是必须是具有iterator 接口， 且返回的每个成员都是pro mise对象。

如果传入的参数不是 Promise 实例，就会先调用下面讲到的`Promise.resolve()`方法，将参数转为 Promise 实例，再进一步处理。

p的状态是有p1,p2,p3决定，分成两种情况。

1. 只有`p1`、`p2`、`p3`状态都变成`fulfilled`，p的状态才会变成`fulfilled`。此p1,p2,p3的返回值组成一个数组，传递给p的回调函数。
2. 只要`p1`、`p2`、`p3`之中有一个被`rejected`，`p`的状态就变成`rejected`，此时第一个被`reject`的实例的返回值，会传递给`p`的回调函数

```javascript
  const timeout1 = new Promise((resolve, reject) => {
    setTimeout(resolve, 1000);
  });

  const timeout2 = new Promise((resolve, reject) => {
    setTimeout(resolve, 2000);
  });
  Promise.all([timeout1, timeout2])
    .then(([data1, data2]) => {
      console.log("都执行完成了");
    })
    .catch(() => {
      console.log("没有catch");
    });
// 都执行完成了
```

所有的Promise实例的状态都是fulfilled。 所有会调成功的回调， 打印："都执行完成了"

```javascript
  const timeout1 = new Promise((resolve, reject) => {
    setTimeout(resolve, 1000);
  });

  const timeout2 = new Promise((resolve, reject) => {
    setTimeout(reject, 2000); // 异步操作设置rejecte， 失败
  });
  Promise.all([timeout1, timeout2])
    .then(([data1, data2]) => {
      console.log("都执行完成了");
    })
    .catch(() => {
      console.log("至少有一个状态是rejected");
    });
}
// 至少有一个状态是rejected
```

timeout2d的状态是rejected， 所以all的最终状态是rejected。



如果作为参数的promise实例， 自己定义了catch方法， 那么它一旦rejected。并且不会触发Pro mi se.all()的catch方法。

```javascript
{
  const timeout1 = new Promise((resolve, reject) => {
    setTimeout(resolve, 1000);
  });

  const timeout2 = new Promise((resolve, reject) => {
    setTimeout(reject, 2000); // 异步操作设置rejecte， 失败
  }).catch(() => {
    console.log("timeou2 的catch");
  });
  Promise.all([timeout1, timeout2])
    .then(([data1, data2]) => {
      console.log("都执行完成了");
    })
    .catch(() => {
      console.log("不会到打这个catch了");
    });
}
//  timeou2 的catch
//  都执行完成了
```

**注意：**这个例子的timeout1会resolve, timeout2首先会rejected, 但是timeout2由自己的catch方法，该方法会返回一个Promise实例。timeout2指向这个实例，该实例执行catch后，也会变成resolved。 导致两个实例都resolved.因此会调用then方法。而不会调用catch方法



如果timeout2没有自己的catch方法， 就会调用Promise.all()的catch方法。

```javascript
  const timeout1 = new Promise((resolve, reject) => {
    setTimeout(resolve, 1000);
  });

  const timeout2 = new Promise((resolve, reject) => {
    setTimeout(reject, 2000); // 异步操作设置rejecte， 失败
  });
  Promise.all([timeout1, timeout2])
    .then(([data1, data2]) => {
      console.log("都执行完成了");
    })
    .catch(() => {
      console.log("参数实例没有自己的catch,所以有错误都会到这里");
    });
// 参数实例没有自己的catch,所以有错误都会到这里
```



### Promise.race()

**作用：**作用也是将多个promise实例，包装成一个新的Promise实例。但是只要其中的一个实例状态发生改变，就会触发回调。

```javascript
const time = Promise.race([timeout1, timeout2])
```

只要timeout1, timeout2其中有一个实例状态发生改变。time的状态就会发生改变。那个先改变的实例的返回值，就传递给time的回调函数。



`Promise.race()`方法的参数与`Promise.all()`方法一样，如果不是 Promise 实例，就会先调用下面讲到的`Promise.resolve()`方法，将参数转为 Promise 实例，再进一步处理。

```javascript

{
  const timeout1 = new Promise((resolve, reject) => {
    setTimeout(() => {
      console.log("timeout 1");
      resolve();
    }, 1000);
  });

  const timeout2 = new Promise((resolve, reject) => {
    setTimeout(() => {
      console.log("timeout2");
      reject();
    }, 500); // 异步操作设置rejecte， 失败
  });
  Promise.race([timeout1, timeout2])
    .then(() => {
      console.log("第一个time状态发生改变了");
    })
    .catch(() => {
      console.log("catch");
    });
}

// timeout2
//  catch
//  timeout 1
```

这个例子中，timeout2 在500ms 之后状态变成rejected了。比timeout1块， 所有就会立即调用time的then方法。然后timeout1 才会调用。



### Promise.allsettled()

**作用：**作用也是将多个promise实例，包装成一个新的Promise实例.。但是只要这些参数的实例都返回结果，不管是fulfilled还是rejected,包装实例才会结束。

```javascript
  const timeout1 = new Promise((resolve, reject) => {
    setTimeout(() => {
      console.log("timeout 1");
      resolve();
    }, 1000);
  });

  const timeout2 = new Promise((resolve, reject) => {
    setTimeout(() => {
      console.log("timeout2");
      reject();
    }, 500); // 异步操作设置rejecte， 失败
  });
  Promise.allSettled([timeout1, timeout2]).then(() => {
    console.log("timeout1 和 timeou2 都结束了");
  });
// timeout2
// timeout 1
// timeout1 和 timeou2 都结束了
// [
//     {status: "fulfilled", value: undefined}
//     {status: "rejected", reason: undefined}
//  ]
```

Timeout1 和timeout2 都返回了结果， 才会调用then方法.then方法接收到的参数是数组`results`。该数组的每个成员都是一个对象，对应传入`Promise.allSettled()`的两个 Promise 实例。每个对象都有`status`属性，该属性的值只可能是字符串`fulfilled`或字符串`rejected`。`fulfilled`时，对象有`value`属性，`rejected`时有`reason`属性，对应两种状态的返回值。



这个点方法的好处在哪里呢？ 好处就在有时候我们不关系一步操作的结果，只关心异步操作有没有结束。这个方法就很有用。



### Promise.any()

**作用：**作用也是将多个promise实例，包装成一个新的Promise实例。 但是是是要参数实例有任何一个状态变成fulfilled。 包装实例对象就会变成fulfilled状态。如果所有的Promise状态都变成rejected, 包装实例对象才会变成rejecte的状态。

```javascript
  const timeout1 = new Promise((resolve, reject) => {
    setTimeout(() => {
      console.log("timeout 1");
      resolve();
    }, 1000);
  });

  const timeout2 = new Promise((resolve, reject) => {
    setTimeout(() => {
      console.log("timeout2");
      reject();
    }, 500); // 异步操作设置rejecte， 失败
  });
  Promise.any([timeout1, timeout2]).then(() => {
    console.log("timeout1 状态变成了 fulfilled");
  });
```

**注意**Promise.any()目前还在草案阶段。

### Promise.resolve()

**作用**: 将现有的对象转成Promise对象。

```javascript
Promise.resolve('name');
// 等价于
new Promise((resolve) => resolve(name))
```

Promise.resolve()方法的参数有四种情况：

1. 参数是一个promise实例， 那么promise.resolve将不做任何的修改， 直接原封不动的返回这个实例

2. 参数是一个thenable对象。Promise.resolve()会将这个对象转成Promise对象， 然后立即执行thenable对象的then方法。

   thenable对象就是指具有then方法的对象。如：

   ````javascript
   
   const thenable = {
     then: (resolve, rejecte) => {
       resolve("ashds");
     },
   };
   ````

   

   ```javascript
     const thenable = {
       then: (resolve, rejecte) => {
         rejecte("ashds");
       },
     };
     Promise.resolve(thenable)
       .then((data) => {
         console.log("thenable 的then方法返回的状态是fulfilled", data);
       })
       .catch((error) => {
         console.log("thenable的then方法返回的状态是rejected", error);
       });
   // thenable的then方法返回的状态是rejected ashds
   ```

   这个例子 Promise.resolve将thenable转成Promise对象之后，立即调用thenble对象的then方法。thenable对象的then方法返回的状态是 rejected; 所以执行到catch里面

3. 参数不是thenable对象， 或者根本不是对象。Promise.resolve()会返回一个新的Promise实例，状态直接为resolved。

   ```javascript
   Promise.resolve("shuliqi").then((name) => console.log(name));
   // shuliqi
   ```

   这个例子的shuliqi不属于异步操作(字符串对象没有then方法)。返回的Promise从一开始就是resolved/所以会调函数就会立即执行。Promise.resolve方法的参数，会同时传给回调函数。

4. 不带任何参数。直接返回一个`resolved`状态的 Promise 对象。

   ```javascript
   Promise.resolve()
   .then(() => {
   	console.log("hahahah")
   });
   // hahahah
   ```

   Promise.resolved是没有参数的， 直接返回一个状态为resolved的Promise对象。 所以会调函数立即就调用了。



### Promise.reject()

**作用:**返回一个新的 Promise 实例，该实例的状态为`rejected`。

```javascript
Promise.reject("shuliqi");
// 等同于
new Promise((resolve, rejecte) => reject("shuliqi"))
```



**注意：**`Promise.reject()`方法的参数，会原封不动地作为`reject`的理由，变成后续方法的参数。这一点与`Promise.resolve`方法不一致。

```javascript
 const thenable = {
    then: (resolve, rejecte) => {
      rejecte("ashds");
    },
  };
  Promise.reject(thenable).catch((data) => {
    console.log(data === thenable);
  });
  // true
```

`Promise.reject`方法的参数是一个`thenable`对象，执行以后，后面`catch`方法的参数不是`reject`抛出的“出错了”这个字符串，而是`thenable`对象。



[学习的链接](https://es6.ruanyifeng.com/#docs/promise)







