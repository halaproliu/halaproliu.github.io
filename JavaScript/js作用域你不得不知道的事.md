# js作用域你不得不知道的事

```js
var length = 10
function fn () {
    console.log(this.length)
}

var obj = {
    length: 5,
    method: function (fn) {
        fn()
        arguments[0]()
    }
}

obj.method(fn, 1)
// 10
// 2
```
首先fn()，由于js的this指向取决于调用方法的作用域，因此，fn的this指向为全局作用域，在浏览器端为window，在node端则为global。

至于arguments[0]()中，arguments 是一个对应于传递给函数的参数的类数组对象。其中arguments[0]为函数的第一个参数，即fn，因此fn中的this指向arguments。
因此，this.length === arguments.length,即参数个数，为2。
