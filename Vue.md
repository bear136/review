## 什么是SPA？如何进行seo优化？

### SPA 单页面应用

只有一个页面的应用，即仅在web页面初始化时加载响应的html、css、js文件

#### 优点：

- 用户体验好，内容的改变不需要重新加载，避免了不必要的跳转和重复渲染，对**服务器压力小**
- 前后端分离
- 完全的前端组件化，代码组织方式更加规范，便于修改

#### 缺点：

- 首次加载需要加载大量的静态资源
- 不利于seo优化，由于所有的内容都在一个页面中动态替换显示，所以在 SEO 上其有着天然的弱势
- 前进后退路由管理：由于单页应用在一个页面中显示所有的内容，所以不能使用浏览器的前进后退功能，所有的页面切换需要自己建立堆栈管理；

**多页面：包含多个页面，跳转时全部刷新**

- **多页应用的首屏时间快**：当访问网页的时候，服务器返回一个新的html文档，只经过了一个http请求
- **搜索引擎优化效果好**： 搜索引擎在做网页排名的时候，要根据网页内容才能给网页权重，来进行网页的排名。搜索引擎是可以识别html内容的，而我们每个页面所有的内容都放在Html中，所以这种多页应用，seo排名效果好。
- 切换慢：每次跳转都需要发出http请求

### 单页面应用如何SEO优化

**SEO:**搜索引擎优化

1. 服务端渲染 **SSR** 例如（nuxt.js)

2. 静态化

   1. 一种是通过程序将动态页面抓取并保存为静态页面，这样的页面实际存在于服务器的硬盘中
   2. 通过web服务器的url rewrite方式。通过web服务器内部模块按一定的规则而将外部的url请求转化为内部的文件地址

3. 使用phantonjs针对爬虫处理。原理是通过`Nginx`配置，判断访问来源是否为爬虫，如果是则搜索引擎的爬虫请求会转发到一个`node server`，再通过`PhantomJS`来解析完整的`HTML`，返回给爬虫。下面是大致流程图

   ![img](https://static.vue-js.com/25be6630-3ac7-11eb-ab90-d9ae814b240d.png)

## Vue模板编译原理

vue模板编译也就是将`<template> </template>`  模板编译生成渲染函数

1. 将模板字符串 编译生成 element ASTs（抽象语法树）

2. 对AST进行静态节点标记，主要是用于虚拟DOM的渲染优化

   **优化：**

   - 生成虚拟dom的过程中，如果发现一个节点是静态子树，除了首次渲染外不会生成新的子节点树，而是拷贝已存在的静态子树；
   - 比对虚拟dom的过程中，如果发现当前节点是静态子树，则直接跳过，不需要进行比对。

3. 是使用 element ASTs 生成 render 函数代码字符串

<img src=".\static\template_render.webp" style="zoom:50%;" />

## Vue2响应式原理

**通过数据的改变去驱动dom视图的改变**

**缺陷：**

vue2中**不会触发试图更新**：

1. 对象属性的动态添加和删除
2. 直接更改数组下标来修改数组的值

**注意：**

- 添加对象属性或者删除属性时，使用Vue.set( ) 方法
- 数组应使用 push pop shift unshift splice reserve sort 

```js
//更新视图
function updateView () {
    console.log('试图进行更新')
}

function observe (target) {
    if (typeof target !== 'object' || target === null) {
        // 不是数组或者对象不适合监听
        return target
    }
    for (let key in target) {
        defineReactive(target, key, target[key])
    }
}

function defineReactive (target, key, value) {
    //递归调用，实现对对象的深度监听
    observe(value)
    Object.defineProperty(target, key, {
        get () {
            return value
        },
        set (newVal) {
            //实现了vue.set( )方法，使得对象的添加删除也能被监听到
            observe(newVal)
            // 使用了闭包 value ，若更新的值和原来的值相同则不会赋值 
            if (newVal !== value) {
                value = newVal
                updateView()
            }
        }
    })
}

```

**源码过程：**

[https://juejin.cn/post/6844903597986037768]()

- 在`init`数据初始化的时候，对象内部通过 `defineReactive` 方法，使用 `Object.defineProperty` 将属性进行劫持（这时候只会劫持已经存在的属性）。如果数据是数组类型， Vue2中是通过重写数组方法来实现。多层对象是通过递归来实现劫持的。
- 在初始化流程中的编译阶段，当`render function` 被渲染的时候，会读取Vue实例中和视图相关的响应式数据，此时会触发 `getter` 函数进行 **依赖收集**（将观察者`Watcher`对象存放到当前闭包的订阅者`Dep`的`subs`中），此时的数据劫持功能和观察者模式就实现了一个MVVM模式中的Binder，之后就是正常的渲染和更新流程。
- 当数据发生变化或者视图导致的数据发生变化时，会触发数据劫持的`setter`函数，`setter`会通知初始化依赖收集中的`Dep`中和视图相应的 `Watcher` ，告知需要重新渲染视图，`Watcher` 就会再次通过 `update` 方法来更新视图。

<img src=".\static\vueObserver.webp" style="zoom:50%;" />

### Vue2检测数组响应式

- vue 没有对数组的每一项用 `definedProperty()` 来数据拦截，而是通过重写数组的方法`push pop shift unshift reverse sort splice`。

- 手动调用 notify,通知 render watcher,执行 update

- 数组中如果有对象类型(`对象和数组`)的话会进行数据拦截。

- 所以通过修改数组下标和数组长度是不会进行数据拦截的，也就不会有响应式变化。例如`arr[0] = 1, arr.length = 2` 都不会有响应式

## Vue3的响应式原理

Vue3的响应式是基于ES6的`Proxy`来实现的

**优点：**

1. 深度监听，性能更好，`vue2`是一次性递归遍历所有，`vue3`是在`set`属性时，才做递归处理
2. 可监听对象 新增、删除 操作
3. 可监听数组变化

```js

// 使用一个全局变量存储被注册的副作用函数
let activeEffect
// 注册副作用函数
function effect (fn) {
    activeEffect = fn
    fn()
}
const obj = new Proxy(data, {
    // getter 拦截读取操作
    get (target, key) {
        // 将副作用函数 activeEffect 添加到存储副作用函数的全局变量 targetMap 中
        track(target, key)
        // 返回读取的属性值
        return Reflect.get(target, key)
    },
    // setter 拦截设置操作
    set (target, key, val) {
        // 设置属性值
        const result = Reflect.set(target, key, val)
        // 把之前存储的副作用函数取出来并执行
        trigger(target, key)
        return result
    }
})
// 存储副作用函数的全局变量
const targetMap = new WeakMap()
// 在 getter 拦截器内追踪依赖的变化
function track (target, key) {
    // 没有 activeEffect，直接返回
    if (!activeEffect) return
    // 根据 target 从全局变量 targetMap 中获取 depsMap
    let depsMap = targetMap.get(target)
    if (!depsMap) {
        // 如果 depsMap 不存，那么需要新建一个 Map 并且与 target 关联
        depsMap = new Map()
        targetMap.set(target, depsMap)
    }
    // 再根据 key 从 depsMap 中取得 deps, deps 里面存储的是所有与当前 key 相关联的副作用函数
    let deps = depsMap.get(key)
    if (!deps) {
        // 如果 deps 不存在，那么需要新建一个 Set 并且与 key 关联
        deps = new Set()
        depsMap.set(key, deps)
    }
    // 将当前的活动的副作用函数保存起来
    deps.add(activeEffect)
}
// 在 setter 拦截器中触发相关依赖
function trigger (target, key) {
    // 根据 target 从全局变量 targetMap 中取出 depsMap
    const depsMap = targetMap.get(target)
    if (!depsMap) return
    // 根据 key 取出相关联的所有副作用函数
    const effects = depsMap.get(key)
    // 执行所有的副作用函数
    effects && effects.forEach(fn => fn())
}
```

## 虚拟DOM

从本质上来说，Virtual Dom是一个JavaScript对象，通过对象的方式来表示DOM结构。将页面的状态抽象为JS对象的形式，配合不同的渲染工具，使跨平台渲染成为可能。通过事务处理机制，将多次DOM修改的结果一次性的更新到页面上，从而有效的减少页面渲染的次数，减少修改DOM的重绘重排次数，提高渲染性能。

会把每个标签转成一个对象，对象可以有三个属性：`tag`、`props`、`children`

- tag : 标签 可以是组件或者函数
- props ： 标签上的属性和方法
- children : 这个标签的内容或者子节点，如果是文本的话就是字符串，如果有子节点就是数组

```vue
<template>
    <div id="app" class="container">
        <h1>沐华</h1>
    </div>
</template>

//转换后
{
  tag:'div',
  props:{ id:'app', class:'container' },
  children: [
    { tag: 'h1', children:'沐华' }
  ]
}
```

## Vue中的diff算法

### 一、vue2的diff算法 （双端diff算法）

[](https://juejin.cn/post/6994959998283907102#heading-9)

​	**主要步骤：patch方法**

1. 对比当前同层的虚拟节点是否为同一种类型的标签，通过sameVnode方法进行判断，如果相同则通过**patchVnode进行深层次比较**，不同则**整个节点替换成新的虚拟节点**

   > 同一种类型的标签的判定准则：
   >
   > 1. key值是否一样
   > 2. 标签名是否一样
   > 3. 是否都为注释节点
   > 4. 是否都定义了data
   > 5. 标签为input时，type必须一样

2. patchVnode函数进行以下判断

   - 找到对应的真实DOM称之为**el**
   - 判断newNode和oldNode是否指向同一个对象，如果是则直接return
   - 如果新旧节点都有文本节点，则el的文本节点设置为newNode的文本节点
   - 如果新节点没有子节点，而旧节点有子节点，则直接删除el的子节点
   - 如果旧节点没有子节点，新节点有，则将newNode的子节点编译后真实化添加到el
   - 如果两个都有子节点，通过updateChildren进行判断

3. updateChildren方法判断子节点（首尾指针法）

   ![](.\static\patchOldAndNewNode.webp)

   1、`oldS 和 newS `使用`sameVnode方法`进行比较，`sameVnode(oldS, newS)`

   2、`oldS 和 newE `使用`sameVnode方法`进行比较，`sameVnode(oldS, newE)`

   3、`oldE 和 newS `使用`sameVnode方法`进行比较，`sameVnode(oldE, newS)`

   4、`oldE 和 newE `使用`sameVnode方法`进行比较，`sameVnode(oldE, newE)`

   5、如果以上逻辑都匹配不到，再把所有旧子节点的 `key` 做一个映射到旧节点下标的 `key -> index` 表，然后用新 `vnode` 的 `key` 去找出在旧节点中可以复用的位置。

### 二、vue3中的diff算法 （快速diff算法）

[](https://juejin.cn/post/7119428616391229471)

- #### 没有key时 patchUnkeydChildren

  ① 比较新老children的length获取最小值 然后对于公共部分，进行重新patch工作。
  ② 如果老节点数量大于新的节点数量 ，移除多出来的节点。
  ③ 如果新的节点数量大于老节点的数量，从新 mountChildren新增的节点。

- #### 有key时 patchKeyChildren

  1. **开始位置节点类型相同**

     从左向右进行diff遍历，如果新旧节点类型相同则进行patch处理，不相同则break结束遍历

  2. **结束位置节点类型相同**

     从右向左进行diff遍历，如果新旧节点类型相同则进行patch处理，不相同则break结束遍历

  3. **相同部分遍历结束，新序列中有新增节点，进行挂载**

  4. **相同部分遍历结束，新序列中有少节点，进行卸载**

  5. **新序列中有多余，旧序列中也有剩余**

     - 为新的子节点构建key：index映射     ---> 作用：后面遍历旧节点后可以知道可复用的节点在新序列中的位置

     - 从左向右遍历旧序列，patch相同的节点，卸载不存在的旧节点

       变量`newIndexToOldIndexMap`用于映射**新的子序列中的节点下标** 对应于 **旧的子序列中的节点的下标**

       并且会将`newIndexToOldIndexMap`初始化为一个全0数组，`[0, 0, 0, 0]`
       通过newIndex初始化`newIndexToOldIndexMap`

     - 移动可复用的节点，挂载新增的节点

       前面通过`newIndexToOldIndexMap`，记录了新旧子节点变化前后的下标映射。

       这里会通过`getSequence`方法获取一个**最大递增子序列**。用于记录相对位置没有发生变化的子节点的下标。

       

## v-for中的key的作用

- key 的作用主要是为了更高效的更新虚拟 DOM，因为它可以非常精确的找到相同节点，因此 patch 过程会非常高效

- Vue 在 patch 过程中会判断两个节点是不是相同节点时，key 是一个必要条件。比如渲染列表时，如果不写 key，Vue 在比较的时候，就可能会导致频繁更新元素，使整个 patch 过程比较低效，影响性能

- 应该避免使用数组下标作为 key，因为 key 值不是唯一的话可能会导致数组插入时，插入位置后的元素都会被重新渲染，使 Vue 无法区分它他，还有比如在使用相同标签元素过渡切换的时候，就会导致只替换其内部属性而不会触发过渡效果

- 从源码里可以知道，Vue 判断两个节点是否相同时主要判断两者的元素类型和 key 等，如果不设置 key，就可能永远认为这两个是相同节点，只能去做更新操作，就造成大量不必要的 DOM 更新操作，明显是不可取的

## v-if 和 v-show 的区别

- v-if 是**动态的向DOM树内添加或者删除**DOM元素，而 v-show 则是通过**设置DOM元素的display样式属性**控制 元素的显示和隐藏
- v-if 切换有一个局部编译/卸载的过程，切换时会销毁和重建组件内部的事件监听。v-show则是简单的css样式变换
- v-if 只有为真的时候才开始编译，而v-show则是在任何条件下都会编译，然后缓存起来
- v-if 消耗的性能较多，而v-show则较少

## Vue生命周期

|                        vue2                         |      vue3       |
| :-------------------------------------------------: | :-------------: |
|                    beforeCreate                     |      setup      |
|                       created                       |      setup      |
|                     beforeMount                     |  onBeforeMount  |
|                       mounted                       |    onMounted    |
|                    beforeUpdate                     | onBeforeUpdate  |
|                       updated                       |    onUpdated    |
|                    beforeDestroy                    | onBeforeDestroy |
|                       destroy                       |   onUnmounted   |
|  activeted    (被 keep-alive 缓存的组件激活时调用)  |   onActivited   |
| deactiveted    (被 keep-alive 缓存的组件卸载时调用) |  onDeactiveted  |

### 请求数据是放在 created 还是 mounted 中

**created：** 组件实例一旦创建完成就会立即调用，这时候页面的dom节点并未生成 。建议不需要dom操作的请求放在created声明周期中

**mounted:** 页面的dom节点渲染并挂载完成后立即执行，放在`mounted`中的请求有可能导致页面闪动（因为此时页面`dom`结构已经生成）

## 为什么data属性是一个函数不是一个对象

- 根实例对象`data`可以是对象也可以是函数（根实例是单例），不会产生数据污染情况
- 组件实例对象`data`必须为函数，目的是为了防止多个组件实例对象之间共用一个`data`，产生数据污染。采用函数的形式，`initData`时会将其作为工厂函数都会返回全新`data`对象

## Vue组件间传参

### 一、通过props传参

- 子组件设置`props`属性，定义接收父组件传递过来的参数
- 父组件在使用子组件标签中通过字面量来传递值

```vue
//子组件
props:{
		name:String,
		data:{
		type:Number,
		defalut:18,
		require:true
	}
}
//父组件

<Children name='hell0' data='18'/>
```

### 二、使用$emit触发自定义事件

- 子组件通过`$emit触发`自定义事件，`$emit`第二个参数为传递的数值
- 父组件绑定监听器获取到子组件传递过来的参数

```vue
	//子组件
	this.$emit('add',18)
	//父组件
	<Children @add = 'addNum(data)'/>
```

### 三、ref

父组件使用ref直接使用子组件中的数据

```vue
<Children ref='child'/>
console.log(this.$refs.child.name) 
```

### 四、EventBus （mitt.js）

Vue2中使用EventBus即全局事件总线，但是在Vue3中官方推荐使用mitt.js

```js
Vue.prototype.$bus = new Vue() // Vue已经实现了Bus的功能  

this.$emit('sendData')   //发布
this.$on('sendData',this.getDate)    //订阅
```

### 五、依赖注入（provide / inject）

该方法可以用与祖孙组件以及父子组件之间的传递，适用于层数较深时的数据传输，依赖注入所提供的属性是**非响应式**的。

- `provide` 钩子用来发送数据或方法
- `inject`钩子用来接收数据或方法

```vue
//父组件
procide(){
	return {
		num:11,
		data:this  //提供父组件的所有属性
	}
}
//子组件
inject：['num','data']   //接收

console.log(this.data)  //父组件的所有属性
```

### 六、$attrs / $listeners

- `$attrs`：继承所有的父组件属性（除了prop传递的属性、class 和 style ），一般用在子组件的子元素上
- `$listeners`：该属性是一个对象，里面包含了作用在这个组件上的所有监听器，可以配合 `v-on="$listeners"` 将所有的事件监听器指向这个组件的某个特定的子元素。（相当于子组件继承父组件的事件）

```vue
//父组件
<template>
	<div>
        <Children @sendChild = 'sendChild' name='hello'/>
    </div>
</template>
//子组件
<template>
	<div>
        <Child v-on="$listeners" v-bind="$attrs"/>       //可将父组件的数据传递到下一层
    </div>
</template>
//子孙组件
<script>
	export default {
		mounted(){
            	console.log(this.$listener,this.$attrs) //sendChild   name
        }
    }
</script>
```

### 七、$parent / $children / $root

- 使用`$parent`可以让组件访问父组件的实例（访问的是上一级父组件的属性和方法）
- 使用`$children`可以让组件访问子组件的实例，但是，`$children`并不能保证顺序，并且访问的数据也不是响应式的。

- 使用`$root` 访问根组件的实例

```vue
//父组件
<script>
	export default {
		mounted(){
            	console.log(this.$children[0].message )    //通过获取子组件的实例获取里面的属性
        },
        data(){
            return {
                msg:'hello'
            }
        }
    }
</script>
//子组件
<script>
	export default {
		mounted(){
            	console.log(this.$parent.msg)   // 通过获取父组件的实例获取里面的属性
            	console.log(this.$root)  //根组件的实例
        },
        data(){
            return {
                message:'world'
            }
        }
    }
</script>
```

### 八、 VueX / Pinia  

使用状态管理工具进行数据传递

## Vue双向绑定的原理

- new Vue()初始化，对data执行响应化处理，在observer函数中劫持监听所有属性

- 同时对模板执行编译，找到其中动态绑定的数据，从`data`中获取并初始化视图，这个过程发生在`Compile`中

- 同时定义⼀个更新函数和`Watcher`，将来对应数据变化时`Watcher`会调用更新函数

- 由于`data`的某个`key`在⼀个视图中可能出现多次，所以每个`key`都需要⼀个管家`Dep`来管理多个`Watcher`

- 将来data中数据⼀旦发生变化，会首先找到对应的`Dep`，通知所有`Watcher`执行更新函数

  ![img](https://static.vue-js.com/e5369850-3ac9-11eb-85f6-6fac77c0c9b3.png)

利用proxy或者object.defineproperty生成的observer劫持监听所有的属性，在属性变化后通知dep订阅者。同时compile解析指令，收集指令所依赖的方法和数据,等待数据变化然后进行渲染，将来data中数据⼀旦发生变化，会首先找到对应的`Dep`，触发setter通知之前的所有`Watcher`执行更新函数，告知视图更新，重新渲染页面

## Vue.nextTick() 原理

将回调函数延迟在下一次DOM更新数据后调用，简单的理解就是：当数据更新了，在DOM中渲染后，自动执行该函数

Vue在修改数据后，视图不会立即更新，二是等到同一事件循环中的所有数据都变化完成后，在进行同一的视图更新

**实现原理：**

将传入的回调函数包装成宏微任务加入到Vue异步队列

vue是异步执行dom更新的，一旦观察到数据变化，vue就会开启一个队列，将同一个事件循环中观察到 数据变化的watcher推送进这个队列，如果这个watcher被触发多次，只会被推送到队列一次。这种缓冲行为可以有效的去掉重复数据造成的不必要的计算和Dom 操作，而在下一个事件循环时，vue会清空队列，并进行必要的dom更新。

源码三个参数:

- callback：我们要执行的操作，可以放在这个函数当中，我们每执行一次`$nextTick`就会把回调函数放到一个异步队列当中；
- pending：标识，用以判断在某个事件循环中是否为第一次加入，第一次加入的时候才触发异步执行的队列挂载
- timerFunc：用来触发执行回调函数，也就是`Promise.then`或`MutationObserver`或`setImmediate` 或`setTimeout`的过程  

**使用场景：**  

- 在created钩子函数进行的dom操作一定要放在vue.nextTick()的回调函数中。原因是在 created()钩子函数执行的时候 DOM 其实并未进行任何渲染，而此时进行 DOM 操作无异于徒劳，所以此处一定要将 DOM 操作的 js 代码放进 Vue.nextTick()的回调函数中。与之对应的就是 mounted 钩子函数，因为该钩子函数执行时所有的 DOM 挂载已完成。

- 点击按钮显示原本v-show = false 隐藏起来的输入框，并获取焦点。

## computed和watch的区别

- computed 支持缓存，只有依赖的数据发生变化时，才会重新计算，而watch不支持缓存，数据变则会执行响应的回调函数
- computed不支持异步，当computed内有异步操作时是无法监听数据变化的；watch支持异步操作
- computed属性的属性值是一函数，函数返回值为属性的属性值，computed中每个属性都可以设置set与get方法。watch监听的数据必须是data中声明过或父组件传递过来的props中的数据，当数据变化时，触发监听器

## vue-router

### 路由守卫

- **全局前置路由守卫：** router.beforeEach   当一个导航触发时，全局前置路由守卫会按照创建顺序进行调用，守卫是异步解析执行
- **全局解析路由守卫：**router.beforeResolve  当导航被确认之前，同时在所有组件内守卫和异步路由组件被解析之后，解析守卫就被调用
- **全局后置钩子：** router.afterEach 不会接收next函数也不会改变导航本身

组件内的守卫：

- beforeRouteEnter(to,from,next){}——在渲染该组件的对应路由被confirm前调用，不能获取组件实例this
- beforeRouteUpdate(to,from,next){}——在当前路由改变，但是该组件被复用时调用，可以访问组件实例this
- beforeRouteLeave(to,from,next){}——导航离开组件的对应路由时调用

### hash&history

1. hash模式 （#） 

   后面hash的值的变化不会向服务器发起请求且请求时值会发送#之前的url，每次hash值发生变化就会触发hashchange事件，可以监听hashchange（window.hashchange）实现页面部分内容操作。

   核心通过监听url中hash来进行路由跳转

   **两个方法**：

   this.$router.push()——将新路由添加到浏览器访问历史的栈顶

   this.$router.replace()——替换掉当前栈顶的路由

2. history模式：核心使用html5 history api

   **修改历史状态**：pushState(),replaceState(),popState()——改变url地址且不会发送请求，可以读取历史记录栈，还可以对浏览器历史记录栈进行修改

   **切换历史状态**：back()——后退,forward()——前进,go()——跳转

**区别：**

​	hash模式中修改的是#中的信息，浏览器请求不会讲#之后的数据发送到后台，但是在history模式下，可以自由修改path，当刷新时，如果服务器没有响应的相	应或者资源，则会刷新出来404页面

### keep-alive的原理

[](https://juejin.cn/post/7043401297302650917)

接受三个参数：

- include : 可以传字符串 正则 数组 。 匹配成功的会被缓存
- exclude: 可以传字符串 正则 数组 。 匹配成功的不会被缓存
- max：数字 缓存的最大数量

keep-alive在各个生命周期

**abstract属性：此组件是否需要渲染成虚拟dom**

- created : 初始化cache，keys 。前者缓存虚拟DOM，后者缓存组件的key集合
- mounted: 实时监听 include 和 exclude这两个变化，执行操作
- distory： 清除所有缓存的东西

**渲染时：**

第一步：获取到`keep-alive`包裹的第一个组件以及它的`组件名称`

第二步：判断此`组件名称`是否能被`白名单、黑名单`匹配，如果`不能被白名单匹配 || 能被黑名单匹配`，则直接返回`VNode`，不往下执行，如果不符合，则往下执行`第三步`

第三步：根据`组件ID、tag`生成`缓存key`，并在缓存集合中查找是否已缓存过此组件。如果已缓存过，直接取出缓存组件，并更新`缓存key`在`keys`中的位置（这是`LRU算法`的关键），如果没缓存过，则继续`第四步`

第四步：分别在`cache、keys`中保存`此组件`以及他的`缓存key`，并检查数量是否超过`max`，超过则根据`LRU算法`进行删除

第五步：将此组件实例的`keepAlive`属性设置为true

## VueX

**Vuex的属性：**

(1）state

state是存储的单一状态，是存储的基本数据。

(2）Getters

getters是store的计算属性，对state的加工，是派生出来的数据。就像computed计算属性一样，getter返回的值会根据它的依赖被缓存起来，且只有当它的依赖值发生改变才会被重新计算。

(3）Mutations

mutations提交更改数据，使用store.commit方法更改state存储的状态。（mutations同步函数）

(4）Actions

actions像一个装饰器，使用dispatch提交mutation，而不是直接变更状态。（actions可以包含任何异步操作）

(5）Module

Module是store分割的模块，每个模块拥有自己的state、getters、mutations、actions。

### 1. Vuex中action和mutation的区别

- Mutation专注于修改State，理论上是修改State的唯一途径；Action业务代码、异步请求。
- Mutation：必须同步执行；Action：可以异步，但不能直接操作State。
- 在视图更新时，先触发actions，actions再触发mutation

### 2.vuex的原理

vuex利用了vue的[mixin](https://cn.vuejs.org/api/options-composition.html#mixins)机制，混合 beforeCreate 钩子 将store注入至vue组件实例上，并注册了

vuex store的引用属性 $store!

在beforeCreate钩子函数中，通过vue 的mixin机制。——即每个vue组件实例化过程中，会在beforeCreate钩子前调用vuex Init方法，this指向vue实例 mixin:混入，一个混入对象可以包含任意组件选项，当组件使用混入对象时，所有混入对象的选项将被混合进入该组件本身的选项

- new一个vue实例

- 给实例添加data：{state：xxx}

- vue.mixin混入到Vue.prototype.$store上

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

