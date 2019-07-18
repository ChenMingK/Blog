## DOM（文档对象模型）
根据 W3C 的 HTML DOM 标准，HTML 文档中的所有内容都是节点：
- 整个文档是一个文档节点
- 每个 HTML 元素是元素节点
- HTML 元素内的文本是文本节点
- 每个 HTML 属性是属性节点
- 注释是注释节点

## 节点类型概述
Document 类型表示整个文档，是一组分层节点的根节点。在 JavaScript 中，document 对象是
 Document 的一个实例。使用 document 对象，有很多种方式可以查询和取得节点。
Element 节点表示文档中的所有 HTML 或 XML 元素，可以用来操作这些元素的内容和特性。
另外还有一些节点类型，分别表示文本内容、注释、文档类型、CDATA 区域和文档片段。
## Element 类型
Element 类型可以说是 Web 编程中最常用的类型，
Element 节点具有以下特征：
- nodeType 的值为 1； 
- nodeName 的值为元素的标签名； 
- nodeValue 的值为 null； 
- parentNode 可能是 Document 或 Element； 
- 其子节点可能是 Element、Text、Comment、ProcessingInstruction、CDATASection 或 EntityReference。

要访问元素的标签名，可以使用 nodeName 属性，也可以使用 tagName 属性；这两个属性会返回相同的值（使用后者主要是为了清晰起见）。
需要注意的是 div.tagName 实际上输出的是 'DIV' 而非 'div'，所以最好是这么比较
```js
// 这样最好（适用于任何文档）
if (element.tagName.toLowerCase() == "div"){ 
 // do something...
} 
```
### 1.HTML元素
所有 HTML 元素都由 HTMLElement 类型表示，不是直接通过这个类型，也是通过它的子类型来表示。HTMLElement 类型直接继承自 Element 并添加了一些属性。每个 HTML 元素中都存在的下列标准特性：
- id，元素在文档中的唯一标识符。
- title，有关元素的附加说明信息，一般通过工具提示条显示出来。
- lang，元素内容的语言代码，很少使用。
- dir，语言的方向，值为"ltr"（left-to-right，从左至右）或"rtl"（right-to-left，从右至左），
也很少使用。
- className，与元素的 class 特性对应，即为元素指定的 CSS 类。没有将这个属性命名为 class，是因为 class 是 ECMAScript 的保留字

```js
<div id="myDiv" class="bd" title="Body text" lang="en" dir="ltr"></div> 

var div = document.getElementById("myDiv")

alert(div.id)        // "myDiv""
alert(div.className) // "bd"
alert(div.title)     // "Body text"
alert(div.lang)      // "en"
alert(div.dir)       // "ltr" 
```
所有 HTML 元素都是由 HTMLElement 或者其更具体的子类型来表示的，每一种类型都有与之相关的特性和方法，
比如 A 元素和 IMG 元素它们的特性和对应的方法是不完全相同的。
### 2.取得特性
每个元素都有一或多个特性，这些特性的用途是给出相应元素或其内容的附加信息。操作特性的 DOM 方法主要有三个，
分别是 getAttribute()、setAttribute() 和 removeAttribute()，
这三个方法可以针对任何特性使用。


`getAttribute()`：由于 `div.getAttribute('id')` 和 `div.id` 的效果是一样的，所以一般不使用，直接使用后者来访问元素节点的特性（属性）即可


关于自定义特性（一般用 data- 表示）可以通过 dataset 属性来访问（当然用 getAttribute 方法也行），比如
```html
<img id="test" src="" class="image-item" 
     lazyload="true" 
     data-original="https://xxx.webp" 
     alt="" />


// document.getElementById('test').dataset.original 可以访问到 data-original 的值
```
参考：[https://blog.csdn.net/qq\_39579242/article/details/81779170](https://blog.csdn.net/qq_39579242/article/details/81779170)

### 3.设置和移除特性
`setAttribute()`：这个方法接受两个参数：要设置的特性名和值。如果特性已经存在，setAttribute() 会以指定的值替换现有的值；
如果特性不存在，setAttribute() 则创建该属性并设置相应的值。
用法如下：
```js
div.setAttribute("id", "someOtherId")
div.setAttribute("class", "ft")
div.setAttribute("title", "Some other text")
```
> 通过这个方法设置的特性名会被统一转换为小写形式，即"ID"最终会变成"id"。

`removeAttribute()`：这个方法用于彻底删除元素的特性。调用这个方法不仅会清除特性的值，而且也会从元素中完全删除特性
```
div.removeAttribute("class");
```
### 4.attributes 属性
Element 类型是使用 attributes 属性的唯一一个 DOM 节点类型。attributes 属性中包含一个 NamedNodeMap，与 NodeList 类似。
虽然我们可以用它做以上三个方法的操作，但是不是很方便（额外的操作 API），一般在需要遍历元素的特性时可以使用这个属性


以下代码展示了如何迭代元素的每一个特性，然后将它们构造成 name="value" 这样的字符串格式
```js
function outputAttributes (element){
 var pairs = new Array(),
 attrName,
 attrValue,
 i,
 len;
 for (i = 0, len = element.attributes.length; i < len; i++) {
   attrName = element.attributes[i].nodeName;
   attrValue = element.attributes[i].nodeValue;
   pairs.push(attrName + "=\"" + attrValue + "\"");
 }
 return pairs.join(" ");
} 
```
### 5.classList 属性（DOM 扩展）
```html
<div class="bd user disabled">...</div>
```
这个`<div>`元素一共有三个类名。要从中删除一个类名，需要把这三个类名拆开，删除不想要的那个，然后再把其他类名拼成一个新字符串。请看下面的例子。
```js
// 删除"user"类
// 首先，取得类名字符串并拆分成数组
var classNames = div.className.split(/\s+/)
// 找到要删的类名
var pos = -1,
    i,
    len

for (i = 0, len = classNames.length; i < len; i++) {
 if (classNames[i] == "user") {
   pos = i
   break
 }
}
// 删除类名
classNames.splice(i,1)
// 把剩下的类名拼成字符串并重新设置
div.className = classNames.join(" "); 
```

HTML5 新增了一种操作类名的方式，可以让操作更简单也更安全，那就是为所有元素添加 classList 属性。这个 classList 属性是新集合类型 DOMTokenList 的实例。与其他 DOM 集合类似 DOMTokenList 有一个表示自己包含多少元素的 length 属性，而要取得每个元素可以使用 item() 方法，也可以使用方括号语法。此外，这个新类型还定义如下方法。
- add(value)：将给定的字符串值添加到列表中。如果值已经存在，就不添加了。
- contains(value)：表示列表中是否存在给定的值，如果存在则返回 true，否则返回 false。
- remove(value)：从列表中删除给定的字符串。
- toggle(value)：如果列表中已经存在给定的值，删除它；如果列表中没有给定的值，添加它。

这样，前面那么多行代码用下面这一行代码就可以代替了
```js
div.classList.remove("user")
```
```js
// 删除 "disabled" 类
div.classList.remove("disabled")

// 添加 "current" 类
div.classList.add("current")

// 切换 "user" 类
div.classList.toggle("user")

// 确定元素中是否包含既定的类名
if (div.classList.contains("bd") && !div.classList.contains("disabled")) {
  // do something...
)

// 迭代类名
for (var i = 0, len = div.classList.length; i < len; i++) {
 doSomething(div.classList[i])
}
```
有了 classList 属性，除非你需要全部删除所有类名，或者完全重写元素的 class 属性，否则也就用不到 className 属性了
# 节点关系
<img src="https://box.kancloud.cn/e3e6da51171ba916d958366256b7185b_661x335.png" />


注意关系指针都是只读的，见下表


|  关系指针   |  描述   |
| :---- | :---- |
|   childNodes  |  每个节点都有一个 childNodes 属性，其中保存着一个 NodeList 对象，NodeList 是一种类数组对象，用于保存一组有序的节点，可以通过下标来访问这些节点。   |
|   parentNode  |  指向该节点在文档树中的父节点   |
|   previousSibling  |  上一个兄弟，没有则为 null   |
|   nextSibling  |  下一个兄弟，没有则为 null   |
|    firstChild |   第一个子节点  |
|  lastChild   |   最后一个子节点  |
|  ownerDocument|   该属性指向表示整个文档的文档节点  |

contains() 方法用于判断某个节点是否为另一个节点的后代，调用 contains 方法的应该是祖先节点，也就是搜索开始的节点，
这个方法接收一个参数，即需要检测的节点。
```js
console.log(document.documentElement.contains(document.body))  // true
```

这个例子检测了\<body\>元素是不是\<html\>元素的后代
<br>

需要注意的是，用上面的方式访问节点返回的不一定是元素节点（Element 类型），而我们往往需要操作的只是元素节点，所以就有了以下的 DOM 扩展
Element Traversal API 为 DOM 元素添加了以下 5 个属性：
- childElementCount：返回子元素（不包括文本节点和注释）的个数。
- firstElementChild：指向第一个子元素；firstChild 的元素版。
- lastElementChild：指向最后一个子元素；lastChild 的元素版。
- previousElementSibling：指向前一个同辈元素；previousSibling 的元素版。
- nextElementSibling：指向后一个同辈元素；nextSibling 的元素版。


支持的浏览器为 DOM 元素添加了这些属性，利用这些元素不必担心空白文本节点，从而可以更方便地查找 DOM 元素了。
下面来看一个例子。过去，要跨浏览器遍历某元素的所有子元素，需要像下面这样写代码。
```js
var i,
    len,
    child = element.firstChild;

while (child != element.lastChild) {
 if (child.nodeType === 1) { // 检查是不是元素
   processChild(child)
 }
 child = child.nextSibling
}
```
而使用 Element Traversal 新增的元素，代码会更简洁：
```js
var i,
    len,
    child = element.firstElementChild

while (child !== element.lastElementChild) {
 processChild(child) // 已知其是元素
 child = child.nextElementSibling
} 
```
# 节点操作
## 创建节点
1.document.createElement(tagName)


创建一个元素节点，tagName 为HTML标签的名称（不区分大小写，如'div'），并将返回一个节点对象。


2.document.createTextNode(text)


创建一个文本节点，text 为文本节点的内容，并将返回一个节点对象。


3.document.createDocumentFragment()


创建一个文档片段文档片段用于保存将来可能添加到文档中的节点，
可以通过 appendChild() 或insertBefore() 方法将文档片段中的内容添加到文档中。
在将文档片段作为参数传给这两个方法时，实际上只有文档片段的所有子节点会被添加到文档树中，**文档片段本身永远不会成为文档树中的一部分**。
``` js
var fragment = document.createDocumentFragment()
var ul = document.getElementById("myList")
var li = null
for (var i = 0; i < 3; i++){
  li = document.createElement("li")
  li.appendChild(document.createTextNode("Item " + (i+1)))
  fragment.appendChild(li)
}
ul.appendChild(fragment)
```
这样做的好处是：如果逐个地添加列表项，将会导致浏览器的反复渲染（呈现）新信息，使用文档片段可以一次性地将多个节点添加到文档树。

## 获取元素节点
带 s 的就是返回一个 NodeList......


1.document.getElementById()：根据 id 获取节点。


2.document.getElementsByTagName()：返回带有指定标签名的对象的集合。


3.document.getElementsByClassName()：返回文档中所有指定类名的元素的集合，作为 NodeList 对象。


4.document.getElementsByName()：查询的是元素的 name 属性。因为一个文档中的 name 属性可能不唯一（如 HTML 表单中的单选按钮通常具有相同的 name 属性），
所以 getElementsByName() 方法返回的是元素的 NodeList，而不是一个元素。


5.document.querySelector()：该方法接受一个 CSS 选择符，返回与该模式匹配的第一个元素，如果没有找到匹配的元素则返回 null。
选择符可以为 id、类、 类型、 属性、 属性值等；对于多个选择器，使用逗号隔开，返回一个匹配的元素。


``` js
// 取得 body 元素
var body = document.querySelector("body")

// 取得 ID 为 "myDiv" 的元素
var myDiv = document.querySelector("#myDiv")

// 取得类为 "selected" 的第一个元素
var selected = document.querySelector(".selected")

// 取得类为 "button" 的第一个图像元素
var img = document.body.querySelector("img.button")

// 获取文档中 class="example" 的第一个 <p> 元素:
document.querySelector("p.example")

// 获取文档中有 "target" 属性的第一个 <a> 元素
document.querySelector("a[target]")
```
注意：通过 Document 类型调用 querySelector()方法时，会在文档元素的范围内查找匹配的元素。
而通过 Element 类型调用 querySelector() 方法时，只会在该元素后代元素的范围内查找匹配的元素。
CSS 选择符可以简单也可以复杂，视情况而定。如果传入了不被支持的选择符，querySelector() 会抛出错误。

6.document.querySelectorAll()：querySelectorAll() 方法接收的参数与 querySelector() 方法一样，都是一个 CSS 选择符，
但返回的是所有匹配的元素而不仅仅是一个元素。**这个方法返回的是一个 NodeList 的实例**，如果没有找到匹配的元素则 NodeList 为空。
``` js
// 取得某 <div> 中的所有 <em> 元素(类似于 getElementsByTagName("em"))
var ems = document.getElementById("myDiv").querySelectorAll("em")

// 取得类为 "selected" 的所有元素
var selecteds = document.querySelectorAll(".selected")

// 取得所有<p>元素中的所有<strong>元素
var strongs = document.querySelectorAll("p strong")
```

## 改变节点关系
|   方法  |  描述   |
| :---- | :---- |
|  appendChild(newNode)	|  向 childNodes 列表末尾添加一个节点，添加节点后，childNodes 的新增节点、父节点以及最后一个节点的关系指针都会相应地得到更新。更新完成后，该方法返回新增的节点	|
|  insertBefore(newNode, 参照 Node)	|  该方法接收两个参数：要插入的节点和作为参照的节点。插入节点后，插入的节点会成为参照节点的 previousSibling，同时被方法返回。	|
|  replaceChild(要插入的节点，要替换的节点)	|  要替换的节点将由这个方法返回并从文档树中被移除，同时由要插入的节点占据其位置	|
|  removeChild(要移除的节点)	|  被移除的节点将成为方法的返回值	|


这四个方法操作的都是某个节点的子节点，也就是说，要使用这几个方法必须先取得父节点（使用 parentNode 属性）。另外，并不是所有类型的节点都有子节点，如果在不支持子节点的节点上调用了这些方法，将会导致错误发生。
