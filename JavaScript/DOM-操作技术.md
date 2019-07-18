# DOM 操作技术
## 动态脚本
```js
function loadScript (url) {
  var script = document.createElement("script")
  script.type = "text/javascript"
  script.src = url
  document.body.appendChild(script)
} 
```
## 动态样式
```js
function loadStyles (url) {
  var link = document.createElement("link")
  link.rel = "stylesheet"
  link.type = "text/css"
  link.href = url
  var head = document.getElementsByTagName("head")[0]
  head.appendChild(link)
} 
```
href 表示超文本引用（hypertext reference），在 link 和 a 等元素上使用。src 表示来源地址，在 img、script、iframe 等元素上。  

src 的内容，是页面必不可少的一部分，是引入。href 的内容，是与该页面有关联，是引用。两者的区别就是对于当前页面来说是引入还是引用（只是从意义上讲）。
## 使用 NodeList
理解 NodeList 及其“近亲” NamedNodeMap 和 HTMLCollection，是从整体上透彻理解 DOM 的关键所在。


这三个集合都是“动态的”，换句话说，每当文档结构发生变化时，它们都会得到更新。


因此，它们始终都会保存着最新、最准确的信息。从本质上说，所有 NodeList 对象都是在访问 DOM 文档时实时运行的查询。

例如，下列代码会导致无限循环
```js
var divs = document.getElementsByTagName("div"),
    i,
    div
    
for (i=0; i < divs.length; i++) {
  div = document.createElement("div")
  document.body.appendChild(div)
}
```
第一行代码会取得文档中所有\<div>元素的 HTMLCollection。


由于这个集合是“动态的”，因此只要有新\<div>元素被添加到页面中，这个元素也会被添加到该集合中。浏览器不会将创建的所有集合都保存在一个列表中，
而是在下一次访问集合时再更新集合。

结果，在遇到上例中所示的循环代码时，
就会导致一个有趣的问题。每次循环都要对条件 i < divs.length 求值，意味着会运行取得所有\<div>元素的查询。


考虑到循环体每次都会创建一个新\<div>元素并将其添加到文档中，因此 divs.length 的值在每次循环后都会递增。
既然 i 和 divs.length 每次都会同时递增，结果它们的值永远也不会相等。


如果想要迭代一个 NodeList，最好是使用 length 属性初始化第二个变量，然后将迭代器与该变量进行比较，如下面的例子所示：
```js
var divs = document.getElementsByTagName("div"),
    i,
    len,
    div
    
for (i = 0, len = divs.length; i < len; i++) {
  div = document.createElement("div")
  document.body.appendChild(div)
} 
```
一般来说，应该尽量减少访问 NodeList 的次数。因为每次访问 NodeList，都会运行一次基于文档的查询。所以，可以考虑将从 NodeList 中取得的值缓存起来

## 焦点管理
HTML5 也添加了辅助管理 DOM 焦点的功能。首先就是 document.activeElement 属性，这个属性始终会引用 DOM 中当前获得了焦点的元素。元
素获得焦点的方式有页面加载、用户输入（通常是通过按 Tab 键）和在代码中调用 focus() 方法。来看几个例子。
```js
var button = document.getElementById("myButton")
button.focus()
alert(document.activeElement === button) // true 
```
默认情况下，文档刚刚加载完成时，document.activeElement 中保存的是 document.body 元素的引用。
文档加载期间，document.activeElement 的值为 null。 另外就是新增了 document.hasFocus() 方法，这个方法用于确定文档是否获得了焦点
```js
var button = document.getElementById("myButton")
button.focus()
alert(document.hasFocus()) // true
```
通过检测文档是否获得了焦点，可以知道用户是不是正在与页面交互。查询文档获知哪个元素获得了焦点，以及确定文档是否获得了焦点，
这两个功能最重要的用途是提高 Web 应用的无障碍性。无障碍 Web 应用的一个主要标志就是恰当的焦点管理，
而确切地知道哪个元素获得了焦点是一个极大的进步，至少我们不用再像过去那样靠猜测了。

## 自定义数据属性
HTML5 规定可以为元素添加非标准的属性，但要添加前缀 data-，目的是为元素提供与渲染无关的信息，或者提供语义信息。这些属性可以任意添加、随便命名，只要以 data-开头即可。来看一个例子
```html
<div id="myDiv" data-appId="12345" data-myname="Nicholas"></div> 
```
添加了自定义属性之后，可以通过元素的 dataset 属性来访问自定义属性的值。dataset 属性的值是 DOMStringMap 的一个实例，
也就是一个名值对儿的映射。在这个映射中，每个 data-name 形式的属性都会有一个对应的属性，只不过属性名没有 data-前缀（比如，自定义属性是 data-myname， 那映射中对应的属性就是 myname）。还是看一个例子吧
```js
// 本例中使用的方法仅用于演示
var div = document.getElementById("myDiv")
// 取得自定义属性的值
var appId = div.dataset.appId
var myName = div.dataset.myname
// 设置值
div.dataset.appId = 23456
div.dataset.myname = "Michael"
// 有没有"myname"值呢？
if (div.dataset.myname) { 
 alert("Hello, " + div.dataset.myname)
} 
```
如果需要给元素添加一些不可见的数据以便进行其他处理，那就要用到自定义数据属性。在跟踪链接或混搭应用中，
通过自定义数据属性能方便地知道点击来自页面中的哪个部分

## innerHTML 与 outerHTML
### innerHTML
在读模式下，innerHTML 属性返回与调用元素的所有子节点（包括元素、注释和文本节点）对应的 HTML 标记。
在写模式下，innerHTML 会根据指定的值创建新的 DOM 树，然后用这个 DOM 树完全替换调用元素原先的所有子节点。下面是一个例子
```html
<div id="content">
 <p>This is a <strong>paragraph</strong> with a list following it.</p>
 <ul>
   <li>Item 1</li>
   <li>Item 2</li>
   <li>Item 3</li>
 </ul>
</div>
对于上面的<div>元素来说，它的 innerHTML 属性会返回如下字符串。
<p>This is a <strong>paragraph</strong> with a list following it.</p>
<ul>
 <li>Item 1</li>
 <li>Item 2</li>
 <li>Item 3</li>
</ul> 
```
但是，不同浏览器返回的文本格式会有所不同。IE 和 Opera 会将所有标签转换为大写形式，
而 Safari、 Chrome 和 Firefox 则会原原本本地按照原先文档中（或指定这些标签时）的格式返回 HTML，包括空格和缩进。
不要指望所有浏览器返回的 innerHTML 值完全相同。 


在写模式下，innerHTML 的值会被解析为 DOM 子树，替换调用元素原来的所有子节点。因为它的值被认为是 HTML，
所以其中的所有标签都会按照浏览器处理 HTML 的标准方式转换为元素（同样， 这里的转换结果也因浏览器而异）。
如果设置的值仅是文本而没有 HTML 标签，那么结果就是设置纯文本。
### outerHTML
在读模式下，outerHTML 返回调用它的元素及所有子节点的 HTML 标签。在写模式下，outerHTML 会根据指定的 HTML 字符串创建新的 DOM 子树，
然后用这个 DOM 子树完全替换调用元素。


使用 outerHTML 属性以下面这种方式设置值：
```html
div.outerHTML = "<p>This is a paragraph.</p>";
```
这行代码完成的操作与下面这些 DOM 脚本代码一样：
```js
var p = document.createElement("p")
p.appendChild(document.createTextNode("This is a paragraph."))
div.parentNode.replaceChild(p, div)
```
结果，就是新创建的\<p>元素会取代 DOM 树中的\<div>元素。
一般来说，在插入大量新 HTML 标记时，使用 innerHTML 属性与通过多次 DOM 操作先创建节点再指定它们之间的关系相比，
效率要高得多。这是因为在设置 innerHTML 或 outerHTML 时，就会创建一个 HTML 解析器。


这个解析器是在浏览器级别的代码（通常是 C++ 编写的）基础上运行的，因此比执行 JavaScript 快得多。
不可避免地，创建和销毁 HTML 解析器也会带来性能损失，所以最好能够将设置 innerHTML 或 outerHTML 的次数控制在合理的范围内。


例如，下列代码使用 innerHTML 创建了很多列表项：
```js
for (var i = 0, len = values.length; i < len; i++) {
 ul.innerHTML += "<li>" + values[i] + "</li>" // 要避免这种频繁操作！！
}
```
这种每次循环都设置一次 innerHTML 的做法效率很低。而且，每次循环还要从 innerHTML 中读取一次信息，就意味着每次循环要访问两次 innerHTML。


最好的做法是单独构建字符串，然后再一次性地将结果字符串赋值给 innerHTML，像下面这样：
```js
var itemsHtml = ""
for (var i = 0, len = values.length; i < len; i++) {
 itemsHtml += "<li>" + values[i] + "</li>"
}
ul.innerHTML = itemsHtml
```
这个例子的效率要高得多，因为它只对 innerHTML 执行了一次赋值操作

## scrollIntoView() 方法
scrollIntoView() 可以在所有 HTML 元素上调用，通过滚动浏览器窗口或某个容器元素，调用元素就可以出现在视口中。如果给这个方法传入 true 作为参数，或者不传入任何参数，那么窗口滚动之后会让调用元素的顶部与视口顶部尽可能平齐。如果传入 false 作为参数，调用元素会尽可能全部出现在视口中，（可能的话，调用元素的底部会与视口顶部平齐。）不过顶部不一定平齐，例如：
```js
// 让元素可见
document.forms[0].scrollIntoView()
```
当页面发生变化时，一般会用这个方法来吸引用户的注意力。实际上，**为某个元素设置焦点也会导致浏览器滚动并显示出获得焦点的元素**

## 访问元素的样式
对于使用短划线（分隔不同的词汇，例如 background-image）的 CSS 属性名，必须将其转换成驼峰大小写形式，
才能通过 JavaScript 来访问。下表列出了几个常见的 CSS 属性及 其在 style 对象中对应的属性名。


|  CSS 属性   | JavaScript 属性    |
| :---- | :---- |
|  background-image   | style.backgroundImage    |
|  color   | style.color|
|  display  |   style.display  |
|  font-family   | style.fontFamily|

# 元素大小
## 偏移量
首先要介绍的属性涉及偏移量（offset dimension），包括元素在屏幕上占用的所有可见的空间。元素的可见大小由其高度、宽度决定，包括所有内边距、滚动条和边框大小（注意，不包括外边距）。通过下列 4 个属性可以取得元素的偏移量。
- offsetHeight：元素在垂直方向上占用的空间大小，以像素计。包括元素的高度、（可见的）水平滚动条的高度、上边框高度和下边框高度。
- offsetWidth：元素在水平方向上占用的空间大小，以像素计。包括元素的宽度、（可见的）垂直滚动条的宽度、左边框宽度和右边框宽度。
- offsetLeft：元素的左外边框至包含元素的左内边框之间的像素距离。
- offsetTop：元素的上外边框至包含元素的上内边框之间的像素距离。

<img src="https://box.kancloud.cn/25813e8ca7ee75147831b0e86d283632_605x346.png" />


## 客户区大小
元素的客户区大小（client dimension），指的是元素**内容及其内边距**所占据的空间大小。
有关客户区大小的属性有两个：clientWidth 和 clientHeight。
其中，clientWidth 属性是元素内容区宽度加上左右内边距宽度；clientHeight 属性是元素内容区高度加上上下内边距高度。


下图形象地说明了这些属性表示的大小。


<img src="https://box.kancloud.cn/e48023b6abe1f39ad7b8147c1564c426_575x345.png" />


可以用来确认浏览器视口大小（这不是用 window.innerWidth 和 window.innerHeight 就行了吗？）
```js
function getViewport() {
 return {
   width: document.documentElement.clientWidth,
   height: document.documentElement.clientHeight
 }
} 
```
## 滚动大小
以下是 4 个与滚动大小相关的属性：
- scrollHeight：在没有滚动条的情况下，元素内容的总高度。
- scrollWidth：在没有滚动条的情况下，元素内容的总宽度。
- scrollLeft：被隐藏在内容区域左侧的像素数。通过设置这个属性可以改变元素的滚动位置。
- scrollTop：被隐藏在内容区域上方的像素数。通过设置这个属性可以改变元素的滚动位置。

<img src="https://box.kancloud.cn/39713029899f60e71bf608f8ac365562_647x393.png" />


通过 scrollLeft 和 scrollTop 属性既可以确定元素当前滚动的状态，也可以设置元素的滚动位置。


在元素尚未被滚动时，这两个属性的值都等于 0。如果元素被垂直滚动了，那么 scrollTop 的值会大于 0，
且表示元素上方不可见内容的像素高度。如果元素被水平滚动了，那么 scrollLeft 的值会大于 0，且表示元素左侧不可见内容的像素宽度。


这两个属性都是可以设置的，因此将元素的 scrollLeft 和 scrollTop 设置为 0，就可以重置元素的滚动位置。


下面这个函数会检测元素是否位于顶部，如果不是就将其回滚到顶部。
```js
function scrollToTop (element) {
 if (element.scrollTop != 0) {
   element.scrollTop = 0
 }
}
```
这个函数既取得了 scrollTop 的值，也设置了它的值
