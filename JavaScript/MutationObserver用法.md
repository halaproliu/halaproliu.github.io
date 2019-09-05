# MutationObserver用法

Mutation Observer API 用来监视 DOM 变动。比如节点的增减、属性的变动、文本内容的变动。

### observe方法
MutationObserver使用observe方法进行监听指定的元素节点变化，observe方法接受两个参数：
| 属性                  | 描述                                                 | 类型                     |
| --------------------- | ---------------------------------------------------- | ------------------------ |
| childList             | 子节点的变动（指新增，删除或者更改）                 | Boolean                  |
| attributes            | 属性的变动                                           | Boolean                  |
| characterData         | 节点内容或节点文本的变动                             | Boolean                  |
| subtree               | 表示是否将该观察器应用于该节点的所有后代节点         | Boolean                  |
| attributeOldValue     | 表示观察attributes变动时，是否需要记录变动前的属性值 | Boolean                  |
| characterDataOldValue | 表示观察characterData变动时，是否需要记录变动前的值  | Boolean                  |
| attributeFilter       | 表示需要观察的特定属性                               | Array，如['class','src'] |
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Event Loop</title>
  <style>
    input {
      width: 200px;
      height: 30px;
    }

    input {
      -webkit-transition: all 0.30s ease-in-out;
      -moz-transition: all 0.30s ease-in-out;
      -ms-transition: all 0.30s ease-in-out;
      -o-transition: all 0.30s ease-in-out;
      outline: none;
      padding: 3px 0 3px 12px;
      margin: 5px 1px 3px 0;
      border: 1px solid #ddd;
      font-size: 18px;
    }

    input:focus {
      box-shadow: 0 0 5px rgba(216, 76, 41, 1);
      padding: 3px 0 3px 12px;
      margin: 5px 1px 3px 0;
      border: 1px solid rgba(216, 76, 41, 1);
    }

    #btn {
      font-size: 18px;
      font-family: Helvetica, Tahoma, Arial;
      line-height: 1em;   /*使用em作为单位，即使字体变化，按钮的整体样式也会按比例跟随变化*/
      color: #fff;
      background: linear-gradient(135deg,#fce,#cce);
      padding: .5em 1.5em;
      border-radius: 2em;
      display: inline-block;
      outline: none;
    }
  </style>
  <script>
    function addCount(el) {
      el.value++
    }

    function modifyAttribute (el) {
      console.log(++el.dataset.count)
    }

    function observe (el, options, callback) {
      var MutationObserver = window.MutationObserver || window.WebKitMutationObserver || window.MozMutationObserver
      var observer = new MutationObserver(callback)
      observer.observe(el, options)
    }
    window.onload = () => {
      var text = document.getElementById('text')
      var btn = document.getElementById('btn')
      btn.onclick = (e) => {
        addCount(text)
        modifyAttribute(text)
      }
      var options = {
        childList: true,
        subtree: true,
        attributes: true,
        attributeOldValue: true,
        characterData: true
      }
      observe(text, options, (records, instance) => {
        console.log(records)
        console.log(instance)
        records.map(record => {
          console.log('Mutation Type: ' + record.type)
          console.log('Mutation Change Attribute: ' + record.attributeName)
          console.log('Previous attribute value: ' + record.oldValue)
        })
      })


    }
  </script>
</head>
<body>
  <input type="text" id="text" value="0" data-count="0">
  <button id="btn">增加</button>
</body>
</html>
```
![image.png](https://upload-images.jianshu.io/upload_images/2377129-152d788f78aac2fe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### disconnect方法
当不需要监听节点变化时，可以使用[`disconnect()`](https://developer.mozilla.org/zh-CN/docs/Web/API/MutationObserver/disconnect "此页面仍未被本地化, 期待您的翻译!")方法取消监听。

### takeRecords方法
takeRecords方法用来清除变动记录，即不再处理未处理的变动。该方法返回变动记录的数组。

### MutationRecord对象
DOM 每次发生变化，就会生成一条变动记录（MutationRecord 实例）。该实例包含了与变动相关的所有信息。Mutation Observer 处理的就是一个个MutationRecord实例所组成的数组。

MutationRecord对象包含了DOM的相关信息，有如下属性：
>type：观察的变动类型（attribute、characterData或者childList）。
target：发生变动的DOM节点。
addedNodes：新增的DOM节点。
removedNodes：删除的DOM节点。
previousSibling：前一个同级节点，如果没有则返回null。
nextSibling：下一个同级节点，如果没有则返回null。
attributeName：发生变动的属性。如果设置了attributeFilter，则只返回预先指定的属性。
oldValue：变动前的值。这个属性只对attribute和characterData变动有效，如果发生childList变动，则返回null。
