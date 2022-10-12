---
title: http的长连接VSwebsocket的长连接
date: 2022-08-03 10:27:18
cover: 'https://img2.baidu.com/it/u=3240559155,1066004125&fm=253&fmt=auto&app=138&f=JPEG?w=773&h=500'
tags: 
- http长连接
- websocket
- socket
---
{% blockquote 诺浅 https://bxoon.blog.csdn.net/article/details/97943282?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1-97943282-blog-106675621.pc_relevant_show_downloadRating&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1-97943282-blog-106675621.pc_relevant_show_downloadRating&utm_relevant_index=1 %}

### HTTP 协议的“缺陷”
HTTP 协议只能是客户端向服务器发出请求，服务器返回查询结果。HTTP 协议做不到服务器主动向客户端推送信息的， 这种单向请求的特点，注定了如果服务器有连续的状态变化，客户端要获知就非常麻烦，客户端只能使用"轮询"的方式，即每隔一段时间，就发出一个请求，请求服务器有没有状态变化 。可见轮询的效率低，非常浪费资源。
在http1.1中，默认开启了一个叫Keep-alive的参数，官方的说法是可以用这个来作为长连接。

#### 关于Keep-alive的缺点
Keep-alive的确可以实现长连接，但是这个长连接是有问题的，本质上依然是客户端主动发起-服务端应答的模式，是没法做到服务端主动发送通知给客户端的。也就是说，在一个HTTP连接中，客户端可以发送多个Request，接收多个Response。但是请记住 Request = Response ， 在HTTP中永远是这样，一个request只能有一个response。而且这个response也是被动的，不能主动发起。
需要注意的是Keep-alive是指不需要多次建立连接，而非http2.0里面的多路复用，多路复用的意思是客户端可以同时向服务器端发送多个请求，服务器端也可以同时响应多个Response.

### 关于websocket
相对于传统 HTTP每次请求-应答都需要客户端与服务端建立连接的模式，WebSocket是类似 Socket 的 TCP 长连接的通讯模式，一旦 WebSocket 连接建立后，后续数据都以帧序列的形式传输。在客户端断开 WebSocket 连接或 Server 端断掉连接前，不需要客户端和服务端重新发起连接请求。在海量并发及客户端与服务器交互负载流量大的情况下，`极大的节省了网络带宽资源的消耗`，有明显的性能优势，且客户端发送和接受消息是在同一个持久连接上发起，`实时性优势明显`。
WebSocket API 是 HTML5 标准的一部分， 但这并不代表 WebSocket 一定要用在 HTML 中，或者只能在基于浏览器的应用程序中使用。在WebSocket中，只需要服务器和浏览器通过TCP协议进行一个握手的动作，然后单独建立一条TCP的通信通道进行数据的传送。WebSocket同HTTP一样也是应用层的协议，但是它是一种`双向通信协议`，是建立在TCP之上的。
- websocket的流程大概是以下几步
    - 浏览器、服务器建立TCP连接，三次握手。这是通信的基础，传输控制层，若失败后续都不执行
    - TCP连接成功后，浏览器通过HTTP协议向服务器传送WebSocket支持的版本号等信息。（开始前的HTTP握手）
    - 服务器收到客户端的握手请求后，同样采用HTTP协议回馈数据
    - 当收到了连接成功的消息后，通过TCP通道进行传输通信
也就是说WebSocket在建立握手时，数据是通过HTTP传输的。但是建立之后，在真正传输时候是不需要TCP协议的。

### WebSocket与Socket的关系
Socket其实并不是一个协议，而是为了方便使用TCP或UDP而抽象出来的一层，是位于应用层和传输控制层之间的一组接口
Socket是应用层与TCP/IP协议族通信的中间软件抽象层，它是一组接口。在设计模式中，Socket其实就是一个门面模式，它把复杂的TCP/IP协议族隐藏在Socket接口后面，对用户来说，一组简单的接口就是全部，让Socket去组织数据，以符合指定的协议。
当两台主机通信时，必须通过Socket连接，Socket则利用TCP/IP协议建立TCP连接。TCP连接则更依靠于底层的IP协议，IP协议的连接则依赖于链路层等更低层次。

WebSocket则是一个典型的应用层协议。
socket是对网络协议实现的封装，长连接是tcp协议实现的，所以可以使用socket实现长连接。
{% endblockquote %}