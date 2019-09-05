# vue源码学习-performance性能检测和proxy对象

vue在初始化过程中，进行了一系列的操作，今天我们就来介绍下其中的performance和proxy的使用。

```javascript
vm._uid = uid++
    
let startTag, endTag
// 初始化开始时打个开始的tag
if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
    startTag = `vue-perf-start:${vm._uid}`
    endTag = `vue-perf-end:${vm._uid}`
    mark(startTag)
}

// 初始化开始时打个结束的tag，并进行计算时间
if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
  vm._name = formatComponentName(vm, false)
  mark(endTag)
  measure(`vue ${vm._name} init`, startTag, endTag)
}
```
从代码中可知，在非生产环境下，会根据配置判断是否使用window.performance的api进行时间监测。
> [Web Performance API](https://developer.mozilla.org/en-US/docs/Web/API/Performance)允许网页访问某些函数来测量网页和Web应用程序的性能，包括 Navigation Timing API和高分辨率时间数据。

> 在vue的[performance API](https://cn.vuejs.org/v2/api/#performance)中介绍，设置为 true 以在浏览器开发工具的性能/时间线面板中启用对组件初始化、编译、渲染和打补丁的性能追踪。只适用于开发模式和支持 performance.mark API 的浏览器上。


```javascript
// 初始化代理
if (process.env.NODE_ENV !== 'production') {
  initProxy(vm)
} else {
  vm._renderProxy = vm
}
```
主要是在vm._renderProxy上挂载一个proxy对象，用作拦截操作。

```javascript
// core/instance/proxy.js 9-14行
// 定义了允许的全局对象类型。
const allowedGlobals = makeMap(
    'Infinity,undefined,NaN,isFinite,isNaN,' +
'parseFloat,parseInt,decodeURI,decodeURIComponent,encodeURI,encodeURIComponent,' +
    'Math,Number,Date,Array,Object,Boolean,String,RegExp,Map,Set,JSON,Intl,' +
    'require' // for Webpack/Browserify
  )
```


```javascript
// core/instance/proxy.js 27-28行
const hasProxy =
    typeof Proxy !== 'undefined' && isNative(Proxy)
```

首先判断Proxy是否为原生支持的函数。


```javascript
// core/instance/proxy.js 30-43行
if (hasProxy) {
    const isBuiltInModifier = makeMap('stop,prevent,self,ctrl,shift,alt,meta,exact')
    config.keyCodes = new Proxy(config.keyCodes, {
      set (target, key, value) {
        if (isBuiltInModifier(key)) {
          warn(`Avoid overwriting built-in modifier in config.keyCodes: .${key}`)
          return false
        } else {
          target[key] = value
          return true
        }
      }
    })
  }
```
这几行代码主要是为了监控是否有修改vue内建的一些按键值，若传入的key值为内建的，则提示用户禁止修改，并且可以添加额外的自定义按键值。


```javascript
// core/instance/proxy.js 45-76行
// 添加拦截器判断是否是允许全局类型或者是否是_开头的
const hasHandler = {
    has (target, key) {
      const has = key in target
      const isAllowed = allowedGlobals(key) || key.charAt(0) === '_'
      if (!has && !isAllowed) {
        warnNonPresent(target, key)
      }
      return has || !isAllowed
    }
}

// 获取对应key的值
const getHandler = {
    get (target, key) {
      if (typeof key === 'string' && !(key in target)) {
        warnNonPresent(target, key)
      }
      return target[key]
    }
}
// 当存在_withStripped时，使用getHandler，否则hasHandler
//（前提是支持原生Proxy，目前对Proxy的支持还不是很好）
initProxy = function initProxy (vm) {
    if (hasProxy) {
      // determine which proxy handler to use
      const options = vm.$options
      const handlers = options.render && options.render._withStripped
        ? getHandler
        : hasHandler
      vm._renderProxy = new Proxy(vm, handlers)
    } else {
      vm._renderProxy = vm
    }
}
```

**总结来看proxy就是用作拦截，类似vm._renderProxy.isFinite或是用来判断是否存在属性'test' in vm._renderProxy去测试是否有test属性，若没有则给出提示。**

