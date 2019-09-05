# hashchange事件

hashchange事件是html5新增的api，用来监听浏览器链接的hash值变化。
目前流行的spa框架的路由都有使用到该特性，接下来简单介绍下：

当URL的片段标识符更改时，将触发hashchange事件 (跟在＃符号后面的URL部分，包括＃符号)
| 属性       | 类型                                                                                                                                                                                  | 描述               |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------ |
| target     | [`EventTarget`](https://developer.mozilla.org/zh-CN/docs/Web/API/EventTarget "EventTarget 是一个由可以接收事件的对象实现的接口，并且可以为它们创建侦听器。")                          | 上下文为window对象 |
| type       | [`DOMString`](https://developer.mozilla.org/zh-CN/docs/Web/API/DOMString "DOMString 是一个UTF-16字符串。由于JavaScript已经使用了这样的字符串，所以DOMString 直接映射到 一个String。") | event类型          |
| bubbles    | [`Boolean`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Boolean)                                                                                                | 事件是否能冒泡     |
| cancelable | [`Boolean`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Boolean)                                                                                                | 事件是否能被取消   |
| oldURL     | [`String`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/String)                                                                                                  | 跳转前的URL        |
| newURL     | [`String`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/String)                                                                                                  | 跳转后的当前URL    |

下面是一个使用hashchange的一个例子。
```js
function getUUID () {
  return Math.floor(Math.random() * 1000000)
}
window.onload = function () {
  const el = document.getElementById('toggle')
  el.onclick = (e) => {
    e.preventDefault()
    const uuid = getUUID()
    location.hash = '#' + uuid
  }
  window.onhashchange = (e) => {
    console.log('oldURL:', e.oldURL)
    console.log('newURL:', e.newURL)
  }
}
```
![hashchange.gif](https://upload-images.jianshu.io/upload_images/2377129-2fab550debf499d2.gif?imageMogr2/auto-orient/strip)

浏览器兼容性：
| Feature       | Chrome | Firefox (Gecko)                                                                                                                               | Internet Explorer | Opera | safari |
| ------------- | ------ | --------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | ----- | ------ |
| Basic support | 5.0    | [3.6](https://developer.mozilla.org/en-US/Firefox/Releases/3.6 "Released on 2010-01-21.") (1.9.2)Firefox 6 中加入对 oldURL/newURL 属性的支持. | 8.0               | 10.6  | 5.0    |

**为了兼容所有浏览器，以下为onhashchange的polyfill:**
```js
// 兼容低版本的hashchange
(function(window) {  
  if ('onhashchange' in window) { return; }

  var location = window.location,
    oldURL = location.href,
    oldHash = location.hash

  setInterval(function() {
    var newURL = location.href, newHash = location.hash
    if (newHash != oldHash && typeof window.onhashchange === 'function') {     
      window.onhashchange({
        type: 'hashchange',
        oldURL: oldURL,
        newURL: newURL
      })

      oldURL = newURL
      oldHash = newHash
    }
  }, 100)
})(window)
```
