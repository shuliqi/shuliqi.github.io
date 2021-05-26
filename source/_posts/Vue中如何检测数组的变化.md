---
title: Vue中如何检测数组的变化?
date: 2019-06-03 19:54:12
tags: Vue
categories: Vue
---



之前学习了关于`Vue`的响应式数据的原理： [Vue 的双向绑定原理及手把手实现](https://shuliqi.github.io/2019/04/20/Vue%E7%9A%84%E5%8F%8C%E5%90%91%E7%BB%91%E5%AE%9A%E5%8E%9F%E7%90%86-%E6%89%8B%E6%8A%8A%E6%89%8B%E5%AE%9E%E7%8E%B0/)。它的原理其实就是通过`Object.defineProperty`控制	`getter`和`setter`结合发布订阅者模式完成响应式设计的，

**1. 但是这种数据劫持对数组有什么影响呢？**

这种递归方式无论对于数组还是对象都进行了观测。但是我们的数组有成千上万个元素，每一个元素下标都添加`get` 和`set`.这样对于性能来说代价太大了。那么`Object.property`只用来劫持对象。

**2. Object.property这种劫持方式有什么缺点呢？**

对于新增的或者删除的属性是无法被检测到的，只有对象本身存在的属性才会被劫持。

对于数组来说也是一样，新增加的元素和删除的元素无法对他们的下表进行劫持。

 <!--more-->

根据以上两点可以得出这就是为什么`Vue`官方说如下的话：

> 由于`JavaScript`的限制，`Vue`无法检测到以下数组的变动：
>
> - 当你使用索引设置一项时；如：`vm.items[indexOfItem] = newValue`
> - 当修改数组的长度时；如：vm.items.length =  newLength

我们举个例子：

之前的文章  [Vue 的双向绑定原理及手把手实现](https://shuliqi.github.io/2019/04/20/Vue%E7%9A%84%E5%8F%8C%E5%90%91%E7%BB%91%E5%AE%9A%E5%8E%9F%E7%90%86-%E6%89%8B%E6%8A%8A%E6%89%8B%E5%AE%9E%E7%8E%B0/)写好了`Observer`。 我们来做一个实验：

```html
<html>
  <header></header>
  <body>  <div id="app"></div></body>
  <script src="./js/observer.js"></script>
  <script src="./js/index.js"></script>
  <script src="./js/compile.js"></script>
  <script src="./js/watcher.js"></script>
  <script>
    const vm = new MyVue({
      el: "#app",
      data: {
        name: "舒丽琦",
        list: ["1", "2", "3"]
      }
    })
    // 设置普调对象的值
    vm.name = " 小小舒" 
    // 获取普调对象的值
    vm.name;

    // 设置数组的值
    vm.list.push("4");
    // 获取数组的值
    vm.list[0];
    console.log(vm.list);
  </script>
</html>
```

> 关上面的代码的代码可到  [Vue 的双向绑定原理及手把手实现](https://shuliqi.github.io/2019/04/20/Vue%E7%9A%84%E5%8F%8C%E5%90%91%E7%BB%91%E5%AE%9A%E5%8E%9F%E7%90%86-%E6%89%8B%E6%8A%8A%E6%89%8B%E5%AE%9E%E7%8E%B0/)找到

我们看看实验的结果：

{% asset_img 1.png %}

从结果可以看出：`data` 中的 `name`,` list` 均发生了变化; `name`发生了变化能检测到，但是是`list`发生变化无法检测到。这是为什么呢？

原来操作数组的方法是在 `Array.prototype`上的； 挂在在`Array.prototype`上的方法并不能触发属性的`getter`和`setter`。

# Vue 检测数组变化-重写数组

那解决这个问题的办法是什么呢？`Vue2.x`使用的是将数组常用的方法进行重写。

基本的思路是之前我们调用数组的常用的方法的时候（如`push`·）；我们是从 `Array.prototype`上面寻找这个方法，现在我们改成一个空对象 {} 继承  `Array.prototype`，然后给 空对象添加 `push`方法；

```javascript
{
  push： function() {}
  // 其他常用的方法
}
```

这样，我们调用常用的办法，实际上就是在调用 这个空对象的方法。因为常用的方法使我们自己重写的，肯定就知道当前操作是什么，新数据是什么等等，就可以做到检测数组的变化了。

既然是这样， 那么我们的`observer`是需要区分当前需要监听的数据是对象还是数组，如果是数组，则改变数组的原型链，只有改变了原型链才能改变调用数组常用的方法如`push`方法时，是调用我们自己的设置的常用的方法的。

```javascript
function Observer(data) {
  const _this = this;
  // 如果数据不存在，或者 data 不是一个对象的话， 则不处理
  if (!data || typeof data !== "object") {
    return;
  }
  if (Array.isArray(data)) {
    data.__proto__ = arrayMethods;
  } else {
    // 监听不是数组的数据
    Object.keys(data).forEach(function (key) {
      defineReactive(data, key, data[key]);
    })
  }
}
```

上面代码的`arrayMethods`就是我们所说的空对象。它里面添加数组常用的方法如：`push`等

```javascript

// 老的数组的原型
const oldArrayProperty = Array.prototype;

// 创建一个新的空对象，但是会继承原数组的一些方法，因为当使用我们没有重写的方法的时候，能使用到
const arrayMethodObj = Object.create(oldArrayProperty);

// 要重写的数组
const arrayMethods =  ["push", "shift", "unshift", "pop","reverse", "sort", "splice"];
arrayMethods.forEach(method => {
  arrayMethodObj[method] = function(...arg) {
     // 执行老的数组的方法，得到结果
    const result = oldArrayProperty[method].apply(this, arg);
    console.log(`数组有变化了，方法：${method}, 新增加的值为: ${inserted}`);
    return result;
  }
})

```

下面我们实现对新增属性的监听。基本的思路：

- 获取到新的属性
- 对新的属性进行监听

```javascript

// 老的数组的原型
const oldArrayProperty = Array.prototype;

// 创建一个新的空对象，但是会继承原数组的一些方法，因为当使用我们没有重写的方法的时候，能使用到
const arrayMethodObj = Object.create(oldArrayProperty);

// 要重写的数组
const arrayMethods =  ["push", "shift", "unshift", "pop","reverse", "sort", "splice"];
arrayMethods.forEach(method => {
  arrayMethodObj[method] = function(...arg) {
     // 执行老的数组的方法，得到结果
    const result = oldArrayProperty[method].apply(this, arg);
    // 进行监听： 1.找到增加的元素； 2.实现监听
    // 1.找到新增加的元素
    let inserted;
    switch(method) {
      case "push":
      case "unshift":
        inserted = arg;
        break
      case "splice":
         // vm.list.splice(3, 0,"哈哈哈", "怎么着") ---> arg = [3, 0, "哈哈哈", "怎么着"]
        inserted = arg.slice(2);
        break
      default:
        break;
    }
    if (inserted) {
      // 2.实现对新增加元素的监听：当然是新增的元素是数组的话才会去监听，observer函数有处理
      observerArray(inserted);
    }
    console.log(`数组有变化了，方法：${method}, 新增加的值为: ${inserted}`);
    // 剩下的就需要通知订阅者去更新视图了
    // ...
    return result;
  }
})

```

上面代码中有一个 `observeArray`方法去监听新增加的数组的元素。我们看看 `observeArray`方法。

```javascript
// 劫持数组元素
function observeArray(items) {
  //  循环监听每一个新增的属性
  for (var i = 0, l = items.length; i < l; i++) {
    observer(items[i]); // 注意这里是observer 不是 Observer
  }
}
```

`observeArray`方法中对`inserted`进行遍历，对每一项进行监听。为什么要遍历呢？因为`inserted`不一定是一个值。也有可能是多个如：[].splice(0,0,"1", "2", "3")；[].push(1,2,3)等。

我们来看 `observer`方法：

```javascript
function observer(value) {
  // 如果数据不存在，或者data 不是一个对象的话， 则不处理
  if (!value || typeof value !== "object") {
    return;
  }
  return new Observer(value);
}
```

这里有很重要的一点：`value`不是一个对象的话， 我们是不做任何处理的。就比如：`  observer(items[i]); // 注意这里是observer 不是 Observer`这一句，这里的`items[i]`有可能就只是一个数据，而不是对象或者数组， 我们就是不处理的。不然跟对数组的所有下表监听的有啥区别。



目前实现对了数组方法的拦截。但是还有一个问题，就是我们在初始化的时候，data可能就是数组，因此要把这个数组也进行监听。

```javascript
function Observer(data) {
  if (Array.isArray(data)) {
    data.__proto__ = arrayMethodObj;
    observerArray(data);
  } else {
    Object.keys(data).forEach((key) => {
      defineObserver(data, key, data[key]);
    })
  }
}
```

最后我们是实现了对数据的监听，不过这里还是有个问题没解决，也就是`Vue2.x`还没有解决的问题：并没有实现对数组的每一项进行监听：如下面这样的就不会被监听到

```javascript
const vm = new MyVue({
  el: "#app",
  data: {
    name: "舒丽琦",
    list: ["1", "2", "3"]
  }
})
vm.list[0] = "我改变了"
```

这是因为我们数据劫持的时候没有对数组的下标进行监听。因为性能的代价太高了。除此之外，改变数组的长度也是无限监听的 `vm.list.length = 9`。

# 结果

最后我们演示一下结果

我们的`html`代码为：

```html
<html>
  <header></header>
  <body>
    <div id="app"></div>
  </body>
  <script src="./js/observer.js"></script>
  <script src="./js/index.js"></script>
  <script src="./js/compile.js"></script>
  <script src="./js/watcher.js"></script>
  <script>
    const vm = new MyVue({
      el: "#app",
      data: {
        name: "舒丽琦",
        list: ["舒", "丽", "琦"]
      }
    })
    // 给数组添加一个元素， 这个元素是一个数组
    vm.list.push(["小小舒", "sha"]);

    // 对新增的元素（新增的是一个数组：["小小舒", "sha"]）再添加元素
    vm.list[3].push("哈哈哈")

    // 添加数组
    vm.list.splice(3, 0,"哈哈哈", "怎么着");

    console.log(vm.list);
  </script>
</html>
```

`observer.js`完整的代码：

```javascript

// 老的数组的原型
const oldArrayProperty = Array.prototype;

// 创建一个新的空对象，但是会继承原数组的一些方法，因为当使用我们没有重写的方法的时候，能使用到
const arrayMethodObj = Object.create(oldArrayProperty);

// 要重写的数组
const arrayMethods =  ["push", "shift", "unshift", "pop","reverse", "sort", "splice"];
arrayMethods.forEach(method => {
  arrayMethodObj[method] = function(...arg) {
     // 执行老的数组的方法，得到结果
    const result = oldArrayProperty[method].apply(this, arg);
    // 进行监听： 1.找到增加的元素； 2.实现监听
    // 1.找到新增加的元素
    let inserted;
    switch(method) {
      case "push":
      case "unshift":
        inserted = arg;
        break
      case "splice":
         // vm.list.splice(3, 0,"哈哈哈", "怎么着") ---> arg = [3, 0, "哈哈哈", "怎么着"]
        inserted = arg.slice(2);
        break
      default:
        break;
    }
    if (inserted) {
      // 2.实现对新增加元素的监听：当然是新增的元素是数组的话才会去监听，observer函数有处理
      observerArray(inserted);
    }
    console.log(`数组有变化了，方法：${method}, 新增加的值为: ${inserted}`);
    // 剩下的就需要通知订阅者去更新视图了
    // ...
    return result;
  }
})



function Observer(data) {
  if (Array.isArray(data)) {
    data.__proto__ = arrayMethodObj;
    observerArray(data);
  } else {
    Object.keys(data).forEach((key) => {
      defineObserver(data, key, data[key]);
    })
  }
}
function observerArray(items) {
  for (let i = 0; i < items.length; i++) {
    observer(items[i]);
  }
}

function observer(value) {
  // 如果数据不存在，或者data 不是一个对象的话， 则不处理
  if (!value || typeof value !== "object") {
    return;
  }
  return new Observer(value);
}

function defineObserver(data, key, value) {
  // 监听子元素
  observer(value);
  const dep = new Dep();
  Object.defineProperty(data, key, {
    get: function() {
      // 把订阅者添加到容器里面，统一管理
      if (Dep.target) {
        dep.addSub(Dep.target)
      }
      return value;
    },
    set: function(newValue) {
      if (value !== newValue) {
        console.log("监听对象属性到变化了，新的值为：", newValue)
        value = newValue;
        // 通知收集的容器的 notify，notify 去更新每一个订阅者的 update 方法去更新视图
        dep.notify();
      }
    }
  })
}

// 管理每一个订阅者的容器:
// 该容器维护一个数组，用来收集订阅者。
// 该容器有一个 notify 方法 去触发订阅者的 update 去更新视图。
function Dep() {
  this.subs = [];
}
Dep.prototype = {
  addSub: function(sub) {
    this.subs.push(sub);
  },
  notify: function() {
    this.subs.forEach((sub) => {
      sub.update();
    })
  }
};
Dep.target = null;
```

结果：

{% asset_img 2.png %}



上面例子的代码： [vue检测数组-重写数组常用的方法](https://github.com/shuliqi/MyVue/tree/array_methids)







