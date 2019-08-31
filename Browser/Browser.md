# 浏览器有哪些线程？
1.&ensp;**GUI 渲染线程**

负责渲染浏览器界面，解析 HTML，CSS，构建 DOM 树和 RenderObject 树，布局和绘制等。

当界面需要重绘（repaint）或由于某种操作引发回流（reflow）时，该线程就会执行。

注意，GUI 渲染线程与 JS 引擎线程是互斥的，当 JS 引擎执行时 GUI 线程会被挂起（相当于被冻结了），GUI 更新会被保存在一个队列中等到 JS 引擎空闲时立即被执行。

2.&ensp;**JS 引擎线程**

也称为 JS 内核，负责处理 Javascript 脚本程序（例如 V8 引擎）。

JS 引擎线程负责解析 Javascript 脚本，运行代码。JS 引擎一直等待着任务队列中任务的到来，然后加以处理，
一个 Tab 页（renderer 进程）中无论什么时候都只有一个 JS 线程在运行 JS 程序

同样注意，GUI 渲染线程与 JS 引擎线程是互斥的，所以如果 JS 执行的时间过长，这样就会造成页面的渲染不连贯，导致页面渲染加载阻塞。（比如你的代码陷入无限循环那么页面就将会是一片空白）

3.&ensp;**事件触发线程**

归属于浏览器而不是 JS 引擎，用来控制事件循环（具体见 JS 章节事件循环部分）。

当 JS 引擎执行代码块如 setTimeout 时（也可来自浏览器内核的其他线程，如鼠标点击、AJAX 异步请求等），会将对应任务添加到事件线程中。

当对应的事件符合触发条件被触发时，该线程会把事件添加到待处理队列的队尾，等待 JS 引擎的处理。

4.&ensp;**定时触发器线程**

setInterval 与 setTimeout 所在线程。

浏览器定时计数器并不是由 JavaScript 引擎计数的,（因为 JavaScript 引擎是单线程的, 如果处于阻塞线程状态就会影响计时的准确性）
，因此通过单独线程来计时并触发定时（计时完毕后，添加到事件队列中，等待 JS 引擎空闲后执行）

> W3C在HTML标准中规定，规定要求 setTimeout 中低于 4ms 的时间间隔算为 4ms。

5.&ensp;**异步 http 请求线程**

XMLHttpRequest 在连接后是通过浏览器新开一个线程来处理请求。

检测到状态变更时，如果设置有回调函数，异步线程就产生状态变更事件，将这个回调再放入事件队列中。再由 JavaScript 引擎执行。

# 浏览器的渲染过程
## 浏览器内核指的是什么？

> 浏览器最重要或者说核心的部分是“Rendering Engine”，可大概译为“渲染引擎”，不过我们一般习惯将之称为“浏览器内核”。
负责对网页语法的解释（如[标准通用标记语言](https://baike.baidu.com/item/%E6%A0%87%E5%87%86%E9%80%9A%E7%94%A8%E6%A0%87%E8%AE%B0%E8%AF%AD%E8%A8%80/6805073)下的一个应用[HTML](https://baike.baidu.com/item/HTML)、[JavaScript](https://baike.baidu.com/item/JavaScript)）并渲染（显示）网页。 所以，通常所谓的[浏览器内核](https://baike.baidu.com/item/%E6%B5%8F%E8%A7%88%E5%99%A8%E5%86%85%E6%A0%B8/10602413)也就是浏览器所采用的[渲染引擎](https://baike.baidu.com/item/%E6%B8%B2%E6%9F%93%E5%BC%95%E6%93%8E/10982158)，渲染引擎决定了浏览器如何显示网页的内容以及页面的格式信息。不同的浏览器内核对网页编写语法的解释也有不同，因此同一网页在不同的内核的浏览器里的渲染（显示）效果也可能不同，这也是网页编写者需要在不同内核的浏览器中测试网页显示效果的原因。
             

主流浏览器：IE、Firefox、Safari、Chrome 及 Opera。
四大内核：Trident（也称IE内核）、webkit、Blink、Gecko
其他常见浏览器及其内核：
1. IE 浏览器内核：Trident 内核，也是俗称的 IE 内核；  
2. Chrome 浏览器内核：统称为 Chromium 内核或 Chrome 内核，以前是 Webkit 内核，现在是 Blink 内核；  
3. Firefox 浏览器内核：Gecko 内核，俗称 Firefox 内核；  
4. Safari 浏览器内核：Webkit 内核；  
5. Opera 浏览器内核：最初是自己的 Presto 内核，后来是  Webkit，现在是Blink 内核；  
6. 360 浏览器、猎豹浏览器内核：IE + Chrome 双内核；  
7. 搜狗、遨游、QQ浏览器内核：Trident（兼容模式）+ Webkit（高速模式）；  
8. 百度浏览器、世界之窗内核：IE 内核；  
9. 2345浏览器内核：以前是 IE 内核，现在是 IE + Chrome 双内核

好吧回到正题：浏览器的渲染过程见下面这张经典的图

<img src="https://github.com/ChenMingK/WebKnowledges-Notes/blob/master/images/e17edc7ae0617a002b2d77ef1d26ad4a_1057x372.png" />

浏览器的渲染机制一般分为以下几个步骤
1.  处理 HTML 并构建 DOM 树。
2.  处理 CSS 构建 CSSOM 树。
3.  将 DOM 与 CSSOM 合并成一个渲染树。
4.  根据渲染树来布局，计算每个节点的位置。
5.  调用 GPU 绘制，合成图层，显示在屏幕上。

在构建 CSSOM 树时，会阻塞渲染，直至 CSSOM 树构建完成。并且构建 CSSOM 树是一个十分消耗性能的过程，所以应该尽量保证层级扁平，减少过度层叠，越是具体的 CSS 选择器，执行速度越慢。

当 HTML 解析到 script 标签时，会暂停构建 DOM，完成后才会从暂停的地方重新开始。也就是说，如果你想首屏渲染的越快，就越不应该在首屏就加载 JS 文件。并且 CSS 也会影响 JS 的执行，只有当解析完样式表才会执行 JS，所以也可以认为这种情况下，CSS 也会暂停构建 DOM。

## 图层
一般来说，可以把普通文档流看成一个图层。特定的属性可以生成一个新的图层。**不同的图层渲染互不影响**，所以对于某些频繁需要渲染的建议单独生成一个新图层，提高性能。**但也不能生成过多的图层，会引起反作用。**

通过以下几个常用属性可以生成新图层（PS：未查证）

*   3D 变换：translate3d、ranslateZ
*   will-change
*   video、iframe标签
*   通过动画实现的 opacity 动画转换
*   position: fixed

当一个元素在自己图层内发生变化的时候它的回流和重绘只会在图层内部发生，利用这一特性可以减少了浏览器对于重绘与回流的运算量。
## 回流（Reflow）与重绘（Repaint）
重绘和回流是渲染步骤中的一小节，但是这两个步骤对于性能影响很大。
*   回流是布局或者几何属性需要改变就称为回流。
*   重绘是当节点需要更改外观而不会影响布局的，比如改变 color 就叫称为重绘

回流必定会发生重绘，重绘不一定会引发回流。回流所需的成本比重绘高的多，改变深层次的节点很可能导致父节点的一系列回流。
关于如何减少回流与重绘这一问题就转到 Performance 章节吧。
# 浏览器的同源策略
同源指的是三个相同：
- 协议相同
- 域名相同
- 端口相同

举例来说，`http://www.example.com/dir/page.html` 这个网址，协议是 `http://`，域名是 `www.example.com`，端口是 `80`（默认端口可以省略）,
它的同源情况如下。


*   `http://www.example.com/dir2/other.html`：同源
*   `http://example.com/dir/other.html`：不同源（域名不同）
*   `http://v2.www.example.com/dir/other.html`：不同源（域名不同）
*   `http://www.example.com:81/dir/other.html`：不同源（端口不同）

## 同源策略限制了什么？   
1. Cookie、LocalStorage 和 IndexDB 无法读取。
2. DOM 无法获得。
3. AJAX 请求不能发送。

> 同源政策的目的，是为了保证用户信息的安全，防止恶意的网站窃取数据。
> 设想这样一种情况：A 网站是一家银行，用户登录以后，又去浏览其他网站。如果其他网站可以读取 A 网站的 Cookie，会发生什么？很显然，如果 Cookie 包含隐私（比如存款总额），这些信息就会泄漏。更可怕的是，Cookie 往往用来保存用户的登录状态，如果用户没有退出登录，其他网站就可以冒充用户，为所欲为。因为浏览器同时还规定，提交表单不受同源政策的限制。
> 由此可见，"同源政策"是必需的，否则 Cookie 可以共享，互联网就毫无安全可言了。

## 如何规避同源策略？（跨域）
### JSONP
利用 script 标签没有跨域限制的漏洞，通过 script 标签指向一个需要访问的地址并提供一个回调函数来接收数据。就是给 url 加上一个 callback 参数

缺点：仅限于 GET 请求
``` js
<script src="http://domain/api?param1=a&param2=b&callback=jsonp"></script>
<script>
    function jsonp (data) {
    	console.log(data)
	}
</script>    
```
在开发中可能会遇到多个 JSONP 请求的回调函数名是相同的，这时候就需要自己封装一个 JSONP，以下是简单实现：
``` js
function jsonp(url, jsonpCallback, success) {
  let script = document.createElement("script");
  script.src = url;
  script.async = true;
  script.type = "text/javascript";
  window[jsonpCallback] = function(data) {
    success && success(data);
  };
  document.body.appendChild(script);
}
jsonp(
  "http://xxx",
  "callback",
  function(value) {
    console.log(value);
  }
);
```
### CORS
实现CORS通信的关键是后端。只要后端实现了 CORS，就实现了跨域。

服务端设置（添加响应头部）`Access-Control-Allow-Origin`就可以开启 CORS。该属性表示哪些域名可以访问资源，如果设置通配符则表示所有网站都可以访问资源。
* Access-Control-Allow-Origin: 指定允许哪些源的网页发送请求.
* Access-Control-Allow-Credentials: 指定是否允许 cookie 发送.
* Access-Control-Allow-Methods: 指定允许哪些请求方法.
* Access-Control-Allow-Headers: 指定允许哪些常规的头域字段, 比如说 Content-Type.
* Access-Control-Expose-Headers: 指定允许哪些额外的头域字段, 比如说 X-Custom-Header.
### document.domain
该方式只能用于二级域名相同的情况下，比如 `a.test.com` 和 `b.test.com` 适用于该方式。只需要给页面添加

`document.domain = 'test.com'`

表示二级域名都相同就可以实现跨域
### postMessage
这种方式通常用于获取嵌入页面中的第三方页面数据。一个页面发送消息，另一个页面判断来源并接收消息（这个 API 是用于多个页面之间传递消息的）
``` js
// 发送消息端
window.parent.postMessage('message', 'http://test.com');
// 接收消息端
var mc = new MessageChannel();
mc.addEventListener('message', (event) => {
    var origin = event.origin || event.originalEvent.origin; 
    if (origin === 'http://test.com') {
        console.log('验证通过')
    }
});
```
### 配置代理服务器
开发 vue 或者 react 项目的时候可以很方便地配置代理服务器，底层依赖的是 webpack 的 devServer 配置项

代理服务器的作用大致如下图：

<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/BlogImgs/808b899f9511b72bd3737262be0d1dcc_600x401.png" />


使用 create-react-app 创造的应用已经具备了代理功能，只需要在 package.json 中添加如下一行： 

`"proxy": "http://www.weather.com.cn/"`  

这一行配置告诉我们的应用，当接收到不是要求本地资源（localhost）的 HTTP 请求时，
这个 HTTP 请求的协议和域名部分"替换"为 `http://www.weather.com.cn` 转发出去（代理服服务器会根据配置分析 入 和 出 的关系，重新构建新的请求），
并将收到的结果返还给浏览器。

注意在线上环境应该开发自己的代理服务器（如 Nginx 的代理配置）

简单来说就是帮我们重新构建请求伪造成同源的请求，当然还要把响应数据转发回来

# 优雅降级与渐进增强
**渐进增强（Progressive Enhancement）**：一开始就针对低版本浏览器进行构建页面，完成基本的功能，然后再针对高级浏览器进行效果、交互、追加功能达到更好的体验。


**优雅降级（Graceful Degradation）**：一开始就构建站点的完整功能，然后针对浏览器测试和修复。比如一开始使用 CSS3 的特性构建了一个应用，然后逐步针对各大浏览器进行 hack 使其可以在低版本浏览器上正常浏览。

# 存储：cookie session token indexedDB 

| 特性 | cookie | localStorage | sessionStorage | indexDB |
| :---- | :---- | :---- | :---- | :---- |
| 数据生命周期 | 一般由服务器生成，可以设置过期时间 | 除非被清理，否则一直存在 | 页面关闭就清理 | 除非被清理，否则一直存在 |
| 数据存储大小 | 4K | 5M | 5M | 无限 |
| 与服务端通信 | 每次都会携带在header中，影响请求性能 | 不参与 | 不参与 | 不参与 |
 

从上表可以看到，cookie 已经不建议用于存储（一般用于登录验证）。
如果没有大量数据存储需求的话，可以使用 localStorage 和 sessionStorage。
对于不怎么改变的数据尽量使用 localStorage 存储，否则可以用 sessionStorage 存储。

对于`cookie`，我们还需要注意安全性。

| 属性 | 作用 |
| --- | --- |
| value | 如果用于保存用户登录态，应该将该值加密，不能使用明文的用户标识 |
| http-only | 不能通过 JS 访问 cookie，减少 XSS 攻击 |
| secure | 只能在协议为 HTTPS 的请求中携带 |
| same-site | 规定浏览器不能在跨域请求中携带 cookie，减少 CSRF 攻击 |


如何设置？

利用 HttpResponse 的 addHeader 方法，设置Set-Cookie的值

cookie字符串的格式：`key=value; Expires=date; Path=path; Domain=domain; Secure; HttpOnly`

```js
// 设置cookie
response.addHeader("Set-Cookie", "uid=112; Path=/; HttpOnly");

// 设置多个cookie
response.addHeader("Set-Cookie", "uid=112; Path=/; HttpOnly");
response.addHeader("Set-Cookie", "timeout=30; Path=/test; HttpOnly");

// 设置 https 的cookie
response.addHeader("Set-Cookie", "uid=112; Path=/; Secure; HttpOnly");
```

## 登录验证的流程

这就要从登录验证的技术迭代说起，假设我们使用最原始的方法：用户输入用户名和密码验证成功后，服务器向客户端设置 cookie，

我们假设这个 cookie 存储一个 username 字段（显然这是一个很愚蠢的行为），那么在用户首次登录之后他下次再登录的时候就拥有了这个 cookie，

前端可以设置在用户一打开应用时就向服务器发送一个请求（自动携带 cookie），后端就通过检测 cookie 中的信息就可以使得用户直接进入登录状态了。

整理一下：

① 首次登录拥有了 cookie 

② 再次登录通过检验 cookie 的存在与否来确定登录状态
*****
在 cookie 中直接暴露用户信息是愚蠢的行为，下面我们来升级一下。

我们在 cookie 中存储一个 userid，服务器根据传来的 userid 来得到对应的 username，
那么就需要花费空间来存储这一映射关系，假设我们用全局变量来存储（即存储在内存中），这就是所谓的 session 了，即 server 端存储用户信息。

那么现在就变成了：

① 首次登录拥有了 cookie，但这次记录的是 userid 

② 再次登录发送 cookie，服务器分析 cookie 并根据存储的映射关系判断登录状态

看上去不错，但是仍然存在一些问题：假设我们是 node.js 的一个进程做服务，用户数量不断增加，内存将会暴增，而 OS 是会限制一个进程所能使用的最大内存的；

另外，假设我为了充分利用 CPU 的多核特性我开个多进程一起来做服务，那么这些进程之间的内存无法共享，即用户信息无法共享，这就不太妙了。
*****

于是我们可以通过使用 redis 来解决这一问题，redis 不同于 mysql，其数据存放在内存中（虽然昂贵但访问存快）

我们把原先要在各个进程中存储的全局变量改为统一存储在 redis 中，这样就可以做到多进程共享信息（全部通过访存 redis 来实现）

粗略地画个图就差不多是这样吧：


<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/BlogImgs/837a4844bf835d3fa2d14a6d95ed4fab_468x609.png" />
