# vue源码学习-初始化生命周期和事件队列

> initLifecycle主要是进行一些变量的初始化

```javascript
export function initLifecycle (vm: Component) {
  const options = vm.$options

  // locate first non-abstract parent
  let parent = options.parent
  if (parent && !options.abstract) {
    while (parent.$options.abstract && parent.$parent) {
      parent = parent.$parent
    }
    parent.$children.push(vm)
  }

  vm.$parent = parent
  vm.$root = parent ? parent.$root : vm

  vm.$children = []
  vm.$refs = {}

  vm._watcher = null
  vm._inactive = null
  vm._directInactive = false
  vm._isMounted = false
  vm._isDestroyed = false
  vm._isBeingDestroyed = false
}
```
以上这段代码，首先先定位到没有abstract属性的parent对象，即最上层的parent对象，然后初始化一些变量。

![initLifeCycle](https://upload-images.jianshu.io/upload_images/2377129-2e5e5aa393cb2670.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> **Vue的事件循环**

```js
export function initEvents (vm: Component) {
  // 首先创建一个空的事件对象
  vm._events = Object.create(null)
  // 定义一个是否有事件钩子，默认为false,一旦注册了带
  // 'hook:'字符串的则为true
  vm._hasHookEvent = false
  // init parent attached events（初始化父对象组件的事件）
  const listeners = vm.$options._parentListeners
  // 当存在_parentListeners时，进行事件更新
  if (listeners) {
    updateComponentListeners(vm, listeners)
  }
}
```

以下为事件循环的主要实现方法。add方法进行事件的注册，remove方法则是进行事件的移除操作。

```js
let target: any

// 添加事件绑定，当once为true，即绑定事件只执行一次就注销，内部调用Vue实例上的对应方法
function add (event, fn, once) {
  if (once) {
    target.$once(event, fn)
  } else {
    target.$on(event, fn)
  }
}

// 移除事件绑定，内部调用Vue实例上的对应方法
function remove (event, fn) {
  target.$off(event, fn)
}

// 更新事件监听队列
export function updateComponentListeners (
  vm: Component,
  listeners: Object,
  oldListeners: ?Object
) {
  target = vm
  updateListeners(listeners, oldListeners || {}, add, remove, vm)
  target = undefined
}
```

在具体实现中，主要是使用当前事件队列中的定义，和原先缓存的事件队列进行对比，并进行更新的操作。

```js
export function updateListeners (
  on: Object,
  oldOn: Object,
  add: Function,
  remove: Function,
  vm: Component
) {
  let name, def, cur, old, event
  for (name in on) {
    def = cur = on[name]
    old = oldOn[name]
    // 格式化事件名称，返回一个数组,数组包含，name,passive,once,capture四个属性
    // 当name以&开头时，passive === true
    // 当name以~开头时，once === true
    // 当name以!开头时，capture === true
    event = normalizeEvent(name)
    /* istanbul ignore if */
    if (__WEEX__ && isPlainObject(def)) {
      cur = def.handler
      event.params = def.params
    }
    // 在开发环境下，当cur未定义时，警告当前的事件处理器为null或undefined
    if (isUndef(cur)) {
      process.env.NODE_ENV !== 'production' && warn(
        `Invalid handler for event "${event.name}": got ` + String(cur),
        vm
      )
    // 当old为Undefined或Null，并且当事件没有定义处理函数的时候，为当前事件创建一个事件处理函数
    } else if (isUndef(old)) {
      if (isUndef(cur.fns)) {
        cur = on[name] = createFnInvoker(cur)
      }
      add(event.name, cur, event.once, event.capture, event.passive, event.params)
    // 当cur和old都存在，且不相等，则更新old的listener的事件处理函数
    } else if (cur !== old) {
      old.fns = cur
      on[name] = old
    }
  }
  // 遍历old事件队列，并确认在当前事件队列是否定义了相应事件处理函数，若没有，则移除相应的事件。
  for (name in oldOn) {
    if (isUndef(on[name])) {
      event = normalizeEvent(name)
      remove(event.name, oldOn[name], event.capture)
    }
  }
}
```
