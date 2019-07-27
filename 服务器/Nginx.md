# 正向代理与反向代理
**正向代理**：用户想从服务器拿资源数据，但是只能通过 proxy 服务器才能拿到，
所以用户 A 只能去访问 proxy 服务器然后通过 proxy 服务器去服务器 B 拿数据，
这种情况用户是明确知道你要访问的是谁，在我们生活中最典型的案例就是“翻墙“了，也是通过访问代理服务器最后访问外网的。

**反向代理**：客户端访问服务器时，并不知道会访问哪一台，感觉就是客户端访问了 Proxy 一样，
而实则就是当 Proxy 关口拿到用户请求的时候会转发到代理服务器中的随机（算法）某一台。
而在用户看来，他只是访问了 Proxy 服务器而已，典型的例子就是负载均衡。

<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/nginx1.png" width=45%/>&emsp;&emsp;<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/nginx2.png" width=45%/>

# 配置文件结构
```shell
...              #全局块
 
events {         #events块
   ...
}
 
http      #http块
{
    ...   #http全局块
    server        #server块
    { 
        ...       #server全局块
        location [PATTERN]   #location块
        {
            ...
        }
        location [PATTERN] 
        {
            ...
        }
    }
    server
    {
      ...
    }
    ...     #http全局块
}
```
1.全局块

配置影响 nginx 全局的指令。一般有运行 nginx 服务器的用户组，nginx 进程 pid 存放路径，
日志存放路径，配置文件引入，允许生成 worker process 数等。

2.events 块

配置影响 nginx 服务器或与用户的网络连接。有每个进程的最大连接数，
选取哪种事件驱动模型处理连接请求，是否允许同时接受多个网路连接，开启多个网络连接序列化等。

3.http 块

可以嵌套多个 server，配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置。
如文件引入，mime-type 定义，日志自定义，是否使用 sendfile 传输文件，连接超时时间，单连接请求数等。

4.server 块

配置虚拟主机的相关参数，一个 http 块中可以有多个 server。

5.location 块

配置请求的路由，以及各种页面的处理情况。

# 常用功能及其配置
## 做静态资源服务器
将静态资源（如jpg | png | css | js等）放在如下配置的`f:/nginx-1.12.2/static`目录下

然后在 nginx 配置文件中做如下配置(注意：静态资源配置只能放在 location / 中)，浏览器中访问  `http://localhost:80/1.png` 

即可访问到 `f:/nginx-1.12.2/static` 目录下的 1.png 图片

<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/nginx3.png" />

即访问 80 端口就相当于找到 root 这个目录。

location 修饰符：[https://www.cnblogs.com/xiaoliangup/p/9175932.html](https://www.cnblogs.com/xiaoliangup/p/9175932.html)

## 负载均衡
>负载均衡(Load Balance)其意思就是分摊到多个操作单元上进行执行，例如 Web 服务器、FTP服务器、企业关键应用服务器和其它关键任务服务器等，从而共同完成工作任务。

<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/nginx4.png" />


服务器 `localhost:8080 `挂掉时，nginx 能将请求自动转向服务器 `192.168.101.9:8080` 

上面还加了一个 weight 属性，此属性表示各服务器被访问到的权重，weight 越高被访问到的几率越高。

## CORS跨域配置
<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/nginx5.png" />


上面与跨域先关的其实只有 `add_header Access-Control-Allow-Origin *` 这一行，这里再介绍下其他设置

注意文件目录是 / 而不是 \\
- autoindex on 表示列出文件目录（访问时可以看到一个文件列表）
- location / 表示对所有经过的请求的操作
这里添加一个 `header Access-Control-Allow-Origin \* ` 指定允许哪个域的请求，* 表示任意
- Cache-Control 一行表示每次访问时不做缓存，如果文件经常变动,缓存可能导致文件未及时更新
  - `no-cache` 表示每次使用静态资源时都需要向服务端请求（进行过期认证），通常情况下，过期认证需要配合 etag 和 Last-Modified 进行一个比较
  - `must-revalidate` 作用与 no-caceh 相同，但更严格，强制意思更明显（理论上）

与跨域相关的头部信息如下：

Access-Control-Allow-Origin: 指定允许哪些源的网页发送请求.

Access-Control-Allow-Credentials: 指定是否允许 cookie 发送.

Access-Control-Allow-Methods: 指定允许哪些请求方法.

Access-Control-Allow-Headers: 指定允许哪些常规的头域字段, 比如说 Content-Type.

Access-Control-Expose-Headers: 指定允许哪些额外的头域字段, 比如说 X-Custom-Header.

## 开启 gzip 压缩
使用 gzip 压缩可以大大减小文件的体积，Nginx 配置如下：
```shell
server {
    gzip on;
    gzip_types    text/plain application/javascript application/x-javascript
text/javascript text/xml text/css;
}
```

## 设定缓存时间（一般对静态资源才做设置）
```shell
location ~* \.(jpg|jpeg|png|gif|webp)$ {
    expires 30d;
}
location ~* \.(css|js)$ {
    expires 7d;
}
```
如果要强制不缓存，可以把 expires 时间设置为 0 。
另外，开启 etag 只需要一行
```shell
etag on;
```
由于 etag 要使用少数的字符表示一个不定大小的文件（如 Etag: "58c4e2a1-f7"），所以 etag 是有重合的风险的，如果网站的信息特别重要，连很小的概率如百万分之一都不允许，那么就不要使用 etag 了。使用 etag 的代价是增加了服务器的计算负担，特别是当文件比较大时。

## 使用 HTTP 2.0
HTTP/2 需要使用 nginx 1.10.0 和 openssl 1.0.2 以上版本，nginx.conf 配置如下：
```shell
listen 443 ssl http2;
```
对于不支持 HTTP/2 的浏览器，nginx 会自动处理，因为 HTTP/2 的实现基本只支持 HTTPS，HTTPS 连接过程中需要先握手，浏览器会发送一个 Client Hello 的包给服务，这个包里面会有它是否支持 h2 的信息。如果没有这些信息，nginx 会自动切换到 HTTP/1.1，所以能够兼容老的浏览器和客户端。

## 一个 node.js 例子
一个十分常见的需求：处理请求，如果是静态文件，Nginx 直接返回，否则交给 Node 服务器处理。首先创建了一个 Node 服务器：
``` js
const http = require('http');
http.createServer((req, res) => {
    res.end('hello world');
}).listen(9000);
```
任何请求过来都返回 `hello world`，简版的 Nginx 配置如下，
```server
events {
    # 这里可不写东西
    use epoll;
}
http {
    server {
        listen 127.0.0.1:8888;
        # 如果请求路径跟文件路径按照如下方式匹配找到了，直接返回
        try_files $uri $uri/index.html;
        location ~* ^/(js|css|image|font)/$ {
            # 静态资源都在 static 文件夹下
            root /home/barret/www/static/;
        }
        location /app {
            # Node.js 在 9000 开了一个监听端口
            proxy_pass http://127.0.0.1:9000;
        }
        # 上面处理出错或者未找到的，返回对应状态码文件
        error_page    404            /404.html;
        error_page    502  503  504  /50x.html;
    }
}
```
首先 try_files，尝试直接匹配文件；没找到就匹配静态资源；还没找到就交给 Node 处理；否则就返回 4xx/5xx 的状态码。

try_files 指令可以用在 server 部分，不过最常见的还是用在 location 部分，它会按照给定的参数顺序进行尝试，第一个被匹配到的将会被使用。$uri 是内置预定义变量，表示当前请求的标准化 URI。

# 常用命令
1.检查 80 端口是否被占用的命令：
```shell
netstat -ano | findstr 0.0.0.0:80` 或 `netstat -ano | findstr "80"
```
2.修改 nginx 的配置文件后重新加载
```shell
nginx -s reload
```
3.关闭 nginx
```shell
nginx -s stop # 快速停止 nginx  
nginx -s quit # 完整有序地停止 nginx

taskkill /f /t /im nginx.exe # windows?
```
4.列出所有 nginx 进程
```shell
ps -aux | grep nginx # Linux

tasklist /fi "imagename eq nginx.exe" # windows
```
5.查找 nginx 配置文件的位置

先找出 nginx 可执行文件的路径`ps -ef | grep nginx`

<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/nginx6.png" />

这个`/usr/local/nginx/conf/nginx.conf`就是配置文件了
