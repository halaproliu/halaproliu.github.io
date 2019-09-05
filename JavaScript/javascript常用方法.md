# javascript常用方法

### 类型判断

此方法为jquery判断函数类型的方式，通过事先缓存所有所有类型的结果，然后进行object的key值查找

```js
var class2type = {}
var toString = class2type.toString
var str =
  'String Number Boolean Symbol Null Object Array Date RegExp Function Error'
var arr = str.split(' ')
arr.forEach(function(name) {
  class2type['[object ' + name + ']'] = name.toLowerCase()
})

function type(obj) {
  if (obj === null) return obj + ''
  return typeof obj === 'object' || typeof obj === 'function'
    ? class2type[toString.call(obj)] || 'object'
    : typeof obj
}
```

### 获取类型的简单方式

```js
const type = obj => Object.prototype.toString.call(obj).slice(8, -1).toLowerCase()
```

### 防抖函数
防抖函数原理：在事件被触发n秒后再执行回调，如果在这n秒内又被触发，则重新计时。
```js
const debounce = (fn, delay, immediate) => {
  let timeout
  return function(...args) {
    let context = this
    let callNow = immediate && !timeout
    clearTimeout(timeout)
    timeout = setTimeout(() => {
      timeout = null
      if (!immediate) fn.apply(context, args)
    }, delay)
    callNow && fn.apply(context, args)
  }
}
```

使用场景：
- 防止按钮多次提交，只执行最后提交的一次
- 表单验证需要服务端验证，只执行一段连续的输入事件的最后一次
- 搜索联想词

### 节流函数
节流函数原理:规定在一个单位时间内，只能触发一次函数。如果这个单位时间内触发多次函数，只有一次生效。
```js
const throttle = (fn, delay) => {
  let flag = false
  return function(...args) {
    if (!flag) return
    flag = true
    setTimeout(() => {
      fn.apply(this, args)
      flag = false
    }, delay)
  }
}
```
使用场景：
*   拖拽场景：固定时间内只执行一次，防止超高频次触发位置变动
*   缩放场景：监控浏览器resize
*   动画场景：避免短时间内多次触发动画引起性能问题

### 函数柯里化
函数柯里化指的是将能够接收多个参数的函数转化为接收单一参数的函数，并且返回接收余下参数且返回结果的新函数的技术。

函数柯里化的主要作用和特点就是参数复用、提前返回和延迟执行。

```js
// 柯里化
const curry = fn => {
  if (fn.length <= 1) return fn;
  const generator = args =>
    args.length === fn.length ?
    fn(...args) :
    arg => generator([...args, arg]);
  return generator([], fn.length);
};
```

使用场景：
假设有一个方法，接收三个参数，第一个参数为name，我们可以确定name为固定的，那么，可以使用柯里化，先传入一个参数，作为默认使用函数，避免后续的重复传参。


### 判断多个条件都为true
```js
function isAllTrue() {
  var sum = 0
  for (var i = 0; i < arguments.length; i++) {
    if (arguments[i]) {
      sum += arguments[i]
    }
  }
  // sum的数量为true的个数
  return sum === arguments.length
}
```
使用场景：
- 判断多个条件都为true的状况
