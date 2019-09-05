
# ES6的Symbol类型介绍

### Symbol特性
 1. Symbol类型不可被Object.keys或for...in枚举
```js
const key = Symbol('value')
const obj = {}
obj[key] = 1

console.log(Object.keys(obj)) // []
for(let i in obj) {console.log(i)} // undefined
```
2. Symbol类型是唯一值，即使传入相同的key也不相等
```js
Symbol('name') === Symbol('name') // false
```
3. Symbol可以使用以下api进行获取值

```js
const obj = {
  Symbol('name'): ''
}
// 使用Object的API
Object.getOwnPropertySymbols(obj) // [Symbol(name)]

// 使用新增的反射API
Reflect.ownKeys(obj) // [Symbol(name), 'age', 'title']
```

4. 获取全局Symbol，可以在多个浏览器窗口使用，如iframe

```js
Symbol.for('global') === Symbol.for('global') // true
```

### 使用场景
1. Object的key值
2. 定义常量
3. 控制输出变量，假设不想输出某个值，就可以使用Symbol创建key值
