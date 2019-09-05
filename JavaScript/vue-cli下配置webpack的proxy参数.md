# vue-cli下配置webpack的proxy参数

在平时本地开发过程中，容易遇到跨域问题，导致接口无法调通。这时候有多种方式解决，
1. 后端支持CORS
2. nginx反向代理
3. 使用jsonp请求
4. 使用http-proxy-middleware代理
下面介绍的这种方式是webpack自带的devServer，devServer中集成了http-proxy-middleware。配置devServer的proxy选项即可。代码如下：
```js
// 以下代理作用 =>  替换/api为localhost:3000/api，并且修改cookie的path
proxy: {
  '/api': {
    target: 'http://localhost:3000',
    changeOrigin: true,
    onProxyRes: function(proxyRes, req, res) {
      var cookies = proxyRes.headers['set-cookie'];
      var cookieRegex = /Path=\/XXX\//i;
      //修改cookie Path
      if (cookies) {
        var newCookie = cookies.map(function(cookie) {
          if (cookieRegex.test(cookie)) {
            return cookie.replace(cookieRegex, 'Path=/');
          }
          return cookie;
        });
        //修改cookie path
        delete proxyRes.headers['set-cookie'];
        proxyRes.headers['set-cookie'] = newCookie;
      }
    }
  }
}
```
假设使用axios进行http请求
```js
const api = axios.create({
    baseURL: '/api'
})
```
而接口配置类似如下，则会请求http://localhost:3000/api/getMenuList的接口：
```js
export default {
    // 获取菜单列表
    async getMenuList (params) {
        let res = await api.get('/getMenuList', {
            params: params
        })
        return res
    }
}
```
proxy还有一个rewrite功能,如下
以下代理作用 =>  替换/api为localhost:3000，并且修改cookie的path
```js
proxy: {
  '/api': {
    target: 'http://localhost:3000',
    changeOrigin: true,
    pathRewrite: {
       '^/api', ''
    }
    onProxyRes: function(proxyRes, req, res) {
      var cookies = proxyRes.headers['set-cookie'];
      var cookieRegex = /Path=\/XXX\//i;
      //修改cookie Path
      if (cookies) {
        var newCookie = cookies.map(function(cookie) {
          if (cookieRegex.test(cookie)) {
            return cookie.replace(cookieRegex, 'Path=/');
          }
          return cookie;
        });
        //修改cookie path
        delete proxyRes.headers['set-cookie'];
        proxyRes.headers['set-cookie'] = newCookie;
      }
    }
  }
}
```

