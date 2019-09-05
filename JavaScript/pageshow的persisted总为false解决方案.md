# pageshow的persisted总为false解决方案

之前在项目中使用pageshow，发现页面返回的时候persisted依然为false，这时候只好找其他方案解决。
这时候发现有一个window.performance对象，performance.navigation.type是一个无符号短整型
TYPE_NAVIGATE (0)：
当前页面是通过点击链接，书签和表单提交，或者脚本操作，或者在url中直接输入地址，type值为0
TYPE_RELOAD (1)
点击刷新页面按钮或者通过Location.reload()方法显示的页面，type值为1
TYPE_BACK_FORWARD (2)
页面通过历史记录和前进后退访问时。type值为2
TYPE_RESERVED (255)
任何其他方式，type值为255

这真是我们需要的部分，于是可以预见，解决方案如下：
```js
window.addEventListener('pageshow', () => {
  if (e.persisted || (window.performance && 
    window.performance.navigation.type == 2)) {
    location.reload()
  }
}, false)
```
