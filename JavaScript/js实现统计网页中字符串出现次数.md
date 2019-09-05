# js实现统计网页中字符串出现次数

```js
function pageStrCount(str) {
    var count = 0 // 统计字数
    var allStr = document.body.innerHTML // 网页文字包含标签
    allStr = allStr.replace(/<\w+?>/g, '').replace(/<\/\w+?>/g, '') // 过滤网页标签
    console.log(allStr)
    var reg = new RegExp(str, 'g') // 匹配的文字正则
    reg.exec(allStr)
    console.log(reg.lastIndex)
    while (reg.lastIndex) {
        count++
        reg.exec(allStr)
    }
    console.log('字数统计结果：', count)
}
```
