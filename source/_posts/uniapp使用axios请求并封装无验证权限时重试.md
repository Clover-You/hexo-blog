title: uniapp使用axios请求并封装无验证权限时重试
categories: 小玩意
tags: [JavaScript,TypeScript,Vue]
date: 2022-01-02 15:44:45
---
在uniapp中，使用axios进行请求时，uniapp无法使用axios的适配器，需要基于`uni.request`来定义适配器。
安装完成axios后在项目utils目录下建一个axios文件夹
> 文中根目录代表utils中的axios文件夹

在根目录新建一个axios.js文件，在该文件中配置一个新的axios
```javascript
import axios from "axios";
const service = axios.create({
  withCredentials: true,
  crossDomain: true,
  baseURL: '***',
  timeout: 6000
})

```

在根目录建一个lib文件夹，在这个文件夹里建一个adapter.js文件，该文件配置了基于uniapp的axios适配，记得抛出这个适配器

```javascript

import settle from "axios/lib/core/settle"
import buildURL from "axios/lib/helpers/buildURL"

/* 格式化路径 */
const URLFormat = function (baseURL, url) {
  return url.startsWith("http") ? url : baseURL
}

/* axios适配器配置 */
const adapter = function (config) {
  return new Promise((resolve, reject) => {
    uni.request({
      method: config.method.toUpperCase(),
      url: buildURL(URLFormat(config.baseURL, config.url), config.params, config.paramsSerializer),
      header: config.headers,
      data: config.data,
      dataType: config.dataType,
      responseType: config.responseType,
      sslVerify: config.sslVerify,
      complete: function complete(response) {
        response = {
          data: response.data,
          status: response.statusCode,
          errMsg: response.errMsg,
          header: response.header,
          config: config
        };
        settle(resolve, reject, response);
      }
    })
  })
}

export default adapter;
```
在根目录的axios.js文件中，使用这个适配器并设置重新发起请求的次数以及每次重新请求的间隔时间
```javascript
import adapter from "./lib/adapter"
service.defaults.adapter = adapter;
service.defaults.retry = 5; // 设置请求次数
service.defaults.retryDelay = 1000;// 重新请求时间间隔
```

设置一个请求完成后的拦截器，如果响应头中的状态码为200表示成功，将请求得到的数据返回，否则一律视为错误请求，需要返回一个Promise。在lib中建立一个axiosError.js在里面处理失败的请求。
```javascript
service.interceptors.response.use(res => {
  if (res.status == 200) {
    return res;
  } else {
    return Promise.reject(res);
  }
}, err => axiosError(err, service))
```
axiosError.js中需要传入axios拦截器截到的错误以及我们新创建的这个axios，这个错误处理页面只是充当一个分配器的角色，我们可以根据响应头中的状态进行处理该错误，未处理的错误在使用时处理，返回Promise.reject
```javascript
// 处理401错误代码
import Error401 from "../handlers/401Error";

export default function (err, axios) {
  const errStatus = err.response.status;
  if (errStatus == 401) {
    return Error401(err); // 401没有权限，重新请求设置token
  } else {
    return Promise.reject(err);
  }
}
```
处理401错误代码，当请求失败并且响应头中的状态码为401时，是我没没有权限去请求，可以根据项目来进行处理，我们是需要携带token，所以401为token未携带或失效，请求时无需传入token，axios遇到401会自动携带这个token重新去请求。在根目录建一个handlers文件夹，在里面建一个401Error.js用于处理401的错误。
这里使用到Vuex，需要引入Vuex，因为获取token、设置token的方法以及token都放在里面！！！使用`store.dispatch("getToken")`得到token后将token设置到请求头Authorization
```javascript
import interceptorsError from "../lib/interceptorsError";
import store from 'store/index'

/* 需要传入axios错误配置 */
export default function (err, axios) {
  const config = err.config;// axios请求配置
  store.dispatch("getToken").then(function () {
    config.headers["Authorization"] = store.state.cnrToken.cnr_token;
  });
  err.config = config;
  return interceptorsError(axios, config);
}
```
一切准备就绪之后需要重新请求，在根目录建一个interceptorsError.js文件，用于重新执行请求，这个方法需要一个请求配置，只需要把我们上一个请求的配置传入即可。

```javascript
export default function (axios, config) {
  // 如果配置不存在或未设置重试选项，reject
  if (!config || !config.retry) return Promise.reject(err);
  // 设置变量以跟踪重试计数
  config.__retryCount = config.__retryCount || 0;
  // 如果重试次数大于最大重试次数，reject
  if (config.__retryCount >= config.retry) {
    return Promise.reject(err);
  }
  // 每重试一次记录一次重试次数
  config.__retryCount += 1;
  // 重试间隔时间
  const backoff = new Promise(function (resolve) {
    setTimeout(function () {
      resolve();
    }, config.retryDelay || 1000);
  });

  return backoff.then(function () {
    return axios(config);
  });
}
```

这是我Vuex中的代码
```javascript
/*
 * @Author: UpYou
 * @Date: 2020-09-25 16:30:13
 * @LastEditTime: 2020-09-25 21:32:56
 * @Descripttion: token
 * 
 */

const state = {
  cnr_token: '',
  POST_KEYS: {
    ...获取token需要的验证信息...
  }
};

const mutations = {
  /* 设置token */
  SET_CNRTOKEN(state, Payload) {
    if (Payload.startsWith("Bearer"))
      state.cnr_token = Payload;
    else state.cnr_token = "Bearer" + Payload;
  }
}

const actions = {
  /* 向服务器获取token */
  getToken(context, args) {
    return new Promise(function (resolve, reject) {
      uni.request({
        url: "token服务器地址",
        data: { ...context.state.POST_KEYS },
        method: "get",
        async success(res) {
          await context.commit('SET_CNRTOKEN', res.data.access_token)
          await resolve(res.data)
        },
        fail: reject
      });
    })
  }
}

export default {
  state, mutations, actions,
}
```

[源码下载](https://github.com/Clover-You/fornt-end-quick-demo/blob/542fa04a90407086eab4471b7ab07b71ff46fc99/uniapp%E4%BD%BF%E7%94%A8axios%E8%AF%B7%E6%B1%82%E5%B9%B6%E5%B0%81%E8%A3%85%E6%97%A0%E9%AA%8C%E8%AF%81%E6%9D%83%E9%99%90%E6%97%B6%E9%87%8D%E8%AF%95.zip)