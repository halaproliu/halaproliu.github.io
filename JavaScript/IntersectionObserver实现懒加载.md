# IntersectionObserver实现懒加载

前端开发过程中，为了丰富页面的表现力，总是需要插入各种各样的图片。但是图片过多，会给页面显示性能带来一些影响。

传统方式是监听scroll事件，再调用[`getBoundingClientRect()`](https://developer.mozilla.org/en/docs/Web/API/Element/getBoundingClientRect)
方法判断元素左上角的坐标是否在视窗内，但是scroll事件触发的太密集，会造成内存不断消耗，带来性能问题。

现在有一个API，IntersectionObserver可以实现监听元素是否可见。

### 实现懒加载

interserverObserver对象可以通过isIntersecting或者intersectionRatio是否大于0判断元素是否在视窗内，并通过data-src来实现懒加载，以下为具体实现：

```html
<ul class="box">
  <li><img src="http://placehold.it/450x300/caaa8e/ccc.png" data-src="https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1549272664908&di=4bb90ffd078e31348159c07e78947f0a&imgtype=0&src=http%3A%2F%2Fb-ssl.duitang.com%2Fuploads%2Fitem%2F201610%2F08%2F20161008151749_2VAKU.jpeg" alt=""></li>
  <li><img src="http://placehold.it/450x300/caaa8e/ccc.png" data-src="https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1549272664908&di=4bb90ffd078e31348159c07e78947f0a&imgtype=0&src=http%3A%2F%2Fb-ssl.duitang.com%2Fuploads%2Fitem%2F201610%2F08%2F20161008151749_2VAKU.jpeg" alt=""></li>
  <li><img src="http://placehold.it/450x300/caaa8e/ccc.png" data-src="https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1549272664908&di=4bb90ffd078e31348159c07e78947f0a&imgtype=0&src=http%3A%2F%2Fb-ssl.duitang.com%2Fuploads%2Fitem%2F201610%2F08%2F20161008151749_2VAKU.jpeg" alt=""></li>
  <li><img src="http://placehold.it/450x300/caaa8e/ccc.png" data-src="https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1549272664908&di=4bb90ffd078e31348159c07e78947f0a&imgtype=0&src=http%3A%2F%2Fb-ssl.duitang.com%2Fuploads%2Fitem%2F201610%2F08%2F20161008151749_2VAKU.jpeg" alt=""></li>
  <li><img src="http://placehold.it/450x300/caaa8e/ccc.png" data-src="https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1549272664908&di=4bb90ffd078e31348159c07e78947f0a&imgtype=0&src=http%3A%2F%2Fb-ssl.duitang.com%2Fuploads%2Fitem%2F201610%2F08%2F20161008151749_2VAKU.jpeg" alt=""></li>
  <li><img src="http://placehold.it/450x300/caaa8e/ccc.png" data-src="https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1549272664908&di=4bb90ffd078e31348159c07e78947f0a&imgtype=0&src=http%3A%2F%2Fb-ssl.duitang.com%2Fuploads%2Fitem%2F201610%2F08%2F20161008151749_2VAKU.jpeg" alt=""></li>
  <li><img src="http://placehold.it/450x300/caaa8e/ccc.png" data-src="https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1549272664908&di=4bb90ffd078e31348159c07e78947f0a&imgtype=0&src=http%3A%2F%2Fb-ssl.duitang.com%2Fuploads%2Fitem%2F201610%2F08%2F20161008151749_2VAKU.jpeg" alt=""></li>
  <li><img src="http://placehold.it/450x300/caaa8e/ccc.png" data-src="https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1549272664908&di=4bb90ffd078e31348159c07e78947f0a&imgtype=0&src=http%3A%2F%2Fb-ssl.duitang.com%2Fuploads%2Fitem%2F201610%2F08%2F20161008151749_2VAKU.jpeg" alt=""></li>
  <li><img src="http://placehold.it/450x300/caaa8e/ccc.png" data-src="https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1549272664908&di=4bb90ffd078e31348159c07e78947f0a&imgtype=0&src=http%3A%2F%2Fb-ssl.duitang.com%2Fuploads%2Fitem%2F201610%2F08%2F20161008151749_2VAKU.jpeg" alt=""></li>
  <li><img src="http://placehold.it/450x300/caaa8e/ccc.png" data-src="https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1549272664908&di=4bb90ffd078e31348159c07e78947f0a&imgtype=0&src=http%3A%2F%2Fb-ssl.duitang.com%2Fuploads%2Fitem%2F201610%2F08%2F20161008151749_2VAKU.jpeg" alt=""></li>
  <li><img src="http://placehold.it/450x300/caaa8e/ccc.png" data-src="https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1549272664908&di=4bb90ffd078e31348159c07e78947f0a&imgtype=0&src=http%3A%2F%2Fb-ssl.duitang.com%2Fuploads%2Fitem%2F201610%2F08%2F20161008151749_2VAKU.jpeg" alt=""></li>
  <li><img src="http://placehold.it/450x300/caaa8e/ccc.png" data-src="https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1549272664908&di=4bb90ffd078e31348159c07e78947f0a&imgtype=0&src=http%3A%2F%2Fb-ssl.duitang.com%2Fuploads%2Fitem%2F201610%2F08%2F20161008151749_2VAKU.jpeg" alt=""></li>
  <li><img src="http://placehold.it/450x300/caaa8e/ccc.png" data-src="https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1549272664908&di=4bb90ffd078e31348159c07e78947f0a&imgtype=0&src=http%3A%2F%2Fb-ssl.duitang.com%2Fuploads%2Fitem%2F201610%2F08%2F20161008151749_2VAKU.jpeg" alt=""></li>
  <li><img src="http://placehold.it/450x300/caaa8e/ccc.png" data-src="https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1549272664908&di=4bb90ffd078e31348159c07e78947f0a&imgtype=0&src=http%3A%2F%2Fb-ssl.duitang.com%2Fuploads%2Fitem%2F201610%2F08%2F20161008151749_2VAKU.jpeg" alt=""></li>
  <li><img src="http://placehold.it/450x300/caaa8e/ccc.png" data-src="https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1549272664908&di=4bb90ffd078e31348159c07e78947f0a&imgtype=0&src=http%3A%2F%2Fb-ssl.duitang.com%2Fuploads%2Fitem%2F201610%2F08%2F20161008151749_2VAKU.jpeg" alt=""></li>
  <li><img src="http://placehold.it/450x300/caaa8e/ccc.png" data-src="https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1549272664908&di=4bb90ffd078e31348159c07e78947f0a&imgtype=0&src=http%3A%2F%2Fb-ssl.duitang.com%2Fuploads%2Fitem%2F201610%2F08%2F20161008151749_2VAKU.jpeg" alt=""></li>
  <li><img src="http://placehold.it/450x300/caaa8e/ccc.png" data-src="https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1549272664908&di=4bb90ffd078e31348159c07e78947f0a&imgtype=0&src=http%3A%2F%2Fb-ssl.duitang.com%2Fuploads%2Fitem%2F201610%2F08%2F20161008151749_2VAKU.jpeg" alt=""></li>
  <li><img src="http://placehold.it/450x300/caaa8e/ccc.png" data-src="https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1549272664908&di=4bb90ffd078e31348159c07e78947f0a&imgtype=0&src=http%3A%2F%2Fb-ssl.duitang.com%2Fuploads%2Fitem%2F201610%2F08%2F20161008151749_2VAKU.jpeg" alt=""></li>
</ul>
```

```css
li {
  list-style: none;
}

img {
  width: 450px;
  height: 300px;
}
```

```js
const imgs = document.querySelectorAll('img')
const io = new IntersectionObserver(entries => {
  entries.forEach(entry => {
    if (entry && entry.isIntersecting) {
      entry.target.src = entry.target.dataset.src
      io.unobserve(entry.target)
    }
  })
})
imgs.forEach(item => {
  io.observe(item)
})
```
IntersectionObserver还有许多其他功能，各位若有兴趣，可以去具体研究下IntersectionObserver这个对象。

目前IntersectionObserver已经在chrome51+实现了。
