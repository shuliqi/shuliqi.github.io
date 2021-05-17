---
title: Vue2.x的生命周期详解
date: 2018-04-17 19:31:36
tags: Vue
categories: Vue
---

使用`Vue`开发已经有一段时间了。但发现自己对`vue`的理解还是不深刻。于是想着从观看`Vue`文档开始。在阅读文档之前，觉得需要先了解整个`Vue`的生命周期，清楚的认识到`Vue`在每个阶段的钩子函数没这样才能更好的让我们去使用`Vue`。

我们先来看一张图，上面解释每个步骤是做什么的。

{% asset_img 3.png %}

每个`Vue`实例在被创建之前都要经历过一系列的初始化过程，这个过程就是`Vue`的生命周期。浅显的来说，生命周期的钩子函数就是回调函数，在不同的阶段有不同的回调函数供用户处理自定义的事件。也可以说生命周期是一套流程，而这套流程里面的方法会有先后顺序的执行（调用就执行，不调用就不执行）。

# 生命周期钩子

从上面的图中会看到生个`vue`的生命周期钩子：

- beforeCreate

- created

- beforeMount

- mounted

- beforeUpdate

- updated

- beforeDestroy

- destroyed

  

## beforeCreate 之前

在`beforeCreate`生命周期之前。首先是使用`new Vue`来开始创建一个`Vue`实例、接下来会初始化这个实例的生命周期事件。如下图：

{% asset_img 1.png %}

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>vue2.x生命周期学习</title>
  <script src="https://cdn.bootcss.com/vue/2.4.2/vue.js"></script>
</head>
<body>
  <div id="app"> <h1>{{message}}</h1></div></body>
 <script>
   var vm = new Vue({
     el: '#app',
      data: {
        message: "舒丽琦"
      },
      methods: {
        getMessage() { 
          return `方法返回：${this.message}`;
        }
    	},
      beforeCreate () {
        console.log("------beforeCreate------");
        console.log("el:", this.$el);
        console.log("data:", this.$data);
        console.log("methods:", this.getMessage);
      },
   })
</script>
</html>
```

打印结果：

```
------beforeCreate------
el: undefined
data: undefined
methods: undefined
```



## beforeCreate 和 created 之间

在这个生命周期之前，会初始化当前实例上的`data` 和`methods`。说明`created`的时候数据已经绑定，是可以访问到了。在这里可以做`ajax`请求了。

**注意：**这里还没有 `el` 选项。

{% asset_img 2.png %}

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>vue2.x生命周期学习</title>
  <script src="https://cdn.bootcss.com/vue/2.4.2/vue.js"></script>
</head>
<body>
  <div id="app"> <h1>{{message}}</h1></div></body>
 <script>
   var vm = new Vue({
    	el: '#app',
      data: {
        message: "舒丽琦"
      },
      methods: {
        getMessage() { 
          return `方法返回：${this.message}`;
        }
    	},
      beforeCreate () {
        console.log("------beforeCreate------");
        console.log("el:", this.$el);
        console.log("data:", this.$data);
        console.log("methods:", this.getMessage);
    	},
      created () {
      	console.log("------created------");
        console.log("el:", this.$el);
        console.log("data:", this.$data.message);
        console.log("methods:", this.getMessage());
      }
   })
</script>
</html>
```

结果打印：

```
------beforeCreate------
el: undefined
data: undefined
methods: undefined
------created------
el: undefined
data: 舒丽琦
methods: 方法返回：舒丽琦
```

可以看出，`created`可以获取`data`和`methods`

## created 和 beforeMount 之间的生命周期

{% asset_img 5.png %}

这阶段里面首先会判断是否有`el`选项。如果有的话，那么继续往下编译，如果没有`el`选项。则停止编辑。也就意味停止了生命周期。直到该`Vue`实例上调用了`Vm.$mount(el)`。

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>vue2.x生命周期学习</title>
  <script src="https://cdn.bootcss.com/vue/2.4.2/vue.js"></script>
</head>
<body>
  <div id="app"> <h1>{{message}}</h1></div></body>
 <script>
   var vm = new Vue({
    	// el: '#app', // 被注释掉了
      data: {
        message: "舒丽琦"
      },
      methods: {
        getMessage() { 
          return `方法返回：${this.message}`;
        }
    	},
      created() {
        console.log("------created--------")
      },
      beforeMount() {
        console.log("------beforeMount------");
        console.log("el:", this.$el);
        console.log("data:", this.$data.message)
        console.log("methods:", this.getMessage());
      },
   })
</script>
</html>
```

结果打印：

```javascript
------created--------
```

手动调用实例的`Vm。$mount(#app)`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>vue2.x生命周期学习</title>
  <script src="https://cdn.bootcss.com/vue/2.4.2/vue.js"></script>
</head>
<body>
  <div id="app"> <h1>{{message}}</h1></div></body>
 <script>
   var vm = new Vue({
    	// el: '#app', // 被注释掉了
      data: {
        message: "舒丽琦"
      },
      methods: {
        getMessage() { 
          return `方法返回：${this.message}`;
        }
    	},
      created() {
        console.log("------created--------")
      },
      beforeMount() {
        console.log("------beforeMount------");
        console.log("el:", this.$el);
        console.log("data:", this.$data.message)
        console.log("methods:", this.getMessage());
      },
   })
   vm.$mount("#app"); // 手动调用了 Vm.$mount(el)
</script>
</html>
```

结果打印：

{% asset_img 4.png %}

然后会判断是否有`template`。有没有`template`对生命周期没有影响。

- 如果有`template`则直接作为模板编译成render函数；
- 如果没有`template`则直接将外部的`html`作为模板编译；

**注意：**`template`模板的优先级要高于外部`html`的优先级。

如下的例子：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>vue2.x生命周期学习</title>
  <script src="https://cdn.bootcss.com/vue/2.4.2/vue.js"></script>
</head>
<body>
  <div id="app">
    <span>这是外部的html模板:{{message}}</span>
  </div>
</body>
 <script>
   var vm = new Vue({
      el: '#app',
    	data: {
        message: "舒丽琦"
      },
      template: "<span>这是内部的模板：{{message}}</span>",
      methods: {
        getMessage() { 
          return `方法返回：${this.message}`;
        }
      }
   })
</script>
</html>
```

打开之后，我们看到的页面文案是："这是内部的模板：舒丽琦"

如果我们把`template` 删除掉那么就会显示："这是外部的html模板:舒丽琦"6

## beforeMount 和 mounted之前的生命周期

{% asset_img 5.png %}

这期间`Vue`实例对象添加`$el`。把内存中渲染好的html 替换到页面上。覆盖`$el`指定的页面

