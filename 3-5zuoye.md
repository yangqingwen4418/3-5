##### 1、Vue 3.0 性能提升主要是通过哪几方面体现的？

​	答：

​	一、Vue 3的体积变小。 目前，Vue运行时的减压和压缩后的Vue运行时的大小约为20kB（当前2.6.10版本为22.8kB）。Vue 3体积的大小估计只有一半左右，所以只有约10kB!

​	二、hoistStatic静态提升。Vue2中无论元素是否参与更新, 每次都会重新创建, 然后再渲染，而 Vue3中对于不参与更新的元素, 会做静态提升, 只会被创建一次, 在渲染时直接复用即可。

​	三、cacheHandlers 事件侦听器缓存。默认情况下onClick会被视为动态绑定, 所以每次都会去追踪它的变化，但是因为是同一个函数，所以没有追踪变化, 直接缓存起来复用即可。

​	四、SSR服务端渲染。当开启SSR了之后，如果我们模版中有一些静态标签，这些静态标签会被直接转化成文本。当有大量静态的内容时候，这些内容会被当做纯字符串推进一个buffer里面，  即使存在动态的绑定，会通过模板插值嵌入进去。这样会比通过虚拟dmo来渲染的快上很多很多。
​		当静态内容大到一定量级时候，会用_createStaticVNode方法在客户端去生成一个static node， 这些静态node，会被直接innerHtml，就不需要创建对象，然后根据对象渲染。

​	五、diff方法优化。Vue2 中的虚拟dom是进行全量的杜比，而Vue3 新增了静态标记（PatchFlag），只比对带有 PF 的节点，并且通过 Flag 的信息得知当前节点要比对的具体内容。

​	六、全局API tree-shaking。Vue 3的源码将是tree-shakeable。这意味着，如果你不使用它的一些功能（如<keep-alive>组件或v-show指令），它们将不会被包含在你的生产包中。目前，无论我们从Vue core中使用了什么特性，它们都会在我们的生产代码中出现，因为Vue实例是作为一个单一的对象导出的，而bundlers无法检测到这个对象的哪些属性在代码中使用了。为了使全局API  tree-shakeable，Vue团队决定通过命名的导出方式导入大部分的API，这样捆绑者就可以检测并删除未使用的代码。

​	七、Composition API。随着Vue组件的增大，组件内代码变得越来越难以理解和维护。其中的一些可以复用的代码很难被抽离出来。同时 Vue2.0还缺少 TS支持。在Vue2中，逻辑概念（功能）被管理在组件中，但是功能和组件并不是一对一关系。一个功能可以被多个组件使用同时一个组件可以有多个功能。在Vue中，一个功能可能需要依赖多个Options（components、props、data、omputed、methods及生命周期方法）。在 Composition API中提供可 setup 方法。




##### 2、Vue 3.0 所采用的 Composition Api 与 Vue 2.x使用的Options Api 有什么区别？

​		答：

​		在vue2中会在一个vue文件中methods，computed，watch，data中等等定义属性和方法，共同处理页面逻辑，我们称这种方式为Options API。

​		**Options API缺点**：一个功能往往需要在不同的vue配置项中定义属性和方法，`比较分散`，项目小还好，清晰明了，但是`项目大了后，一个methods中可能包含20多个方法`，往往分不清哪个方法对应着哪个功能。

​		**vue3 Composition API** **，**代码是根据逻辑功能来组织的，`一个功能所定义的所有api会放在一起（更加的高内聚，低耦合）`，这样做，即时项目很大，功能很多，我们都能`快速的定位到这个功能所用到的所有API`，而不像vue2 Options API 中一个功能所用到的API都是分散的，需要改动功能，到处找API的过程是很费劲的。

​		**为什么要使用 Composition API：**

- Composition API 是根据逻辑相关性组织代码的，提高可读性和可维护性

- 基于函数组合的 API 更好的重用逻辑代码`（在vue2 Options API中通过Mixins重用逻辑代码，容易发生命名冲突且关系不清）`

  

##### **3、Proxy 相对于 Object.defineProperty 有哪些优点？**

​		答：

Proxy的优势：

​		1、可以直接监听对象而非属性

​		2、可以直接监听数组的变化

​		3、拦截方式较多

​		4、Proxy返回一个新对象，可以只操作新对象达到目的，而Object.defineProperty只能遍历对象属性直接修改

​		5、Proxy作为新标准将受到浏览器厂商重点持续的性能优化

Object.defineProperty的缺点：

​		1、Object.defineProperty无法监控到数组下标的变化，导致直接通过数组的下标给数组设置值，不能实时响应。

​		2、Object.defineProperty只能劫持对象的属性,因此我们需要对每个对象的每个属性进行遍历。Vue 2.x里，是通过 递归 + 遍历 data 对象来实现对数据的监控的，如果属性值也是对象那么需要深度遍历,显然如果能劫持一个完整的对象是才是更好的选择。

Object.defineProperty的优势为兼容性好,支持IE9。

##### 4、Vue 3.0 在编译方面有哪些优化？

​		答：

**1.生成block tree**

- Vue.js 2.x 的数据更新并触发重新渲染的粒度是组件级的，单个组件内部需要遍历该组件的整个 vnode 树。
- Vue.js 3.0 做到了通过编译阶段对静态模板的分析，编译生成了 Block tree。Block tree 是一个将模版基于动态节点指令切割的嵌套区块，每个区块内部的节点结构是固定的。每个区块只需要追踪自身包含的动态节点。

**2.slot编译优化**

- Vue.js 2.x 中，如果有一个组件传入了slot，那么每次父组件更新的时候，会强制使子组件update，造成性能的浪费。
- Vue.js 3.0 优化了slot的生成，使得非动态slot中属性的更新只会触发子组件的更新。动态slot指的是在slot上面使用v-if，v-for，动态slot名字等会导致slot产生运行时动态变化但是又无法被子组件track的操作。

**3.diff算法优化**





##### 5、Vue.js 3.0 响应式系统的实现原理？

​	答：

Vue3 使用 Proxy 对象重写响应式系统，这个系统主要有以下几个函数来组合完成的：

- 1、reactive:
  - 接收一个参数，判断这参数是否是对象。不是对象则直接返回这个参数，不做响应式处理
  - 创建拦截器对象 handler, 设置 get/set/deleteProperty
    - get
      - 收集依赖（track）
      - 返回当前 key 的值。
        - 如果当前 key 的值是对象，则为当前 key 的对象创建拦截器 handler, 设置 get/set/deleteProperty
        - 如果当前的 key 的值不是对象，则返回当前 key 的值
    - set
      - 设置的新值和老值不相等时，更新为新值，并触发更新（trigger）
    - deleteProperty
      - 当前对象有这个 key 的时候，删除这个 key 并触发更新（trigger）
  - 返回 Proxy 对象
- 2、effect: 接收一个函数作为参数。作用是：访问响应式对象属性时去收集依赖
- 3、track:
  - 接收两个参数：target 和 key
  - 如果没有 activeEffect，则说明没有创建 effect 依赖
  - 如果有 activeEffect，则去判断 WeakMap 集合中是否有 target 属性，
    - WeakMap 集合中没有 target 属性，则 set(target, (depsMap = new Map()))
    - WeakMap 集合中有 target 属性，则判断 target 属性的 map 值的 depsMap 中是否有 key 属性
      - depsMap 中没有 key 属性，则 set(key, (dep = new Set()))
      - depsMap 中有 key 属性，则添加这个 activeEffect
- 4、trigger:
  - 判断 WeakMap 中是否有 target 属性
    - WeakMap 中没有 target 属性，则没有 target 相应的依赖
    - WeakMap 中有 target 属性，则判断 target 属性的 map 值中是否有 key 属性，有的话循环触发收集的 effect()