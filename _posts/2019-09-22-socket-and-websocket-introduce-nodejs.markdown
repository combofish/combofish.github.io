---
layout: post
title: "Socket和WebSocket使用(nodejs)"
date: 2019-09-22
tag: Node.js Socket WebSocket 
---

- [基础函数](#org3608edb)
  - [trim()](#orgb91615f)
- [net](#org1cf5fe3)
  - [socket 类](#org38c9248)
  - [server 端](#org5daef10)
  - [client 端](#orga803025)
  - [socket.pipe()](#orga446b14)
- [WebSocket](#org59ed88d)
  - [与 HTTP 的关系](#org136e34c)
  - [与socket 的不同](#orga47a9f0)
  - [socket.io](#org04e8fab)
  - [websocket server](#org34b02ba)
  - [websocket client](#org6d518b6)
  - [参考连接](#org3795cc1)


<a id="org3608edb"></a>

# 基础函数


<a id="orgb91615f"></a>

## trim()

-   去除头尾字符串
-   不会改变原始字符串
    
    ```javascript
    var str = "    hello world   "
    console.log(str.trim())
    coneole.log(str)
    ```


<a id="org1cf5fe3"></a>

# net

-   net 模块提供一个原始的TCP/IP socket接口。
-   http模块在net模块的基础之上实现了HTTP协议。


<a id="org38c9248"></a>

## socket 类

socket类同时用在 net模块的客户端和服务器端，它是Stream的子类，即可读也可写，当有数据要读出来时， 触发data事件，当要发送输出数据时它有write(),end()函数。


<a id="org5daef10"></a>

## server 端

```javascript
var  net = require('net')

net.createServer((socket) => {
    socket.write('Hello World!\r\n')
    socket.end()
}).listen(1337)

console.log('Listen on port 1337!')

```


<a id="orga803025"></a>

## client 端

```javascript
var net = require('net')

var socket = net.connect({host: process.argv[2], port: 22})
socket.setEcoding('utf8')
socket.once('data',(chunk) => {
    console.log('SSH server version:%j', chunk.trim())
    socket.end();
})

```


<a id="orga446b14"></a>

## socket.pipe()

```javascript
var  net = require('net')

net.createServer((socket) => {
    console.log('socket connect')

    socket.on('data',(data) => {
        console.log('"data" event',data)
    })
    socket.on('end',() =>{
        console.log('"end" event')
    })
    socket.on('close',() =>{
        console.log('"close event')
    })
    socket.on('error',(e) => {
        console.log('"error" event',e)
    })
    socket.pipe(socket)

}).listen(1337)

console.log('Listen on port 1337!')
```

-   net.connect()

指定一个 host, port 值的配置项参数，并返回一个socket实例，返回的socket 开始并没有连接到服务器上， 所以一般要在作神魔事之前监听connect事件

```javascript
var net = require('net'),
    host = process.argv[2],
    port = Number(process.argv[3])

var socket = net.connect(port,host)

socket.on('connect',() => {
    process.stdin.pipe(socket)
    socket.pipe(process.stdout)
    process.stdin.resume()
})
socket.on('end'() =>{
    process.stdin.pause()
})


```


<a id="org59ed88d"></a>

# WebSocket

WebSocket 协议本质上是一个基于TCP的协议，它由通信协议和API 组成，能够在浏览器和服务器之间建立双向连接，以基于 事件的方式，赋予浏览器实时通信的能力。双向通信，服务器端和客户端之间可以同时发送并响应请求，而不在像HTTP的请求和响应。


<a id="org136e34c"></a>

## 与 HTTP 的关系

-   HTTP 协议是一个懒惰的协议，server 只有在接收到请求时才会作出反应。
-   两者都是应用层协议，都是基于TCP 协议，与HTTP 兼容，却不融于HTTP 协议，仅作为 HTML5的一部分，
-   websocket建立连接需要借助HTTP 协议，建立好后，client 与 server 的双向通信就与HTTP无关了。


<a id="orga47a9f0"></a>

## 与socket 的不同

WebSocekt 是 HTML5 规范中的一部分，其借鉴了 socket 的思想， 为 client 和 server 之间提供了类似的双向通信机制。 同时，WebSocket 又是一种新的应用层协议，包含一套标准的 API； 而 socket 并不是一个协议，而是一组接口，其主要方便大家直接使用更底层的协议（比如 TCP 或 UDP）


<a id="org04e8fab"></a>

## socket.io

socket.io 是一个开源的websocket库，它通过nodejs 实现websocket服务端，同时也提供客户端js库。 socket.io 支持 websocket,htmlfile,xhr-polling,jsonp-polling,协议。

```sh
npm install --save express
npm install --save socket.io
```


<a id="org34b02ba"></a>

## websocket server

```javascript
var app = require('express')(),
    http = require('http').Server(app),
    io = require('socket.io')(http)

app.get('/',(req,res) =>{
    res.send('<h1>Welcome Realtime Server</h1>')
})

var onlineUsers = {},
    onlineClient = 0

io.on('connection',(socket) =>{
    console.log('a user connected')
    socket.on('login',(obj) =>{
        io.emit('login',{onlineUsers:onlineUsers,onlineClient:onlineClient,user:obj})
        console.log(obj.username + 'join')
    })

    socket.on('disconnect',() =>{
        if (onlineUsers.hasOwnProperty(socket.name)){
            io.emit('logout',{onlineUsers:onlineUsers,onlineClient,onlineClient,user:obj})
            console.log(obj.username + 'out')
        }
    })

    socket.on('message',(obj) =>{
        io.emit('message',obj)
        console.log(obj.username + 'say : ' + obj.content)
    })
})

http.listen(3000,()=>{
    console.log('listening on 3000')
})
```


<a id="org6d518b6"></a>

## websocket client


<a id="org3795cc1"></a>

## 参考连接

-   第十二天 长连接(net和socket.io) <https://www.jianshu.com/p/6966f12284b5>
-   WebSocket 与 Socket.IO <https://zhuanlan.zhihu.com/p/23467317>
