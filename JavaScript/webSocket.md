# webSocket入门（消息中间件开发）

## 什么是webSocket

<font color=red>WebSocket</font>是一种网络传输协议，可在单个TCP链接上进行<font color=red>全双工</font>通信，位于OSI模型的应用层

特点：

- TCP链接，与HTTP协议兼容
- 双向通信，主动推送（服务端向客户端）
- 无同源限制，协议的标识符是 ws （加密wss）

应用场景：

- 聊天、消息、点赞

- 直播评论（弹幕）

- **游戏、协同编辑、基于位置的应用、在线多人编辑功能**

ws常用库

- ws（实现原生协议，特点：通用、性能高、定制性强、**较稳定**）
- socket.io（向下兼容协议，特点：适配性强、性能一般）

## 第一个简单WebSocket应用

### ws

创建目录结构

- demo01
  - client
    - index.html
  - server
    - 通过 `npm init -y` 初始化
    - index.js

```js
// 先在server文件夹 npm i ws -S
// server/index.js
const webSocket = require('ws')
const wss = new webSocket.Server({port: 3000})
wss.on('connection',(ws) => {
  console.log('one client is connected')
  // 接收客户端消息
  ws.on('message',(msg)=>{
    console.log(msg)
  })
  // 发消息
  ws.send('message from server')
})
```

客户端代码

```html
<script>
    var ws = new WebSocket("ws://localhost:3000")
    ws.onopen = () => {
      ws.send('hello from client!')
    }
    ws.onmessage = (event) => {
      console.log(event)
    }
</script>
```

命令行 node index.js 运行 server 

使用vscode插件：`Live Server`  开启index.html 控制台可以看到发送的信息。

### socket.io

首先 ` npm i socket.io -S  `

广播：同时打开两个客户端窗口，一个窗口发送的数据将由广播的形式在其他窗口的控制台打印

index.js

```js
const app = require('express')()
const http = require('http').createServer(app)
const io = require('socket.io')(http)
// 静态服务器
app.get('/',(req,res)=>{
  res.sendFile(__dirname+"/index.html")
})

io.on('connection',(socket)=>{
  console.log('a socker is connected')
  socket.on('chatEvent',function(msg){
    console.log('msg from client' + msg)
    // socket.send('server says:' + msg)
    // 广播
    socket.broadcast.emit('ServerMsg', msg)
  })
})

http.listen(3000,()=>{
  console.log('server at 3000')
})
```

index.html

```html
<script src="/socket.io/socket.io.js"></script>

<body>
    hello server
    <input type="text" id="msg">
    <button type="button" id="submit">发送</button>
    <script>
      var socket = io()
      const obtn = document.querySelector("#submit")
      const oinput = document.querySelector("#msg")
      obtn.addEventListener('click', () => {
        const value = oinput.value
        socket.emit('chatEvent', value)
        oinput.value = ''
      })

      // 普通接收
      // socket.on('message', (msg) => {
      //   console.log(msg)
      // })

      // 广播  这个ServerMsg需对应 index.js socket.broadcast.emit派发的事件
      socket.on('ServerMsg', function (msg) {
        console.log(msg)
      })
    </script>
</body>
```

可以看出，socket.io是通过emit和on派发事件，去接受和发送信息。

### 一个简易入门版聊天室

...

###  一个简易多人聊天室

...

### ws如何鉴权

[ws](https://www.npmjs.com/package/ws#client-authentication)

