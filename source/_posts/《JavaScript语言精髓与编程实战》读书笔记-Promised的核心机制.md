---
title: 《JavaScript语言精髓与编程实战》读书笔记-Promised的核心机制
date: 2020-07-15 08:46:49
tags: ES6
categories: ES6
---



最近参部门的读书会，读书会会推荐一些书来供我们学习， 这次的读书会推荐读的书是《JavaScript语言精髓与编程实战》。感觉写的很不错，我这次学习的是【Promise的核心机制】, 虽然之前也看过关于`Promise`  [ES6学习笔记-Promise](https://shuliqi.github.io/2018/03/20/ES6学习-Promise/), 但是在看这本书之后，讲的更多关于`Promise`的原理。还是有值得很深入的点。

最后感觉跟着大神们学习感觉很不错呀，哈哈嘻😊！

 <!--more-->

# 什么是Promise?

Promise对象是一个代理对象（代理一个值）。被代理的值在Promise创建的时候可能是未知的。它允许你为异步操作的成功与失败分别绑定相应的处理方法。这让异步可以像同步方法那样返回值，但并不是立即返回最终的执行结果，而是一个能代表未来的出现的结果的promise对象。

# promise的状态

Promise 一种有三种值的状态： **pending(进行中)**, **fulfiiled(已成功)**，**rejected(已失败)‘**

# Promise的特点

* **对象的状态不受外界的影响**

  只有异步的结果可以决定当前是哪一种状态（pending， fulfilled, rejected）。 其他的手段都是无法改变的。

* **一旦状态改变，就不会再变，任何时候都可以拿到这个状态**

  Promise的状态只有两种可能：从pending 到 fulfilled， 从peding 到rejected。只要这两种发生了，状态也就凝固了，不会再变了。

  如果在对Promise对象添加回调函数，也会立刻拿到这个结果。

# 创建Promise

创建Promise对象，主要有三种方式：

* 使用 `new Promise()`来创建一个`promise`(即`new Promise() `构造器)
* 使用类方法Promise.xxx()-——`Promise.all()`，`Promise.race()`，`Promise.allsettled()`，`Promise.any()`，`Promise.resolve()`，`Promise.reject()`

* 使用原型形方法`Promise.prototype.xxx()`——`promise.then()`，`promise.catch()`，`promise.fanally()`都将返回一个新的promise

  > 任何方法得到的`promise`对象都具有`.then`，`.catch`等方法，也是`Promise.prototype.xxx`的原型方法，`Javascript`调用这些方法将绝对不会抛出异常，所以这就是上面第三种得到一个新的`promise`对象的方法

**注意：**任何一种方法都是立即得到`promise`对象的

# Promise的构造方法

`Promise`通常使用一个简单的构造器来让用户方便的创建`promise`对象：

```javascript
// 创建promise对象的构造器
const p = new Promise(executor);

// 需要用户声明的执行函数
const executor = function(resolve, reject) {
  
}
```

上面代码的`executor` 是用户声明的执行函数，当`JavaScript`引擎通过new 运算符来创建`promise`对象时，它事实上在调用`executor()`之前就创建好一个新的`promise`对象的实例，并且得到关联给这个实例的两个置值器：`resolve()`，`reject()`函数，然后，它会调用`executor()`，并且把这两个置值器作为入口参数传入，而`executor()`函数会被执行直到退出。

**注意：**`executor()`函数并不通过退出时所返回的值来对系统造成影响——该返回值将会被忽略（无论是显式返回结果，还是默认返回值`undefined`）。`executor()`中的用户代码可以利用上述的两个置值器，来向`promise`对象“代理的那个数据”置值。也就是说，为`promise`绑定值的过程是由用户代码来触发的。这个过程看起来像”让用户代码回调`JavaScript`引擎”。例如：

```javascript
// 用户代码通过resolve(或reject）来回调引擎以置值
const p = new Promise((resolve, reject) => {
  resolve("shuliqi");
})
```

**注意：`executor()`**函数中的`resolve`·置值器可以接受任何值（除了当前的promise本身之外）,如果使用自身来置值时，`JavaScript`会抛出一个异常。如：

```javascript
// 暂存resolve置值的变量
let delayResolve;
 const p = new Promise((resolve, reject) => {
    delayResolve = resolve;
 });
// 尝试使用自身来置值
 delayResolve(p)

// 结果：TypeError: Chaining cycle detected for promise #<Promise>
```

# 构建Promise对象：没有延迟

在构建整个`Promise`对象的过程中， 是没有任何的延迟的。`Promise`机制中没有延迟， 也没有被延迟的行为，更没有对"时间"这个维度进行控制。因此`JavaScript`中创建一个`promise`时，创建过程时立即完成的；使用原型的方法`promise.xxx`来得到一个新的`promise`时也是立即完成的。所有`promise`对象都是在你需要的时候立即就生成的。

只不过重要的是---->**这些`promise`所代理的那个值/数据还没有“就绪”。这个就绪的过程要推迟到“未知的将来”才会发生，而一旦数据就绪，promise.then(fun)中的fun就会被触发了**

```javascript
const p = new Promise((resolve, reject) => {
    console.log('shuliqi');
    setTimeout(() => resolve('1'), 3000)
})
p.then((result) => {
   console.log(result);
})
```

只要我们创建了`promise` 对象，这个过程是立即的，所以立马输出“shuliqi”。然后过3秒，输出 “1”。



# Then 链

两个`Promise `对象之间顺序执行的关系，在`JavaScript`中称为“Then 链”。

```javascript
// 创建一个Promise 对象P
const p1 = new Promise((resolve, reject) => {
    resolve('shuliqi')
})
// 通过p.then()来得到新的promise对象 p2
const p2 = p.then(foo() => {
  
});
```

通过调用`p.then(`)的方式来约定当前的`promise `对象与下一个`promise`对象之间的“链”关系，并且这事实上也也代表了它们之前的顺序执行的关系（**所谓的“顺序执行”是指它们的置值逻辑以及触发的行为之间的关系----> 因为Promise本身是数据的代理所以并不是执行体**）。



并且then链是Promise机制的基本用法和关键的机制。上面例子中的p.then()代表了对顺序的逻辑的理解， 同时它也隐含的说明：p2 与p1两者所代理的数据之间是有关联的。这个例子的foo()函数是作为p1的成功回调，如下面这个例子的作为onFulfilled参数：

```javascript
// 创建一个Promise 对象P
const p1 = new Promise((resolve, reject) => {
    resolve('shuliqi');
})
// 通过p.then()来得到新的promise对象 p2
const p2 = p.then(onFulfilled, onRejected);
function onFulfilled(value) {
   console.log(value); // shuliqi
}
function onRejected() {

}
```

在当前的promise数据就绪时，JavaScript就将根据就绪状态立即出发有p.then()方法所关联的onFulfilled/ onRejected之一，并且这个函数**退出时返回的值**或者**终止执行时的状态**作为值来调用p2

p.then()实际上是Promise.prototype.then这个原型的方法，它的作用主要是完成三件事：

* 创建新的promise2对象
* 登记当前promise 与 promise2 之间的关系（顺序执行的关系和代理的数据之间是有关联的）
* 将onFulfilled ，onRejected关联给promise2 的resolve置值器，并且确保早promise1的数据就绪时调用onFulfilled， onRejected。

## promise1的置值逻辑

所以在构建promise的执行器中，可以向resolve/reject 传入任意的JavaScript数据类型：

```javascript
const p = new Promise((resolve, reject) => {
  try {
    resolve('我显示使用resolve置值器置值')
  } catch (error) {
    reject('我显示使用reject 置值器置值')
  }
})
```

这个例子显式的使用resolve / reject 置值器来置值。

如果执行器在执行的过程中触发异常，javaScript引擎也将调用reject， 并且把异常作为reason。

```javascript
const p = new Promise((resolve, reject) => {
  throw new Error(); // 创建异常，并且抛出，相当于reject(new Error())
})
```

接下来，如果在它的then 链上有promise2,那么如前所诉：onFulfilled/onRejected将被触发，例如：

````javascript
const p = new Promise((resolve, reject) => {
  try {
    resolve('我显示使用resolve置值器置值')
  } catch (error) {
    reject('我显示使用reject 置值器置值')
  }
})
const promise2 =p.then(onFulfilled, onRejected);

function onFulfilled() {
    console.log('我是resolve置值器置值成功的回调')
}
function onRejected() {
    console.log("我是reject置值器置值或者是resolve置值器置值失败的回调")
}
// 结果：
// 我是resolve置值器置值成功的回调
````

```javascript
const p = new Promise((resolve, reject) => {
    throw new Error(); // 创建异常，并且抛出，相当于reject(new Error())
  })
const promise2 =p.then(resolved, rejected);

function resolved() {
    console.log('我是resolve置值器置值成功的回调')
}
function rejected() {
    console.log("我是reject置值器置值或者是resolve置值器置值失败的回调")
}
// 结果：
// 我是reject置值器置值或者是resolve置值器置值失败的回调
```

以上的的 resolved， rejected 都只是 p 的状态，他们只是promise2的置值前提而已， 那么promise2的置值逻辑是啥呢？

## promise2的置值逻辑

一个Promise可能被置入两种值之一（并且一旦置值将不可改变，称为“终态”），这两种值时指：

* 如果promise 被 resolve置值器置值成功，则该值称为有效值（value）
* 如果promise 被 reject置值器置值 或者 resolve置值器置值 失败，则该值用于记录原因（reason）。

**无论rejected，rejected函数无论返回何值，都将作为resolve值直接绑定给promise2**(即：resolved, rejected函数会关联在promise2的resolve 值值器上)

promise2 的置值逻辑伪代码如下：

```javascript
try {
    // 判断上一个promise的置值状态
    if (isRejected(p)) {
        x2 = rejected(result);  // 执行rejected函数，result为reason（result是p代理的数据）
    } else {
        x2 = resolved(result); // 执行resolved函数
    } 
    resolve(x2); // promise2
} catch (error) {
    reject(error)
}
```

具体例子：

```javascript
const resolvedForP1 = function(value) {
    return value;
}
const rejectedForP1 = function(reason) {
    return reason;
}
const resolvedForP2 = function(value) {
    console.log('我是p2 resolved状态,', value);
}
const rejectedForP2= function(reason) {
    console.log('我是p2 rejected状态,', reason);
}

const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
      // 顺便调用p1的resolve置值器，还是reject置值器
      // resolve('p1 resolve');
      reject('p1 reject');
  }, 1000)
})
const p2 = p1.then(resolvedForP1, rejectedForP1);
p2.then(resolvedForP2, rejectedForP2)
    
// 结果：
// 我是p2 resolved状态, p1 reject
```

这个例子： 无论p1 我调用的是 resolve 置值器还是拍，导致无论调用 resolvedForP1还是 rejectedForP1，返回值都是作为resolve值绑定给p2。

调用（p1  reject 置值器 ----> 调用rejectedForP1 ----> rejectedForP1 返回值 ----> 通过p2 的resolve置值器直接绑定给p2）。

注意：**这不是完成的promise2 的置值逻辑**， 完整的请往下看。

## then 链中“产生” reject值的方法

在then 链中“产生” reject值的方法只有两种：

* 通过抛出异常来使得JavaScript引擎捕获异常对象。

  ```javascript
  const p1 = new Promise((resolve, reject) => {
      resolve('我是舒丽琦')
  });
  // 方法1: 通过抛出异常来使得JavaScript引擎捕获异常对象
  const p2 = p1.then((value)=> {
      throw new Error('通过抛出异常来使得JavaScript引擎捕获异常对象');
  });
  p2.then((value) => {
      console.log(value);
  })
  
  // 结果：
  // UnhandledPromiseRejectionWarning: Error: 通过抛出异常来使得JavaScript引擎捕获异常对象....
  ```

* 通过显式的调用Promise.reject()。

  ```javascript
  const p1 = new Promise((resolve, reject) => {
      resolve('我是舒丽琦')
  });
  // 方法2: 显式的调用Promise.reject() 
  const p2 = p1.then((value)=> {
      return Promise.reject('显式的调用Promise.reject() ')
  });
  p2.then((value) => {
      console.log(value);
  })
  // 结果：
  // UnhandledPromiseRejectionWarning: 显式的调用Promise.reject() ...
  ```

## Then 链对值的传递

如果promise2 的resolve 并没有关联有效的resolved，rejected 呢？ 或者 promise2 根本没有resolved， rejected 函数呢？那么还会生成promise2 吗？ 如果生成， 那么它的值时什么呢？

答案：**如果没有有效的响应函数仍会产生新的promise2, 并且它的resolve 将以then 链中的当前的promise的值为值**，

因此promise2 的完整的伪置值逻辑如下：

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

我们来举例子检测一下：

```javascript
const p1 = new Promise((resolve, reject) => {
  reject('调用p1的reject置值器置值')
});
const p2 = p1.then();
```

{% asset_img 1.png %}

P1代理的值是reject 置值器置值的，在创建p2的时候， rejected 函数不存在(无效), 则抛出异常, 值为当前promise(p1)的值。





```javascript
p1 = new Promise((resolve, reject) => {
  resolve('调用p1的reject置值器置值')
});
p2 = p1.then();
```

{% asset_img 2.png %}

P1代理的值是resolve置值器置值的，在创建p2的时候， resolved函数不存在(无效), 则直接返回当前promise(p1)的值。

## Then 链 catch() 的处理

promise2的置值过程也解释了使用.catch作为then链结尾的原因：

```javascript
const p1 = new Promise((resolve, reject) => {
    reject('调用p1的reject置值器置值')
});
const p2 = p1.then();
const p3 = p2.then();
p3.catch((reason) => {
    console.log(reason)
})
// 结果：
// UnhandledPromiseRejectionWarning: 调用p1的reject置值器置值
```

在任意长的then链中，如果链的前端出现了rejected 值，无论经过多少级.then()响应(只有onFulfilled响应而没有处理)， 最终rejected 值都能够持续向后传递并且被链尾的.catch()响应到。这也就是Promise机制的第一原则：**始于Promise， 终于catch**。

# Promise的类方法

Promise()类方法Promise.xxx主要作用于获得一个promise

promise类方法有四个：

* **Promise.resolve()**

* **Promise.reject()**

* **Promise.all()**

* **Promise.race()**

  

## Promise.resolve()

作用：`Promise.resolve()`将现有的对象转成`Promise`对象

```javascript
const p = Promise.resolve(x)
```

x是任意值，如果不指定则是undefined

* 如果 x 是Promise的一个实例，那么Promise.resolve(x)， 将不会产生新的计算结果，而是直接返回x这个实例

  ```javascript
  const obj = {}
  const p1 = new Promise((resolve, reject) => {
    reject('reject')
  });
  const p2 = Promise.resolve(p1);
  console.log(p1 === p2); // true
  // p1 与 p2 是相同的promise
  ```
  
  ```javascript
  const obj = {}
  const p1 = new Promise((resolve, reject) => {
    resolve('resolve')
  });
  const p2 = Promise.resolve(p1);
  console.log(p1 === p2); // true
  // p1 与 p2 是相同的promise
  ```
  
  
  
  说明：如果x是一个`rejected`的`promise`对象，那么`Project.resolve(x)`将会是一个`rejected`状态的`promise`对象
  
  ```javascript
  const p = new Promise((resolve, reject) => {
      reject('rejected状态')
  })
  const p2 = Promise.resolve(p);
  p2.then().catch((reason) => console.log(reason));
  
  // 结果：
  // rejected状态
  ```
  
  {% asset_img 3.png %}
  
  
  
* 如果x是一个thenable

  thenable对象：指任意带有.then()方法的对象。如：

  ```javascript
  const thenable = {
     then: function() {
       console.log('我是thenable对象')
     }
  }
  ```

  再看下面这个例子：

  ```javascript
  const obj = {
    then: (reaolve, reject) => {
      console.log("我是thenable对象")
    }
  }
  p1 = Promise.resolve(obj);
  ```

  {% asset_img 4.png %}

  `x.then()`是立即执行了， 但是这个`thenable`对象与最终的p没有关系，所以目前这个执行过程是没有什么意义的。因为这个t`henable`对象与最终的`p`对像之间并没有建立关系。为什么没有建立关系呢？这是因为`JavaScript`是试图将x.then()作为一个“类似new Promise()中的执行器（executor）”来使用的。`Promise.resolve(tenable)`导致`.then() `被执行时，`p`这个`promise`对象的`resolve`与r`eject`两个置值器被作为`.then()`的参数传入的，因此(与`new Promise`类似)，用户代码需要在`.then(resolve,reject)`调用这两个置值器才能真正的建立thenable与p之间的关系。

  

  

  所以下面这个例子，p对象与thenable才真正建立起了关系：

  ```javascript
  const obj = {
    then: (resolve, reject) => {
      console.log("我是thenable对象");
      resolve("结果")
    }
  }
  p1 = Promise.resolve(obj);
  ```

  {% asset_img 5.png %}

  

* x以上情况都不是，那么将会返回一个新的`resolved`的`promise`对像，并且代理的数据与`x`的相同。

  ```javascript
  p1 = Promise.resolve("111");
  ```

  {% asset_img 6.png %}

  

## Promise.reject()

`Pomise`类方法`Promise.reject()`得到一个状态为`rejected `的`promise`；

```javascript
Promise.reject(x)； // x是任意值，如果不指定则为undefined   
```

`Promise.reject(x)`与`Promise.resolve(x)`不同，`Promise.reject(x)`是将`x`作为一个普通的对象。这就意味着仍然会得到一个`rejected promise`，并且它的值时一个（类型为Promise）的对象。

```javascript
p1 = Promise.resolve("111");
p2 = Promise.reject(p1);
```

{% asset_img 7.png %}

## Promise.all()

尝试`resolve`所有元素。当所有的元素都`resolved`，得到一个将所有结果作为`resolved` array的`promise`。当任意一个元素`rejected`，得到一个该结果 `reason` 的`rejected promise`。

```javascript
Promise.all(x); // x 必须是可迭代对象（集合对象，或者是有迭代器的对象）
```

**可迭代的对象**：数组， Map/Set集合，字符串可迭代的对象：数组， Map/Set集合，字符串 。

```javascript
const p1 = Promise.resolve("111");
const p2 = Promise.resolve("222");
const p3 = Promise.resolve("333");
const p4 = Promise.resolve("444");

p = Promise.all([p1,p2,p3,p4]);
```

{% asset_img 8.png %}



```javascript

const p1 = Promise.resolve("111");
const p2 = Promise.resolve("222");
const p3 = Promise.reject("333");
const p4 = Promise.resolve("444");

p = Promise.all([p1,p2,p3,p4]);

```

{% asset_img 9.png %}

Promise.all()会对所有的元素进行预处理（Promise.resolve(x)）所以无论x 是一个普通值还是一个“潜在的promise”， 它都将作为一个resolved promise进入后续的处理，即使x是一个rejected promise， 那么它的状态(Promise.resolve(rejected_promise))依然会影响Promise.all()的最终状态的判断。 一旦发现rejected，则返回rejected的结果。

promise.all(element)只有element完全resolved时，会在then()中得到一个与原始element 存在映射关系的数组。

## Promise.any()

**作用：**作用也是将多个promise实例，包装成一个新的Promise实例。 但是是是要参数实例有任何一个状态变成fulfilled。 包装实例对象就会变成fulfilled状态。如果所有的Promise状态都变成rejected, 包装实例对象才会变成rejecte的状态。

```javascript
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("1111")
  }, 1000)
})
const p2 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject("2222");
  }, 500)
})

p = Promise.any([p1,p2]);
p.then((value) => {
  console.log("resolve", value)
}, (reason) =>{
  console.log("reject", reason)
})
// 111
```



## Promise.race()

尝试resolve 所有元素。只要其任一元素resolved 或 rejected，都将以该结果作为结果promise

注意：所有的其他的元素的状态都是未确定的，并且他们的执行过程与结果不确定

```javascript
Promise.race(x); // x 必须是可迭代对象（集合对象，或者是有迭代器的对象）
```

Promise.race()与Promise.all()类似， 都会对所有的元素进行预处理（Promise.resolve(x)）。

```javascript
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("1111")
  }, 1000)
})
const p2 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject("2222");
  }, 500)
})

p = Promise.race([p1,p2]);
p.then((value) => {
  console.log("resolve", value)
}, (reason) =>{
  console.log("reject", reason)
})
// reject 2222
```

## Promise.allSettled()

**作用：**作用也是将多个promise实例，包装成一个新的Promise实例.。但是只要这些参数的实例都返回结果，不管是fulfilled还是rejected,包装实例才会结束。

```javascript
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("1111")
  }, 1000)
})
const p2 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject("2222");
  }, 500)
})

p = Promise.allSettled([p1,p2]);
p.then((value) => {
  console.log("resolve", value)
}, (reason) =>{
  console.log("reject", reason)
})

// resolve [
//   { status: 'fulfilled', value: '1111' },
//   { status: 'rejected', reason: '2222' }
// ]
```



# Promise对象的原型方法

Promise对象方法Promise.prototype.xxx(p.xxx)主要用于响应一个promise 状态，并且对象方法的响应结果也必然是返回一个新的promise2。除了.then()之外，promise对象最主要的的原型方法—.catch() 和.finally()都是通过通过.then()来间接实现的。

## Promise.Prototype.catch()

.catch(onRejected) 与.then(_, onRejected)中的参数一致， 两种用法的效果也近似。

但是使用.catch()的方法， 尤其是在Then链末端的.catch()是非常必要的和安全的， 因为.then(onFulfilled, onRejected)中的onRejected函数并不能捕获到onFulfilled函数里面的异常(以及其中返回的rejected promise)。 如下面的例子：

```javascript
const onFulfilled = () => {
    return Promise.reject('我抛出异常，产生rejected promise')
};
const onRejected = () => {
    console.log('没有能捕获onFulfilled函数返回的rejected promise')
};
const p = Promise.resolve();
p.then(onFulfilled, onRejected)
```

这个例子中的onRejected 函数并不能响应onFulfilled函数中抛出的异常以及JavaScript为该异常而创建的rejected promise，将会遗漏对rejected Promise的处理，争取的处理应该是遵循“始于promise，终于catch”的原则。如下例子：

```javascript
const onFulfilled = () => {
    return Promise.reject('我抛出异常，产生rejected promise')
};
const onRejected = () => {
    console.log('没有能捕获onFulfilled函数返回的rejected promise')
};
const p = Promise.resolve();
p.then(onFulfilled, onRejected).catch((reason) => {
    console.log('我来处理onFulfilled产生的异常:', reason)
})
// 结果：
// 我来处理onFulfilled产生的异常： 我抛出异常，产生rejected promise
```

## Promise.Prototype.finally()

finally()方法，与try...catch/finally中的finally子句的语法类似；使代码无论.catch()还是.then()调用结束后，总是能得到一次调用.fanally()的机会。

**该方法不接受任何的参数，意思就是最后的Promise代理的值是什么， 是没有办法知道的。finally()方法跟最后的promise的状态是无关的。**

```javascript
// finally（）方法的响应函数是无传入参数的
const onFinally = (value) => {
    console.log("不管promise状态怎么样， 我都会执行, 并且我是无参数参入的哦", value);
}
const p = new Promise((resolve, reject) => {
    throw new Error("我抛出错误");
});
p
.then(() => {
    console.log("成功");
})
.catch((error) => {
    console.log("发生错误", error);
})
.finally(onFinally);
```

上面例子的结果为：

```javascript
发生错误 Error: 我抛出错误
    at <anonymous>:6:15
    at new Promise (<anonymous>)
    at <anonymous>:5:15
VM134:3 不管promise状态怎么样， 我都会执行, 并且我是无参数参入的哦 undefined
```

这个例子中的onFinally函数中，我们打印value 值，值为undefined。说明.finally()的onFinally函数是不接受任何参数的。

**默认.finally()方法不会改变Then 链上前端所产生的值，也包括其状态。**

JavaScript 不处理在响应函数onFinally中的任何返回值，.finally()方法的结果---只是简单的“得到了”链上的前一个promise对象所代理的数据。如：

```javascript
const x = "我是舒力气"
const p1 = Promise.resolve(x);
const p2 = p1.finally(() => {
    return '我的年龄是21'
});
p2.then((value) => {
    console.log('我的值是：', value)
});
// 我的值是：我是舒力气
```

以上例子中.finally()的onFinally中的返回值自动被忽略。所以p2的值是前一个promise对象所代理的值（p1的值）。

当然，也有例外的时候。如果.finally()的响应函数onFinally()中发生了异常，或者用户代码显式的返回了rejected promise。 那么就会替代了then链上之前的promise对象所代理的值了。如：

```javascript
const x = "我是舒力气"
const p1 = Promise.resolve(x);
const p2 = p1.finally(() => {
  return Promise.reject('我在onFinally函数中显式的返回了rejected promise了')
});
p2.catch((reason) => {
    console.log('我的值是：', reason)
});
// 我的值是： 我在onFinally函数中显式的返回了rejected promise了
```

## Promise.Prototype.then()

其实`then`方法是最重要的， 在前面我们都是讲过知识点。



