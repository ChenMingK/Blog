# XSS
跨网站指令码（Cross-site scripting，通常简称为：XSS） 是一种网站应用程式的安全漏洞攻击，是代码注入的一种。它允许恶意使用者将程式码注入到网页上，其他使用者在观看网页时就会受到影响。这类攻击通常包含了 HTML 以及使用者端脚本语言。

XSS 分为三种：反射型，存储型和 DOM-based

## 如何攻击
### 反射型
反射型 XSS 只是简单地把用户输入的数据 “反射” 给浏览器，这种攻击方式往往需要攻击者诱使用户点击一个恶意链接，或者提交一个表单.
### 存储型
存储型 XSS 会把用户输入的数据 "存储" 在服务器端，当浏览器请求数据时，脚本从服务器上传回并执行。这种 XSS 攻击具有很强的稳定性。
比较常见的一个场景是攻击者在社区或论坛上写下一篇包含恶意 JavaScript 代码的文章或评论，文章或评论发表后，所有访问该文章或评论的用户，都会在他们的浏览器中执行这段恶意的 JavaScript 代码。
### 基于 DOM
基于 DOM 的 XSS 攻击是指通过恶意脚本修改页面的 DOM 结构，是纯粹发生在客户端的攻击。

## 如何防御
最普遍的做法是转义输入输出的内容，对于引号，尖括号，斜杠进行转义
``` js
function escape(str) {
	str = str.replace(/&/g, "&amp;");
	str = str.replace(/</g, "&lt;");
	str = str.replace(/>/g, "&gt;");
	str = str.replace(/"/g, "&quto;");
	str = str.replace(/'/g, "&#39;");
	str = str.replace(/`/g, "&#96;");
    str = str.replace(/\//g, "&#x2F;");
    return str
}
```
对于显示富文本来说，不能通过上面的办法来转义所有字符，因为这样会把需要的格式也过滤掉。

这种情况通常采用白名单过滤的办法，当然也可以通过黑名单过滤，但是考虑到需要过滤的标签和标签属性实在太多，更加推荐使用白名单的方式。

``` js
var xss = require("xss");
var html = xss('<h1 id="title">XSS Demo</h1><script>alert("xss");</script>');
// -> <h1>XSS Demo</h1>&lt;script&gt;alert("xss");&lt;/script&gt;
console.log(html);
```
以上示例使用了 `js-xss` 来实现。可以看到在输出中保留了 `h1` 标签且过滤了 `script` 标签
```js
/*
  通过 whiteList 来指定，格式为：{'标签名': ['属性1', '属性2']}。
  不在白名单上的标签将被过滤，不在白名单上的属性也会被过滤。以下是示例：
  只允许 a 标签，该标签只允许 href, title, target 这三个属性
*/
var options = {
  whiteList: {
    a: ["href", "title", "target"]
  }
};
// 使用以上配置后，下面的HTML
// <a href="#" onclick="hello()"><i>大家好</i></a>
// 将被过滤为
// <a href="#">大家好</a>
```

### CSP
内容安全策略（Content Security Policy，CSP ） 是一个额外的安全层，用于检测并削弱某些特定类型的攻击，包括跨站脚本 XSS 和数据注入攻击等。

无论是数据盗取、网站内容污染还是散发恶意软件，这些攻击都是主要的手段。

我们可以通过 CSP 来尽量减少 XSS 攻击。CSP 本质上也是建立白名单，规定了浏览器只能够执行特定来源的代码。

通常可以通过 HTTP Header 中的 `Content-Security-Policy` 来开启 CSP

*   只允许加载本站资源
```html
Content-Security-Policy: default-src 'self'
```
*   只允许加载 HTTPS 协议图片
```html
Content-Security-Policy: img-src https://*
```
*   允许加载任何来源框架
``` html
Content-Security-Policy: child-src 'none'
```
也可以通过 HTML 的 meta 标签
```html
<meta http-equiv="Content-Security-Policy" content="script-src 'self' ">
```
具体使用方式见 [https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CSP](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CSP)

# CSRF
跨站请求伪造（Cross-site request forgery / one-click attack / session riding / CSRF）是一种挟制用户在当前已登录的 Web 应用程序上执行非本意的操作的攻击方法

。XSS 利用的是用户对指定网站的信任，CSRF 利用的是网站对用户网页浏览器的信任。

简单点说，CSRF 就是利用用户的登录态发起恶意请求（一般是利用浏览器自动发送 cookie 的机制）。

## 如何攻击
假设网站中有一个通过 Get 请求提交用户评论的接口，那么攻击者就可以在钓鱼网站中加入一个图片，图片的地址就是评论接口
```html
<img src="http://www.domain.com/xxx?comment='attack'"/>
```
如果接口是 Post 提交的，就相对麻烦点，需要用表单来提交接口
```html
<form action="http://www.domain.com/xxx" id="CSRF" method="post">
    <input name="comment" value="attack" type="hidden">
</form>
```

这里多提一句 Cookie 的发送机制：当向 HTTP 服务器请求某个 URL 时，浏览器将该 URL 与客户主机中存储的所有 Cookie 比较，如果发现域名相匹配 Cookie，则匹配  Cookie 中包含名字/值的那一行将被包含在 HTTP 请求头中，以保证依赖于 Cookie 的功能得以实现。

虽然一般情况下浏览器的跨域限制会保护我们的 cookie，但是就如上面的简单的例子 img 标签的 src 是不受跨域限制的，那么 domain 属性与这个 GET 请求的 domain 相同的 Cookies 都会被自动携带在请求头中。

<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/BlogImgs/1595efac21dbf0bddbd78bb9a42abb12_754x338.png" />

## 如何防范
防范 CSRF 可以遵循以下几种规则：

1.  Get 请求不对数据进行修改

2.  不让第三方网站访问到用户 Cookie

3.  阻止第三方网站请求接口

4.  请求时附带验证信息，比如验证码或者 token（比如利用 axios 拦截器）

### SameSite
可以对 Cookie 设置 `SameSite` 属性。该属性设置 Cookie 不随着跨域请求发送，该属性可以很大程度减少 CSRF 的攻击，但是该属性目前并不是所有浏览器都兼容。

### 验证 Referer
对于需要防范 CSRF 的请求，我们可以通过验证 Referer 来判断该请求是否为第三方网站发起的。HTTP Referer 是 HTTP 请求头部的一部分，浏览器向 Web 服务器发送请求时一般都会自动带上 Referer 以告诉服务器我是从哪个页面来的。

### 使用验证码
验证码被认为是对抗 CSRF 攻击最简洁而有效的防御方法。

CSRF 攻击往往是在用户不知情的情况下构造了网络请求。而验证码会强制用户必须与应用进行交互（验证码由服务器随机给出），才能完成最终请求。因此通常情况下，验证码能够很好地遏制 CSRF 攻击。

但验证码并不是万能的，因为出于用户考虑，不能给网站所有的操作都加上验证码。因此，验证码只能作为防御 CSRF 的一种辅助手段，而不能作为最主要的解决方案。
### Token

服务器下发一个随机 Token（算法不能复杂），每次发起请求时将 Token 携带上，服务器验证 Token 是否有效。

客户端可以把 token 存在 cookie（如果 token 不是用于验证的话，就防范 CSRF 而言就不存在 cookie 中了）或者 Local / Session Storage中。

# OAuth2
OAuth2 是一个开放授权标准，它允许用户让第三方应用访问该用户在某服务的特定私有资源但是不提供账号密码信息给第三方应用。

## OAuth2 的四个重要角色
1、`Resource Owner`：资源拥有者

2、`Resource Server`：资源服务器

3、`Client`：第三方应用客户端

4、`Authorization Server`：授权服务器，管理 Resource Owner，Client 和 Resource Server 的三角关系的中间层

OAuth2 解决问题的关键在于使用 Authorization server 提供一个访问凭据给 Client，使得 Client 可以在不知道 Resource owner 在 Resource server 上的用户名和密码的情况下消费 Resource owner 的受保护资源。

我们使用的第三方登录很多功能都遵守 OAth2 规范，如 [QQ OAuth2 API](https://wiki.connect.qq.com/%E4%BD%BF%E7%94%A8authorization_code%E8%8E%B7%E5%8F%96access_token) [微信公众号获取 access_token](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421140183)

## 部署 OAuth2 需要完成的工作
由于 OAuth2 引入了 Authorization server 来管理 Resource Owner，Client 和 Resource Server 的三角关系，那么想要用上 OAuth2，是必须要实现以下功能的：

1.  增加一个 Authorization server，提供授权的实现，一般由 Resource server 来提供。

2.  Resource server 为第三方应用程序提供注册接口。

3.  Resource server 开放相应的受保护资源的 API。

4.  Client 注册成为 Resource server 的第三方应用。

5.  Client 消费这些 API。

作为资源服务提供商来说，1，2，3 这三件事情是需要完成的。

作为第三方应用程序，要完成的工作是在 4 和 5 这两个步骤中。

其中作为 Resource owner 来说，是不用做什么的，是 OAuth2 受益的千千万万的最终人类用户。

## 资源服务提供商和用户的职责
在一般情况下，Resource server 提供 Authorization server 服务，主要提供两类接口：

- 授权接口：接受 Client 的授权请求，引导用户到 Resource server 完成登陆授权的过程。
- 获取访问令牌接口：使用授权接口提供的许可凭据来颁发 Resource owner 的访问令牌给 Client，或者由 Client 更新过期的访问令牌。

除此之外，还需要提供一个第三方应用程序注册管理的服务。通常情况下会为注册完成的第三方应用程序分配两个成对出现的重要参数：

- client\_id：第三方应用程序的一个标识 id，这个信息通常是公开的信息，用来区分哪一个第三方应用程序。
- client\_secret：第三方应用程序的私钥信息，这个信息是私密的信息，不允许在 OAuth2 流程中传递的，用于安全方面的检测和加密。


作为 Client：在 Client 取得 client\_id 和 client\_secret 之后。使用这些信息来发起授权请求、获取 access\_token 请求和消费受保护的资源。

## OAuth2 授权流程
1\. 消费方：把用户带到服务器提供方登录

2\. 服务提供方：获得授权

3\. 服务提供方：把用户重定向到消费方

4\. 消费方：使用授权码请求访问令牌

5\. 服务提供方：核发访问令牌

6\. 消费方：使用访问令牌访问受保护的资源

<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/BlogImgs/75a30332c17413082c2b07d8778ca225_762x684.png" />

这其中比较重要的一个概念是 **访问令牌**，它代表的信息是整个 OAuth2 的核心，也是前 4 步最终要得到的信息。

访问令牌是对 **消费方可访问的用户的哪些信息** 这个完整权限的一个抽象

访问令牌背后抽象的信息有哪些呢？如下 3 类信息。

1.  客户端标识

2.  用户标识

3.  客户端能访问资源所有者的哪些资源以及其相应的权限。

有了这三类信息，那么资源服务器（Resouce Server）就可以区分出来是哪个第三方应用（Client）要访问哪个用户（Resource Owner）的哪些资源（以及有没有权限）。

# DDoS 攻击与防范
DDoS 的攻击原理，往简单说，其实就是利用 TCP/UDP 协议规律，通过占用协议栈资源或者发起大流量拥塞，达到消耗目标机器性能或者网络的目的。

按照攻击对象的不同，将对攻击原理和攻击危害的分析分成 3 类，分别是攻击网络带宽资源、系统以及应用。具体可以参考 [这篇文章](https://www.cnblogs.com/163yun/archive/2018/06/01/9121857.html)

其中，比较常见的一种攻击是 cc 攻击。它就是简单粗暴地送来大量正常的请求，超出服务器的最大承受量，导致宕机。

可以看看阮一峰老师的个人博客遭受 DDoS 攻击时 [采取的措施](https://www.cnblogs.com/southx/p/9414695.html)

<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/BlogImgs/72390a784c7146b959214e37a68aa790_1720x783.png" />

# 参考链接

[xss 与 csrf](https://github.com/dwqs/blog/issues/68) 
 
[oauth2](https://www.cnblogs.com/linianhui/p/oauth2-authorization.html)

[token](https://juejin.im/post/58da720b570c350058ecd40f)

[node 中常用的加密算法](https://www.cnblogs.com/laogai/p/4664917.html)

[http://louiszhai.github.io/2016/03/05/xss/#Content-Security-Policy](http://louiszhai.github.io/2016/03/05/xss/#Content-Security-Policy)

[https://content-security-policy.com/](https://content-security-policy.com/)

[https://www.cnblogs.com/southx/p/9414695.html](https://www.cnblogs.com/southx/p/9414695.html)

[https://www.cnblogs.com/163yun/archive/2018/06/01/9121857.html](https://www.cnblogs.com/163yun/archive/2018/06/01/9121857.html)
