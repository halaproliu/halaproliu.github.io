# let,const和var的区别

let,const和var的区别
===

| 声明方式 | 变量提升 | 作用域 | 初始值 | 重复定义 |
| :------: | :------: | :----: | :----: | :------: |
|  const   |    否    |  块级  |  需要  |  不允许  |
|   let    |    否    |  块级  | 不需要 |  不允许  |
|   var    |    是    | 函数级 | 不需要 |   允许   |

**变量提升**：const 和 let 必须先声明再使用，不支持变量提升

```javascript
console.log(c1, l1, v1);
// 报错
// Uncaught ReferenceError: c1 is not defined
  
const c1 = 'c1';
let l1 = 'l1';
var v1 = 'v1';
```

**作用域**：const，let 支持块级作用域，有效避免变量覆盖

```javascript
const c21 = 'c21';
let l21 = 'l21';
var v21 = 'v21';
  
if (0.1 + 0.2 != 0.3) {
 const c21 = 'c22';
 let l21 = 'l22';
 var v21 = 'v22';
  
 console.log(c21, l21, v21);
 // 输出 c22 l22 v22
}
  
console.log(c21, l21, v21);
// 输出 c21 l21 v22
```

块级作用域，在外层不能直接访问内层变量

```javascript
if (0.1 + 0.2 != 0.3) {
 const c22 = 'c22';
 let l22 = 'l22';
 var v22 = 'v22';
  
 console.log(c22, l22, v22);
 // 输出 c22 l22 v22
}
  
console.log(c22, l22, v22);
// 报错
// Uncaught ReferenceError: c22 is not defined
// 同样地, l22 is not defined
```

const 定义常量，该常量不能赋值，但该常量的属性可以赋值

```javascript
const c231 = {};
const c232 = [];
  
c231.name = 'seven';
c232.push(27);
  
console.log(c231, c232);
// 输出 {name: "seven"} [27]
  
// 禁止给对象赋值，应该使用 Object.freeze
  
const c233 = Object.freeze({});
const c234 = Object.freeze([]);
  
c233.name = 'seven';
// 普通模式下不报错
// 严格模式下报错
// Uncaught TypeError: Cannot add property name, object is not extensible
   
c234.push(27);
// 普通模式下就会报错
// Uncaught TypeError: Cannot add property 0, object is not extensible
  
console.log(c233, c234);
// 输出 {} []
```
全局变量不再设置为顶层对象（window）的属性，有效避免全局变量污染

```javascript
const c24 = 'c24';
let l24 = 'l24';
  
console.log(c24, l24);
// 输出 c24 l24
  
console.log(window.c24, window.l24);
// 输出 undefined undefined
```
符合预期的 for 循环

```javascript
for (var i = 0; i != 3; i++) {
 setTimeout(function() {
  console.log(i);
 },10);
}
// 依次打印
 
for (let i = 0; i != 3; i++) {
 setTimeout(function() {
  console.log(i);
 },10);
}
// 依次打印，为啥呢
```
可以看到在 for 循环中使用 let 方式声明变量才是符合预期。

在 for 中每一次循环，let 都是重新声明变量，并且因为 JavaScript 引擎会记住上一次循环的值，初始化 i 时在上一轮的基础上计算。

可以看到在 for 循环中至少有两层作用域，看下面的例子更容易理解。

```javascript
for (let i = 0; i != 3; i++) {
 let i = 'seven';
 console.log(i);
}
console.log('eight');
// 依次打印
seven
seven
seven
eight
```

**初始值**：const 声明的变量必须设置初始值，且不能重复赋值。

```javascript
const c3 = 'c3';
let l3 = 'l3';
var v3 = 'v3';
  
console.log(c3, l3, v3);
// 输出 c3 l3 v3
  
c3 = 2; // Uncaught TypeError: Assignment to constant variable
l3 = 2;
v3 = 2;
  
console.log(c3, l3, v3);
// 输出 c3 2 2
  
const c32;
// 报错
// Uncaught SyntaxError: Missing initializer in const declaration
```
重复定义：const 和 let 不支持重复定义

const、let 缩小了变量作用域，完美避免变量污染；const 固定变量（即固定变量类型），对于弱类型 JavaScript 来说，可以明显提升性能。推荐在应用中使用 const、let 声明变量。
