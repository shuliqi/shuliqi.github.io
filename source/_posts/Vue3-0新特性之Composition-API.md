---
title: Vue3.0新特性之Composition API
date: 2021-05-08 17:55:25
tags: Vue
categories: Vue
---

`Vue3.0`发布了很多新的特性和一些语法上的变更。其中	`Composition API`是`Vue3.0`版本中主要特色语法，这是一个全新的逻辑重用和代码组织的方法。这边文章就带你看看为啥会有`Composition API`以及如何使用`Composition API`

 <!--more-->

# Option API 和 Composition API 的比较

我们知道`Vue2.0`（选项）所有数据都定义在`data`中，方法定义在`methods`中。所以给组件添加逻辑， 我们可能需要填充（选项）属性`data`, `methods`,`computed`等。但是`Vue3.0`我们可以不这么写了。具体怎么写， 我们先看看`Vue2.0`的写法有什么缺陷。

## Option API

使用`Vue2.0`的`option API`实现如下功能：

<iframe height="691" style="width: 100%;" scrolling="no" title="option api" src="https://codepen.io/shuliqi/embed/rNjPvPg?height=691&theme-id=dark&default-tab=js,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/rNjPvPg'>option api</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

我们实现一个加减法，之后数值显示出来，就需要分别在`data`,`computed`,`methods`添加代码，如果还有其他功能，可能还得在生命周期选项等添加代码。那如果需要添加其他的需求。我们的代码结构可能就变成这样了：

{% asset_img 1.png %}

## Composition API

在`Vue3.0`中， 我们可以使用`Composition API`的方式来实现：

<iframe height="717" style="width: 100%;" scrolling="no" title="vue3.0" src="https://codepen.io/shuliqi/embed/yLgwLwX?height=717&theme-id=dark&default-tab=js,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/yLgwLwX'>vue3.0</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

代码按照逻辑（`Composition API`的方式来实现）来分割，上面的图变成下面的图（相同业务逻辑的代码颜色），是不是就清晰了？

{% asset_img 2.png %}

可以看的出来： `Option API`相同业务的代码分散在各处，这样后期维护起来就很麻烦。而`Composition API`就解决了这个问题。那么下面就来讲解`Composition API`怎么使用。

# Composition API 的入口及API

关于`Composition API`有哪些`API`呢？ 先总结一下：

{% asset_img 3.png %}

下面根据这些知识点分别来讲解。

## setup

`setup`功能是新的组件选项，是`Composition API`使用的入口。

### 执行的的时机

`setup`是在创建`vue`组件实例并完成`props`的初始化之后执行（在`beforeCreate`钩子之前执行）。这就直接限制了在`setup`中无法使用其他的选项(`option`)中的数据；如：`data`,`methods`，`computed`等。 但是其他的选项(`option`)可以使用`setup`中返回的变量。

> 由于在执行 `setup`函数的时候，还没有执行 `Created `生命周期方法，所以在 `setup` 函数中，无法使用 `data `和 `methods` 的变量和方法

```javascript
export default {
  beforeCreate () {
    console.log("------beforeCreate------")
  },
  created () {
    console.log("------created------")
  },
  setup () {
    console.log("------setup------");
  }
}
```

打印的结果为：

{% asset_img 8.png %}

### setup中的上下文

`setup`中是没有`this`上下文的.为什么呢？ `javascript`函数中都是应该有`this`。但是由于 **执行的的时机**的原因，`setup`中的`this` 与`Vue2.x`中的`this` 已经不是一个东西了。所以为了防止错误的使用，直接将`this`改成了 `undefined`。

> 由于我们不能在 `setup`函数中使用` data `和 `methods`，所以 Vue 为了避免我们错误的使用，直接将 `setup`函数中的`this`修改成了 `undefined`

```javascript
export default {
  setup () {
    console.log("this", this); // undefined
  }
}
```

### 与模板的使用

`setup`如果返回的是一个对象的话，那么这个对象的所有属性会合并到`template`的渲染上下文中（也就是说可以在`template`中使用`setup`的返回的对象的属性）。

<iframe height="563" style="width: 100%;" scrolling="no" title="setup 与模板一起使用" src="https://codepen.io/shuliqi/embed/yLgwJZb?height=563&theme-id=dark&default-tab=js,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/yLgwJZb'>setup 与模板一起使用</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

在`template`中，`vue`已经帮我们自动获取`value`属性， 所有我们只需要`{{ people.name }}`,`{{ people.age }}`,`{{ sex }}`

### 参数

`setup`是一个函数，它接受两个参数：`props`，`context`

下面我们一个具体的例子项目来讲解。假如我们有这样的项目：

{% asset_img 4.png %}

#### props

`props`是父组件传给子组件或者`vue-router`传给页面的参数。`setup`中的`props` 是响应式，当传入新的`props`，就会被更新。

```javascript
export default  {
  props: {
    name: String,
    age: Number
  },
  setup(props) {
    console.log(props): // { name, age }
  } 
}
```

> 注意： 因为`props`是响应式的， 所以不能使用`ES6`解构，如果这样做将会失去响应性。

如果需要`ES6`解构并且需要数据的响应性的的话， 可以使用`toRefs`来完成。下面也会讲到。

```javascript
import { toRefs } from "vue"
export default  {
  props: {
    name: String,
    age: Number
  },
  setup(props) {
    const { name, age } = toRefs(props);
  } 
}
```

#### context

`context`是一个`javascript`对象，它暴露了3 个`property`：`attrs`、`slot` 和`emit`。分别对应`Vue2.x`的`$attr`属性、`slot`插槽 和`$emit`。

```javascript
export default  {
  emits: ["updateName"],
  setup(props, { emit }) {
   emit("updateName", "shuliqi");
  } 
}
```

`props` 和 `context`具体的代码如下：

{% asset_img 5.png %}

## ref

`ref` 函数接受一个值，用于初始化的值，然后返回一个响应式且可修改的`ref`对象。该对象有一个`value`属性。`valus`保存`ref`对象的值。所以修改变量的话需要修改变量的`value`值。

```javascript

import { ref }  from "vue";
export default {
  setup() {
    // ref 函数返回一个响应式
    const sex = ref("女");    // { value: "女"}
    console.log("sex", sex); 

    const isOk = ref(false);  // { value:  false }
    console.log("isOk", isOk); 

    const tag = ref(null);    // { value: null }
    console.log("tag", tag); 

    const people = ref({      // { value: { name: "shuliqi", age: 12 }}
      name:  "shuliqi",
      age: 12
    });
    console.log("people", people); 
    setTimeout(() => {
        // 修改变量需要修改对象的value值
        people.value.name = "zhangdada";
        console.log("更新后的people", people)
    }, 1000)
  }
}
```

我们看看这段代码打印的结果:

{% asset_img 6.png %}

说明:

- 使用`ref`初始化的变量都是一个对象（`ref` 函数接受一个值，用于初始化的值，然后返回一个响应式且可修改的`ref`对象）。`valus`保存`ref`对象的值。
- 在`setTimeout`时候， 想要修改`people`这个响应式对象的值，则需要通过赋值操作` people.value.name = "zhangdada"`来实现。

再继续看视图页面

<iframe height="643" style="width: 100%;" scrolling="no" title="ref" src="https://codepen.io/shuliqi/embed/yLgwJZb?height=643&theme-id=dark&default-tab=js,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/yLgwJZb'>ref</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>
## reactive

`reactive` 和`ref` 很相似。 也是一个函数，但是只接受一个对象。并返回一个对这个对象的响应式代理

```javascript
import { reactive, toRefs }  from "vue";
export default {
  setup() {
    // 使用 reactive 初始化一个变量
    const state = reactive({
      name: "shuliqi",
      age: 12,
      sex: "女"
    }) 
    setTimeout(() => {
        // reactive 中的变量 的取值和赋值不需要 取其 value 属性
        state.name = "更改之后的名字";
        console.log("更新后的state的name值", state.name)
    }, 2000);

    return  {
      // ...state  会失去响应性
      ...toRefs(state) // 会保留响应性
    }
  }
```

我们看打印的结果：

{% asset_img 7.png %}

再看看`template`中的使用：

<iframe height="732" style="width: 100%;" scrolling="no" title="reactive" src="https://codepen.io/shuliqi/embed/oNBVaPZ?height=732&theme-id=dark&default-tab=js,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shuliqi/pen/oNBVaPZ'>reactive</a> by shuliqi
  (<a href='https://codepen.io/shuliqi'>@shuliqi</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

最后可得出结论：

- `reactive`函数返回一个对这个对象的响应式代理
- `reactive` 可以将零散的变量聚焦在一个对象
- `reactive`中的变量的取值和赋值不需要取其 `value`值

**注意的点：**使用`reactive`时记得使用`toRefs`保证`reactive`对象属性保持响应性。

## isRef

`isRef`用于判断变量是为`ref`对象

```javascript
import { ref, reactive, isRef } from "vue";
export default {
  setup (){
    const name = ref("shuliqi");
    const people = reactive({
      age: 12,
      sex: "女"
    });
    console.log(isRef(name)); // true
    console.log(isRef(name) ? name.value : name); // shuliqi
    console.log(isRef(people)) // false
    console.log(isRef(people) ? people.value.age : people.age) // 12
  }
}
```

## toRefs

`toRefs` 用于将一个`reactive`对象转化为属性为`ref`对象的普通对象。

```javascript
export default {
  setup (){
    const people = reactive({
      age: 12,
      sex: "女"
    });
    const { age, sex } = toRefs(people);
    console.log(isRef(age), isRef(sex)) // true true
    console.log(age.value, sex.value) // 12 shuliqi
  }
}
```

## watch

`watch`与选项式`API`（this.$watch/`watch`选项）完全等效的。`watch`需要监听特定的`data`数据源，并且在单独的回调函数中副作用。默认的情况下，是惰性的，也就是说回调函数仅在监听源数据发生变更时才会回调。

语法：

```javascript
watch(source, callback, [options])
```

参数说明：

- `source:`要监听的响应式变量。支持`String`,`Object`,`Function`, `Array`。
- `callback`： 要执行的回调函数。回调函数的第一个参数是监听的数据更新后的值， 第二个参数之前监听的数据之前得值。
- `options`: 支持deep、immediate 和 flush 选项。
  - 当我们监听复杂的嵌套数据对象的时候，需要传入第三个参数的`deep:true`。这个参数的意思是进行深拷贝，目的是为了真正的监听复杂数据对象
  - 希望`watch` 不是惰性的（立即执行回调函数）可以设置`immediate: true`

###  监听`ref`定义的数据

```javascript
import { ref, watch } from "vue";
export default {
  setup () {
    const name = ref("shuliqi");
    setTimeout(() => {
      name.value = "张大大";
    }, 1000);
    watch(name, (newName,oldName) => {
      console.log("发生了变化：", "newName:", newName, "oldName:", oldName )
    })
  }
}

// 发生了变化： newName: 张大大 oldName: shuliqi
```

### 监听`reactive`定义的数据

```javascript

import { watch, reactive} from "vue";
export default {
  setup () {
    // 监听`reactive`定义的数据
    const  obj = reactive({
      name: "shuliqi",
      age: 12
    })
    setTimeout(() => {
      obj.name = "张大大"
    }, 500)
    watch(() => obj.name, (newName,oldName) => {
      console.log("发生了变化：", "newName:", newName, "oldName:", oldName)
    })

  }
}
// 发生了变化： newName: 张大大 oldName: shuliqi
```

### 监听多个数据

```javascript
  import { ref, watch } from "vue";
  export default {
    setup () {
      // 监听多个数据源
      const age = ref(12);
      const sex = ref("女");
      setTimeout(() => {
        sex.value = "男";
      })
      watch([age, sex], (newName,oldName) => {
        console.log("发生了变化：", "newName", newName, "oldName", oldName)
      })
    }
  }
```

### 监听复杂的嵌套数据

```javascript
  import { watch, reactive} from "vue";
  export default {
    setup () {
      // 监听复杂的嵌套数据
      const state = reactive({
        total: 12,
        data: {
          titile: "标题",
          result: {
            nane: "shuliqi",
            age: 12
          }
        }
      });
      setTimeout(() => {
        state.data.result.name = "张大大";
      }, 500)
      watch(() => state.data, (newName,oldName) => {
        console.log("发生了变化：", "newName:", newName, "oldName:", oldName)
      },  {
        deep: true
      })
    }
  }
```

注意：当我们监听复杂的嵌套数据对象的时候，需要传入第三个参数的`deep:true`。这个参数的意思是进行深拷贝，目的是为了真正的监听复杂数据对象。

### 设置`watch`为立即执行

设置`watch`的第三个参数：`immediate： true`。 则`watch`不是惰性的，即立即执行回调函数。

```javascript
  import { watch, ref } from "vue"; 
  export default {
    setup () {
      // 设置`watch`为立即执行
      const  sex = ref("女");
      watch(sex, (newName, oldName) => {
        console.log("我不是惰性的：", "newName:", newName, "oldName:", oldName)
      }, { immediate: true })
    }

  }
// 我不是惰性的： newName: 女 oldName: undefined
```

### 停止监听

组件中创建的`watch`监听，会在组件被销毁的时候自动停止，如果在组件销毁之前我们想停止某个监听，可以调用`watch()`函数的返回值。

```javascript
import { watch, ref } from "vue";
export default {
  setup () {
    // 停止监听示例
    const age = ref(12);
    const stop = watch(age, (newName, oldName) => {
       console.log("我不是惰性的：", "newName:", newName, "oldName:", oldName)
    });
    // 停止监听
    stop();
    setTimeout(() => {
      age.value = 27;
    }, 3000)
  }
}
```

结果： 不会监听到变化了

### 不能监听非响应式的值

```javascript
// 不能监听非响应式的值
let age = 0;
setTimeout(() => {
  age++;
  console.log("改变值", age);

}, 1000);
watch(() => age, () => {
  console.log("asdas");
})
// 改变值 1
```

## watchEffect

`watchEffect`和`watch`类似，不过`watchEffect`函数会自动收集依赖。 只需要指定一个回调函数。在组件初始化的时候，会先执行一次手机依赖。然后当收集到的依赖中数据发生变化时，会再次执行回调函数。有以下特点：

- `watchEffect`不需要手动传入依赖（不需要手动传入需要监听的值）；
- `watchEffect`会先执行一次用来自动收集依赖;
- `watchEffect`无法获取到变化前的值，只能获取到变化后的值

```javascript
import { ref, watchEffect } from "vue";
export default {
  setup () {
    const name = ref("shuliqi");
    const age = ref(0);
    setTimeout(() => {
      name.value = "张大大";
      age.value = 20;
    }, 1000);
    watchEffect(() => {
      console.log("name值：", name.value, "age值：", age.value);
    });

  }
}
// name值： shuliqi age值： 0
// watchEffect.vue?25bf:15 name值： 张大大 age值： 20
```

结果可以看出来：组件初始化的时候先执行一次回调函数，收集依赖。1秒之后，依赖发生变化。回调函数会再次被调用。

### 停止监听

跟`watch`的停止监听一样。组件中创建的`watchEffect`监听，会在组件被销毁的时候自动停止，如果在组件销毁之前我们想停止某个监听，可以调用`watchEffect()`函数的返回

```javascript
import { ref, watchEffect } from "vue";
export default {
  setup () {
    const name = ref("shuliqi");
    const age = ref(0);
    setTimeout(() => {
      name.value = "张大大";
      age.value = 20;
    }, 1000);
    const stop = watchEffect(() => {
      console.log("name值：", name.value, "age值：", age.value);
    });
    // 停止监听
    stop();
  }
}
// name值： shuliqi age值： 0
```

回调函数只会在组件初始化的时候回调一次。因为停止了监听`stop();`所以依赖发生了变化也不会再触发回调了。

### 不能监听非响应式的值

```javascript
{
  // 不能监听非响应式的值
  let age = 0;
  setTimeout(() => {
    age++;
    console.log("改变值", age);

  }, 1000);
  watchEffect(() => age, () => {
    console.log("asdas");
  })
}
// 改变值 1
```

## computed

`computed`函数与`Vue.x`的中的`computed`功能一样。`computed`接受一个函数并返回一个`value`值为`getter`返回值不可改变的响应式`ref`对象。

```javascript
import { ref, computed, isRef } from "vue";
export default {
  setup () {
    const age = ref(0);
    // computed函数的返回值是响应式的ref对象
    const myComputedAge = computed(() => age.value + 1);
    console.log(isRef(myComputedAge), myComputedAge.value); // true 1

    // omputed函数的返回值是不可改变的
    myComputedAge.value = 3; //  Write operation failed: computed value is readonly
  }
}
```

##  readonly

`readonly`获取一个对象（响应式/纯对象）并返回原始代理的只读代理，只读代理是深层次的：访问的任何嵌套的`property`也是只读的。返回的代理对象不可改变。但是传入的原始对象改变时，返回的代理对象也会相应的改变。如果传入的是`ref`对象或者`reactive`对象。那么返回的代理对象也是响应式的。

### readonly 响应式对象

**ref对象**

```javascript
import { ref, readonly } from "vue";
  export default {
    setup () {
      const name = ref("shuliqi");
      const readonlyName = readonly(name);
      console.log("读取只读代理的值：",readonlyName.value );  // 读取只读代理的值: shuliqi
      // 尝试修改会直接警告
      readonlyName.value = "张大大"; // Set operation on key "value" failed: target is readonly.
      setTimeout(() => {
        name.value = "小小舒";
        console.log("原始ref对象改变，只读代理的值也会变：",readonlyName.value); // 原始ref对象改变，只读代理的值也会变：小小舒
      }, 3000)
    }
  }
```

**reactive对象**

```javascript
import { readonly, reactive } from "vue";
  export default {
    setup () {
      const state  = reactive({
        title: "标题",
        subTitle: "二级标题"
      })
      const readonlyState = readonly(state);
      console.log("读取只读代理的值(reactive)：",readonlyState.title ); // 读取只读代理的值(reactive)：标题
      // 尝试修改会直接警告
      readonlyState.title = "修改标题"; // Set operation on key "title" failed: target is readonly

      setTimeout(() => {
        state.title = "标题被修改了";
        console.log("原始reactive对象改变，只读代理的值也会变(reactive)：",readonlyState.title); 
        // 原始reactive对象改变，只读代理的值也会变(reactive)：标题被修改了
      }, 3000)
    }
  }
```

**重要：**`readonly` 响应式对象.只读代理是具有响应性的:

{% asset_img 9.gif %}

###  readonly 普调对象

```javascript
import { readonly } from "vue";
  export default {
    setup () {
      const obj = {
        size: 12,
        isNew: true,
      }
      const readonlyObj = readonly(obj);
      console.log("读取readonly一个普调对象的只读代理的值：",readonlyObj );
      // 读取readonly一个普调对象的只读代理的值:{ size: 12, isNew: true }
      
      
      //  尝试修改 readonlyObj, 直接警告
      readonlyObj.size = 100;  // Set operation on key "size" failed: target is readonly
      //  修改原始数据
      obj.size = 3000;
      console.log("原始数据obj改变， readonlyObj也会改变",  readonlyObj); 
      // 原始数据obj改变， readonlyObj也会改变 { size: 3000, isNew: true }
    }
  }
```

 ###  readonly 一个非对象

```javascript
import { readonly } from "vue";
  export default {
    setup () {
      // readonly 一个非对象: 就不具有只读特性了
      const str = "oldDate";
      let readonlyStr = readonly(str);
      console.log("readonlyStr:",readonlyStr)
      // 可以直接修改，说明不具有只读特性
      readonlyStr = "newDate";
      console.log("newReadonlyStr:",readonlyStr)
    }
  }
```

## 生命周期

`Composition API`有人提供了组件生命周期的钩子回调。我们先看`vue3.0`生命周期图：

{% asset_img 10.png %}

我们可以看出，`vue3.0`增加了`setup`,然后为了更加语义化将`beforeDestroy`改成了`beforeUnmount`；`destotredy`改成了`unmounted`。其他`Vue2`中的生命周期仍然保留。这个图没有显示全部的声声明周期。我们看看全部的生命周期的钩子图示。

{% asset_img 11.png %}

`beforeCreate`和`created`被`setup`替换了（但是Vue3中你仍然可以使用， 因为Vue3是向下兼容的， 也就是你实际使用的是vue2的）。钩子命名都增加了`on`; Vue3.x还新增用于调试的钩子函数数`onRenderTriggered`和`onRenderTricked`

```javascript

import { 
  onBeforeMount,
  onMounted,
  onBeforeUpdate,
  onUpdated,
  onBeforeUnmount,
  onUnmounted,
  onErrorCaptured, 
  onRenderTracked,
  onRenderTriggered
}  from "vue";
export default {
  beforeCreate () {
    console.log("beforecreate")
  },
  created () {
    console.log("created")
  },
  setup () {
    onBeforeMount(() => {
      console.log("Composition API: onBeforeMount")
    });

    onMounted(() => {
      console.log("Composition API: onMounted")
    });

    onBeforeUpdate(() => {
      console.log("Composition API: onBeforeUpdate")
    })
    // ...

  },
  // ...
}

```

# 最后

当然关于`Composition	API`的还有很多的钩子，具体可以看中文官网 [组合式 API？](https://vue3js.cn/docs/zh/guide/composition-api-introduction.html) 英文官网：[Composition API?](https://juejin.cn/post/6844904066103902215#heading-15)

最后关于以上的例子的`github`链接: [vue3.0--Composition-API](https://github.com/shuliqi/vue3.0--Composition-API/tree/master)

通过修改`src/main.js`中的不同的模板来看不同的`API 示例`。

{% asset_img 12.png %}























































