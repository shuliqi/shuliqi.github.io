---
title: Vue 的双向绑定原理及手把手实现
date: 2019-04-20 18:22:44
tags: Vue
categories: Vue

---

最近在学习`Vue`，之前一直对`Vue`的双向数据绑定只算是了解。经过这几天的深入学习。对它的原理有了更加深刻的认识。虽然`Vue`并没有有完全遵循`MVVM模型`， 但是它的设计也是受到了`MVVM模型`的启发。所以它也是能实现双向数据绑定的， 那我们来看`mvvm模型`的双向数据绑定是什么？其实双向数据绑定实现的效果就是指：模型(`model`)`javaScript`中定义的对象，它改变了会同步视图(view)。修改视图()view)也会同步修改数据层；

 <!--more-->

演示如下：

{% asset_img 6.gif %}

# Vue 双向数据绑定的原理



## 数据绑定

从开发者的角度去看，视图到`data`的的改动只需要监听`DOM`的变化再同步赋值给`JavaScript`变量即可；如：`<input>`标签添加`change`或者`input`监听事件并在事件处理函数中给变量赋值。其实这也是`Vue`中`v-model`指令做的很重要事情。另外数据到视图关键是监听数据的变化，再去更新相应的`DOM`。那么监听数据的变化有哪些方式呢？。 

我们知道常见架构模式有`MVC`, `MVP`,`MVVM`模式，目前前端框架基本上都是采用`MVVMM`实现双向数据绑定。`Vue`也不例外。各个框架实现双向数据板绑定的方法有有所不同，目前大概有以为这三种：

- 发布订阅模式
- 数据劫持
- [Angular 的脏查机制](https://www.jianshu.com/p/2b61cd0bcbce)

而`Vue`采用的是**数据劫持**和**发布订阅者模式**相结合的方式来实现双向数据绑定。而**数据劫持**主要是通过`Object.defineProperty`来实现。

## Object.defineProperty

关于Object.defineProperty 可以看我之前写的一篇文章： [Object.defineProperty](https://shuliqi.github.io/2018/02/19/%E6%B7%B1%E5%85%A5%E4%BA%86%E8%A7%A3Object-defineProperty/)。let ，我们主要看它的`get` 和`set`能帮我们实现什么？

我们对一个`Javacript`变量进行设置值和获取值：

```javascript
const obj = {
  labelName: "标签"
}
obj.labelName = "更新标签名字"
obj.labelName;
```

我们对一个`Javacript`变量进行设置值和获取值， 除了设置成功和获取成功， 我们是没办法看到其他的变化了。

```javascript
const obj = {}
let labelName;
Object.defineProperty(obj, "labelName", {
  get: function() {
    console.log("获取标签名字");
    return labelName;
  },
  set: function(newValue) {
    console.log("设置新的标签名");
    labelName = newValue
  }
})
obj.labelName = "标签";
console.log(obj.labelName);
```

具体的效果如下：

{% asset_img 2.gif %}

从上面的例子可以看出，我们在访问`JavaScript`变**量的时候会自动执**行`get`函数。设置值得时候会自动执行`set`函数。



## 思路整理

根据上面的两点我们知道`Vue`是通过数据劫持结合发布订阅模式来实现双向数据绑定，所以实现双向数据绑定就必须实现以下几点：

- 实现一个监听器 `Observer`能够对数据对象的所有属性进行监听；`Observer`监听器里面使用`Object.defineProperty`来监听。当属性有变动拿到最新的值和通知订阅者。
- 实现一个指令解析器`Compile`。对每个元素的节点的指令进行扫描和解析，根据指令模板替换数据，以及绑定相应的更新函数。
- 实现一个订阅者`Watcher`，能够订阅并收到每个属性的变动的通知，执行指令绑定的相应的回调函数，从而更新视图；
- 实现入口函数，整合以上三者。

上面的流程如下图：

{% asset_img 3.png %}



# 监听器Oberver

监听器的作用是去监听数据的每一个属性，使用上面我们讲的`Object.defineProperty`来监听属性的变动。我们需要对`Observer`的数据对象进行递归遍历。使得子属性对象的属性都加上`get` 和 `set`函数。这样就能监听到数据的变化了

```javascript
function Observer(data) {
  // 如果数据不存在，或者data 不是一个对象的话， 则不处理
  if (!data || typeof data !== "object") {
    return;
  }
  Object.keys(data).forEach((key) => {
    defineObserver(data, key, data[key]);
  })
}

function defineObserver(data, key, value) {
  // 监听子元素
  Observer(data[key]);
  Object.defineProperty(data, key, {
    get: function() {
      return value;
    },
    set: function(newValue) {
      if (value !== newValue) {
        console.log("监听到变化了", newValue)
        value = newValue;
      }
    }
  })
}
```

当我们监听的属性发生变化之后我们需要去通知订阅者`Wtcher`去执行更新函数更新视图。这个过程中会有很多的订阅者（一个属性就是一个订阅者`Watcher`）， 所以我们创建一个容器`Dep`去做一个统一的管理。这个容器维护一个数组，用来收集订阅者。当数据变动触发`notify`， 然后容器（订阅器）触发订阅者的`update`方法。最终的的`Observer`如下：

```java
function Observer(data) {
  // 如果数据不存在，或者data 不是一个对象的话， 则不处理
  if (!data || typeof data !== "object") {
    return;
  }
  Object.keys(data).forEach((key) => {
    defineObserver(data, key, data[key]);
  })
}

function defineObserver(data, key, value) {
  // 监听子元素
  Observer(data[key]);
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
        console.log("监听到变化了", newValue)
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

大家可能对下面这段代码可能不是很理解：

```javascript
get: function() {
  // 把订阅者添加到容器里面，统一管理
  if (Dep.target) {
    dep.addSub(Dep.target)
  }
  return value;
}
```

这一段代码的目的是为了后面写的`Watcher`. 到哪里我会解释的。

那么到目前为止，就实现了一个`Observer`了。已经有监听数据变化和通知订阅者的功能了。

# Compile解析器

`Compile`主要做的事情就两点:

- 解析模板的指令，将模板中的变量替换成数据，然后初始化渲染页面。
- 对每个指令对应的节点绑定订阅者（添加更新视图的函数uodate）

如下图：

{% asset_img 4.png %}

因为在解析`DOM`加点的过程中我们会频繁的操作`DOM`所以我们利用好文档片段[DocumentFragment]](https://developer.mozilla.org/zh-CN/docs/Web/API/DocumentFragment) 来帮助我们解析`DOM。

```javascript
function Compile(vm) {
  this.vm = vm;
  this.el = vm.$el;
  this.fragment = null;
  this.init();
}
Compile.prototype = {
  init: function() {
    // 文档片段
    this.fragment = this.nodeFragment(this.el);
    
    // 解析DOM元素
    this.compileNode(this.fragment);
    
    // 解析完成添加到元素中去
    this.el.appendChild(this.fragment); 
  },
  // 获取当前 el 下面的所有元素 放到 文档片段 fragment 里面去
  nodeFragment: function(el) {
    // 创建一个空白的文档片段
    const fragment = document.createDocumentFragment();
    
    // 把页面 el 下面的所有 子节点都放到文档片段里面去
    // appendChild:Node.appendChild() 方法将一个节点附加到指定父节点的子节点列表的末尾处。
    // 如果将被插入的节点已经存在于当前文档的文档树中，那么 appendChild() 只会将它从原先的位置移动到新的位置
    //（不需要事先移除要移动的节点）。
    let child = el.firstChild;
    while(child) {
      fragment.appendChild(child);
      child = el.firstChild;
    }
    return fragment;
  },
  // 解析节点： 也就是替换 {{}}
  compileNode: function(fragment) {
    // Node.childNodes 返回包含指定节点的子节点的集合
    const childNodes = fragment.childNodes;
    [...childNodes].forEach(node => {
      if (this.isElementNode(node)) {
        //如果是元素节点
        this.compile(node);
      } else {
        // 文本元素的内容
        // textContent 属性表示一个节点及其后代的文本内容。
        const text = node.textContent;

        // 匹配插槽如： {{ name }}
        const reg = /\{\{(.*)\}\}/; 

        // test() 方法执行一个检索，用来查看正则表达式与指定的字符串是否匹配。返回 true 或 false。
        if(reg.test(text)) {
          // exec() 方法在一个指定字符串中执行一个搜索匹配。返回一个结果数组或 null
          const prop = reg.exec(text)[1].trim();
          this.compileText(node, prop);
        }
      }
      if (node.childNodes && node.childNodes.length) {
        this.compileNode(node);
      }
    });
  },
  compile: function(node) {
    // Element.attributes 属性返回该元素所有属性节点的一个实时集合。该集合是一个 NamedNodeMap 对象
    let nodeAttrs = node.attributes;
    [...nodeAttrs].forEach((attr) => {
      const name = attr.name;
      if (this.isDirective(name)) {
        if (name === "v-model") {
          const value = attr.value;
          this.compileModel(node, value)
        }
      }
    })
  },

  compileModel: function(node, prop) {
    const val = this.vm.$data[prop];
    // 更新 model 类型的 value 值
    this.updateModel(node, val);
    // 给 input 框添加 input事件

    // 添加订阅者
    new Watcher(this.vm, prop, (newValue) => {
      this.updateModel(node, newValue);
    })

    node.addEventListener('input', (e) => {
      const newValue = e.target.value;
      if (newValue === val) {
        return;
      }
      // 设置新值
      this.vm.$data[prop] = newValue;
    })
  },
  compileText: function(node, prop) {
    const val = this.vm.$data[prop];
    this.updateView(node, val);
    // 添加订阅者
    new Watcher(this.vm, prop, (newValue) => {
      this.updateView(node, newValue);
    })
  },

  updateView: function(node, value) {
    node.textContent = value == 'undefined' ? "" : value;
  },
  updateModel: function(node, value) {
    node.value = typeof value == 'undefined' ? "" : value;
  },

  // 是否是一个指令
  isDirective: function(attr) {
    return attr.indexOf('v-') !== -1;
  },
  isElementNode: function(node) {
    // nodeType 属性返回节点类型。
    // 如果节点是一个元素节点，nodeType 属性返回 1。
    // 如果节点是属性节点, nodeType 属性返回 2。
    // 如果节点是一个文本节点，nodeType 属性返回 3。
    // 如果节点是一个注释节点，nodeType 属性返回 8。

    // 元素节点：构成了DOM的基础。文档结构中，<html>是根元素，代表整个文档，其他的还有<head>,<body>,<p>,<span>等等。元素节点之间可以相互包含(当然遵循一定的规则)
    // 文本节点：包含在元素节点中。
    // 属性节点：元素都可以包含一些属性，属性的作用是对元素做出更具体的描述，比如id,name之类的。
     return node.nodeType === 1;
  }
}
```

在写`Compile`时候对下面这段代码比较疑惑：

```javascript
  nodeFragment: function(el) {
    // 创建一个空白的文档片段
    const fragment = document.createDocumentFragment();
    let child = el.firstChild;
    while(child) {
      fragment.appendChild(child);
      child = el.firstChild;
    }
    return fragment;
  },
```

`while`为啥一直是` child = el.firstChild;`呢？那岂不是一直都一个元素节点？，其实不是的，这是因为：

> appendChild:Node.appendChild() 方法将一个节点附加到指定父节点的子节点列表的末尾处。如果将被插入的节点已经存在于当前文档的文档树中，那么 appendChild() 只会将它从原先的位置移动到新的位置.

哈哈， 豁然开朗！！！

这里一个`Compile`就完成了。

# Watcher 订阅者

`Watcher` 订阅器主要做的事情就两件:

- 在自身实例化的时候网订阅器（`Dep` 容器）添加自己。
- 必须有一个`uodate`方法，目的是为了调用每个订阅者的更新视图的函数（即：接收到通知，执行更新函数）。

```javascript
// 订阅者
function Watcher(vm, prop, callback) {
  this.vm = vm;
  this.prop = prop;
  this.callback = callback;
  this.value = this.get();
}
Watcher.prototype = {
  update: function() {
    const value = this.vm.$data[this.prop];
    const oldValue = this.value;
    if (value !== oldValue) {
      this.callback(value);
    }
  },
  get: function() {
    Dep.target = this;
    // 这一步很关键：因为属性被监听，这一步会执行监听器的 get 方法
    const value = this.vm.$data[this.prop];
    Dep.target = null; // 清空订阅器，因为上一步订阅器已经被加上了
    return value;
  }
}
```

这里的这段代码很关键：

```javascript
 const value = this.vm.$data[this.prop];
```

注意这里是获取`属性`。在实现`Observer`监听了所有属性的获取，监听属性的时候我们在`get`方法有如下的代码：

```javascript
get: function() {
  // 把订阅者添加到容器里面，统一管理
  if (Dep.target) {
    dep.addSub(Dep.target)
  }
  return value;
}
```

这里这么写的时候就是跟`Watcher`的` const value = this.vm.$data[this.prop];`相呼应的。我们的订阅者就是在这时候添加到容器`Dep`里面去的。

# 实现的入口函数

上面三个重要的点实现完了。最后就只需要一个入口函数来整合了。

```javascript
function MyVue(options) {
  this.$el = document.querySelector(options.el);
  this.$data = options.data;
  // 监听数据
  new Observer(this.$data);
  // 解析指令
  new Compile(this);
}
```

我们尝试去修改数据，也完全没问题；

{% asset_img 5.gif %}

但是有个问题就是我们修改数据时时通过 `vm.$data.name` 去修改数据，而不是想 Vue 中直接用 `vm.name` 就可以去修改，那这个是怎么做到的呢？其实很简单，Vue 做了一步数据代理操作。最新代码如下：

```javascript
// index.js ---> 入口函数
function MyVue(options) {
  this.$el = document.querySelector(options.el);
  this.$data = options.data;
  //数据代理
  Object.keys(this.$data).forEach(key => {
    this.proxyData(key);
  });
  this.init();
}
MyVue.prototype = {
  init: function() {
    new Observer(this.$data);
    new Compile(this);
  },
  proxyData: function(key) {
    // 代理第一层即可
    Object.defineProperty(this, key, {
      get: function () {
        return this.$data[key]
      },
      set: function (value) {
        this.$data[key] = value;
      }
    });
  }
}
```

实现效果如下：

{% asset_img 6.gif %}

# 最终效果

最后实现的相关如下：

{% asset_img 7.gif %}

如果需要代码的可点击 [MyVue](https://github.com/shuliqi/MyVue/tree/master)。最后一点感触， 原理性的东西看懂了原理，还是需要手写一遍， 其中会发现很多意想不到的情况。收获颇多。























