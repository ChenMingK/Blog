# 第 8 章 构建 Web 应用
## Cookie
Cookie 的处理分为如下几步：
- 服务器向客户端发送 Cookie
- 浏览器将 Cookie 保存
- 之后每次浏览器都会将 Cookie 发向服务器端

Cookie 是被放在请求头中的而不是请求体中，原生 Node 可以通过 req.headers.cookie 来获取到。

我们来看下设置 Cookie 的 Set-Cookie 字段：
```js
Set-Cookie: name=value; Path=/; Expires=Sun, 23-Apr-23 09:01:35 GMT; Domain=.domain.com;
```
其中 name = value 是必须包含的部分，其余部分皆是可选参数。
- path

表示这个 Cookie 影响到的路径，当前访问的路径不满足该匹配时，浏览器则不发送这个 Cookie
- Expires 和 Max-Age

告知浏览器这个 Cookie 何时过期，如果不设置该选项，在关闭浏览时会丢失掉这个 Cookie。

如果设置了过期时间，浏览器会把 Cookie 内容写入到磁盘并保存，下次打开浏览器依旧有效。

Expires 的值是一个 UTC 格式的时间字符串，告知浏览器此 Cookie 何时将过期，Max-Age 则告知浏览器此 Cookie 多久后过期。

如果服务器端和客户端的时间不匹配，使用 Expires 就会存在偏差，为此可以使用 Max-Age。
- HttpOnly

告知浏览器不允许通过脚本 document.cookie 去更改这个 Cookie 值。

事实上，设置 HttpOnly 后，这个值在 document.cookie 中不可见，但是在 HTTP 请求的过程中，依然会发送这个 Cookie 到服务器端。

- Secure

当 Secure 值为 true 时，在 HTTP 中是无效的，在 HTTPS 中才有效，表示创建的 Cookie 只能在 HTTPS 连接中被浏览器传递到服务器端进行会话验证，
如果是 HTTP 连接则不会传递该信息。

```js
// 封装方法快速设置 Cookie
/**
 * 
 * @param {*} name 该 Cookie 的键
 * @param {*} val  该 Cookie 的值
 * @param {*} opt  可选参数
 */
const serialize = function (name, val, opt) {
  const pairs = [`${name}=${val}`]
  opt = opt || {}

  if (opt.maxAge) pairs.push(`Max-Age=${opt.maxAge}`)
  if (opt.domain) pairs.push(`Domain=${opt.domain}`)
  if (opt.path) pairs.push(`Path=${opt.path}`)
  if (opt.expires) pairs.push(`Expires=${opt.expires}`)
  if (opt.httpOnly) pairs.push(`HttpOnly`)
  if (opt.secure) pairs.push(`Secure`)

  return pairs.join('; ')
}

const handler = function (req, res) {
  if (!req.cookies.isVisit) {
    res.setHeader('Set-Cookie', serialize('isVisit', 1))
    res.writeHead(200)
    res.end('欢迎第一次到来')
  } else {
    res.writeHead(200)
    res.end('再次欢迎你')
  }
}

// res.setHeader 的第二个参数可以是一个数组
res.setHeader('Set-Cookie', [serialize('foo', 'bar'), serialize('baz', 'val')])
// 这会在报文头部形成两条 Set-Cookie 字段
// Set-Cookie: foo=bar; Path=/; Expires= ...; Domain=.domain.com
// Set-Cookie: baz=val; Path=/; Expires= ...; Domain=.domain.com
```
## Cookie 的性能影响：
一旦 Cookie 设置过多，将会导致报头较大，大多数的 Cookie 并不需要每次都用上。

如果在域名的根节点设置 Cookie(Path = /)，几乎所有子路径下的请求都会带上这些 Cookie，而它们在有些情况下是无用的，比如静态文件。
- 将静态文件放在不同的域名下，使得业务相关的 Cookie 不再影响静态资源。
- 使用额外的域名的好处是减少了无效 Cookie 的传输，还可以突破浏览器下载线程数量的限制。缺点是 ￥ 以及额外的一次 DNS 查询

## Session
Cookie 最严重的的问题就是前后端都可以进行修改，Session 的数据只保留在服务器端，客户端无法修改，
但是仍然需要使用 Cookie 实现用户和数据的映射，一旦服务器端启用了 Session，它将约定一个键值作为 Session 的口令。

一旦服务器检查到用户请求 Cookie 中没有携带该值，它就会为之生成一个值，这个值是唯一且不重复的值，并设定超时时间。

PS：以下代码为原生 node.js 实现的，现在一般使用 express-session 插件配置一下就可以实现相同的功能（我用的时候有 bug 搞不定），所以有时候还是了解下原生实现的好。另外，这里的 session 是存在内存中的，现在一般存在 redis。
```js
const sessions = {}
const key = 'session_id'
const EXPIRES = 20 * 60 * 1000

const generate = function () {
  const session = {}
  session.id = (new Date()).getTime() + Math.random()
  session.cookie = {
    expires: (new Date()).getTime() + EXPIRES
  }
  sessions[session.id] = session
  return session
}

function (req, res) {
  const id = req.cookies[key]
  if (!id) {
    req.session = generate()
  } else {
    const session = sessions[id]
    if (session) {
      if (session.cookie.expire > (new Date()).getTime()) {
        // 更新超时时间
        session.cookie.expire = (new Date()).getTime() + EXPIRES
        req.session = session
      } else {
        // 超时了，删除旧的数据，并重新生成
        delete session[id]
        req.session = generate()
      }
    } else {
      // 如果 sesion 过期或口令不对，重新生成 session
      req.session = generate()
    }
  }
  handler(req, res)
}

// 我们还需要再响应时添加相应头部
// hack 响应对象的 writeHead() 方法
let writeHead = res.writeHead
res.writeHead = function () {
  const cookies = res.getHeader('Set-Cookie')
  const session = serialize('Set-Cookie', req.session.id)
  cookies = Array.isArray(cookies) ? cookies.concat(session) : [cookies, session]
  res.setHeader('Set-Cookie', cookies)
  return writeHead.apply(this, arguments)
}

// 业务逻辑
const handler = function (req, res) {
  if (!req.session.isVisit) {
    res.session.isVisit = true
    res.writeHead(200)
    res.end('欢迎第一次来到动物园')
  } else {
    res.writeHead(200)
    res.end('动物园再次欢迎你')
  }
}
```
## 缓存
缓存需要浏览器与服务器共同协作来完成。

通常来说 POST、DELETE、PUT 这类待行为性的请求操作一般不做任何缓存，大多数缓存只应用在 GET 请求中。

本地没有文件时，浏览器必然会请求服务器端的内容，并将这部分内容放置在本地的某个缓存目录中。

在第二次请求时，它将对本地文件进行检查，如果不能确定这份本地文件是否可以直接使用，它将会发起一次条件请求。

所谓条件请求，就是在普通的 GET 请求报文中附带 If-Modified-Since 字段，如下所示：

If-Modified-Since: Sun, 03 Feb 2019 06:01;12 GMT

它将询问服务器端是否有更新的版本，本地文件的最后修改时间。如果服务器端没有新的版本，只需响应一个 304 状态码，客户端就使用本地版本；如果服务器有新版本，就将新的内容发送给客户端，客户端放弃本地版本，代码如下：
```js
const handler = function (req, res) {
  fs.stat(filename, function (err, stat) {
    const lastModified = stat.mtime.toUTCString()
    if (lastModified === req.headers['if-modified-since']) {
      res.writeHead(304, 'Not Modified')
      res.end()
    } else {
      fs.readFile(filename, function (err, file) {
        const lastModified = stat.mtime.toUTCString()
        res.setHeader('Last-Modified', lastModified)
        res.writeHead(200, 'OK')
        res.end(file)
      })
    }
  })
}
```
浏览器在收到 Etag："83-13591232132"这样的响应后，下次请求会将其放置在请求头中：If-None-Match："83-13591232132"

设置 Expires 或 Cache-Control 头，浏览器就可以不向服务器发送 HTTP 请求而知晓是否直接使用本地版本。

其区别之前已经提到过，Expires 可能会出现服务器端和浏览器端时间不同步的情况，Cache-Control 设置 max-age 是倒计时的方式。

如果同时设置了 max-age 和 Expires，max-age 会覆盖 Expires。
```js
const handler = function (req, res) {
  fs.readFile(filename, function (err, file) {
    const expires = new Date()
    expires.setTime(expires.getTime() + 10 * 365 * 24 * 60 * 60 * 1000)
    res.setHeader('Expires', expires.toUTCString())
    res.writeHead(200, 'OK')
    res.end(file)
  })
}

const handelr = function (req, res) {
  fs.readFile(filename, function (err, file) {
    res.setHeader('Cache-Control', 'max-age=' + 10 * 365 * 24 * 60 * 60 * 1000)
    res.writeHead(200, 'OK')
    res.end(file)
  })
}
```
> 如果使用 Nginx 做静态资源服务器就看看 Nginx 的缓存配置即可

## 清除缓存
缓存一旦设定，当服务端意外更新内容时，却无法通知客户端更新。一般有两种更新机制：
- 每次发布，路径中跟随 Web 应用的版本号：`http://url.com?v=20190502`
- 每次发布，路径中跟随该文件内容的 hash 值：`http://url.com?hash=sadsadsa`

一般采用的是第二种方式（所以 webpack 打包生成的文件都要哈希啊）

## MVC
MVC 模型的主要思想是将业务逻辑按职责分离
- 控制器（Controller），一组行为的集合
- 模型（Model），数据相关的操作和封装
- 视图（View），视图的渲染

它的工作模式如下：
- 路由解析，根据 URL 寻找到对应的控制器和行为
- 行为调用相关的模型，进行数据操作
- 数据操作结束后，调用视图和相关数据进行页面渲染，输出到客户端

<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/diveintonode11.png" />

## RESTFUL
REST 全称是 Representational State Transfer，它是一个关于 URL 的设计规范。

比如我们过去对用户的增删改查或许是这么设计 URL 的：
```shell
POST /user/add?username=jacksontian
GET /user/remove?username=jacksontian
POST /user/update/username=jacksontian
GET /user/get?username=jacksontian
```
在 RESTFUL 设计中，它应该是这样的：
```shell
POST /user/username=jacksontian
DELETE /user/username=jacksontian
PUT /user/username=jacksontian
GET /user/username=jacksontian
```
过去设计资源的格式与后缀有很大的关联，比如：
```shell
GET /user/jacksontian.json
GET /user/jacksontian.xml
```
在 RESTFUL 设计中，资源的具体格式由请求报头中的 Accept 字段和服务器端的支持情况来决定。

如果客户端同时接受 JSON 和 XML 格式的响应，那么它的 Accept 字段值是如下这样的：

`Accept: application/json,application/xml`

靠谱的服务器应该要顾及这个字段，然后根据自己能响应的格式做出响应，在响应报文中，通过 Content-Type 字段告知客户端是什么格式，如下

`Content-Type: application/json`

所以 RESTful 的设计就是：**通过 URL 设计资源，请求方法定义资源的操作，通过 Accept 决定资源的表现形式**

## 中间件
中间件的行为类似 Java 中过滤器的工作原理，就是在进入具体的业务处理之前，先让过滤器处理，比如对于每个请求我们一般都要解析 cookie，querystring 什么的，那么就设计对应的中间件处理完成后存储在上下文中（req 和 res，Koa2 合并为一个 context）

<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/diveintonode12.png" />

```js
// 模拟中间件的实现
const http = require('http')
const slice = Array.prototype.slice

class LikeExpress {
  constructor() {
    // 存放中间件的列表
    this.routes = {
      all: [], // 存放 app.use 注册的中间件
      get: [], // 存放 app.get 注册的中间件
      post: []
    }
  }

  // 内部实现注册的方法
  register(path) {
    const info = {}
    if (typeof path === 'string') { // 字符串 - 路由
      info.path = path
      // 从第二个参数开始，转换为数组，存入stack
      info.stack = slice.call(arguments, 1) // 取出剩余的参数
    } else { // 没有显式地传入路由则默认是根路由
      info.path = '/' // 省略第一个参数 -> 根目录
      // 从第一个参数开始，转换为数组，存入stack
      info.stack = slice.call(arguments, 0)
    }
    // { path: '', stack: [middleware, ...] }
    return info
  }

  use() {
    const info = this.register.apply(this, arguments) // 把当前函数的所有参数传入
    this.routes.all.push(info)
  }

  get() {
    const info = this.register.apply(this, arguments) // 把当前函数的所有参数传入
    this.routes.get.push(info)
  }

  post() {
    const info = this.register.apply(this, arguments) // 把当前函数的所有参数传入
    this.routes.post.push(info)
  }

  match(method, url) {
    let stack = [] // resultList
    if (url === '/favicon.ico') { // 小图标无视
      return stack
    }
    // 获取 routes
    let curRoutes = []
    curRoutes = curRoutes.concat(this.routes.all)
    curRoutes = curRoutes.concat(this.routes[method])

    curRoutes.forEach(routeInfo => {
      if (url.indexOf(routeInfo.path === 0)) {
        // url === '/api/get-cookie' 且 routeInfo.path === '/'
        // url === '/api/get-cookie' 且 routeInfo.path === '/api'
        // url === '/api/get-cookie' 且 routeInfo.path === '/api/get-cookie'
        stack = stack.concat(routeInfo.stack)
      }
    })
    return stack
  }
  
  // 核心的 next 机制
  handle(req, res, stack) {
    const next = () => {
      // 拿到第一个匹配的中间件
      const middleware = stack.shift()
      if (middleware) {
        // 执行中间件函数
        middleware(req, res, next)
      }
    }
    next()
  }

  callback() {
    return (req, res) => {
      // 自己定义 res.json 方法
      res.json = data => {
        res.setHeader('Content-Type', 'application/json')
        res.end(
          JSON.stringify(data)
        )
      }

      // 获取 url 和 method ：通过这两个来获得需要经过的中间件
      const url = req.url
      const method = req.method.toLowerCase()

      // match 函数匹配可用的中间件列表
      const resultList = this.match(url, method)
      this.handle(req, res, resultList)
    }
  }

  listen(...args) {
    const server = http.createServer(this.callback)
    server.listen(...args)
  }
}

// 工厂函数
module.exports = () => {
  return new LikeExpress()
}
```
