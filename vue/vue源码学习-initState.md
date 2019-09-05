# vue源码学习-initState

接下来来学习一下initState方法

![image.png](https://upload-images.jianshu.io/upload_images/2377129-99700aa9b0afb00a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```js
export function initState (vm: Component) {
  vm._watchers = []
  const opts = vm.$options
  if (opts.props) initProps(vm, opts.props)
  if (opts.methods) initMethods(vm, opts.methods)
  if (opts.data) {
    initData(vm)
  } else {
    observe(vm._data = {}, true /* asRootData */)
  }
  if (opts.computed) initComputed(vm, opts.computed)
  if (opts.watch && opts.watch !== nativeWatch) {
    initWatch(vm, opts.watch)
  }
}
```



首先创建vm._watchers数组，用于存储watcher对象实例，接着获取组件的$options，即挂载在实例上的属性，包含我们平时写的data,props,methods,computed,watch等对象。


### initProps

```js
function initProps (vm: Component, propsOptions: Object) {
  const propsData = vm.$options.propsData || {}
  const props = vm._props = {}
  const keys = vm.$options._propKeys = []
  const isRoot = !vm.$parent
  if (!isRoot) {
    toggleObserving(false)
  }
  for (const key in propsOptions) {
    keys.push(key)
    const value = validateProp(key, propsOptions, propsData, vm)
    if (process.env.NODE_ENV !== 'production') {
      ...... // 隐去部分代码
    } else {
      defineReactive(props, key, value)
    }
    if (!(key in vm)) {
      proxy(vm, `_props`, key)
    }
  }
  toggleObserving(true)
}
```

其中propsData是父组件传递给子组件的props值，propsOptions则为组件中定义的props属性。

```js
const keys = vm.$options._propKeys = []
for (const key in propsOptions) {
  keys.push(key)
}
```

其中根节点的props需要进行转化，接着遍历定义的组件props属性，缓存key值到_propKeys数组中。

```js
const value = validateProp(key, propsOptions, propsData, vm)
```

通过validateProp方法获取props值。

```js
defineReactive(props, key, value)
```

接着使用defineReactive方法为props添加响应式监听。


```js
if (!(key in vm)) {
  proxy(vm, `_props`, key)
}
```

最后挂载props值到vm._props属性上作为备份。


### initMethods

```js
if (opts.methods) initMethods(vm, opts.methods)
```
接着是初始化methods方法，具体实现如下：

```js
function initMethods (vm: Component, methods: Object) {
  const props = vm.$options.props
  for (const key in methods) {
    ...
    vm[key] = methods[key] == null ? noop : bind(methods[key], vm)
  }
}
```

省略了部分代码，主要是绑定所有的methods到当前组件上。

#### initData
```js
if (opts.data) {
  initData(vm)
} else {
  observe(vm._data = {}, true /* asRootData */)
}
```

当opts.data存在的时候，则调用initData方法，否则定义一个vm._data为空对象。


```js
function initData (vm: Component) {
  let data = vm.$options.data
  data = vm._data = typeof data === 'function'
    ? getData(data, vm)
    : data || {}
  if (!isPlainObject(data)) {
    data = {}
    process.env.NODE_ENV !== 'production' && warn(
      'data functions should return an object:\n' +
      'https://vuejs.org/v2/guide/components.html#data-Must-Be-a-Function',
      vm
    )
  }
  // proxy data on instance
  const keys = Object.keys(data)
  const props = vm.$options.props
  const methods = vm.$options.methods
  let i = keys.length
  while (i--) {
    const key = keys[i]
    if (process.env.NODE_ENV !== 'production') {
      if (methods && hasOwn(methods, key)) {
        warn(
          `Method "${key}" has already been defined as a data property.`,
          vm
        )
      }
    }
    if (props && hasOwn(props, key)) {
      process.env.NODE_ENV !== 'production' && warn(
        `The data property "${key}" is already declared as a prop. ` +
        `Use prop default value instead.`,
        vm
      )
    } else if (!isReserved(key)) {
      proxy(vm, `_data`, key)
    }
  }
  // observe data
  observe(data, true /* asRootData */)
}
```

initData方法首先判断data是否是一个function，如果是则执行function初始化data，否则直接获取data，若data为null或undefined，则为一个{}空对象。当data不是一个纯对象时，则重新赋值为空对象，并报错。接着会判断data的key值，是否在props，methods中定义，如果是，则报错。同时检测data是否以$或者_开头，如果不是，则定义一个vm._data对象，挂载data的值。最后调用observe方法，为每个data属性，添加监听。关于如何监听，后续会有个专门的篇幅进行介绍。

#### initComputed

```js
if (opts.computed) initComputed(vm, opts.computed)
```

当computed存在时，则调用initComputed方法，进行初始化computed监听对象

```js
function initComputed (vm: Component, computed: Object) {
  // $flow-disable-line
  const watchers = vm._computedWatchers = Object.create(null)
  // computed properties are just getters during SSR
  const isSSR = isServerRendering()

  for (const key in computed) {
    const userDef = computed[key]
    const getter = typeof userDef === 'function' ? userDef : userDef.get
    if (process.env.NODE_ENV !== 'production' && getter == null) {
      warn(
        `Getter is missing for computed property "${key}".`,
        vm
      )
    }

    if (!isSSR) {
      // create internal watcher for the computed property.
      watchers[key] = new Watcher(
        vm,
        getter || noop,
        noop,
        computedWatcherOptions
      )
    }

    // component-defined computed properties are already defined on the
    // component prototype. We only need to define computed properties defined
    // at instantiation here.
    if (!(key in vm)) {
      defineComputed(vm, key, userDef)
    } else if (process.env.NODE_ENV !== 'production') {
      if (key in vm.$data) {
        warn(`The computed property "${key}" is already defined in data.`, vm)
      } else if (vm.$options.props && key in vm.$options.props) {
        warn(`The computed property "${key}" is already defined as a prop.`, vm)
      }
    }
  }
}
```

首先initComputed方法会使用Object.create(null)创建一个空对象，同时调用isServerRendering方法，判断是否是ssr页面。接着遍历computed对象，通过computed[key]获取对应的function，若computed[key]不是function，则获取computed[key].get，当computed[key].get也不是function是，则报错。

```js
const computedWatcherOptions = { lazy: true }
<!--more-->
if (!isSSR) {
  // create internal watcher for the computed property.
  watchers[key] = new Watcher(
    vm,
    getter || noop,
    noop,
    computedWatcherOptions
  )
}
```
当不是ssr页面的时候，创建一个lazy模式的watcher对象，关于watcher对象，后续我们会再详细介绍。目前创建的watcher对象是监听computed中的字段变化，当lazy为true时，watcher对象不默认调用get方法，获取值。

接下来循环computed，创建computed监听，具体实现如下：
```js
if (!(key in vm)) {
  defineComputed(vm, key, userDef)
} else if (process.env.NODE_ENV !== 'production') {
  if (key in vm.$data) {
    warn(`The computed property "${key}" is already defined in data.`, vm)
  } else if (vm.$options.props && key in vm.$options.props) {
    warn(`The computed property "${key}" is already defined as a prop.`, vm)
  }
}
```

接下来查看definedComputed的逻辑：

![image.png](https://upload-images.jianshu.io/upload_images/2377129-34e57e04ec2d8bf3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

defineComputed主要是使用Object.definedProperty来实现响应式，针对浏览器环境构建getter部分，即使用createComputedGetter方法创建getter，createComputedGetter通过watcher对象构建一个自己的watcher数组，当属性变化时，调用watcher的evaluate计算value，通过watcher的depend方法收集依赖。

#### initWatch

```js
function initWatch (vm: Component, watch: Object) {
  for (const key in watch) {
    const handler = watch[key]
    if (Array.isArray(handler)) {
      for (let i = 0; i < handler.length; i++) {
        createWatcher(vm, key, handler[i])
      }
    } else {
      createWatcher(vm, key, handler)
    }
  }
}
```

initWatch主要是获取watch对象，若是数组则遍历，否则使用createWatcher方法对变量绑定Watcher对象进行响应式监听，
createWatcher则是调用Vue.$watch方法。关于$watch方法，我们会在后续一起说明。
