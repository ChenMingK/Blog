# 第 6 章 理解 Buffer
在网络流和文件的操作中需要处理大量二进制数据，JavaScript 自有的字符串远远不能满足这些需求，于是 Buffer 对象应运而生。

Buffer 是一个像 Array 的对象，但它主要用于操作字节，其是一个典型的 JavaScript 与 C++ 结合的模块，它将性能相关部分用 C++ 实现，将非性能相关部分应用 JavaScript 实现。

Node 在进程启动时就已经加载了 Buffer 模块，并将其放在全局对象 global 上，使用时无须 require。
## Buffer 对象
Buffer 对象类似于数组，它的元素为 16 进制的两位数，即 0 到 255 的数值
```js
let str = '深入浅出node.js'
let buf = new Buffer(str, 'utf-8')
console.log(buf)
// => <Buffer e6 b7 b1 e5 85 a5 e6 b5 85 e5 87 ba 6e 6f 64 65 2e 6a 73>
```
可以看到，不同编码的字符串占用的元素个数不同，中文字在 UTF-8 编码下占用 3 个元素，字母和半角标点符号占用 1 个元素。

Buffer 与 Array 类型很相似，可以访问 length 属性得到长度，也可以通过下标访问元素，构造对象时也十分相似，代码如下：
```js
let buf = new Buffer(100) // 分配一个长 100 字节的 Buffer 对象
console.log(buf.length) // 100

console.log(buf[10]) // 初始化为 0?

buf[10] = 100 // 可以通过下标进行赋值
console.log(buf[10]) // => 100

// 给元素的赋值如果小于 0，就将该值逐次加 256，直到得到一个 0 ~ 255 之间的整数；
// 如果得到的数值大于 255，就逐次减 256，直到得到 0 ~ 255 区间内的整数；
// 如果是小数，舍弃小数部分，只保留整数部分
buf[20] = -100
console.log(buf[20]) // 156
buf[21] = 300
console.log(buf[21]) // 44
buf[22] = 3.1415
console.log(buf[22]) // 3
```
## Buffer 内存分配
Buffer 对象的内存分配不是在 V8 的堆内存中，而是在 Node 的 C++ 层面实现内存的申请的。

因为处理大量的字节数据不能采用需要一点内存就向操作系统申请一点内存的方式，这可能造成大量的内存申请的系统调用，对操作系统有一定压力。

为此 Node 在内存的使用上应用的是在 C++ 层面申请内存，在 JavaScript 中分配内存的策略。

为了高效地使用申请来的内存，Node 采用了 slab 分配机制，这里不做记录了。

## Buffer 的转换
Buffer 对象可以与字符串之间相互转换，其支持的字符串编码类型包括以下几种：
- ASCII
- UTF-8
- UTF-16LE/UCS-2
- Base64
- Binary
- Hex

字符串转 Buffer：通过构造函数完成，存储的只能是一种编码类型。
```js
new Buffer(str, [encoding]) // encoding 参数不传递时，默认按 UTF-8 编码进行转码和存储
```
Buffer 转字符串：Buffer 对象的 toString() 方法可以将 Buffer 对象转换为字符串：
```js
buf.toString([encoding], [start], [end])
// encoding 默认为 UTF-8
// start、end 可实现局部的转换
```
## Buffer 的拼接
......
## Buffer 与性能
......

# 第 7 章 网络编程
## 构建 TCP 服务
```js
// 构造 TCP 服务器
const net = require('net')
const server = net.createServer(function (socket) {
  // 新的连接
  socket.on('data', function (data) {
    socket.write('你好')
  })

  socket.on('end', function () {
    console.log('连接断开')
  })

  socket.write('TCP 连接示例 \n')
})

server.listen(8124, function () {
  console.log('server bound')
})
```
```js
// 构造 TCP 客户端
const net = require('net')
const client = net.connect({ port: 8124 }, function () { // 'connect' listener
  console.log('client connected')
  client.write('world!\r\n')
})

client.on('data', function (data) {
  console.log(data.toString())
  client.end()
})

client.on('end', function () {
  console.log('client disconnected')
})
```
TCP 服务的事件：在上面的示例中，代码分为服务器事件和连接事件。

1.服务器事件，通过 net.createServer() 创建的服务器而言，它具有如下事件
- listening: 调用 server.listen() 后触发
- connection：每个客户端套接字连接到服务器端时触发
- close：服务器关闭时触发
- error：服务器发生异常时触发

2.连接事件：服务器可以同时与多个客户端保持连接，每个连接是典型的**可读可写 Stream 对象**，既可以通过 data 事件从一端读取另一端发来的数据，也可以通过 write() 方法从一端向另一端发送数据，它具有如下事件：
- data：当一端调用 write() 发送数据时，另一端会触发 data 事件，事件传送的数据即是 write() 发送的数据
- end：当连接中的任意一端发送了 FIN 数据时，将会触发该事件
- connect：与服务器端连接成功时触发
- drain：当任意一端调用 write() 发送数据时，当前这端会触发该事件
- error：异常发生时触发
- close：套接字完全关闭时触发
- timeout：当一定时间后连接不再活跃时，该事件将会被触发，通知用户当前该连接已经被闲置了
## 构建 UDP 服务
TCP 中所有的会话基于连接完成，UDP 中一个套接字可以与多个 UDP 服务通信，其提供面向事务的简单不可靠信息传输服务，在网络差的情况下存在丢包严重的问题，但是由于它无须连接，资源消耗低，处理快速且灵活，所有常用于那种即使丢失一两个数据包也不会产生重大影响的场景，比如音频、视频等。DNS 服务也是基于 UDP 实现的。
```js
// 构建 UDP 服务器
// UDP 套接字一旦创建，既可以作为客户端发送数据，也可以作为服务器端接收数据
const dgram = require('dgram')
const server = dgram.createSocket('udp4')

server.on('message', function (msg, rinfo) {
  console.log(`server got: ${msg} from ${rinfo.address}:${rinfo.port}`)
})

server.on('listening', function () {
  const address = server.address()
  console.log(`server is listening at ${address.address}: ${address.port}`)
})

server.bind(41234)
// 该套接字将接收所有网卡上 41234 端口上的消息，绑定完成后将触发 listening 事件
```
```js
// 构建 UDP 客户端
const dgram = require('dgram')

const message = new Buffer('深入浅出Node.js')
const client = dgram.createSocket('udp4')
client.send(message, 0, message.length, 41234, 'localhost', function (err, bytes) {
  client.close()
})

// socket.send(buf, offset, length, port, address, [callback])
/*
  buf: 要发送的 Buffer
  offset: Buffer 的偏移
  length: Buffer 的长度
  port: 目标端口
  address: 目标地址
  callback: 发送后的回调
*/
```
UDP 套接字事件：
- message：当 UDP 套接字侦听网卡端口后，接收到消息触发该事件
- listening：UDP 套接字开始侦听时触发该事件
- close：调用 close() 方法后触发
- error：异常发生时触发
## HTTP
### 1.HTTP 请求
报文头第一行如 GET / HTTP/1.1 被解析之后分解为如下属性：
- req.method：请求方法，值为 GET、POST、DELETE、PUT、CONNECT 等
- req.url：这里值为 /
- req.httpVersion：值为 1.1

其余报头是很规律的 Key:Value 形式，被解析后放置在 req.headers 属性上，例如
```js
headers: 
{
    'user-agent': ...,
    host: '127.0.0.1:1337',
    accept: '*/*'
}
```
报文体部分则抽象为一个只读流对象，如果业务逻辑需要读取报文体中的数据，则要在这个数据流结束后才能进行操作：
```js
function (req, res) {
  let buffers = []
  req.on('data', function (trunk) {
    buffers.push(trunk)
  }).on('end', function () {
    let buffer = Buffer.concat(buffers)
    // TODO
    res.end('hello world')
  })
}
```
这里的 request 对象和 response 对象都是相对较为底层的封装，express 等框架在这两个对象的基础上进行了高层封装。
### 2.HTTP 响应
HTTP 响应对象封装了对底层连接的写操作，可以将其看成一个可写的流对象。它影响响应报文头部信息的 API 为 res.setHeader() 和 res.writeHead()。

我们可以调用 setHeader 进行多次设置，但只有调用 writeHead 后，报头才会写入到连接中。

报文体部分则是调用 res.write() 和 res.end() 方法实现的，其差别在于 res.end() 会先调用 write() 发送数据，然后发送信号告知服务器这次响应结束。

需要注意以下几点：
- 报头是在报文体发送前发送的，一旦开始了数据的发送，writeHead() 和 setHeader() 将不再生效，这是由协议的特性决定的
- 务必调用 res.end() 以结束请求，否则客户端将一直处于等待的状态。当然，也可以通过延迟 res.end() 的方式实现客户端与服务器之间的长连接

### 3.HTTP 服务的事件
- connection：开启 HTTP 请求和响应前，客户端与服务器需要建立底层的 TCP 连接，该连接建立时触发一次 connection 事件
- request：服务器解析出 HTTP 请求头后触发
- close：调用 server.close() 后触发
- checkContinue：某些客户端再发送较大的数据时，并不会将数据直接发送，而是先发送一个头部带 Expect：100-continue 的请求到服务器，服务器将会触发 checkContinue 事件；如果没有为服务器监听这个事件，服务器将会自动响应客户端 100 Continue 的状态码，表示接收数据上传；如果不接受较大的数据，响应客户端 400 Bad Request 拒绝客户端继续发送即可。需要注意的是，当该事件发生时不会触发 request 事件，两个事件之间互斥。当客户端收到 100 Continue 后重新发送请求时，才会触发 request 事件。
- connect：当客户端发起 CONNECT 请求时触发，发起 CONNECT 请求通常在 HTTP 代理时出现；如果不监听该事件，发起该请求的连接将会关闭。
- upgrade：当客户端要求升级连接的协议时，需要和服务器协商，客户端会在请求中带上 Upgrade 字段，服务器会在接收到这样的请求时触发该事件，如果不监听该事件，发起该请求的连接将会关闭。（对 WebSocket 很重要）
- clientError：连接的客户端发 error 事件时，这个错误会传送到服务器端，触发该事件。

### 4.HTTP 客户端
```js
const options = {
  hostname: '127.0.0.1',
  port: 1334,
  path: '/',
  method: 'GET'
}
/*
  options 参数决定了 HTTP 请求头中的内容
  host: 服务器的域名或 IP 地址，默认 localhsot
  hostname: 服务器名称
  port: 服务器端口，默认 80
  localAddress: 建立网络连接的本地网卡
  socketPath: Domain 套接字路径
  method: HTTP 请求方法，默认 GET
  path: 请求路径，默认 /
  headers: 请求头对象
  auth: Basic 认证，这个值将被计算成请求头中的 Authorization 部分
*/
const req = http.request(options, function (res) {
  console.log(`STATUS: ${res.statusCode}`)
  console.log(`HEADERS: ${JSON.stringify(res.headers)}`)
  res.setEncoding('utf-8')
  res.on('data', function (chunk) {
    console.log(chunk)
  })
})
```
HTTP 客户端事件：
- response
- socket
- connect
- upgrade
- continue

## 构建 WebSocket 服务
- WebSocket 客户端基于事件的编程模型与 Node 中自定义事件相差无几
- WebSocket 实现了客户端与服务器端之间的长连接，而 Node 事件驱动的方式十分擅长与大量的客户端保持高并发连接

WebSocket 与传统 HTTP 相比有如下好处：
- 客户端与服务器端只建立一个 TCP 连接，可以使用更少的连接
- WebSocket 服务器端可以推送数据到客户端，这远比 HTTP 请求响应模式更灵活高效
- 有更轻量级的协议头，减少数据传送量

WebScoket 协议主要分为两个部分：握手和数据传输

客户端建立连接时，通过 HTTP 发起请求报文，如下所示：
```shell
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
```
与普通的 HTTP 协议略有区别的部分在于如下这些请求头：
```js
Upgrade: websocket 
Connection: Upgrade
// 以上表示请求服务器端升级协议为 WebSocket

Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==  // 用于安全校验
Sec-WebSocket-Protocol: chat, superchat // 子协议
Sec-WebSocket-Version: 13 // 版本号
```
服务端在处理完请求后，响应如下报文：告知客户端正在更换协议，更新应用层协议为 WebSocket 协议，并在当前的套接字上应用新协议
```shell
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
Sec-WebSocket-Protocal: caht
```
一旦 WebSocket 握手成功，服务器端与客户端将会呈现对等的效果，都能接收和发送消息。

*****
在握手顺利完成后，当前连接将不再进行 HTTP 的交互，而是开始 WebSocket 的数据帧协议。

<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/diveintonode9.png" />

使用的话直接用社区的 socket.io 比较好。

## HTTPS
Node 在网络安全上提供了 3 个模块，分别是 crypto、tls、https。其中 crypto 主要用于加密解密，SHA1、MD5 等加密算法都在其中有体现；tls 模块用于建立 TLS/SSL 加密的 TCP 连接；https 模块与 http 模块接口一致，区别仅在于它建立于安全的连接上。
> TLS（Transport Layer Security，安全传输层协议）可以认为就是 SSL（Secure Socket Layer，安全套阶层）的标准化后的名称，在传输层提供对网络连接加密的功能

中间人攻击：客户端与服务器端在交换公钥的过程中，中间人对客户端扮演服务器的角色，对服务器端扮演客户端的角色，因此客户端和服务器端几乎感受不到中间人的存在。

<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/diveintonode10.png" />

因此，需要引入数字证书来对公钥进行验证。

数字证书中包含了服务器的名称和主机名、服务器的公钥、签名颁发机构的名称、来自签名颁发机构的签名。在连接建立前，会通过证书中的签名确认收到的公钥是来自目标服务器的，从而产生信任关系。

CA（Certificate Authority，数字证书认证中心）的作用是为站点颁发证书，且这个证书中具有 CA 通过自己的公钥和私钥实现的签名。

为了得到签名证书，服务器端需要通过自己的私钥生成 CSR（Certificate Signing Request，证书签名请求）文件。

CA 机构将通过这个文件颁发属于该服务器端的签名证书，只要通过 CA 机构就能验证证书是否合法。

通过 CA 机构颁发证书是一个烦琐的过程，需要付出一定的经理和费用，对于中小型企业，多半是采用自签名证书来构建安全网络的。

所谓自签名证书，就是自己扮演 CA 机构，给自己的服务器端颁发签名证书。

......
