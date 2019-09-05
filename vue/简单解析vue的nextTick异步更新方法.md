# 简单解析vue的nextTick异步更新方法

vue有一个nextTick方法，涉及到vue中dom的异步更新，为了更好的了解结果，下面来看个例子。    

```html
<div id="app">
  <div ref="msg">{{msg}}</div>
  <div v-if="msg1">Without $nextTick: {{msg1}}</div>
  <div v-if="msg2">$nextTick: {{msg2}}</div>
  <div v-if="msg3">Without $nextTick: {{msg3}}</div>
  <button class="btn" @click="change">修改</button>
</div>
```

```js
new Vue({
  el: '#app',
  data: {
      msg: 'before change',
      msg1: '',
      msg2: '',
      msg3: ''
  },
  methods: {
      change() {
          this.msg = "after change"
          this.msg1 = this.$refs.msg.innerHTML
          this.$nextTick(() => {
              this.msg2 = this.$refs.msg.innerHTML
          })
          this.msg3 = this.$refs.msg.innerHTML
      }
  }
})
```
![nextTick.gif](https://upload-images.jianshu.io/upload_images/2377129-ca5588a3b078a91f.gif?imageMogr2/auto-orient/strip)

从上面可以看到，在nextTick之前，虽然msg在页面上的显示改变了，但是获取到的值并没有改变。只有在nextTick后，才能取到变化后的值。

>在下次 DOM 更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，获取更新后的 DOM。

>可能你还没有注意到，Vue 异步执行 DOM 更新。只要观察到数据变化，Vue 将开启一个队列，并缓冲在同一事件循环中发生的所有数据改变。如果同一个 watcher 被多次触发，只会被推入到队列中一次。这种在缓冲时去除重复数据对于避免不必要的计算和 DOM 操作是非常重要的。然后，在下一个的事件循环“tick”中，Vue 刷新队列并执行实际 (已去重的) 工作。Vue 在内部尝试对异步队列使用原生的 Promise.then 和 MessageChannel，如果执行环境不支持，会采用 setTimeout(fn, 0) 代替。

>例如，当你设置 vm.someData = 'new value' ，该组件不会立即重新渲染。当刷新队列时，组件会在事件循环队列清空时的下一个“tick”更新。多数情况我们不需要关心这个过程，但是如果你想在 DOM 状态更新后做点什么，这就可能会有些棘手。虽然 Vue.js 通常鼓励开发人员沿着“数据驱动”的方式思考，避免直接接触 DOM，但是有时我们确实要这么做。为了在数据变化之后等待 Vue 完成更新 DOM ，可以在数据变化之后立即使用 Vue.nextTick(callback) 。这样回调函数在 DOM 更新完成后就会调用。

### nextTick实现浅析
nextTick函数接受两个参数，回调函数cb和环境上下文ctx, 如果没有提供回调函数则返回一个promise对象。
源码：
```js
import { noop } from 'shared/util'
import { handleError } from './error'
import { isIE, isIOS, isNative } from './env'

export let isUsingMicroTask = false

const callbacks = []
let pending = false

function flushCallbacks () {
  pending = false
  const copies = callbacks.slice(0)
  callbacks.length = 0
  for (let i = 0; i < copies.length; i++) {
    copies[i]()
  }
}

if (typeof Promise !== 'undefined' && isNative(Promise)) {
  const p = Promise.resolve()
  timerFunc = () => {
    p.then(flushCallbacks)
    if (isIOS) setTimeout(noop)
  }
  isUsingMicroTask = true
} else if (!isIE && typeof MutationObserver !== 'undefined' && (
  isNative(MutationObserver) ||
  MutationObserver.toString() === '[object MutationObserverConstructor]'
)) {
  let counter = 1
  const observer = new MutationObserver(flushCallbacks)
  const textNode = document.createTextNode(String(counter))
  observer.observe(textNode, {
    characterData: true
  })
  timerFunc = () => {
    counter = (counter + 1) % 2
    textNode.data = String(counter)
  }
  isUsingMicroTask = true
} else if (typeof setImmediate !== 'undefined' && isNative(setImmediate)) {
  timerFunc = () => {
    setImmediate(flushCallbacks)
  }
} else {
  timerFunc = () => {
    setTimeout(flushCallbacks, 0)
  }
}

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
    timerFunc()
  }
  if (!cb && typeof Promise !== 'undefined') {
    return new Promise(resolve => {
      _resolve = resolve
    })
  }
}

```

重点关注timerFunc, pending, flushCallbacks这几个方法以及变量
- $\color{red}{timerFunc是异步执行的核心函数} $ 
当promise存在时，使用promise，否则使用MutationOberserver,依次类推。

![image.png](https://upload-images.jianshu.io/upload_images/2377129-b517d3d012247215.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- $\color{red}{pending变量} $ 
当执行函数的时候，pending置为true是为了防止异步操作未完成时的重复操作。

- $\color{red}{flushCallbacks函数} $ 

flushCallbacks函数则是执行当前callbacks队列中需要执行的回调函数（异步更新）
