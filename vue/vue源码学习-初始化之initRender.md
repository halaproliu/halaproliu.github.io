# vue源码学习-初始化之initRender

接下来我们继续来学习初始化调用的初始化时调用的initRender函数

#### 获取父节点和执行环境

```js
vm._vnode = null // the root of the child tree
vm._staticTrees = null // v-once cached trees
const options = vm.$options
const parentVnode = vm.$vnode = options._parentVnode // the placeholder node in parent tree
const renderContext = parentVnode && parentVnode.context
```

开头的部分，首先获取到了父组件的vnode节点和执行环节。

#### 组件之slot插槽

```js
vm.$slots = resolveSlots(options._renderChildren, renderContext)
vm.$scopedSlots = emptyObject
```

接着进行组件的slots（插槽）的初始化.包含匿名slot，具名slot，区别在于，在组件定义中slot是否包含name属性。

```html
// a.vue
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>
```

```html
// 匿名插槽的使用方式
<a>
  <div>main is here</div>
</a>
```

```html
<a>
  <div slot="header">header is here</div>
  <div slot="footer">footer is here</div>
</a>
```
以上方法为vue2.5版本的使用方法， 在2.6.0的版本中，废弃了这种方法，改为使用v-slot进行具名操作
```html
<a>
  <div v-slot:header>header is here</div>
  <div v-slot:footer>footer is here</div>
</a>
```
在slots的操作中，vue主要做了以下几点操作：
- 若组件children子节点为空，则组件中this.$slots对象为{}空对象
- 若组件子节点存在，则先移除slot属性，接着匿名slot设置一个默认default作为key值，存储到this.$slots对象中，具名slot则按照定义的name进行存储
- 当定义slot的标签为template时，则获取它的子节点，进行渲染。
- 最后删除只包含空白字符（即不包含具体内容的slot）

#### 组件之$createElement方法

```js
vm._c = (a, b, c, d) => createElement(vm, a, b, c, d, false)
vm.$createElement = (a, b, c, d) => createElement(vm, a, b, c, d, true)
```

$createElement方法的主要作用是解析用户写的模板html，从而成为浏览器可以识别的格式。

以上两个方法的区别在于，当子节点只是functional组件时，返回的是一个数组格式的节点内容，同时内部已经做了解析，所以只需要简单的进行数组合并，而当子节点是<template>,<slot>,v-for等复杂节点时，需要深入遍历来解析。

#### 组件之$attrs和$listeners

```js
defineReactive(vm, '$attrs', parentData && parentData.attrs || emptyObject, null, true)
defineReactive(vm, '$listeners', options._parentListeners || emptyObject, null, true)
```

$attrs和$listeners是vue暴露出来，用于编写高阶组件的两个属性，$attrs用于在组件中获取所有未在props中定义的元素属性。

- $attrs

包含了父作用域中不作为 prop 被识别 (且获取) 的特性绑定 (class 和 style 除外)。当一个组件没有声明任何 prop 时，这里会包含所有父作用域的绑定 (class 和 style 除外)，并且可以通过 v-bind="$attrs" 传入内部组件——在创建高级别的组件时非常有用。

- $listeners

包含了父作用域中的 (不含 .native 修饰器的) v-on 事件监听器。它可以通过 v-on="$listeners" 传入内部组件——在创建更高层次的组件时非常有用。
