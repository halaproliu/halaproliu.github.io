# vue源码学习-引入Vue时，Vue做了什么

### 前言
Vue.js作为我们当今前端开发中最常出现的框架之一，其受到了许多前端开发者的喜爱，为了更深入的了解其中的原理，来学习一下其实现机制。

####  引入Vue时，Vue做了什么？
```javascript
// 这里是vue对象的实例化部分
import Vue from './instance/index'
// 这是初始化Vue的全局api
import { initGlobalAPI } from './global-api/index'
// 判断是否是服务端渲染
import { isServerRendering } from 'core/util/env'
import { FunctionalRenderContext } from 'core/vdom/create-functional-component'

// 调用初始化全局api
initGlobalAPI(Vue)

// 在Vue对象上定义$isServer参数，判断是否是服务端渲染
Object.defineProperty(Vue.prototype, '$isServer', {
  get: isServerRendering
})

Object.defineProperty(Vue.prototype, '$ssrContext', {
  get () {
    /* istanbul ignore next */
    return this.$vnode && this.$vnode.ssrContext
  }
})

// expose FunctionalRenderContext for ssr runtime helper installation
Object.defineProperty(Vue, 'FunctionalRenderContext', {
  value: FunctionalRenderContext
})

// Vue的版本号
Vue.version = '__VERSION__'

export default Vue

```

从上述代码中看到，initGlobalAPI(Vue)这句代码就是我们的核心入口了。


```javascript
  // config
  const configDef = {}
  configDef.get = () => config
  if (process.env.NODE_ENV !== 'production') {
    configDef.set = () => {
      warn(
        'Do not replace the Vue.config object, set individual fields instead.'
      )
    }
  }
  Object.defineProperty(Vue, 'config', configDef)

```
代码中首先先定义了一些基本的配置信息，同时不允许后续的修改配置的操作。


```javascript
// 暴露一些方法到全局的util下
Vue.util = {
    warn,
    extend,
    mergeOptions,
    defineReactive
}

// 暴露set方法到全局
Vue.set = set
// 暴露set方法到全局
Vue.delete = del
// 暴露set方法到全局
Vue.nextTick = nextTick
Vue.options = Object.create(null)
// 定义主要自定义的组件，包括components,directives,filters
ASSET_TYPES.forEach(type => {
    Vue.options[type + 's'] = Object.create(null)
})
```
上述代码中，extend主要是一个浅拷贝的方法，set方法则是vue的一个**关键方法**，主要作用就是为对象属性添加动态监控。set方法的核心就是调用了defineReactive方法，具体的实现后续再进行展开。delete方法主要是用来删除响应的监听属性。

```js
  Vue.options._base = Vue

  extend(Vue.options.components, builtInComponents)

  initUse(Vue)
  initMixin(Vue)
  initExtend(Vue)
  initAssetRegisters(Vue)
```

![image.png](https://upload-images.jianshu.io/upload_images/2377129-90ac176cbcbe7921.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从上图中，我们列出了initGlobalApi方法具体做了些什么。

### initUse
初始化Vue.use方法，从而实现Vue的插件引入机制。

### initMixin
初始化Vue.mixin方法，从而实现mixin混入功能。

### initExtend
初始化Vue.extend方法，从而实现组件的深拷贝，包含原型链的指向，props，computed, component, filter directive等属性的拷贝等

### initAssetRegisters
初始化Vue.component, Vue.filter, Vue.directive方法，从而实现全局引入组件，过滤器和指令。

> ### nextTick模块实现
    

```javascript
const callbacks = []
let pending = false
let useMacroTask = false
export function nextTick (cb?: Function, ctx?: Object) {
  let _resolve
  callbacks.push(() => {
    if (cb) {
      try {
        cb.call(ctx)
      } catch (e) {
        handleError(e, ctx, 'nextTick')
      }
    } else if (_resolve) {
      _resolve(ctx)
    }
  })
  if (!pending) {
    pending = true
    if (useMacroTask) {
      macroTimerFunc()
    } else {
      microTimerFunc()
    }
  }
  // $flow-disable-line
  if (!cb && typeof Promise !== 'undefined') {
    return new Promise(resolve => {
      _resolve = resolve
    })
  }
}
```
nextTick方法通过callbacks模拟一个事件队列，将新传入的回调和上下文push进callbacks中。接着通过pending和useMacroTask判断使用macroTimerFunc或者
microTimerFunc，默认使用的microTimerFunc。

![microTimerFunc.jpg](https://upload-images.jianshu.io/upload_images/2377129-44c0175f0e19a991.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```javascript
if (typeof Promise !== 'undefined' && isNative(Promise)) {
  const p = Promise.resolve()
  microTimerFunc = () => {
    p.then(flushCallbacks)
    if (isIOS) setTimeout(noop)
  }
} else {
  // fallback to macro
  microTimerFunc = macroTimerFunc
}
```
从上述代码可知，microTimerFunc默认是使用Promise，若是Promise不存在则使用macroTimerFunc。

**在ios下，会出现microTask队列无法被释放，直到浏览器要做其他动作的时候才会释放，因此添加一个空的定时器，强制清空microTask队列。**

![macroTimerFunc.jpg](https://upload-images.jianshu.io/upload_images/2377129-5ec402ebda42a8b9.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```javascript
if (typeof setImmediate !== 'undefined' && isNative(setImmediate)) {
  macroTimerFunc = () => {
    setImmediate(flushCallbacks)
  }
} else if (typeof MessageChannel !== 'undefined' && (
  isNative(MessageChannel) ||
  MessageChannel.toString() === '[object MessageChannelConstructor]'
)) {
  const channel = new MessageChannel()
  const port = channel.port2
  channel.port1.onmessage = flushCallbacks
  macroTimerFunc = () => {
    port.postMessage(1)
  }
} else {
  macroTimerFunc = () => {
    setTimeout(flushCallbacks, 0)
  }
}
```
从上述代码可知，macroTimerFunc默认使用setImmediate构造，若是不存在则依次使用MessageChannel——>setTimeout.

相信大家对MessageChannel有些陌生，第一次见到也有些摸不着头脑，于是查阅了[MDN文档](https://developer.mozilla.org/en-US/docs/Web/API/Channel_Messaging_API)，以下为对应的描述：

>
The Channel Messaging API allows two separate scripts running in different browsing contexts attached to the same document (e.g., two IFrames, or the main document and an IFrame, two documents via a SharedWorker, or two workers) to communicate directly, passing messages between one another through two-way channels (or pipes) with a port at each end.

MessageChannel允许我们通过该接口提供的两个端口，在两个不同的脚本文件中共享相同的文档信息（包括两个IFrames，或者一个文档和IFrame，或者通过两个[SharedWorker](https://developer.mozilla.org/en-US/docs/Web/API/SharedWorker)线程，或者是两个[Web Worker](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Workers_API/Using_web_workers)线程中来进行直接通话。



