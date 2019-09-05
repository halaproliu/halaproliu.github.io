# html5拖放事件API

##### 拖放（Drag 和 drop）是 HTML5 标准的组成部分。
拖放是一种常见的特性，即抓取对象以后拖到另一个位置。
在 HTML5 中，拖放是标准的一部分，任何元素都能够拖放。

为了使元素可以拖放,将draggable设置为true
```html
<div draggable="true"></div>
```
拖放的主要有如下事件：
- **在拖动目标上触发事件 (源元素)**:
  *   ondragstart用户开始拖动元素时触发
  *   ondrag - 元素正在拖动时触发
  *   ondragend - 用户完成元素拖动后触发

- **释放目标时触发的事件**:
  *   ondragenter - 当被鼠标拖动的对象进入其容器范围内时触发此事件
  *   ondragover - 当某被拖动的对象在另一对象容器范围内拖动时触发此事件
  *   ondragleave - 当被鼠标拖动的对象离开其容器范围内时触发此事件
  *   ondrop - 在一个拖动过程中，释放鼠标键时触发此事件

**注意**： 在拖动元素时，每隔 350 毫秒会触发 ondrag 事件。

以下为元素拖放的一个例子，dataTransfer.setData() 方法设置被拖数据的数据类型和值,例子中类型为Text，值为元素的id。在拖动到元素内部时，会使容器的边框为绿色虚线框。同时监听了拖离容器时，去除边框和被拖放元素。

- dragover
dragover规定何处放置数据，默认是不允许放置的，需要使用e.preventDefault来允许放置。
- drop
drop的默认行为是打开URL链接，因此要阻止默认行为，避免默认打开URL
通过e.dataTransfer.getData来获取之前setData设置的类型值。
```css
.drag-container {
  width: 400px;
  height: 200px;
  border: 1px solid #ccc;
}

.drag-container .input-text {
  text-align: center;
}

.input-text {
  margin-top: 30px;
}

#text {
  color: hotpink;
}
```

```js
function addEventListener(el, type, callback) {
  if (el.addEventListener) {
    el.addEventListener(type, callback, false)
  } else if (el.attachEvent) {
    el.attachEvent('on' + type, callback)
  } else {
    el['on' + type] = callback
  }
}

window.onload = () => {
  const el = document.getElementById('dragtarget')
  const target = document.getElementById('drag-container')
  const text = document.getElementById('text')
  addEventListener(el, 'dragstart', (e) => {
    text.innerText = e.target.id + '开始移动'
    e.dataTransfer.setData('Text', e.target.id)
  })
  addEventListener(el, 'dragenter', (e) => {
    text.innerText = e.target.id + '进入目标'
  })
  addEventListener(target, 'dragover', (e) => {
    e.preventDefault()
    e.target.id === 'drag-container' && (e.target.style.border = '4px dotted green')
  })
  addEventListener(target, 'drop', (e) => {
    e.preventDefault()
    text.innerText = '拖动元素到' + e.target.id
    const data = e.dataTransfer.getData('Text')
    e.target.appendChild(document.getElementById(data))
  })

  addEventListener(target, 'dragleave', (e) => {
    e.target.className === 'drag-container' && (e.target.style.border = '')
    // 判断相对元素，即当前所处的容器父元素。（dragleave会触发两次）
    if (e.relatedTarget.nodeName.toLowerCase() === 'body') {
      const child = e.target.childNodes[0].cloneNode(true)
      e.target.removeChild(e.target.childNodes[0])
      document.body.insertBefore(child, text)
      text.innerText = '拖走元素内的' + e.target.id
    }
  })
  addEventListener(el, 'dragend', (e) => {
    text.innerText = e.target.id + '拖放结束'
  })
}
```


```html
<div id="drag-container" class="drag-container"></div>
<div id="dragtarget" class="input-text" draggable="true">学如逆水行舟，不进则退</div>
<p id="text"></p>
```

##### 使用场景
1. 当需要生成自定义表单的时候，可以定义好一些通用组件，拖动到自定义表单的容器，实现自定义表单。
2. 网上商城的购物车功能。

##### 浏览器支持
Internet Explorer 9、Firefox、Opera 12、Chrome 以及 Safari 5 支持拖放。
**注意**：在 Safari 5.1.2 中不支持拖放。
