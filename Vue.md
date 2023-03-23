## MVVM的理解

-   Model：模型层，负责处理业务逻辑以及和服务器端进行交互
-   View：视图层：负责将数据模型转化为 UI 展示出来，可以简单的理解为 HTML 页面
-   ViewModel：视图模型层，用来连接 Model 和 View，是 Model 和 View 之间的通信桥梁

## 什么是 SPA？如何进行 seo 优化？

### SPA 单页面应用

只有一个页面的应用，即仅在 web 页面初始化时加载响应的 html、css、js 文件

#### 优点：

-   用户体验好，内容的改变不需要重新加载，避免了不必要的跳转和重复渲染，对**服务器压力小**
-   前后端分离
-   完全的前端组件化，代码组织方式更加规范，便于修改

#### 缺点：

-   首次加载需要加载大量的静态资源
-   不利于 seo 优化，由于所有的内容都在一个页面中动态替换显示，所以在 SEO 上其有着天然的弱势
-   前进后退路由管理：由于单页应用在一个页面中显示所有的内容，所以不能使用浏览器的前进后退功能，所有的页面切换需要自己建立堆栈管理；

**多页面：包含多个页面，跳转时全部刷新**

-   **多页应用的首屏时间快**：当访问网页的时候，服务器返回一个新的 html 文档，只经过了一个 http 请求
-   **搜索引擎优化效果好**： 搜索引擎在做网页排名的时候，要根据网页内容才能给网页权重，来进行网页的排名。搜索引擎是可以识别 html 内容的，而我们每个页面所有的内容都放在 Html 中，所以这种多页应用，seo 排名效果好。
-   切换慢：每次跳转都需要发出 http 请求

### 单页面应用如何 SEO 优化

**SEO:**搜索引擎优化

1. 服务端渲染 **SSR** 例如（nuxt.js)

2. 静态化

    1. 一种是通过程序将动态页面抓取并保存为静态页面，这样的页面实际存在于服务器的硬盘中
    2. 通过 web 服务器的 url rewrite 方式。通过 web 服务器内部模块按一定的规则而将外部的 url 请求转化为内部的文件地址

3. 使用 phantonjs 针对爬虫处理。原理是通过`Nginx`配置，判断访问来源是否为爬虫，如果是则搜索引擎的爬虫请求会转发到一个`node server`，再通过`PhantomJS`来解析完整的`HTML`，返回给爬虫。下面是大致流程图

    ![img](https://static.vue-js.com/25be6630-3ac7-11eb-ab90-d9ae814b240d.png)

## Vue 模板编译原理

vue 模板编译也就是将`<template> </template>` 模板编译生成渲染函数

1. **解析器：**将模板字符串 编译生成 element ASTs（抽象语法树）

2. **优化器：**对AST进行静态节点标记，主要是用于虚拟DOM的渲染优化

    **优化：**

   - 生成虚拟dom的过程中，如果发现一个节点是静态子树，除了首次渲染外不会生成新的子节点树，而是拷贝已存在的静态子树；
   - 比对虚拟dom的过程中，如果发现当前节点是静态子树，则直接跳过，不需要进行比对。

3. **代码生成器： **使用 element ASTs 生成 render 函数代码

<img src=".\static\template_render.webp" style="zoom:50%;" />

## Vue2响应式原理

**通过数据的改变去驱动dom视图的改变**

**缺陷：**

vue2 中**不会触发试图更新**：

1. 对象属性的动态添加和删除
2. 直接更改数组下标来修改数组的值

**注意：**

-   添加对象属性或者删除属性时，使用 Vue.set( ) 方法
-   数组应使用 push pop shift unshift splice reserve sort

```js
//更新视图
function updateView() {
    console.log('试图进行更新');
}

function observe(target) {
    if (typeof target !== 'object' || target === null) {
        // 不是数组或者对象不适合监听
        return target;
    }
    for (let key in target) {
        defineReactive(target, key, target[key]);
    }
}

function defineReactive(target, key, value) {
    //递归调用，实现对对象的深度监听
    observe(value);
    Object.defineProperty(target, key, {
        get() {
            return value;
        },
        set(newVal) {
            //实现了vue.set( )方法，使得对象的添加删除也能被监听到
            observe(newVal);
            // 使用了闭包 value ，若更新的值和原来的值相同则不会赋值
            if (newVal !== value) {
                value = newVal;
                updateView();
            }
        },
    });
}
```

**源码过程：**

[https://juejin.cn/post/6844903597986037768]()

-   在`init`数据初始化的时候，对象内部通过 `defineReactive` 方法，使用 `Object.defineProperty` 将属性进行劫持（这时候只会劫持已经存在的属性）。如果数据是数组类型， Vue2 中是通过重写数组方法来实现。多层对象是通过递归来实现劫持的。
-   在初始化流程中的编译阶段，当`render function` 被渲染的时候，会读取 Vue 实例中和视图相关的响应式数据，此时会触发 `getter` 函数进行 **依赖收集**（将观察者`Watcher`对象存放到当前闭包的订阅者`Dep`的`subs`中），此时的数据劫持功能和观察者模式就实现了一个 MVVM 模式中的 Binder，之后就是正常的渲染和更新流程。
-   当数据发生变化或者视图导致的数据发生变化时，会触发数据劫持的`setter`函数，`setter`会通知初始化依赖收集中的`Dep`中和视图相应的 `Watcher` ，告知需要重新渲染视图，`Watcher` 就会再次通过 `update` 方法来更新视图。

<img src=".\static\vueObserver.webp" style="zoom:50%;" />

### Vue2 检测数组响应式

-   vue 没有对数组的每一项用 `definedProperty()` 来数据拦截，而是通过重写数组的方法`push pop shift unshift reverse sort splice`。

-   手动调用 notify,通知 render watcher,执行 update

-   数组中如果有对象类型(`对象和数组`)的话会进行数据拦截。

-   所以通过修改数组下标和数组长度是不会进行数据拦截的，也就不会有响应式变化。例如`arr[0] = 1, arr.length = 2` 都不会有响应式

## Vue3 的响应式原理

Vue3 的响应式是基于 ES6 的`Proxy`来实现的

**优点：**

1. 深度监听，性能更好，`vue2`是一次性递归遍历所有，`vue3`是在`set`属性时，才做递归处理
2. 可监听对象 新增、删除 操作
3. 可监听数组变化

```js
// 使用一个全局变量存储被注册的副作用函数
let activeEffect;
// 注册副作用函数
function effect(fn) {
    activeEffect = fn;
    fn();
}
const obj = new Proxy(data, {
    // getter 拦截读取操作
    get(target, key) {
        // 将副作用函数 activeEffect 添加到存储副作用函数的全局变量 targetMap 中
        track(target, key);
        // 返回读取的属性值
        return Reflect.get(target, key);
    },
    // setter 拦截设置操作
    set(target, key, val) {
        // 设置属性值
        const result = Reflect.set(target, key, val);
        // 把之前存储的副作用函数取出来并执行
        trigger(target, key);
        return result;
    },
});
// 存储副作用函数的全局变量
const targetMap = new WeakMap();
// 在 getter 拦截器内追踪依赖的变化
function track(target, key) {
    // 没有 activeEffect，直接返回
    if (!activeEffect) return;
    // 根据 target 从全局变量 targetMap 中获取 depsMap
    let depsMap = targetMap.get(target);
    if (!depsMap) {
        // 如果 depsMap 不存，那么需要新建一个 Map 并且与 target 关联
        depsMap = new Map();
        targetMap.set(target, depsMap);
    }
    // 再根据 key 从 depsMap 中取得 deps, deps 里面存储的是所有与当前 key 相关联的副作用函数
    let deps = depsMap.get(key);
    if (!deps) {
        // 如果 deps 不存在，那么需要新建一个 Set 并且与 key 关联
        deps = new Set();
        depsMap.set(key, deps);
    }
    // 将当前的活动的副作用函数保存起来
    deps.add(activeEffect);
}
// 在 setter 拦截器中触发相关依赖
function trigger(target, key) {
    // 根据 target 从全局变量 targetMap 中取出 depsMap
    const depsMap = targetMap.get(target);
    if (!depsMap) return;
    // 根据 key 取出相关联的所有副作用函数
    const effects = depsMap.get(key);
    // 执行所有的副作用函数
    effects && effects.forEach(fn => fn());
}
```

## 虚拟 DOM

从本质上来说，Virtual Dom 是一个 JavaScript 对象，通过对象的方式来表示 DOM 结构。将页面的状态抽象为 JS 对象的形式，配合不同的渲染工具，使跨平台渲染成为可能。通过事务处理机制，将多次 DOM 修改的结果一次性的更新到页面上，从而有效的减少页面渲染的次数，减少修改 DOM 的重绘重排次数，提高渲染性能。

会把每个标签转成一个对象，对象可以有三个属性：`tag`、`props`、`children`

-   tag : 标签 可以是组件或者函数
-   props ： 标签上的属性和方法
-   children : 这个标签的内容或者子节点，如果是文本的话就是字符串，如果有子节点就是数组

```vue
<template>
    <div id="app" class="container">
        <h1>沐华</h1>
    </div>
</template>

//转换后 { tag:'div', props:{ id:'app', class:'container' }, children: [ { tag: 'h1', children:'沐华' } ] }
```

## Vue 中的 diff 算法

### 一、vue2 的 diff 算法 （双端 diff 算法）

[](https://juejin.cn/post/6994959998283907102#heading-9)

​ **主要步骤：patch 方法**

1. 对比当前同层的虚拟节点是否为同一种类型的标签，通过sameVnode方法进行判断，如果相同则通过**patchVnode进行深层次比较**，不同则**整个节点替换成新的虚拟节点**

    > 同一种类型的标签的判定准则：
    >
    > 1. key 值是否一样
    > 2. 标签名是否一样
    > 3. 是否都为注释节点
    > 4. 是否都定义了 data
    > 5. 标签为 input 时，type 必须一样

2. patchVnode 函数进行以下判断

    - 找到对应的真实 DOM 称之为**el**
    - 判断 newNode 和 oldNode 是否指向同一个对象，如果是则直接 return
    - 如果新旧节点都有文本节点，则 el 的文本节点设置为 newNode 的文本节点
    - 如果新节点没有子节点，而旧节点有子节点，则直接删除 el 的子节点
    - 如果旧节点没有子节点，新节点有，则将 newNode 的子节点编译后真实化添加到 el
    - 如果两个都有子节点，通过 updateChildren 进行判断

3. updateChildren 方法判断子节点（首尾指针法）

    <img src='.\static\patchOldAndNewNode.webp'/>

    1、`oldS 和 newS `使用`sameVnode方法`进行比较，`sameVnode(oldS, newS)` 相同则将旧节点移到新节点的位置

    2、`oldS 和 newE `使用`sameVnode方法`进行比较，`sameVnode(oldS, newE)` 相同则将旧节点移到新节点的位置

    3、`oldE 和 newS `使用`sameVnode方法`进行比较，`sameVnode(oldE, newS)` 相同则将旧节点移到新节点的位置

    4、`oldE 和 newE `使用`sameVnode方法`进行比较，`sameVnode(oldE, newE)` 相同则将旧节点移到新节点的位置

    5、如果不属于以上四种情况，进行循环 patch

    - 循环 newChildren 列表 ，如果在旧节点中找到了当前循环的新节点，如果相等则直接更新并且移动（**总是移动未为查找的节点前**）
    - 如果找不到，就创建节点插到正确的位置

### 二、vue3 中的 diff 算法 （快速 diff 算法）

[](https://juejin.cn/post/7119428616391229471)

**vue3 在进行 patch 中使用了 patchFlag 进行优化，即会根据节点的特点打上特定的 patchFlag , patch 是就只用比较 patchFlag，减少遍历次数**

-   #### 没有 key 时 patchUnkeydChildren

    ① 比较新老 children 的 length 获取最小值 然后对于公共部分，进行重新 patch 工作。
    ② 如果老节点数量大于新的节点数量 ，移除多出来的节点。
    ③ 如果新的节点数量大于老节点的数量，从新 mountChildren 新增的节点。

-   #### 有 key 时 patchKeyChildren

    1. **开始位置节点类型相同**

        从左向右进行 diff 遍历，如果新旧节点类型相同则进行 patch 处理，不相同则 break 结束遍历

    2. **结束位置节点类型相同**

        从右向左进行 diff 遍历，如果新旧节点类型相同则进行 patch 处理，不相同则 break 结束遍历

    3. **相同部分遍历结束，新序列中有新增节点，进行挂载**

    4. **相同部分遍历结束，新序列中有少节点，进行卸载**

    5. **新序列中有多余，旧序列中也有剩余**

        - 为新的子节点构建 key：index 映射 ---> 作用：后面遍历旧节点后可以知道可复用的节点在新序列中的位置**（最大递增序列的算法）**

        - 移除新节点队列中不存在的旧节点并更新复用节点

-   处理新增节点和移动的节点

## v-for 中的 key 的作用

-   key 的作用主要是为了更高效的更新虚拟 DOM，因为它可以非常精确的找到相同节点，因此 patch 过程会非常高效

-   Vue 在 patch 过程中会判断两个节点是不是相同节点时，key 是一个必要条件。比如渲染列表时，如果不写 key，Vue 在比较的时候，就可能会导致频繁更新元素，使整个 patch 过程比较低效，影响性能你

-   应该避免使用数组下标作为 key，因为 key 值不是唯一的话可能会导致数组插入时，插入位置后的元素都会被重新渲染，使 Vue 无法区分它他，还有比如在使用相同标签元素过渡切换的时候，就会导致只替换其内部属性而不会触发过渡效果

-   从源码里可以知道，Vue 判断两个节点是否相同时主要判断两者的元素类型和 key 等，如果不设置 key，就可能永远认为这两个是相同节点，只能去做更新操作，就造成大量不必要的 DOM 更新操作，明显是不可取的

## v-if 和 v-show 的区别

-   v-if 是**动态的向 DOM 树内添加或者删除**DOM 元素，而 v-show 则是通过**设置 DOM 元素的 display 样式属性**控制 元素的显示和隐藏
-   v-if 切换有一个局部编译/卸载的过程，切换时会销毁和重建组件内部的事件监听。v-show 则是简单的 css 样式变换
-   v-if 只有为真的时候才开始编译，而 v-show 则是在任何条件下都会编译，然后缓存起来
-   v-if 消耗的性能较多，而 v-show 则较少

## Vue 生命周期

|                       vue2                       |      vue3       |
| :----------------------------------------------: | :-------------: |
|                   beforeCreate                   |      setup      |
|                     created                      |      setup      |
|                   beforeMount                    |  onBeforeMount  |
|                     mounted                      |    onMounted    |
|                   beforeUpdate                   | onBeforeUpdate  |
|                     updated                      |    onUpdated    |
|                  beforeDestroy                   | onBeforeDestroy |
|                     destroy                      |   onUnmounted   |
|  activeted (被 keep-alive 缓存的组件激活时调用)  |   onActivited   |
| deactiveted (被 keep-alive 缓存的组件卸载时调用) |  onDeactiveted  |

### 请求数据是放在 created 还是 mounted 中

**created：** 组件实例一旦创建完成就会立即调用，这时候页面的 dom 节点并未生成 。建议不需要 dom 操作的请求放在 created 声明周期中

**mounted:** 页面的 dom 节点渲染并挂载完成后立即执行，放在`mounted`中的请求有可能导致页面闪动（因为此时页面`dom`结构已经生成）

## 为什么 data 属性是一个函数不是一个对象

-   根实例对象`data`可以是对象也可以是函数（根实例是单例），不会产生数据污染情况
-   组件实例对象`data`必须为函数，目的是为了防止多个组件实例对象之间共用一个`data`，产生数据污染。采用函数的形式，`initData`时会将其作为工厂函数都会返回全新`data`对象

## Vue 组件间传参

### 一、通过 props 传参

-   子组件设置`props`属性，定义接收父组件传递过来的参数
-   父组件在使用子组件标签中通过字面量来传递值

```vue
//子组件 props:{ name:String, data:{ type:Number, defalut:18, require:true } } //父组件

<Children name="hell0" data="18" />
```

### 二、使用$emit 触发自定义事件

-   子组件通过`$emit触发`自定义事件，`$emit`第二个参数为传递的数值
-   父组件绑定监听器获取到子组件传递过来的参数

```vue
//子组件 this.$emit('add',18) //父组件
<Children @add="addNum(data)" />
```

### 三、ref

父组件使用 ref 直接使用子组件中的数据

```vue
<Children ref="child" />
console.log(this.$refs.child.name)
```

### 四、EventBus （mitt.js）

Vue2 中使用 EventBus 即全局事件总线，但是在 Vue3 中官方推荐使用 mitt.js

```js
Vue.prototype.$bus = new Vue(); // Vue已经实现了Bus的功能

this.$emit('sendData'); //发布
this.$on('sendData', this.getDate); //订阅
```

### 五、依赖注入（provide / inject）

该方法可以用与祖孙组件以及父子组件之间的传递，适用于层数较深时的数据传输，依赖注入所提供的属性是**非响应式**的。

-   `provide` 钩子用来发送数据或方法
-   `inject`钩子用来接收数据或方法

```vue
//父组件 procide(){ return { num:11, data:this //提供父组件的所有属性 } } //子组件 inject：['num','data'] //接收
console.log(this.data) //父组件的所有属性
```

### 六、$attrs / $listeners

-   `$attrs`：继承所有的父组件属性（除了 prop 传递的属性、class 和 style ），一般用在子组件的子元素上
-   `$listeners`：该属性是一个对象，里面包含了作用在这个组件上的所有监听器，可以配合 `v-on="$listeners"` 将所有的事件监听器指向这个组件的某个特定的子元素。（相当于子组件继承父组件的事件）

```vue
//父组件
<template>
    <div>
        <Children @sendChild="sendChild" name="hello" />
    </div>
</template>
//子组件
<template>
    <div><Child v-on="$listeners" v-bind="$attrs" /> //可将父组件的数据传递到下一层</div>
</template>
//子孙组件
<script>
export default {
    mounted() {
        console.log(this.$listener, this.$attrs); //sendChild   name
    },
};
</script>
```

### 七、$parent / $children / $root

-   使用`$parent`可以让组件访问父组件的实例（访问的是上一级父组件的属性和方法）
-   使用`$children`可以让组件访问子组件的实例，但是，`$children`并不能保证顺序，并且访问的数据也不是响应式的。

-   使用`$root` 访问根组件的实例

```vue
//父组件
<script>
export default {
    mounted() {
        console.log(this.$children[0].message); //通过获取子组件的实例获取里面的属性
    },
    data() {
        return {
            msg: 'hello',
        };
    },
};
</script>
//子组件
<script>
export default {
    mounted() {
        console.log(this.$parent.msg); // 通过获取父组件的实例获取里面的属性
        console.log(this.$root); //根组件的实例
    },
    data() {
        return {
            message: 'world',
        };
    },
};
</script>
```

### 八、 VueX / Pinia

使用状态管理工具进行数据传递

## Vue 双向绑定的原理

​ Vue 数据双向绑定的原理是通过**数据劫持结合发布者-订阅者模式**来实现的

-   在初始化，对 data 执行响应化处理，在 observer 函数中劫持监听所有属性，并为每个属性添加一个 Dep 数组

-   当 get 执行时，会为调用的 dom 节点创建一个 watcher 存放到该数组中，当 set 执行时，调用 Dep 数组的 notify 方法通知

    所有使用了该属性的 watcher，调用 watcher 的 updater 方法触发 Compile 的回调，更新视图

-   在 Compile 解析过程中，此时会调用 get 方法，我们在触发 get 时创建一个 watcher 对象，该对象在初始化时，**为 Dep 添加一个**

    **静态属性 target，为该 DOM 节点的值**。此时调用 CompilerUtil.getValue，获取该 data 的当前值，这时就会触发 getter，将 watcher 存入 Dep 中，获得属性的值然后更更新视图，最后将 Dep.target 设置为空，。因为它是全局变量，也是 watcher 与 dep 关联的唯一桥梁，任何时刻都必须保证`Dep.target`只有一个值。

![img](https://static.vue-js.com/e5369850-3ac9-11eb-85f6-6fac77c0c9b3.png)

利用 proxy 或者 object.defineproperty 生成的 observer 劫持监听所有的属性，在属性变化后通知 dep 订阅者。同时 compile 解析指令，收集指令所依赖的方法和数据,等待数据变化然后进行渲染，将来 data 中数据⼀旦发生变化，会首先找到对应的`Dep`，触发 setter 通知之前的所有`Watcher`执行更新函数，告知视图更新，重新渲染页面

## Vue.nextTick() 原理

**在下次 DOM 更新循环结束之后执行延迟回调**。在修改数据之后立即使用这个方法，获取更新后的 DOM，

Vue 在修改数据后，视图不会立即更新，二是等到同一事件循环中的所有数据都变化完成后，在进行同一的视图更新

**实现原理：**

将传入的回调函数包装成宏微任务加入到 Vue 异步队列

vue 是异步执行 dom 更新的，一旦观察到数据变化，vue 就会开启一个队列，将同一个事件循环中观察到 数据变化的 watcher 推送进这个队列，如果这个 watcher 被触发多次，只会被推送到队列一次。这种缓冲行为可以有效的去掉重复数据造成的不必要的计算和 Dom 操作，而在下一个事件循环时，vue 会清空队列，并进行必要的 dom 更新。

源码三个参数:

-   callback：我们要执行的操作，可以放在这个函数当中，我们每执行一次`$nextTick`就会把回调函数放到一个异步队列当中；
-   pending：标识，用以判断在某个事件循环中是否为第一次加入，第一次加入的时候才触发异步执行的队列挂载
-   timerFunc：用来触发执行回调函数，也就是`Promise.then`或`MutationObserver`或`setImmediate` 或`setTimeout`的过程

**使用场景：**

-   在 created 钩子函数进行的 dom 操作一定要放在 vue.nextTick()的回调函数中。原因是在 created()钩子函数执行的时候 DOM 其实并未进行任何渲染，而此时进行 DOM 操作无异于徒劳，所以此处一定要将 DOM 操作的 js 代码放进 Vue.nextTick()的回调函数中。与之对应的就是 mounted 钩子函数，因为该钩子函数执行时所有的 DOM 挂载已完成。

-   点击按钮显示原本 v-show = false 隐藏起来的输入框，并获取焦点。

## computed 和 watch 的区别

-   computed 支持缓存，只有依赖的数据发生变化时，才会重新计算，而 watch 不支持缓存，数据变则会执行响应的回调函数
-   computed 不支持异步，当 computed 内有异步操作时是无法监听数据变化的；watch 支持异步操作
-   computed 属性的属性值是一函数，函数返回值为属性的属性值，computed 中每个属性都可以设置 set 与 get 方法。watch 监听的数据必须是 data 中声明过或父组件传递过来的 props 中的数据，当数据变化时，触发监听器

**computed 实现原理：**

```vue
<script>
	data(){
	return {
        a:1,
        b:1
	}
}
conputed:{
	num(){
		return this.a+this.b
    }
}
</script>
```

当我们使用 computed 时，**会遍历我们的 computed 对象，将每一个值都定义一个 Wathcer 实例**，**并且其中的每一个 watcher 实例都有一个 dirty 属性，当 dirty 为 false 时，使用缓存的结果，当变为 true 后调用 evaluate 重新计算**

即计算属性重新求值的过程：

-   改变依赖值 --> 触发 set --> 触发 dep.notify --> watcher.update --> 是计算 watcher --> this.dirty = true --> 当 dirty 为 true 时，调用 evaluate 重新求值。
-   改变试图 ：将渲染 watcher 放入 Dep 依赖池中

**watch 实现原理：**

-   首先 watch 在初始化时，会读取一边监听的数据的值，此时那个数据就收集到了 watch 的 watcher 中了
-   当 watch 进行了更新，回调函数就会被调用，将新值和旧的值都进行了传递

## vue-router

### 路由守卫

-   **全局前置路由守卫：** router.beforeEach 当一个导航触发时，全局前置路由守卫会按照创建顺序进行调用，守卫是异步解析执行
-   **全局解析路由守卫：**router.beforeResolve 当导航被确认之前，同时在所有组件内守卫和异步路由组件被解析之后，解析守卫就被调用
-   **全局后置钩子：** router.afterEach 不会接收 next 函数也不会改变导航本身

组件内的守卫：

-   beforeRouteEnter(to,from,next){}——在渲染该组件的对应路由被 confirm 前调用，不能获取组件实例 this
-   beforeRouteUpdate(to,from,next){}——在当前路由改变，但是该组件被复用时调用，可以访问组件实例 this
-   beforeRouteLeave(to,from,next){}——导航离开组件的对应路由时调用

### hash&history

1. hash 模式 （#）

    后面 hash 的值的变化不会向服务器发起请求且请求时值会发送#之前的 url，每次 hash 值发生变化就会触发 hashchange 事件，可以监听 hashchange（window.hashchange）实现页面部分内容操作。

    核心通过监听 url 中 hash 来进行路由跳转

    **两个方法**：

    this.$router.push()——将新路由添加到浏览器访问历史的栈顶

    this.$router.replace()——替换掉当前栈顶的路由

2. history 模式：核心使用 html5 history api

    **修改历史状态**：pushState(),replaceState(),popState()——改变 url 地址且不会发送请求，可以读取历史记录栈，还可以对浏览器历史记录栈进行修改

    **切换历史状态**：back()——后退,forward()——前进,go()——跳转

**区别：**

​ hash 模式中修改的是#中的信息，浏览器请求不会讲#之后的数据发送到后台，但是在 history 模式下，可以自由修改 path，当刷新时，如果服务器没有响应的相应或者资源，则会刷新出来 404 页面

### keep-alive 的原理

[](https://juejin.cn/post/7043401297302650917)

接受三个参数：

-   include : 可以传字符串 正则 数组 。 匹配成功的会被缓存
-   exclude: 可以传字符串 正则 数组 。 匹配成功的不会被缓存
-   max：数字 缓存的最大数量

keep-alive 在各个生命周期

**abstract 属性：此组件是否需要渲染成虚拟 dom**

-   created : 初始化 cache，keys 。前者缓存虚拟 DOM，后者缓存组件的 key 集合
-   mounted: 实时监听 include 和 exclude 这两个变化，执行操作
-   distory： 清除所有缓存的东西

**渲染时：**

第一步：获取到`keep-alive`包裹的第一个组件以及它的`组件名称`

第二步：判断此`组件名称`是否能被`白名单、黑名单`匹配，如果`不能被白名单匹配 || 能被黑名单匹配`，则直接返回`VNode`，不往下执行，如果不符合，则往下执行`第三步`

第三步：根据`组件ID、tag`生成`缓存key`，并在缓存集合中查找是否已缓存过此组件。如果已缓存过，直接取出缓存组件，并更新`缓存key`在`keys`中的位置，**从原来的地方删除重新放到 this.keys 的最后一个**，如果没缓存过，则继续`第四步`

第四步：分别在`cache、keys`中保存`此组件`以及他的`缓存key`，并检查数量是否超过`max`，超过则将 this.keys 的第一个进行删除

第五步：将此组件实例的`keepAlive`属性设置为 true

## VueX

**Vuex 的属性：**

(1）state

state 是存储的单一状态，是存储的基本数据。

(2）Getters

getters 是 store 的计算属性，对 state 的加工，是派生出来的数据。就像 computed 计算属性一样，getter 返回的值会根据它的依赖被缓存起来，且只有当它的依赖值发生改变才会被重新计算。

(3）Mutations

mutations 提交更改数据，使用 store.commit 方法更改 state 存储的状态。（mutations 同步函数）

(4）Actions

actions 像一个装饰器，使用 dispatch 提交 mutation，而不是直接变更状态。（actions 可以包含任何异步操作）

(5）Module

Module 是 store 分割的模块，每个模块拥有自己的 state、getters、mutations、actions。

### 1. Vuex 中 action 和 mutation 的区别

-   Mutation 专注于修改 State，理论上是修改 State 的唯一途径；Action 业务代码、异步请求。
-   Mutation：必须同步执行；Action：可以异步，但不能直接操作 State。
-   在视图更新时，先触发 actions，actions 再触发 mutation

### 2.vuex 的原理

vuex 利用了 vue 的[mixin](https://cn.vuejs.org/api/options-composition.html#mixins)机制，混合 beforeCreate 钩子 将 store 注入至 vue 组件实例上，并注册了

vuex store 的引用属性 $store!

在 beforeCreate 钩子函数中，通过 vue 的 mixin 机制。——即每个 vue 组件实例化过程中，会在 beforeCreate 钩子前调用 vuex Init 方法，this 指向 vue 实例 mixin:混入，一个混入对象可以包含任意组件选项，当组件使用混入对象时，所有混入对象的选项将被混合进入该组件本身的选项

-   new 一个 vue 实例

-   给实例添加 data：{state：xxx}

-   vue.mixin 混入到 Vue.prototype.$store 上

```JavaScript
class Store {
  constructor(Vue, options) {
    var bus = new Vue({
      data: {
        state: options.state
      }
    })

    this.install(Vue, bus)
  }

  install(Vue, bus) {
    Vue.mixin({
      beforeCreate() {
        if (this.$options.store) {
          Vue.prototype.$store = bus
        }
      }
    })
  }
}

const store = new Store(Vue, {
  state: {
    count: 0
  }
})
```

## npm run xxx 发生了什么

npm run xxx 会首先再 package.json 文件中找 **scripts** 里找对应的 xxx

```json
'sctipts':{
	'serve' : 'vue-cli-service serve'
}
```

npm run serve 就相当于执行了 vue-cli-service serve 这个指令，但是由于系统中不存在该指令则会执行一下步骤

1. 首先 npm i xxx 安装依赖时，会在 **node_modules/.bin/** 下创建一个个 xxx 的脚本文件
2. 当执行 vue-cli-service serve 时 ，npm 回到 node_modules/.bin/ 路径下找到 vue-cli-service 文件执行该脚本，相当于执行了 **./node_modules/.bin/vue-cli-service serve**

3. 该 bin 目录下的软连接的实现： npm i 时 npm 读到该配置后，就将该文件软链接到 ./node_modules/.bin 目录下，而 npm 还会自动把 node_modules/.bin 加入$PATH，这样就可以直接作为命令运行依赖程序和开发依赖程序

总结：

npm i 的时候，npm 就帮我们把这种软连接配置好了，其实这种软连接相当于一种映射，执行 npm run xxx 的时候，就会到 node_modules/bin 中找对应的映射文件，然后再找到相应的 js 文件来执行。

1. 运行 npm run xxx 的时候，npm 会先在当前目录的 node_modules/.bin 查找要执行的程序，如果找到则运行；

2. 没有找到则从全局的 node_modules/.bin 中查找，npm i -g xxx 就是安装到到全局目录；

3. 如果全局目录还是没找到，那么就从 path 环境变量中查找有没有其他同名的可执行程序。

## 首页加载白屏

#### 造成白屏的原因

-   网络延迟
-   资源文件体积过大
-   加载脚本时，渲染内容堵塞

#### 白屏时间、首屏时间

白屏时间：**FP**：first paint 。指浏览器从响应用户输入网络地址，到浏览器开始显示内容的时间

首屏时间： **FCP:** 指浏览器从响应用户输入网络地址，到首屏内容渲染完成的时间

有效绘制时间：**FMP：** ajax 请求数据之后，首次有效绘制。

#### 解决方案

1. **预渲染**

    webpack 打包时渲染，通过无头浏览器

    无头浏览器,打包的时候，可以把你 index.html 的内容放入你这个浏览器，但是你这个浏览器是空白的，然后当你进入页面时候直接加载这个 index.html，但是没 ajax 请求

2. **同构：** 一套代码多端使用

3. **SSR 服务端渲染** 在服务端首先准备好首页所需要的 html 文件

4. **路由懒加载**

5. **gzip 压缩，加速:** 启动 gzip 压缩可大幅度缩减传输资源大小，从而缩短资源下载的时间，减少首次白屏时间，提高用户体验。
6. **首页加 loading 或者骨架屏 （优化体验）**
