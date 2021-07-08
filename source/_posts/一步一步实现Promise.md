---
title: 一步一步实现Promise
date: 2018-03-29 14:41:11
tags: ES6
categories: ES6
---

上篇文章学习了 关于 `Promise` [ES6学习笔记-Promise](https://shuliqi.github.io/2018/03/20/ES6%E5%AD%A6%E4%B9%A0-Promise/) ； 那么这篇文章就来看看如何自己实现 一个`Promise `和它的原型方法和类方法的

本文手写`Promise`的主要分为下面这三部分：

- 实现`Promise`大体框架
- 添加异步处理和多次调用`then`方法
- 添加`Promise`的链式调用
- 实现`Promise`的原型方法
- 实现`Promise`的类方法

# 实现大体框架



我们先来看一个例子：

```javascript
const p =  new Promise((resolve, reject) => {
  console.log("1");
  resolve("2")
})
p.then((value) => {
  console.log("成功：", value)
}, (reason) => {
  console.log("失败：", reason)
});
console.log("3")
```

这个例子的输出结果如下：

```javascript
1
3
成功： 2
```

那么根据这个例子我们可以得到如下的规则：

- `Promise` 是构造函数，`new`出来的实例有`then`方法。
- `new Promise`的时候短笛一个参数，这个参数是一个函数（我们一般称为执行器函数--`executor`）并且执行器会立即调用。就是上面的例子中 1 会被最先打印出来。
- `executor`函数有两个参数：`resolve `和 `reject`；这两个参数也是函数。
- `new Promise` 后的实例是具有状态，默认的状态是 `pending`。当置值器`resolve`调用后，实例的状态变为`fulfilled`。当置值器`reject`调用后，实例的状态变为`rejected`。
- `Promise`状态的实例一经改变，就不能再次修改。
- 每一个`Promise`实例都有`then`方法，`then`方法中有两个参数，这两个参数都是函数。第一个函数叫做**成功回调**，第二个参数叫做**失败回调**。当置值器`resolve`调用后，`then`中的第一个参数函数会被执行；当置值器`reject`调用后，`then`中的第二个参数函数会被执行。

那我们根据这个规则来写一个大体的`Promise`:

## 构造函数的参数（executor）会立即执行

```javascript
function MyPromise(executor) {
   // executor 是会被立即执行的
  executor(resolve, reject)
}
```

## new 出来的实例具有then方法

`then`方法有两个参数： 成功回调函数，失败回调函数

```javascript
// new 出来的实例在原型上有 then 方法
MyPromise.prototype.then =function (onFulfilled, onRejected) {
  
}
```

## new 出来的实例的状态

new 出来的实例具有默认的状态，默认状态为`pending`; 需要使用`resolve`置值器或者`reject`置值器来修改状态。

```javascript
function MyPromise(executor) {
   const self = this;
  // 实例的默认状态： pending
  self.status = "pending";

  // resolve 置值器： 用于修改状态
  function resolve(value) {
    self.status = "fulfilled";
  }

  // reject 置值器： 用于修改状态
  function  reject(reason) {
    self.status = "rejected";
  }

  // executor 是会被立即执行的
  executor(resolve, reject)
}
```

> 这里为什么会把 `this` 赋值 给 `self `呢？值得思考！！

答案：这是因为普调函数的 `this` 的指向是在调用的时候确定的，如果还不了解 `this`指向的， 可以先看看这篇文章 [关于this的指向问题](https://shuliqi.github.io/2018/07/02/%E5%85%B3%E4%BA%8Ethis%E7%9A%84%E6%8C%87%E5%90%91%E9%97%AE%E9%A2%98/)。因为我们的 `resolve`和 `reject` 函数里面的`this`指向的是`Window`。 而不是我们的实例。 所以我们需要使用闭包的用法，让函数能够记住并且能够访问当前的词法作用域，即使函数不是在当前的词法作用域中执行的。即我们上线的写法，将实例`this` 赋值给`self`变量，在`resolve` 和` reject `置值器函数中使用 `self`，那么当这两个函数不在当前的词法作用域中调用的时候， 也有对当前词法作用域的使用权限。如果还对闭包不了解的可以先看这边文章 [JavaScript中的闭包](https://shuliqi.github.io/2018/10/23/JavaScript%E4%B8%AD%E7%9A%84%E9%97%AD%E5%8C%85/)

## then中回调函数的触发

当置值器`resolve`调用之后，`then`中的第一个函数参数会被执行（成功回调），当置值器`reject`调用之后，`then`中的第二个函数参数会被执行（失败回调）。

```javascript
// new 出来的实例在原型上有 then 方法
MyPromise.prototype.then =function (onFulfilled, onRejected) {

  // 当置值器resolve调用之后，then中的第一个函数参数会被执行（成功回调）；
  // 即当前的 promise 状态为 fulfilled
  if (this.status === "fulfilled") {
    onFulfilled();
  }

  // 当置值器reject调用之后，then中的第一个函数参数会被执行（失败回调）；
  // 即当前的 promise 状态为 rejected
  if (this.status === "rejected") {
    onRejected()
  }
}

```

## Promise的状态一经更改就不能再次改变

Promise的状态一经更改就不能再次改变，所以只有在pending状态的时候才能修状态

```javascript
  // resolve 置值器： 用于修改状态
  function resolve(value) {
    // Promise的状态一经修改就不能再改变，所以只有在 pending 状态才能修改
    if (self.status === "pending") {
      self.status = "fulfilled";
    }
  }

  // reject 置值器： 用于修改状态
  function  reject(reason) {
    // Promise的状态一经修改就不能再改变，所以只有在 pending 状态才能修改
    if (self.status === "pending") {
      self.status = "rejected";
    }
  }
```

## 置值器被调用之后， 传入的值会传给then中的回调函数

当置值器`resolve`调用后，传入的值是成功的值， 我们称为 `value`。这个值会传给`then`中的第一个参数，当置值器`reject`调用后，实例的值是失败原因，称为`reason`。这个值会传给`then` 中的第二个参数函数

```javascript
function MyPromise(executor) {
  const self = this;
  // 实例的默认状态： pending
  self.status = "pending";

  // resolve 置值器： 用于修改状态
  function resolve(value) {
    // Promise的状态一经修改就不能再改变，所以只有在 pending 状态才能修改
    if (self.status === "pending") {
      self.status = "fulfilled";
      // Promise的值
      self.value = value;
    }
  }

  // reject 置值器： 用于修改状态
  function  reject(reason) {
    // Promise的状态一经修改就不能再改变，所以只有在 pending 状态才能修改
    if (self.status === "pending") {
      self.status = "rejected";
      // Promise的值
      self.value = reason
    }
  }

  // executor 是会被立即执行的
  executor(resolve, reject)
}
// new 出来的实例在原型上有 then 方法
MyPromise.prototype.then =function (onFulfilled, onRejected) {

  // 当置值器resolve调用之后，then中的第一个函数参数会被执行（成功回调）；
  // 即当前的 promise 状态为 fulfilled
  if (this.status === "fulfilled") {
    // 值会传递给成功回调
    onFulfilled(this.value);
  }

  // 当置值器reject调用之后，then中的第一个函数参数会被执行（失败回调）；
  // 即当前的 promise 状态为 rejected
  if (this.status === "rejected") {
    // 值会传递给失败回调
    onRejected(this.value)
  }
}

```

那么到现在位置， 我们大体的结构写完了。

# 添加异步处理和多次调用then方法

## 例子参考

我们还是先来看一个例子：

```javascript
const p = new Promise((resolve, reject) => {
  console.log("1");
  setTimeout(() => {
    resolve("2");
  }, 2000)
})

p.then((value) => {
  console.log("成功1", value)
},  (reason) => {
  console.log("失败1", reason)
})

p.then((value) => {
  console.log("成功2", value)
},  (reason) => {
  console.log("失败2", reason)
})

console.log("3");
```

打印的结果为:

```javascript
1
3
成功1 2
成功2 2
```

## 例子得出的规则

那针对异步这种情况，我们来看上面的例子可得到解决办法：

1. 当创建`Promise`的时候， `executor` 执行器方法中的`resolve("2")`被`setTimeout`放到了异步队列中了

2. 所以创建出来的实例当前的状态还是默认的状态，并没有发生改变

3. 那么当代码执行到`p.then`的时候， 我们并不知道要去执行`then`中的第一个参数（成功回调）还是第二个参数（失败会回调）。

4. 当不知道哪个回调函数会被执行的情况下， 就需要吧把这两个回调保存起来，等到等确定调用哪个回调的时候，再拿出来调用。

5. 因为同一个` promise` 实例的`then`是可以多次调用的（这里不是指链式调用）。所以我们应该使用数组把成功回调和失败回调保存起来。因为`then` 是可以多次被调用的。

   

## 完善异步和多次调用

那么我们现在的`MyPromise`应该添加代码如下：

```javascript
function MyPromise(executor) {
  const self = this;

  // 保存成功回调：then方法中的第一个参数
  self.onResolvedCallbacks = [];

  // 保存失败回调：：then方法中的第二个参数
  self.onRejectedCallbacks = [];

  // ... 此处省略其他逻辑代码
 
}
// new 出来的实例在原型上有 then 方法
MyPromise.prototype.then =function (onFulfilled, onRejected) {

  // ... 此处省略其他逻辑代码

  // 当还不知道实例状态的情况下，把成功回调和失败回调都保存起来
  // 等到确定调用哪个回调之后，再拿出来调用
  if (this.status === "pending") {
    this.onResolvedCallbacks.push(onFulfilled);
    this.onRejectedCallbacks.push(onRejected)
  }
}
```

现在我们的`MyPromise`把我们的成功回调或者失败回调都保存起来了。 那我们该什么时候去调用回调呢？那当然是在状态发生变化的时候去调用。实例的状态只会在 `resolve` 和 `reject`置值器函数中改变， 那么我们就可以在这时候把回调拿出来调用。

```javascript
// resolve 置值器： 用于修改状态
  function resolve(value) {
    if (self.status === "pending") {
      
      // ... 此处省略其他逻辑代码
      
      // 把存起来的成功回调调用
      self.onResolvedCallbacks.forEach((fn) => {
        fn(self.value);
      })
    }
  }

  // reject 置值器： 用于修改状态
  function  reject(reason) {
    if (self.status === "pending") {
      
      // ... 此处省略其他逻辑代码

      // 把保存起来的失败回调拿出来调用
      self.onRejectedCallbacks.forEach((fn) => {
        fn(self.value);
      })
    }
  }
```

最后附上完整代码：

```javascript

function MyPromise(executor) {
  const self = this;

  // 保存成功回调：then方法中的第一个参数
  self.onResolvedCallbacks = [];

  // 保存失败回调：：then方法中的第二个参数
  self.onRejectedCallbacks = [];

  // 实例的默认状态： pending
  self.status = "pending";

  // resolve 置值器： 用于修改状态
  function resolve(value) {
    console.log("置值")
    // Promise的状态一经修改就不能再改变，所以只有在 pending 状态才能修改
    if (self.status === "pending") {
      self.status = "fulfilled";
      // Promise的值
      self.value = value;

      // 把存起来的成功回调调用
      self.onResolvedCallbacks.forEach((fn) => {
        fn();
      })
    }
  }

  // reject 置值器： 用于修改状态
  function  reject(reason) {
    // Promise的状态一经修改就不能再改变，所以只有在 pending 状态才能修改
    if (self.status === "pending") {
      self.status = "rejected";
      // Promise的值
      self.value = reason;

      // 把保存起来的失败回调拿出来调用
      self.onRejectedCallbacks.forEach((fn) => {
        fn();
      })
    }
  }

  // executor 是会被立即执行的
  executor(resolve, reject)
}

// new 出来的实例在原型上有 then 方法
MyPromise.prototype.then =function (onFulfilled, onRejected) {

  // 当置值器resolve调用之后，then中的第一个函数参数会被执行（成功回调）；
  // 即当前的 promise 状态为 fulfilled
  if (this.status === "fulfilled") {
    // 值会传递给成功回调
    onFulfilled(this.value);
  }

  // 当置值器reject调用之后，then中的第一个函数参数会被执行（失败回调）；
  // 即当前的 promise 状态为 rejected
  if (this.status === "rejected") {
    // 值会传递给失败回调
    onRejected(this.value)
  }

  // 当还不知道实例状态的情况下，把成功回调和失败回调都保存起来
  // 等到确定调用哪个回调之后，再拿出来调用
  if (this.status === "pending") {
    this.onResolvedCallbacks.push(() => {
      onFulfilled(this.value)
    });
    this.onRejectedCallbacks.push(() => {
      onRejected(this.value)
    })
  }
}

```

 现在我们来测试一下；还是上面的例子，只不过构造函数是我们自己实现的`MyPromise`:

```javascript
const p = new MyPromise((resolve, reject) => {
  console.log("1");
  setTimeout(() => {
    resolve("2");
  }, 2000)
})

p.then((value) => {
  console.log("成功1", value)
},  (reason) => {
  console.log("失败1", reason)
})

p.then((value) => {
  console.log("成功2", value)
},  (reason) => {
  console.log("失败2", reason)
})

console.log("3");
```

结果如下：

```javascript
1
3
成功1 2
成功2 2
```

结果跟原生的一样， 完美！！！

## then方法是异步执行的处理

我们知道 `then` 方法返回的是一个`Promise   `。 而`Promise   `是一个微任务， 微任务是在同步任务执行完之后再执行的。也就是说 `then`方法是异步执行的.

ES6的原生Promise对象已经实现了这一点，但是我们自己的代码是同步执行，我们试试：

```javascript

const p = new MyPromise((resolve, reject) => {
  console.log("1");
  resolve("2");
})

p.then((value) => {
  console.log("成功1", value)
},  (reason) => {
  console.log("失败1", reason)
})

p.then((value) => {
  console.log("成功2", value)
},  (reason) => {
  console.log("失败2", reason)
})

console.log("3");
```

我们期望的结果：

```javascript
1
3
成功1 2
成功2 2

```

但是实际的结果为：

```javascript
1
成功1 2
成功2 2
3

```



那关于这样情况如何解决呢，我们可以使用`setTimeOut`将同步代码变成异步代码。

```javascript
function MyPromise(executor) {
	// .. 已省略
	
  function resolve(value) {
      self.onResolvedCallbacks.forEach((fn) => {
        setTimeout(() => {
          fn();
        }, 0)
      })
    }
  }

  function  reject(reason) {
      self.onRejectedCallbacks.forEach((fn) => {
        setTimeout(() => {
          // then 方法是异步指执行的，使用setTimeOut 将同步代码变成异步代码。
          fn();
        }, 0)
      })
    }
  }

	// .. 已省略
}


// new 出来的实例在原型上有 then 方法
MyPromise.prototype.then =function (onFulfilled, onRejected) {

  if (this.status === "fulfilled") {
    // then 方法是异步指执行的，使用setTimeOut 将同步代码变成异步代码。
    setTimeout(()=> {
      // 值会传递给成功回调
      onFulfilled(this.value);
    },0)
  }


  if (this.status === "rejected") {
    // then 方法是异步指执行的，使用setTimeOut 将同步代码变成异步代码。
    setTimeout(()=> {
      // 值会传递给失败回调
      onRejected(this.value)
    },0)
  }
  
	// ... 已省略
}

```

这样即可， 再试试我们的例子返回的结果是我们期望的结果。

# 支持链式调用

## then 链的知识点

我们知道得到一个`Promise`对象可以有三种方式：

- 使用`new Promise`来创建
- 使用`Promise`的类方法如：`Promise.all()`，`Promise.race()`，`Promise.allsettled()`，`Promise.any()`，`Promise.resolve()`，`Promise.reject()`等
- 使用`Promise`原型方法 Promise.prototype.xxx(): 如：promise.then()`，`promise.catch()`，`promise.fanally()；

由此我们知道：

1. **每个`then`方法返回的是一个新的`Promise`对象（第三种创建方式）**

   为什么是一个新的`Promise`呢？因为`Promise`实例的状态一旦修改，则不能再修改，所以要返回全新的`Promise`

既然返回的是一个新的`Promise`。 那这个新的`Promise`的置值逻辑是什么样子的呢？那我们先来看`then` 链上的置值逻辑:

```javascript
try {
    // 判断上一个promise的置值状态
    if (isRejected(p)) {
       // rejected 是有效的 ? 执行rejected函数，result为reason : result 
       // 其中result是p代理的数据
        x2 = isValidHandler(rejected) ? rejected(result) :throw result;  
    } else {
    	  // resolved函数是有效的 ？ 执行resolved函数 ： result;
        x2 = isValidHandler(resolve) : resolved(result) : result; 
    } 
    resolve(x2); // promise2
} catch (error) {
    reject(error)
}
```

由此我们知道比较重要的一点：

2. **如果 `rejected` 和 `resolve`是有效的。无论他们返回何值，都将作为`resolve`置值器的值直接绑定给这个新的`Promise`。如果产生了` reject `值。那么就作为`rejecte`置值器的值直接绑定给这个新的`Promise`。**

then链中产生`Promise`值的方法有两种：

- 通过抛出异常来是的`Javascript`引擎获得异常对象。
- 通过显示的调用 `Promise.reject()`。

现在我们知道新的`Promise`的值是重要的一点是由成功回调或者失败回调的返回值来确定的。那成功回调或者失败回调的返回值的类型是什么？如果返回值是一个普调值，该是什么样？ 如果返回值不是一个普调值，那又该怎么样？

我们来试验一下：

### 返回普调值

```javascript

const p1 = new Promise((resolve, reject) => {
  resolve("1")
})
const p2 = p1.then((value) => {
  console.log("成功回调：", value);
  return "2";   // 返回一个普调值
}, (reason) => {
  console.log("失败回调：", reason)
})

p2.then((value) => {
  console.log("p2的值：", value);
})

```

结果：

```
成功回调： 1
p2的值： 2
```

```javascript
const p1 = new Promise((resolve, reject) => {
  reject("1")
})
const p2 = p1.then((value) => {
  console.log("成功回调：", value);
  return "2";   // 返回一个普调值
}, (reason) => {
  console.log("失败回调：", reason);
  return "3";
})

p2.then((value) => {
  console.log("p2的值：", value);
})

```

结果：

```javascript
失败回调： 1
p2的值： 3
```

> 如果返回了一个普调值，那么这个值被`resove` 置值器直接绑定给下一个`Promise`（即 `p2`）。意思就是会调用下一个`then`的第一个参数

这里注意：如果回调里面没有显示的返回值， 那么函数默认是会返回` undefined`。如：

```javascript
const p1 = new Promise((resolve, reject) => {
  reject("1")
})
const p2 = p1.then((value) => {
  console.log("成功回调：", value);
}, (reason) => {
  console.log("失败回调：", reason);
})

p2.then((value) => {
  console.log("p2的值：", value);
})
```

结果：

```javascript
失败回调： 1
p2的值： undefined
```

### 返回非普调值---`Promise`

例子1:

```javascript

const p1 = new Promise((resolve, reject) => {
  resolve("1")
})
const p2 = p1.then((value) => {
  console.log("成功回调：", value);
  // 返回一个 Promise, 这个promise产生 resolve 值
  return new Promise((resolve, reject) => {
    resolve("2");
  })
}, (reason) => {
  console.log("失败回调：", reason);
})

p2.then((value) => {
  console.log("p2的值：", value);
})
```

结果：

````javascript
成功回调： 1
p2的值： 2
````

例子2:

```javascript

const p1 = new Promise((resolve, reject) => {
  reject("1")
})
const p2 = p1.then((value) => {
  console.log("成功回调：", value);
}, (reason) => {
  console.log("失败回调：", reason);
  // 返回一个 Promise, 这个promise产生 reject 值
  return new Promise((resolve, reject) => {
    reject("2");
  })
})

p2.then((value) => {
  console.log("p2的值：", value);
}, (reason) => {
  console.log("p2的reason:", reason)
})

```

结果:

```javascript
失败回调： 1
p2的reason: 2
```



两个例子可得出：

>  如果回调函数返回一个`Promise`. 那么会根据这个`Promise`返回成功还是失败来决定使用`resolve`置值器还是 `reject `置值器（即调用下一个`Promise`的第一个参数还是第二个参数。如果是成功的）。
>
> `Promise`返回成功，则调用`resolve` 置值器，调用下一个`then `的第一个参数；如果返回失败，则调用 `reject `置值器，调用下一个`then`的第二个参数

### 非普调值---报错异常

```javascript

const p1 = new Promise((resolve, reject) => {
  resolve("1")
})
const p2 = p1.then((value) => {
  console.log("成功回调：", value);
  throw new Error("报错了")
}, (reason) => {
  console.log("失败回调：", reason);
})

p2.then((value) => {
  console.log("p2的值：", value);
}, (reason) => {
  console.log("p2的reason", reason)
})

```

结果：

```javascript
成功回调： 1
p2的reason Error: 报错了
```

由此可知：

> 当报错异常（即上面所说的产生了 reject 值）。会采用reject 置值器，直接调用下一个`then` 的第二个参数

以上的知识点都来自于：[《JavaScript语言精髓与编程实战》读书笔记-Promised的核心机制](https://shuliqi.github.io/2020/07/15/%E3%80%8AJavaScript%E8%AF%AD%E8%A8%80%E7%B2%BE%E9%AB%93%E4%B8%8E%E7%BC%96%E7%A8%8B%E5%AE%9E%E6%88%98%E3%80%8B%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0-Promised%E7%9A%84%E6%A0%B8%E5%BF%83%E6%9C%BA%E5%88%B6/#%E5%88%9B%E5%BB%BAPromise)

## 完善 链式调用

那么基于上面的知识点我们来完善我们的` MyPromise`:

### then 返回的是全新的 Promise

我们可以使用 `MyPromise`的`executor`函数包裹了一下。而`executor`函数是立即执行的（在new 实例的时候就会运行）。所以加上这个包裹是不会对原有的逻辑存在影响。又实现了 只要有then方法执行就会返回`Promise`实例。并且是全新的`MyPromise`

````javascript
// new 出来的实例在原型上有 then 方法
MyPromise.prototype.then = function (onFulfilled, onRejected) {

  // then 返回的是全新的 Promise
  return new MyPromise((resolve, reject) => {
     
      if (this.status === "fulfilled") {
				// ...  已省略相关逻辑
      }

      if (this.status === "rejected") {
      	// ...  已省略相关逻辑
      }

      if (this.status === "pending") {
      	// ...  已省略相关逻辑
      }
  })
}

````



### then 链上返回值的处理

由上面的知识点我们可知上一个`Promise`的回调函数的返回值会有三种: 

- 返回普调值： 会被当前的 `Promise`的`resolve`置值器直接绑定给下一个 `Promise`。则调用下一个 `then `的第一个参数
- 返回`Promise`：会根据当前的`Promise`的返回值来决定来使用哪个置置器。当返回 成功值，使用`resolve`置值器，调用下一个`then`的第一个参数。返回失败值。使用`reject` 置值器；调用下一个`then` 的第二个参数。
- 报错异常：如果当发现异常报错了，使用`reject`置值器， 调用下一个then的第二个参数。

#### 对普调值和报错异常的处理

```javascript
// new 出来的实例在原型上有 then 方法
MyPromise.prototype.then =function (onFulfilled, onRejected) {

  // then 返回的是全新的 Promise
  return new MyPromise((resolve, reject) => {
      // 当置值器resolve调用之后，then中的第一个函数参数会被执行（成功回调）；
      // 即当前的 promise 状态为 fulfilled
      if (this.status === "fulfilled") {
        // then 方法是异步指执行的，使用setTimeOut 将同步代码变成异步代码。
        setTimeout(() => {
          try {
            // 回调函数的返回值会被当前的 MyPromise 的 resolve 置值器绑定给下一个 MyPromise
            const result = onFulfilled(this.value);
            resolve(result);
          } catch (error) {
            // 如果报错会被当前的 MyPromise 的 reject 置值器绑定给下一个 MyPromise
            reject (error)
          }
        }, 0)
      }

      // 当置值器reject调用之后，then中的第一个函数参数会被执行（失败回调）；
      // 即当前的 promise 状态为 rejected
      if (this.status === "rejected") {
        // then 方法是异步指执行的，使用setTimeOut 将同步代码变成异步代码。
        setTimeout(() => {
           try {
             // 回调函数的返回值会被当前的 MyPromise 的 resolve 置值器绑定给下一个 MyPromise
             const result = onRejected(this.value)
             resolve(result);
          } catch (error) {
            // 如果报错会被当前的 MyPromise 的 reject 置值器绑定给下一个 MyPromise
            reject(error)
          }
        }, 0)


      }

      // 当还不知道实例状态的情况下，把成功回调和失败回调都保存起来
      // 等到确定调用哪个回调之后，再拿出来调用
      if (this.status === "pending") {
        this.onResolvedCallbacks.push(() => {
          try {
            // 回调函数的返回值会被当前的 MyPromise 的 resolve 置值器绑定给下一个 MyPromise
            const result = onFulfilled(this.value);
            resolve(result);
          } catch (error) {
            // 回调函数的返回值会被当前的 MyPromise 的 reject 置值器绑定给下一个 MyPromise
            reject(error);
          }
        });
        this.onRejectedCallbacks.push(() => {
          try {
            // 回调函数的返回值会被当前的 MyPromise 的 resolve 置值器绑定给下一个 MyPromise
            const result = onRejected(this.value);
            resolve(result);
          } catch (error) {
            // 回调函数的返回值会被当前的 MyPromise 的 reject 置值器绑定给下一个 MyPromise
            reject(error);
          }
        })
      }
  })
}
```

 #### 对返回值是Promise的处理

因为回调函数的返回值有可能是`Promise`。 那么针对这种情况， 我们把处理返回值单独提出来处理。

```javascript
// new 出来的实例在原型上有 then 方法
MyPromise.prototype.then = function (onFulfilled, onRejected) {

  return new MyPromise((resolve, reject) => {
     
      if (this.status === "fulfilled") {
        setTimeout(() => {
           try {
            const result = onFulfilled(this.value);

            //  因为 result 有可能是 promise，抽离出来处理
            resolvePromiseResult(result, resolve, reject);

          } catch (error) {
            // .. 已省略
          }
        }, 0)
        
      }

      if (this.status === "rejected") {
        setTimeout(() => {
           try {
             const result = onRejected(this.value)

            //  因为 result 有可能是 promise，抽离出来处理
             resolvePromiseResult(result, resolve, reject);

          } catch (error) {
            // .. 已省略
          }         
        }, 0)


      }

      console.log("this值：", this)
      if (this.status === "pending") {
       
        this.onResolvedCallbacks.push(() => {
          try {
            const result = onFulfilled(this.value);

            //  因为 result 有可能是 promise，抽离出来处理
            resolvePromiseResult(result, resolve, reject);

          } catch (error) {
          	// .. 已省略
          }
        });
        this.onRejectedCallbacks.push(() => {
          try {
            const result = onRejected(this.value);

            //  因为 result 有可能是 promise，抽离出来处理
            resolvePromiseResult(result, resolve, reject);

          } catch (error) {
          	// .. 已省略
          }
        })
      }
  })
}

```

有一种情况，` Promise` 可能调用了自己本身， 这种是不允许的, 会导致死循环。所以我们`resolvePromiseResult`函数应该再加上一个参数，就是自身的`Promise`。这样在`resolvePromiseResult`函数中判断就好判断了。

```javascript
// new 出来的实例在原型上有 then 方法
MyPromise.prototype.then =function (onFulfilled, onRejected) {

  const newMyPromise = new MyPromise((resolve, reject) => {
     
      if (this.status === "fulfilled") {
        // ...已省略
        try {
          const result = onFulfilled(this.value);
          
          //  因为 result 有可能是 promise，抽离出来处理
          resolvePromiseResult(newMyPromise, result, resolve, reject);
          
        } catch (error) {
          // .. 已省略
        }
        // ...已省略
        
      }

      if (this.status === "rejected") {
        // ...已省略
        try {
           const result = onRejected(this.value)

          //  因为 result 有可能是 promise，抽离出来处理
           resolvePromiseResult(newMyPromise, result, resolve, reject);

        } catch (error) {
          // .. 已省略
        }
       // ...已省略

      }

      if (this.status === "pending") {
       
        this.onResolvedCallbacks.push(() => {
          try {
            const result = onFulfilled(this.value);

            //  因为 result 有可能是 promise，抽离出来处理
            resolvePromiseResult(newMyPromise, result, resolve, reject);

          } catch (error) {
          	// .. 已省略
          }
        });
        this.onRejectedCallbacks.push(() => {
          try {
            const result = onRejected(this.value);

            //  因为 result 有可能是 promise，抽离出来处理
            resolvePromiseResult(newMyPromise, result, resolve, reject);

          } catch (error) {
          	// .. 已省略
          }
        })
      }
  })
  return newMyPromise;
}


```

 下面我们就来看看 `resolvePromiseResult` 函数的处理：

如果返回的是自己的 Promise 对象的话，那么状态永远`pending `状态. 再也无法称为 `fulfilled `或者 `rejected`。程序是会死掉的， 所有首先处理它：

```javascript

function resolvePromiseResult(promise, backResult, resolve, reject) {
  // 如果结果返回的是自身， 那么直接调用reject 置值器处理。不然程序将会死掉
  if (promise === backResult) {
    reject(new Error("Promise发生了循环引用"))
  }
}
```



接下来就是分各种情况判断： 如果 `backResult`是一个普调值， 那么直接调用 `resolve` 置值器置值。如果是一个 `Promise`, 对象（then 对象）那么就执行它。

```javascript
function resolvePromiseResult(promise, backResult, resolve, reject) {
  // 如果结果返回的是自身， 那么直接调用reject 置值器处理。不然程序将会死掉
  if (promise === backResult) {
    reject(new Error("Promise发生了循环引用"))
  };
  if (backResult !== null && (typeof backResult === "object" || typeof backResult === "function")) {
    // Promise 或者then 对象
  } else {
    // 普调值， 直接使用 resove 置值器置值
    resolve(backResult);
  }
}
```



如果 `backResult` 是一个对象 或者 函数的话，那么尝试把`then` 方法取出来，此时如果报错，那么将是使用`reject`置值器置值。	

```javascript

function resolvePromiseResult(promise, backResult, resolve, reject) {
  // 如果结果返回的是自身， 那么直接调用reject 置值器处理。不然程序将会死掉
  if (promise === backResult) {
    reject(new Error("Promise发生了循环引用"))
  };
  if (backResult !== null && (typeof backResult === "object" || typeof backResult === "function")) {
    // Promise 或者then 对象
    try {
      //  取出 then 方法
      const then = backResult.then;

    } catch (error) {

      // 如果此时报错， 将世界使用 reject 置值器置值
      reject(error);
      
    }
  } else {
    // 普调值， 直接使用 resove 置值器置值
    resolve(backResult);
  }
}
```



如果对象中有`then`，并且是函数类型， 那么就可认为是一个 `Promise`对象。之后直接调用这个 `then` 方法。成功回调的话使用` resolve` 置值器，失败回调的话使用` reject` 置值器

```javascript
function resolvePromiseResult(promise, backResult, resolve, reject) {
  // 如果结果返回的是自身， 那么直接调用reject 置值器处理。不然程序将会死掉
  if (promise === backResult) {
    reject(new Error("Promise发生了循环引用"))
  };
  if (backResult !== null && (typeof backResult === "object" || typeof backResult === "function")) {
    // Promise 或者then 对象

    try {

      //  取出 then 方法
      const then = backResult.then;
      
      if (typeof then === "function") {
        // 如果 then 是一个函数，那么直接执行， 
        then((y) => {

          // 成功的话使用 resolve 置值器
          resolve(y)

        }, (error) => {
          // 失败的话使用 reject 置值器
          reject(error);

        })
      }

    } catch (error) {

      // 如果此时报错， 将世界使用 reject 置值器置值
      reject(error);
    }
  } else {
    // 普调值， 直接使用 resove 置值器置值
    resolve(backResult);
  }
}
```



在执行的时候，我们需要绑定一下` this` 值。 因为内部可能用到了`this` 呀

```javascript
    // 如果 then 是一个函数，那么直接执行 
    then.call(promise, () => {
      // 成功的话使用 resolve 置值器
      resolve(y)
    }, (error) => {
      // 失败的话使用 reject 置值器
      reject(error);
    })
  }
```

到现在链式调用已经基本完成了，但是还有一种极端的情况。如果` Promise` 对象转为成功状态或者失败的状态的时候传入的是一个`Promise` 对象。那么此时应该继续执行 ，知道最后的 `Promise`  执行完成。那么久需要递归了。

```javascript
// 如果 then 是一个函数，那么直接执行 
then.call(promise, () => {
  // 递归调用，传入y若是Promise对象，继续循环
  resolvePromiseResult(promise, y, resolve, reject)
}, (error) => {
  // 失败的话使用 reject 置值器
  reject(error);
})
```

那么现在的 链式调用就全部完成了。

最后我们贴上全部的代码：

```javascript
function MyPromise(executor) {
  const self = this;

  // 保存成功回调：then方法中的第一个参数
  self.onResolvedCallbacks = [];

  // 保存失败回调：：then方法中的第二个参数
  self.onRejectedCallbacks = [];

  // 实例的默认状态： pending
  self.status = "pending";

  // resolve 置值器： 用于修改状态
  function resolve(value) {
    // Promise的状态一经修改就不能再改变，所以只有在 pending 状态才能修改
    if (self.status === "pending") {
      self.status = "fulfilled";
      // Promise的值
      self.value = value;

      // 把存起来的成功回调调用
      self.onResolvedCallbacks.forEach((fn) => {
        // then 方法是异步指执行的，使用setTimeOut 将同步代码变成异步代码。
        setTimeout(() => {
          fn();
        }, 0)
      })
    }
  }

  // reject 置值器： 用于修改状态
  function  reject(reason) {
    // Promise的状态一经修改就不能再改变，所以只有在 pending 状态才能修改
    if (self.status === "pending") {
      self.status = "rejected";
      // Promise的值
      self.value = reason;

      // 把保存起来的失败回调拿出来调用
      self.onRejectedCallbacks.forEach((fn) => {
        // then 方法是异步指执行的，使用setTimeOut 将同步代码变成异步代码。
        setTimeout(() => {
          fn();
        }, 0)
      })
    }
  }

  // executor 是会被立即执行的
  executor(resolve, reject)
}


function resolvePromiseResult(promise, backResult, resolve, reject) {
  // 如果结果返回的是自身， 那么直接调用reject 置值器处理。不然程序将会死掉
  if (promise === backResult) {
    reject(new Error("Promise发生了循环引用"))
  };
  if (backResult !== null && (typeof backResult === "object" || typeof backResult === "function")) {
    // Promise 或者then 对象

    try {

      //  取出 then 方法
      const then = backResult.then;
      
      if (typeof then === "function") {
        // 如果 then 是一个函数，那么直接执行 
        then.call(promise, () => {
          // 递归调用，传入y若是Promise对象，继续循环
          resolvePromiseResult(promise, y, resolve, reject)
        }, (error) => {
          // 失败的话使用 reject 置值器
          reject(error);
        })
      }


    } catch (error) {

      // 如果此时报错， 将世界使用 reject 置值器置值
      reject(error);

    }
  } else {
    // 普调值， 直接使用 resove 置值器置值
    resolve(backResult);
  }
}

// new 出来的实例在原型上有 then 方法
MyPromise.prototype.then =function (onFulfilled, onRejected) {

  // then 返回的是全新的 Promise
  const newMyPromise = new MyPromise((resolve, reject) => {
      // 当置值器resolve调用之后，then中的第一个函数参数会被执行（成功回调）；
      // 即当前的 promise 状态为 fulfilled
      if (this.status === "fulfilled") {
        // then 方法是异步指执行的，使用setTimeOut 将同步代码变成异步代码。
        setTimeout(() => {
          try {
            const result = onFulfilled(this.value);
  
            //  因为 result 有可能是 promise，抽离出来处理
            resolvePromiseResult(newMyPromise, result, resolve, reject);
            
          } catch (error) {
            // 如果报错会被当前的 MyPromise 的 reject 置值器绑定给下一个 MyPromise
            reject (error)
          }
        }, 0)
      }

      // 当置值器reject调用之后，then中的第一个函数参数会被执行（失败回调）；
      // 即当前的 promise 状态为 rejected
      if (this.status === "rejected") {
        // then 方法是异步指执行的，使用setTimeOut 将同步代码变成异步代码。
        setTimeout(() => {
          try {
            const result = onRejected(this.value)
 
           //  因为 result 有可能是 promise，抽离出来处理
            resolvePromiseResult(newMyPromise, result, resolve, reject);
 
         } catch (error) {
           // 如果报错会被当前的 MyPromise 的 reject 置值器绑定给下一个 MyPromise
           reject(error)
         }
        }, 0)


      }

      // 当还不知道实例状态的情况下，把成功回调和失败回调都保存起来
      // 等到确定调用哪个回调之后，再拿出来调用
      if (this.status === "pending") {
       
        this.onResolvedCallbacks.push(() => {
          try {
            const result = onFulfilled(this.value);

            //  因为 result 有可能是 promise，抽离出来处理
            resolvePromiseResult(newMyPromise, result, resolve, reject);

          } catch (error) {
            // 回调函数的返回值会被当前的 MyPromise 的 reject 置值器绑定给下一个 MyPromise
            reject(error);
          }
        });
        this.onRejectedCallbacks.push(() => {
          try {
            const result = onRejected(this.value);

            //  因为 result 有可能是 promise，抽离出来处理
            resolvePromiseResult(newMyPromise, result, resolve, reject);

          } catch (error) {
            // 回调函数的返回值会被当前的 MyPromise 的 reject 置值器绑定给下一个 MyPromise
            reject(error);
          }
        })
      }
  })
  return newMyPromise;
}
```





## 测试例子

最后我们来测试一下：

##### 测试用例1

```javascript
const p = new MyPromise((resolve, reject) => {
  console.log("1");
  setTimeout(() => {
    resolve("2");
  })
})

p.then((value) => {
  console.log("成功1", value)
},  (reason) => {
  console.log("失败1", reason)
})

p.then((value) => {
  console.log("成功2", value)
},  (reason) => {
  console.log("失败2", reason)
})

console.log("3");
```

实际结果与期望结果一致：

```javascript
1
3
成功1 2
成功2 2
```

支持异步，支持多次调用`then`

##### 测试用例2

```javascript
const p = new MyPromise((resolve, reject) => {
  console.log("1");
  resolve("2");
})

p.then((value) => {
  console.log("成功1", value)
},  (reason) => {
  console.log("失败1", reason)
})

p.then((value) => {
  console.log("成功2", value)
},  (reason) => {
  console.log("失败2", reason)
})

console.log("3");

```

实际结果与期望结果一致：

```javascript
1
3
成功1 2
成功2 2
```

实现`then`方法是异步的，支持多次调用`then`

##### 测试用例3

```javascript
const p = new MyPromise((resolve, reject) => {
  console.log("1");
  setTimeout(() => {
    resolve("2");
  }, 1000)
})
p.then((value) => {
  console.log("成功1", value);
  return 3; // 返回普调值
}, (reason) => {
  console.log("失败1", reason);
  return 4
})
.then((value) => {
  console.log("成功2", value);
  return 5
},(reason) => {
  console.log("失败2", reason);
  return 6
})
console.log(7)
```

实际结果与期望结构一致：

```javascript
1
7
成功1 2
成功2 3
```

支持链式调用

# Promise 的原型方法

原型除了` then` 方法之外， 当然还有 `catch`, `finally`方法。 那我们来一一实现下。

## Promise.prototype.catch()

`catch`方法是` then `方法的语法糖。 只接受rejected 状态的数据。它就相当于`then` 的第二个参数函数。 那写的时候直接调用 then 方法的第二个参数函数即可。

```javascript
MyPromise.prototype.catch = function(callback) {
  return this.then(null, callback)
}
```

测试：

```javascript
const p = new MyPromise((resolve, reject) => {
  reject("失败");
})
p.catch((err) => {
  console.log(err)
})
// 失败
```

```JAVA

const p = new Promise((resolve, reject) => {
  throw new Error("失败")
})
p.catch((err) => {
  console.log(err)
})
// Error: 失败
```



## Promise.prototype.finally()

`finally`方法 也是`Promise`的一个语法糖，在当前的`Promise`执行完`then`或者执行完`catch`之后均会触发。

注意：`finally`的参数（回调函数） 是不带任何的参数的。

- 考虑到当前返回的` resolve` 可能是个异步函数，所以要通过调用实例上的 `then`方法，添加 `callback` 逻辑

- 成功传 `value` 值， 失败传`error`

  

```javascript

MyPromise.prototype.finally = function(callback) {
  return this.then((value) => {
    Promise.resolve(callback())
    .then(() => value)
  }, (err) => {
    Promise.resolve(callback())
   .then(() => err)
  })
}
```

测试：

```JAVA
const p = new MyPromise((resolve, reject) => {
  resolve("111")
})
p.finally(() => {
  console.log("222")
})
p.then((value) => {
  console.log("value", value)
})
  
// 222
// value 111
```





# Promise 的类方法



## Promise.all()

### 过程

`Promise.all(arr)` 返回一个新的 `Promise`实例，参数是外面传入的多个 `Promise`的实例。对于返回新的实例的，有以下这两种情况：

- 如果传入的所有 promise 实例的状态均变为`fulfilled`，那么返回的 promise 实例的状态就是`fulfilled`，并且其 value 是 传入的所有 promise 的 value 组成的数组。
- 如果有一个 promise 实例状态变为了`rejected`，那么返回的 promise 实例的状态立即变为`rejected`。

### 代码实现

实现思路：

- 传入的参数不一定是数组对象，可以是"遍历器"
- 传入的每个实例不一定是 promise，需要用`Promise.resolve()`包装
- 借助"计数器"，标记是否所有的实例状态均变为`fulfilled`

```javascript
MyPromise.all = function(arg) {
  const arrPromises = [...arg];
  const result = [];
  let num = 0;
  return new Promise((resolve,reject) => {
    arrPromises.forEach((p) => {
      Promise.resolve((p) => {
        p.then((vaue) => {
          result.push(value);
          num++;
          // 必须要等所有的都执行完，才能返回
          if (num === arrPromises.length) {
            resolve(result);
          }
        })
        .catch((err) => {
          reject(err)
        })
      })
      
    })
  })
}
```



## Promise.race()

### 过程

尝试resolve 所有元素。只要其任一元素resolved 或 rejected，都将以该结果作为结果promise

> `Promise.race(iterators)`的传参和返回值与`Promise.all`相同。但其返回的 promise 的实例的状态和 value，完全取决于：传入的所有 promise 实例中，最先改变状态那个（不论是`fulfilled`还是`rejected`）。

### 实现：

- 某传入实例`pending -> fulfilled`时，其 value 就是`Promise.race`返回的 promise 实例的 value
- 某传入实例`pending -> rejected`时，其 error 就是`Promise.race`返回的 promise 实例的 error

```javascript
MyPromise.race = function(arg) {
  const arrPromises = Array.from(arg);
  return new Promise((ressolve, reject) => {
    arrPromises.forEach((p) => {
      p.then((value) => {
        resolve(value);
      })
      .catch((err) => {
        reject(err);
      })
    });
  })
}
```



## Promise.any

### 过程

`Promise.any(iterators)`的传参和返回值与`Promise.all`相同。

如果传入的实例中，有任一实例变为`fulfilled`，那么它返回的 promise 实例状态立即变为`fulfilled`；如果所有实例均变为`rejected`，那么它返回的 promise 实例状态为`rejected`。

⚠️`Promise.all`与`Promise.any`的关系，类似于，`Array.prototype.every`和`Array.prototype.some`的关系。

### 实现

实现思路和`Promise.all`及其类似。不过由于对异步过程的处理逻辑不同，**因此这里的计数器用来标识是否所有的实例均 rejected**。

```javascript
MyPromise.any = function(arg) {
  const arrPromises = Array.from(arg);
  let rejectedNum = 0;
  let errResult = [];
  return new Promise((resolve, reject) => {
    arrPromises.forEach((p) => {
      p.then((value) => {
        rresolve(value)
      })
      .catch((err) => {
        errResult.push(err)
        rejectedNum++;
        if (rejectedNum === arrPromises.length) {
          reject(errResult);
        }
      })
    })
  })
}
```



## Promise.allSettled

### 作用：

作用也是将多个promise实例，包装成一个新的Promise实例.。但是只要这些参数的实例都返回结果，不管是fulfilled还是rejected,包装实例才会结束。

### 过程

`Promise.allSettled(iterators)`的传参和返回值与`Promise.all`相同。

根据[ES2020](https://github.com/tc39/proposal-promise-allSettled)，此返回的 promise 实例的状态只能是`fulfilled`。对于传入的所有 promise 实例，会等待每个 promise 实例结束，并且返回规定的数据格式。

如果传入 a、b 两个 promise 实例：a 变为 rejected，错误是 error1；b 变为 fulfilled，value 是 1。那么`Promise.allSettled`返回的 promise 实例的 value 就是：

```javascript
[{ status: "rejected", value: error1 }, { status: "fulfilled", value: 1 }];

```

### 代码实现

计数器，用于统计所有传入的 promise 实例。

```javascript
MyPromise.allSetlled = function(arg) {
  const arrPromises = Array.from(arg);
  let  result = [];
  let num = 0;
  return new Promise((resolve, reject) => {
    arrPromises.forEach((p) => {
      p.then((value) => {
        result.push(value);
        num++;
        if (num === arrPromises.length) {
          resolve(result)
        }
      })
      .catch((err) => {
        result.push(err);
        num++;
        if (num === arrPromises.length) {
          resolve(result)
        }
      })
    })
  })
}


```

Promise.all、Promise.any 和 Promise.allSettled 中计数器使用对比:

这三个方法均使用了计数器来进行异步流程控制，下面表格横向对比不同方法中计数器的用途，来加强理解：

| 方法名               | 用途                                        |
| -------------------- | ------------------------------------------- |
| `Promise.all`        | 标记 fulfilled 的实例个数                   |
| `Promise.any`        | 标记 rejected 的实例个数                    |
| `Promise.allSettled` | 标记所有实例（fulfilled 和 rejected）的个数 |



## Promise.resolve()

该方法是将现有的对象转成 `promise`对象。

### 过程

- 如果当前的值是一个 `Promise`对象或者是 `thenable` 那么直接返回。

- 反之则返回一个 使用` resolve` 置值器绑定当前值 的`Promise`

### 实现

```javascript
MyPromise.resolve = function(arg) {
  // 如果是一个Promise 对象 或者是 thenable 对象。那么直接返回
  if (arg instanceof Promise || (typeof arg === 'object' && 'then' in arg )) return arg;
  return new Promise((resolve, reject) => {
    resolve(arg);
  })
}

```



## Promise.resolve()

该方法返回一个新的 Promise 实例，该实例的状态为`rejected`。

**注意：**`Promise.reject()`方法的参数，会原封不动地作为`reject`的理由，变成后续方法的参数。这一点与`Promise.resolve`方法不一致。

### 实现

```javascript
MyPromise.reject = function(arg) {
  return new Promise((resolve, reject) => {
    reject(arg);
  })
}
```





# 最后

最后附上完整的代码。 当然在写一写Promise的原型和类方法的时候， 可能会用用到了原生的`Promise`。当然是可以替换成我们自己的`MyPromise`的。

```javascript
function MyPromise(executor) {
  const self = this;

  // 保存成功回调：then方法中的第一个参数
  self.onResolvedCallbacks = [];

  // 保存失败回调：：then方法中的第二个参数
  self.onRejectedCallbacks = [];

  // 实例的默认状态： pending
  self.status = "pending";

  // resolve 置值器： 用于修改状态
  function resolve(value) {
    // Promise的状态一经修改就不能再改变，所以只有在 pending 状态才能修改
    if (self.status === "pending") {
      self.status = "fulfilled";
      // Promise的值
      self.value = value;

      // 把存起来的成功回调调用
      self.onResolvedCallbacks.forEach((fn) => {
        // then 方法是异步指执行的，使用setTimeOut 将同步代码变成异步代码。
        setTimeout(() => {
          fn();
        }, 0)
      })
    }
  }

  // reject 置值器： 用于修改状态
  function  reject(reason) {
    // Promise的状态一经修改就不能再改变，所以只有在 pending 状态才能修改
    if (self.status === "pending") {
      self.status = "rejected";
      // Promise的值
      self.value = reason;

      // 把保存起来的失败回调拿出来调用
      self.onRejectedCallbacks.forEach((fn) => {
        // then 方法是异步指执行的，使用setTimeOut 将同步代码变成异步代码。
        setTimeout(() => {
          fn();
        }, 0)
      })
    }
  }

  // executor 是会被立即执行的
  executor(resolve, reject)
}


function resolvePromiseResult(promise, backResult, resolve, reject) {
  // 如果结果返回的是自身， 那么直接调用reject 置值器处理。不然程序将会死掉
  if (promise === backResult) {
    reject(new Error("Promise发生了循环引用"))
  };
  if (backResult !== null && (typeof backResult === "object" || typeof backResult === "function")) {
    // Promise 或者then 对象

    try {

      //  取出 then 方法
      const then = backResult.then;
      
      if (typeof then === "function") {
        // 如果 then 是一个函数，那么直接执行 
        then.call(promise, () => {
          // 递归调用，传入y若是Promise对象，继续循环
          resolvePromiseResult(promise, y, resolve, reject)
        }, (error) => {
          // 失败的话使用 reject 置值器
          reject(error);
        })
      }


    } catch (error) {

      // 如果此时报错， 将世界使用 reject 置值器置值
      reject(error);

    }
  } else {
    // 普调值， 直接使用 resove 置值器置值
    resolve(backResult);
  }
}

// new 出来的实例在原型上有 then 方法
MyPromise.prototype.then = function (onFulfilled, onRejected) {

  // then 返回的是全新的 Promise
  const newMyPromise = new MyPromise((resolve, reject) => {
      // 当置值器resolve调用之后，then中的第一个函数参数会被执行（成功回调）；
      // 即当前的 promise 状态为 fulfilled
      if (this.status === "fulfilled") {
        // then 方法是异步指执行的，使用setTimeOut 将同步代码变成异步代码。
        setTimeout(() => {
          try {
            const result = onFulfilled(this.value);
  
            //  因为 result 有可能是 promise，抽离出来处理
            resolvePromiseResult(newMyPromise, result, resolve, reject);
            
          } catch (error) {
            // 如果报错会被当前的 MyPromise 的 reject 置值器绑定给下一个 MyPromise
            reject (error)
          }
        }, 0)
      }

      // 当置值器reject调用之后，then中的第一个函数参数会被执行（失败回调）；
      // 即当前的 promise 状态为 rejected
      if (this.status === "rejected") {
        // then 方法是异步指执行的，使用setTimeOut 将同步代码变成异步代码。
        setTimeout(() => {
          try {
            const result = onRejected(this.value)
 
           //  因为 result 有可能是 promise，抽离出来处理
            resolvePromiseResult(newMyPromise, result, resolve, reject);
 
         } catch (error) {
           // 如果报错会被当前的 MyPromise 的 reject 置值器绑定给下一个 MyPromise
           reject(error)
         }
        }, 0)


      }

      // 当还不知道实例状态的情况下，把成功回调和失败回调都保存起来
      // 等到确定调用哪个回调之后，再拿出来调用
      if (this.status === "pending") {
       
        this.onResolvedCallbacks.push(() => {
          try {
            const result = onFulfilled(this.value);

            //  因为 result 有可能是 promise，抽离出来处理
            resolvePromiseResult(newMyPromise, result, resolve, reject);

          } catch (error) {
            // 回调函数的返回值会被当前的 MyPromise 的 reject 置值器绑定给下一个 MyPromise
            reject(error);
          }
        });
        this.onRejectedCallbacks.push(() => {
          try {
            const result = onRejected(this.value);

            //  因为 result 有可能是 promise，抽离出来处理
            resolvePromiseResult(newMyPromise, result, resolve, reject);

          } catch (error) {
            // 回调函数的返回值会被当前的 MyPromise 的 reject 置值器绑定给下一个 MyPromise
            reject(error);
          }
        })
      }
  })
  return newMyPromise;
}

MyPromise.prototype.catch = function(callback) {
  return this.then(null, callback)
}

MyPromise.prototype.finally = function(callback) {
  return this.then((value) => {
    Promise.resolve(callback())
    .then(() => value)
  }, (err) => {
    Promise.resolve(callback())
   .then(() => err)
  })
}


MyPromise.all = function(arg) {
  const arrPromises = Array.from(arg);
  const result = [];
  let num = 0;
  return new Promise((resolve,reject) => {
    arrPromises.forEach((p) => {
      Promise.resolve((p) => {
        p.then((vaue) => {
          result.push(value);
          num++;
          // 必须要等所有的都执行完，才能返回
          if (num === arrPromises.length) {
            resolve(result);
          }
        })
        .catch((err) => {
          reject(err)
        })
      })

    })
  })
}

MyPromise.race = function(arg) {
  const arrPromises = Array.from(arg);
  return new Promise((ressolve, reject) => {
    arrPromises.forEach((p) => {
      p.then((value) => {
        resolve(value);
      })
      .catch((err) => {
        reject(err);
      })
    });
  })
}

MyPromise.any = function(arg) {
  const arrPromises = Array.from(arg);
  let rejectedNum = 0;
  let errResult = [];
  return new Promise((resolve, reject) => {
    arrPromises.forEach((p) => {
      p.then((value) => {
        rresolve(value)
      })
      .catch((err) => {
        errResult.push(err)
        rejectedNum++;
        if (rejectedNum === arrPromises.length) {
          reject(errResult);
        }
      })
    })
  })
}


MyPromise.allSetlled = function(arg) {
  const arrPromises = Array.from(arg);
  let  result = [];
  let num = 0;
  return new Promise((resolve, reject) => {
    arrPromises.forEach((p) => {
      p.then((value) => {
        result.push(value);
        num++;
        if (num === arrPromises.length) {
          resolve(result)
        }
      })
      .catch((err) => {
        result.push(err);
        num++;
        if (num === arrPromises.length) {
          resolve(result)
        }
      })
    })
  })
}

const obj = {
  then: () => {
    
  }
}
console.log('then' in obj)

MyPromise.resolve = function(arg) {
  // 如果是一个Promise 对象 或者是 thenable 对象。那么直接返回
  if (arg instanceof Promise || (typeof arg === 'object' && 'then' in arg )) return arg;
  return new Promise((resolve, reject) => {
    resolve(arg);
  })
}


MyPromise.reject = function(arg) {
  return new Promise((resolve, reject) => {
    reject(arg);
  })
}


```































