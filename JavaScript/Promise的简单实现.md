# Promise的简单实现

随着ES6的出现，Promise成为标准，平时使用的次数也增加。但是Promise的原理是什么，如何实现链式调用。今天就来探究一下，下面简单实现了一个Promise：
```js
class Promise {
  constructor (fn) {
    this.value = ''
    this.resolveFn = null
    this.rejectFn = null
    try {
      fn.call(this, this.resolve.bind(this), this.reject.bind(this))
    } catch (e) {
      this.reject.call(this, e)
    }
  }

  resolve (value) {
    this.value = value
    setTimeout(() => {
      this.resolveFn && this.resolveFn.call(this, value)
      return this
    }, 0)
  }

  reject (value) {
    this.value = value
    setTimeout(() => {
      console.log(this)
      this.rejectFn && this.rejectFn.call(this, value)
      return this
    }, 0)
  }

  then (onFullfilled, onRejected) {
    this.resolveFn = onFullfilled
    this.rejectFn = onRejected
    return this
  }
}
```
接下来来执行一下：
```js
var fn = new Promise((resolve, reject) => {
  console.log(111)
  resolve(333)
  console.log(222)
})

fn.then((val) => {
  console.log(val)
}, (e) => {
  console.log(e)
})

// 111
// 222
// 333
```
执行结果是正确的，但是这段代码有缺陷，只支持一个then方法，不支持多个then方法的调用，后面定义的then方法会覆盖前面的方法。接下来来改造下：
```js
class Promise {
  constructor (fn) {
    this.state = 'pending'
    this.callbacks = []
    try {
      fn.call(this, this.resolve.bind(this), this.reject.bind(this))
    } catch (e) {
      this.value = e
      this.reject.call(this, e)
    }
  }

  resolve (value) {
    this.state = 'fullfilled'
    this.value = value
    this.run()
  }

  reject (value) {
    this.state = 'rejected'
    this.value = value
    this.run()
  }

  then (onFullFilled, onRejected) {
    return new Promise((resolve, reject) => {
      this.handle({
        onFullFilled: onFullFilled,
        onRejected: onRejected,
        resolve: resolve,
        reject: reject
      })
    })
  }

  catch (onRejected) {
    this.state = 'rejected'
    return this.then(undefined, onRejected)
  }

  run () {
    setTimeout(() => {
      this.callbacks.forEach(callback => {
        this.handle(callback)
      })
    }, 0)
  }
  
  handle (callback) {
    if (this.callbacks.length === 0) {
      this.callbacks.push(callback)
      return
    }
    let fn = this.state === 'fullfilled' ? callback.onFullFilled : callback.onRejected
    if (!fn) {
      fn = this.state === 'fullfilled' ? callback.resolve : callback.reject
      fn(this.value)
      return
    }
    try {
      let res = fn(this.value)
      callback.resolve(res)
    } catch (e) {
      this.value = e
      callback.reject(e)
    }
  }
}
```

```js
var fn = new Promise((resolve, reject) => {
  console.log(111)
  resolve(333)
  console.log(222)
})

fn.then((val) => {
  console.log(val)
}, (e) => {
  console.log(e)
})
.then((e) => {
  console.log(555)
  throw new Error('Error11')
}, (e) => {
  console.log(666)
})
.catch(e => {
  console.log(e)
  return 'aa'
})
.then((e) => {
  console.log(e)
})
```
![image.png](https://upload-images.jianshu.io/upload_images/2377129-5558a76a75bcdf1a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

至此，实现了Promise的简单功能。
