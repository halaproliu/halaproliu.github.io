### Vue源码之defineReactive

1. [Vue源码之初始化操作](./index)
2. [Vue源码之defineReactive](./Vue源码之defineReactive)
3. [Vue源码之Observer](./Vue源码之Observer)

### 引言

在Vue.js框架中，框架的核心是使用Object.defineProperty实现数据的双向绑定，即在页面渲染的时候，就对所有的data数据进行数据绑定。传统的原生API操作DOM和jquery类库操作DOM的方式，一旦项目过于庞大，就会产生各种奇葩的问题，让人一把辛酸泪。那么今天我们就来介绍下react实现双向数据绑定的核心方法之一，defineReactive。

### 为什么需要双向数据绑定？

下面我们来看个例子：

```js
let a = 5
let b = a * 10
console.log(b) // 50

a = 7
console.log(b) // 50
```

当a改变时，b依然还是50,不会因为a的数据改变了，而产生新的结果。那么需要b根据a进行变化，需要如何做呢？

```js
const setState = () => {
  b = a * 10
}
a = 11

console.log(b) // 110
```

如上可见，我们添加了一个setState函数来监听a的变化，并重新进行计算,这是React使用的方式。

```js
Object.defineProperty(obj, key, {
  get () {},
  set () {},
  configurable: true,
  writeable: true
})
```
