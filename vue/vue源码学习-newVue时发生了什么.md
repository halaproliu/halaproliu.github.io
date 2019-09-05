# vue源码学习-new Vue时发生了什么

### 前言
Vue框架包含了各种各样的特性，为了更好的理解框架本身，让我们来看看在vue初始化时所做的一些操作

```javascript
// 合并选项，参数处理
if (options && options._isComponent) {
  // 优化内部组件的实例化，并且内部组件的动态合并较慢，且没有需要特殊处理的部分。
  initInternalComponent(vm, options)
} else {
  // 合并选项
  vm.$options = mergeOptions(
    resolveConstructorOptions(vm.constructor),
    options || {},
    vm
  )
}
```

```javascript
// 暴露出真正的this指向
vm._self = vm
// 初始化生命周期
initLifecycle(vm)
// 初始化事件监听
initEvents(vm)
// 渲染页面
initRender(vm)
// 进入beforeCreate生命周期
callHook(vm, 'beforeCreate')
initInjections(vm) // resolve injections before data/props
// 初始化状态
initState(vm)
initProvide(vm) // resolve provide after data/props
// 进入created生命周期
callHook(vm, 'created')

if (vm.$options.el) {
  vm.$mount(vm.$options.el)
}
```

![](https://upload-images.jianshu.io/upload_images/2377129-ea75f3e74c9a636b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

以下按顺序介绍调用方法的具体作用：

#### initLifecycle
主要是进行一些变量的初始化

#### initEvents
初始化事件监听

#### initRender
主要是初始化组件的$slots和$scopedSlots属性，以及$attrs和$listeners属性

#### callHook(vm, 'beforeCreate') 
不用说也知道，这是调用beforeCreate生命周期方法

#### initInjections
vue2.2.0新增的特性，初始化作为HOC高阶组件使用的api，包含一个provide和inject属性，父级组件通过provide注入一个object，子组件不管多少层级，都可以通过inject来获取父组件注入的值。
initInjectctions就是初始化inject属性的

#### initState
初始化data、props、methods、computed、watch等属性

#### initProvide
从方法名可以看出，是用来初始化provide这个HOC组件属性的

#### callHook(vm, 'created')
调用created生命周期方法

#### vm.$mount(vm.$options.el)
当new Vue的时候传入了el属性（即vue模板挂载的dom节点）时，调用$mount方法,初始化dom节点。
