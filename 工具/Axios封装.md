# axios封装以及取消重复请求应用

## 封装代码

```js
// axios.js
// 封装axios请求,返回重新封装的数据格式
import axios from 'axios'
import errHandle from './errHandle'

class HttpRequest {
  constructor(baseUrl) {
    this.baseUrl = baseUrl
  }

  getInsideConfig () {
    const config = {
      baseURL: this.baseUrl,
      headers: {
        'Content-Type': 'application/json;charset=utf-8'
      },
      timeout: 10000
    }
    return config
  }

  // 设定拦截器
  interceptors (instance) {
    // 添加请求拦截器
    instance.interceptors.request.use((config) => {
        // 如果有token 的话 在header添加token 如果没有（登录的情况）就不传
     if () {
            config.headers.Authorization = JSON.parse(window.localStorage.getItem("Login")).token
     }
        
      return config
    }, (error) => {
      // 对错误的统一处理
      // 对请求错误做些什么
      errHandle(error)
      return Promise.reject(error)
    })

    // 添加响应拦截器
    instance.interceptors.response.use((res) => {
      if (res.status === 200) {
        return Promise.resolve(res.data)
      } else {
        return Promise.reject(res)
      }
      // 对响应数据做点什么
    }, (error) => {
      // debugger
      errHandle(error)
      // 对响应错误做点什么
      return Promise.reject(error)
    })
  }

  // 创建实例
  request (options) {
    const instance = axios.create()
    const newOptions = Object.assign(this.getInsideConfig(), options)
    this.interceptors(instance)
    return instance(newOptions)
  }

  get (url, config) {
    const options = Object.assign({
      method: 'get',
      url
    }, config)
    return this.request(options)
  }

  post (url, data) {
    return this.request({
      method: 'post',
      url,
      data
    })
  }
}

export default HttpRequest

```

```js
// request.js
import HttpRequest from './axios'
import config from './../config'

// 根据不同环境 不同的请求url
const baseUrl = process.env.NODE_ENV === 'development' ? config.baseUrl.dev
  : config.baseUrl.prod

const axios = new HttpRequest(baseUrl)

export default axios
```

## 取消重复请求应用

开发项目中，在处理**长列表**点击加载更多，需要考虑到用户网速比较慢的情况，在请求还没有到达之前，如果再次点击加载更多，需要对这种重复请求进行取消。

`axios.js`

```js
const CancelToken = axios.CancelToken
constructor(baseUrl) {
    this.baseUrl = baseUrl
    this.pending = {} // ++
}
// ------------------------- 
// key 判断是否相同的请求
// isRequest 判断请求是否已经得到响应
// this.pending[key] 判断是否发送了
removePending(key, isRequest = false) {
    if (this.pending[key] && isRequest) {
        this.pending[key]('取消重复请求!')
    }
    delete this.pending[key]
}
// ------------------------- 
// 判断上一次的请求是否有回应 请求拦截器
instance.interceptors.request.use((config) => {
    const key = config.url + '&' + config.method
    this.removePending(key, true)
    
    // 核心.. 这个 c 个人理解为：调用则会取消此次请求。
    config.cancelToken = new CancelToken(c => {
        this.pending[key] = c
    })
    return config
},

// -------------------------                
                                  instance.interceptors.response.use((res) => {
    const key = res.config.url + '&' + res.config.method
    this.removePending(key) // 默认 isRequest = false
 
    if (res.status === 200) {
        return Promise.resolve(res.data)
    } else {
        return Promise.reject(res)
    }
    ...
})
```

组件中发送请求：

```js
getList(options).then(res => {
    // ...do sth
}).catch(e => {
    if (e) {
        this.$alert(e.message)
    }
    // e.message(则是调用上面的c传的参数)
})
```

整个流程就是 提交一次请求，此时`this.pending[key]` 有值，如果用户因为网速问题 (可以在浏览器network修改网速调试) 还没有得到响应，再次点击发送请求的时候，`this.pending[key]`就会执行` this.$alert(e.message) 会被执行  `。如果有响应的话，会`delete this.pending[key]`，这样就完成了重复请求的取消，并且对用户进行了提示。

