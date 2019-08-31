# OPTIONS 请求方法
HTTP 的 `OPTIONS 方法` 用于获取目的资源所支持的通信选项。

以下三种情况浏览器会在发送正式请求先自动发送一个 OPTIONS 请求：
- 跨域请求，非跨域请求不会出现 options 请求  
- 请求设置了自定义的 header 字段
- 请求头中的 Content-Type 是 application/x-www-form-urlencoded ，multipart/form-data，text/plain 之外的格式

在 CORS  中，可以使用 OPTIONS 方法发起一个预检请求，以检测实际请求是否可以被服务器所接受。预检请求报文中的 `Access-Control-Request-Method` 首部字段告知服务器实际请求所使用的 HTTP 方法；`Access-Control-Request-Headers` 首部字段告知服务器实际请求所携带的自定义首部字段。服务器基于从预检请求获得的信息来判断，是否接受接下来的实际请求。
```html
OPTIONS /resources/post-here/ HTTP/1.1 
Host: bar.other 
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8 
Accept-Language: en-us,en;q=0.5 
Accept-Encoding: gzip,deflate 
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7 
Connection: keep-alive 
Origin: http://foo.example 
Access-Control-Request-Method: POST 
Access-Control-Request-Headers: X-PINGOTHER, Content-Type
```
服务器所返回的`Access-Control-Allow-Methods`首部字段将所有允许的请求方法告知客户端。
```html
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:15:39 GMT 
Server: Apache/2.0.61 (Unix) 
Access-Control-Allow-Origin: http://foo.example 
Access-Control-Allow-Methods: POST, GET, OPTIONS 
Access-Control-Allow-Headers: X-PINGOTHER, Content-Type 
Access-Control-Max-Age: 86400 
Vary: Accept-Encoding, Origin 
Content-Encoding: gzip 
Content-Length: 0 
Keep-Alive: timeout=2, max=100 
Connection: Keep-Alive 
Content-Type: text/plain
```
# XMLHttpRequest
|  XMLHttpRequest对象方法 |  描述   |
| :---- | :---- |
|   xhr.open('get', 'example.php', false)  |  启动一个请求以备发送，第三个参数为false表示异步，true表示同步发送   |
|   xhr.send(String)  |  发送一个请求，接受一个可选的参数，其作为请求主体；如果请求方法是 GET 或者 HEAD，则应将请求主体设置为 null。   |
|   xhr.onreadystatechange= function () {}  |  只要readyState属性的值由一个值变为另一个值，都会触发一次readystatechange事件  |
|  xhr.setRequestHeader('MyHeader', 'MyValue')  |  用于设置自定义的请求头部信息，必须在调用open方法之后和send方法之前调用  |

|  XMLHttpRequest对象属性 |  描述   |
| :---- | :---- |
|   xhr.status   |    响应的HTTP状态  |
|  xhr.statusText    |    HTTP状态的说明  |
|   xhr.responseXML   |   如果响应的内容类型是"text/html"或"application/xml"，这个属性将保存包含着响应数据的XML DOM文档   |
|   xhr.readyState |  0:未初始化<br>1:启动。已经调用open()方法，但尚未调用send()方法<br>2:发送。已经调用send()方法，但尚未接收到相应。<br>3:接收。已经收到部分响应数据。<br>4:完成:已经收到全部响应数据，而且已经可以在客户端使用了  |

|   进度事件  |  描述   |
| :---- | :---- |
|   loadstart  |  在接收到相应数据的第一个字节时触发   |
|   progress|  在接收响应期间持续不断地触发  |
|   error  | 在请求发生错误时触发   |
|   abort|  在因调用abort()方法而终止连接时触发   |
|   load|  在接收到完整的响应数据时触发   |
|  loadend  |  在通信完成或者触发error、abort或load事件后触发   |

onprogress 事件回调方法在 `readyState==3` 状态时开始触发, 默认传入 ProgressEvent 对象, 可通过 `e.loaded/e.total` 来计算加载资源的进度, 该方法用于获取资源的下载进度.
```javaScript
xhr.onprogress = function(e){ console.log('progress:', e.loaded/e.total); }
```

更完整的整理：[AJAX 知识体系大梳理](http://louiszhai.github.io/2016/11/02/ajax/#%E4%BD%BF%E7%94%A8%E5%91%BD%E4%BB%A4%E6%B5%8B%E8%AF%95OPTIONS%E8%AF%B7%E6%B1%82)
# axios 库的使用
安装：`npm install aoxis`

导入：`import axios from axios`
## 请求发送的格式
1.`axios(config)`：通过向 axios 传递相关配置来创建请求
```js
// 发送 POST 请求
axios({
  method: 'post',
  url: '/user/12345',
  data: {
    firstName: 'Fred',
    lastName: 'Flintstone'
  }
});
```
2.`axios(url[, config])`
```js
// 发送 GET 请求（默认的方法）
axios('/user/12345');
```
**请求方法的别名**
为方便起见，为所有支持的请求方法提供了别名

axios.request(config)

axios.get(url\[, config\])

axios.delete(url\[, config\])

axios.head(url\[, config\])

axios.post(url\[, data\[, config\]\])

axios.put(url\[, data\[, config\]\])

axios.patch(url\[, data\[, config\]\])

在使用别名方法时，`url`、`method`、`data` 这些属性都不必在配置中指定。

1.只传入一个参数，参数写在URL中
```js
// 为给定 ID 的 user 创建请求
axios.get('/user?ID=12345')
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  });
```
2.第一个参数为 URL，第二个参数为配置选项；`params` 是与请求一起发送的URL参数，`data` 是作为请求主体被发送的数据，还有其他很多配置项。
```js
// 可选地，上面的请求可以这样做
axios.get('/user', {
    params: {
      ID: 12345
    }
  })
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  });
```
3.可以自定义新建一个axios的实例，`axios.create(config)`
```js
var instance = axios.create({
  baseURL: 'https://some-domain.com/api/',
  timeout: 1000,
  headers: {'X-Custom-Header': 'foobar'}
});
```
**实例方法**

以下是可用的实例方法。指定的配置将与实例的配置合并

axios#request(config)

axios#get(url\[, config\])

axios#delete(url\[, config\])

axios#head(url\[, config\])

axios#post(url\[, data\[, config\]\])

axios#put(url\[, data\[, config\]\])

axios#patch(url\[, data\[, config\]\])

## 请求配置与响应结构
```
{
  // `url` 是用于请求的服务器 URL
  url: '/user',

  // `method` 是创建请求时使用的方法
  method: 'get', // 默认是 get

  // `baseURL` 将自动加在 `url` 前面，除非 `url` 是一个绝对 URL。
  // 它可以通过设置一个 `baseURL` 便于为 axios 实例的方法传递相对 URL
  baseURL: 'https://some-domain.com/api/',

  // `transformRequest` 允许在向服务器发送前，修改请求数据
  // 只能用在 'PUT', 'POST' 和 'PATCH' 这几个请求方法
  // 后面数组中的函数必须返回一个字符串，或 ArrayBuffer，或 Stream
  transformRequest: [function (data) {
    // 对 data 进行任意转换处理

    return data;
  }],

  // `transformResponse` 在传递给 then/catch 前，允许修改响应数据
  transformResponse: [function (data) {
    // 对 data 进行任意转换处理

    return data;
  }],

  // `headers` 是即将被发送的自定义请求头
  headers: {'X-Requested-With': 'XMLHttpRequest'},

  // `params` 是即将与请求一起发送的 URL 参数
  // 必须是一个无格式对象(plain object)或 URLSearchParams 对象
  params: {
    ID: 12345
  },

  // `paramsSerializer` 是一个负责 `params` 序列化的函数
  // (e.g. https://www.npmjs.com/package/qs, http://api.jquery.com/jquery.param/)
  paramsSerializer: function(params) {
    return Qs.stringify(params, {arrayFormat: 'brackets'})
  },

  // `data` 是作为请求主体被发送的数据
  // 只适用于这些请求方法 'PUT', 'POST', 和 'PATCH'
  // 在没有设置 `transformRequest` 时，必须是以下类型之一：
  // - string, plain object, ArrayBuffer, ArrayBufferView, URLSearchParams
  // - 浏览器专属：FormData, File, Blob
  // - Node 专属： Stream
  data: {
    firstName: 'Fred'
  },

  // `timeout` 指定请求超时的毫秒数(0 表示无超时时间)
  // 如果请求话费了超过 `timeout` 的时间，请求将被中断
  timeout: 1000,

  // `withCredentials` 表示跨域请求时是否需要使用凭证
  withCredentials: false, // 默认的

  // `adapter` 允许自定义处理请求，以使测试更轻松
  // 返回一个 promise 并应用一个有效的响应 (查阅 [response docs](#response-api)).
  adapter: function (config) {
    /* ... */
  },

  // `auth` 表示应该使用 HTTP 基础验证，并提供凭据
  // 这将设置一个 `Authorization` 头，覆写掉现有的任意使用 `headers` 设置的自定义 `Authorization`头
  auth: {
    username: 'janedoe',
    password: 's00pers3cret'
  },

  // `responseType` 表示服务器响应的数据类型，可以是 'arraybuffer', 'blob', 'document', 'json', 'text', 'stream'
  responseType: 'json', // 默认的

  // `xsrfCookieName` 是用作 xsrf token 的值的cookie的名称
  xsrfCookieName: 'XSRF-TOKEN', // default

  // `xsrfHeaderName` 是承载 xsrf token 的值的 HTTP 头的名称
  xsrfHeaderName: 'X-XSRF-TOKEN', // 默认的

  // `onUploadProgress` 允许为上传处理进度事件
  onUploadProgress: function (progressEvent) {
    // 对原生进度事件的处理
  },

  // `onDownloadProgress` 允许为下载处理进度事件
  onDownloadProgress: function (progressEvent) {
    // 对原生进度事件的处理
  },

  // `maxContentLength` 定义允许的响应内容的最大尺寸
  maxContentLength: 2000,

  // `validateStatus` 定义对于给定的HTTP 响应状态码是 resolve 或 reject  promise 。如果 `validateStatus` 返回 `true` (或者设置为 `null` 或 `undefined`)，promise 将被 resolve; 否则，promise 将被 rejecte
  validateStatus: function (status) {
    return status >= 200 && status < 300; // 默认的
  },

  // `maxRedirects` 定义在 node.js 中 follow 的最大重定向数目
  // 如果设置为0，将不会 follow 任何重定向
  maxRedirects: 5, // 默认的

  // `httpAgent` 和 `httpsAgent` 分别在 node.js 中用于定义在执行 http 和 https 时使用的自定义代理。允许像这样配置选项：
  // `keepAlive` 默认没有启用
  httpAgent: new http.Agent({ keepAlive: true }),
  httpsAgent: new https.Agent({ keepAlive: true }),

  // 'proxy' 定义代理服务器的主机名称和端口
  // `auth` 表示 HTTP 基础验证应当用于连接代理，并提供凭据
  // 这将会设置一个 `Proxy-Authorization` 头，覆写掉已有的通过使用 `header` 设置的自定义 `Proxy-Authorization` 头。
  proxy: {
    host: '127.0.0.1',
    port: 9000,
    auth: : {
      username: 'mikeymike',
      password: 'rapunz3l'
    }
  },

  // `cancelToken` 指定用于取消请求的 cancel token
  // （查看后面的 Cancellation 这节了解更多）
  cancelToken: new CancelToken(function (cancel) {
  })
}
```
响应结构：某个请求的响应包含如下信息
```js
{
  data: {},          // `data` 由服务器提供的响应
  status: 200,       // `status` 来自服务器响应的 HTTP 状态码
  statusText: 'OK',  // `statusText` 来自服务器响应的 HTTP 状态信息
  headers: {},       // `headers` 服务器响应的头
  config: {}         // `config` 是为请求提供的配置信息
}
```

# fetch API
参考：[https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch\_API/Using\_Fetch](https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API/Using_Fetch)

Fetch API 提供了一个全局 `fetch()` 方法，该方法提供了一种简单，合理的方式来跨网络异步获取资源。

这种功能以前是使用  `XMLHttpRequest` 实现的，使用 XMLHttpRequest (XHR) 对象可以与服务器交互。

你可以从 URL 获取数据，而无需让整个的页面刷新。这使得 Web 页面可以只更新页面的局部，而不影响用户的操作。

XMLHttpRequest 在 Ajax 编程中被大量使用。Fetch 提供了一个更好的替代方法，可以很容易地被其他技术使用，例如 `Service Workers`。

Fetch 还提供了单个逻辑位置来定义其他 HTTP 相关概念，例如 CORS 和 HTTP 的扩展。

请注意，`fetch` 规范与 `jQuery.ajax()` 主要有两种方式的不同，牢记：

*   当接收到一个代表错误的 HTTP 状态码时，从 `fetch()` 返回的 Promise **不会被标记为 reject，** 即使该 HTTP 响应的状态码是 404 或 500。相反，它会将 Promise 状态标记为 resolve （但是会将 resolve 的返回值的`ok`属性设置为 false ），仅当网络故障时或请求被阻止时，才会标记为 reject。
*   默认情况下，`fetch` **不会从服务端发送或接收任何 cookies**, 如果站点依赖于用户 session，则会导致未经认证的请求（要发送 cookies，必须设置 credentials 选项）。

一个基本的 fetch 请求设置起来很简单。看看下面的代码：

```js
fetch('http://example.com/movies.json')
  .then(function(response) {
    return response.json();
  })
  .then(function(myJson) {
    console.log(myJson);
  });
```
这里我们通过网络获取一个 JSON 文件并将其打印到控制台。最简单的用法是只提供一个参数用来指明想`fetch()`到的资源路径，然后返回一个包含响应结果的 promise 对象。使用 Response 对象的 json() 方法可以获取 JSON 的内容

`fetch()`接受第二个可选参数，一个可以控制不同配置的 `init` 对象：
```js
// Example POST method implementation:

postData('http://example.com/answer', {answer: 42})
  .then(data => console.log(data)) // JSON from `response.json()` call
  .catch(error => console.error(error))

function postData(url, data) {
  // Default options are marked with *
  return fetch(url, {
    body: JSON.stringify(data), // must match 'Content-Type' header
    cache: 'no-cache', // *default, no-cache, reload, force-cache, only-if-cached
    credentials: 'same-origin', // include, same-origin, *omit
    headers: {
      'user-agent': 'Mozilla/4.0 MDN Example',
      'content-type': 'application/json'
    },
    method: 'POST', // *GET, POST, PUT, DELETE, etc.
    mode: 'cors', // no-cors, cors, *same-origin
    redirect: 'follow', // manual, *follow, error
    referrer: 'no-referrer', // *client, no-referrer
  })
  .then(response => response.json()) // parses response to JSON
}
```
## 实践
**上传 JSON 数据**
```js
var url = 'https://example.com/profile';
var data = {username: 'example'};

fetch(url, {
  method: 'POST', // or 'PUT'
  body: JSON.stringify(data), // data can be `string` or {object}!
  headers: new Headers({
    'Content-Type': 'application/json'
  })
}).then(res => res.json())
.catch(error => console.error('Error:', error))
.then(response => console.log('Success:', response));
```
**上传文件**
```js
var formData = new FormData();
var fileField = document.querySelector("input[type='file']");

formData.append('username', 'abc123');
formData.append('avatar', fileField.files[0]);

fetch('https://example.com/profile/avatar', {
  method: 'PUT',
  body: formData
})
.then(response => response.json())
.catch(error => console.error('Error:', error))
.then(response => console.log('Success:', response));
```

