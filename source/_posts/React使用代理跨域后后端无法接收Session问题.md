title: React使用代理跨域后后端无法接收Session问题
categories: 随笔
tags: [JavaScript,TypeScript,React]
date: 2022-01-02 15:31:00
---
将一个 MVC 项目重构为一个前后端分离项目，前端使用了 react + axios + vite...。

在前后端分离项目中，通常都会使用代理来解决跨域问题，vite 需要在 vite.config.js 文件中配置代理：

```js
export default defineConfig({
  server: {
    // 代理配置
    proxy: {
      // 请求前缀
      '/api': {
        target: 'http://localhost:8080/server_war_exploded',
        // 开启跨域
        changeOrigin: true,
        // 正式请求时将前缀替换为空字符
        rewrite: path => path.replace(/^\/api/, '')
      }
    }
  }
})
```

配置了代理之后请求能过去了，但后端保存在 Session 中的用户信息无法拿到。检查发现请求头中并没有携带 Cookie，这是因为 axios  在跨域请求中是默认不提供凭据信息，也就是在跨域请求中不携带 cookie、HTTP认证及客户端SSL证明等。需要在 axios 中手动开启：

```js
axios.defaults.withCredentials = true;
```

开启之后再次请求后检查请求头...嗯...好像携带 Cookie 进行请求了，但后端还是无法获取 Session，再检查发现这个 Cookie 的作用范围不对，`Path` 是 `/server_war_exploded`。

```http
set-cookie: JSESSIONID=DD2CCA381F1EA1AAEEE912E3DCDC5A43; Path=/server_war_exploded; HttpOnly
```

将代理中请求的前缀改为 `/server_war_exploded` 再次发送请求就可以了

```js
export default defineConfig({
  server: {
    // 代理配置
    proxy: {
      '/server_war_exploded': {
        target: 'http://localhost:8080/server_war_exploded',
        changeOrigin: true,
        rewrite: path => path.replace(/^\/server_war_exploded/, '')
      }
    }
  }
})
```

这是因为没改之前你请求的时候地址**看着**是 `http://localhost:3000/api/login` ， 而 Cookie 作用于 `/server_war_exploded` 所以你的请求无权访问 Cookie。修改之后请求地址**看着**就是  `http://localhost:3000/server_war_exploded/login` 。

下面是请求使用代码：

```typescript
private onFinish = (values: {[index: string]: any}) => {

  this.context.http.post('/server_war_exploded/login', {}, {
    params: {
      '__method__': 'doLogin',
      ...values
    }
  })
    .then((response: AxiosResponse) => {
      const {data} = response;
      console.log(data)
    })
    .catch((err: any) => {
      console.log(err)
    })
};
```



---

后端使用 Session 保存用户登录信息，那么后端是如何确定当前会话对应的是哪一个 Session 你知道吗？

在每一次请求中，其实浏览器都会默认携带一个 Cookie `JSESSIONID` ，这个 Cookie 记录了一大串乱七八糟的字符串，后端就是通过这个字符串来确定一个会话的 Session 对象的。