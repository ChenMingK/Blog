# 使用 attributes 属性遍历元素特性
```js
// 迭代元素的每一个特性，将它们构造成 name = value 的字符串形式
function outputAttributes (element) {
  const pairs = []
  let attrName
  let attrValue

  for (let i = 0, len = element.attributes.length; i < len; i++) {
    attrName = element.attributes[i].nodeName
    attrValue = element.attributes[i].nodeValue
    pairs.push(`${attrName}=${attrValue}`)
  }
  return pairs.join(" ")
}
```
# 使用 classList 属性来操作类名
```html
<div class="bd user disabled">...</div>
```
这个 \<div> 元素一共有三个类名。要从中删除一个类名，需要把这三个类名拆开，删除不想要的那个，然后再把其他类名拼成一个新字符串。请看下面的例子：
```js
// div.className = 'bd user disabled'
let classNames = div.className.split(/\s+/)
let pos = -1
for (let i = 0, len = classNames.length; i < len; i++) {
  if (classNames[i] === 'user') {
    pos = i
    break
  }
}

classNames.splice(pos, 1) // 删除类名
div.className = classNames.join(' ') // 把剩下的类名拼接成字符串并重新设置
```
HTML5 新增了一种操作类名的方式，可以让操作更简单也更安全，那就是为所有元素添加 classList 属性，这个新类型定义了如下方法：
- add(value)：将给定的字符串值添加到列表中。如果值已经存在，就不添加了。
- contains(value)：表示列表中是否存在给定的值，如果存在则返回 true，否则返回 false。
- remove(value)：从列表中删除给定的字符串。
- toggle(value)：如果列表中已经存在给定的值，删除它；如果列表中没有给定的值，添加它。

这样，前面那么多行代码用下面这一行代码就可以代替了
```js
div.classList.remove("user")
```

# 使用 Element Traversal API 操作元素节点
过去，要遍历某元素的所有子元素，需要像下面这样写代码：
```js
let child = element.firstChild
while (child !== element.lastChild) {
  if (child.nodeType === 1) { // 检查是否为元素节点
    processChild(child)
  }
  child = child.nextSibling
}
```
Element Traversal API 为 DOM 元素添加了以下 5 个属性：
- childElementCount：返回子元素（不包括文本节点和注释）的个数。
- firstElementChild：指向第一个子元素；firstChild 的元素版。
- lastElementChild：指向最后一个子元素；lastChild 的元素版。
- previousElementSibling：指向前一个同辈元素；previousSibling 的元素版。
- nextElementSibling：指向后一个同辈元素；nextSibling 的元素版。

使用 Element Traversal 新增的 API，代码会更简洁：
```js
let child = element.firstElementChild
while (child !== element.lastElementChild) {
  processChild(child) // 肯定是元素节点
  child = child.nextElementSibling // 遍历下一个元素节点
}
```

# getElementsByTagName('*') 会返回什么？
最近看到一道面试题：找出页面出现最多的标签，或者说出现次数最多的前 2、3 个标签，可以试着自己实现下。
```js
// 获取元素列表，以键值对的形式存储为一个对象
function getElements () {
  // 如果把特殊字符串 "*" 传递给 getElementsByTagName() 方法
  // 它将返回文档中所有元素的列表，元素排列的顺序就是它们在文档中的顺序。
  // 返回一个 HTMLCollection - 类数组对象
  const nodes = document.getElementsByTagName('*')
  const tagsMap = {}
  for (let i = 0, len = nodes.length; i < len; i++) {
    let tagName = nodes[i].tagName
    if (!tagsMap[tagName]) {
      tagsMap[tagName] = 1
    } else {
      tagsMap[tagName] ++
    }
  }
  return tagsMap
}

// n 为要选取的标签个数 - 即出现次数前 n 的标签名
// 将上面的方法获取的对象的键值对取出组成数组，按出现次数排序
function sortElements (obj, n) {
  const arr = []
  const res = []
  for (let key of Object.keys(obj)) {
    arr.push({ tagName: key, count: obj[key] })
  }

  // 冒泡
  for (let i = arr.length - 1; i > 0; i--) {
    for (let j = 0; j < i; j++) {
      if (arr[j].count < arr[j + 1].count) { // 升序
        swap(arr, j, j + 1)
      }
    }
  }

  for (let i = 0; i < n; i++) {
    res.push(arr[i].tagName)
  }
  return res
}

function swap (arr, index1, index2) {
  let temp = arr[index1]
  arr[index1] = arr[index2]
  arr[index2] = temp
}

let res = sortElements(getElements(), 2)
console.log(res)
```

# 动态添加脚本与样式
```js
// 动态添加脚本
function loadScript (url) {
  var script = document.createElement("script")
  script.type = "text/javascript"
  script.src = url
  document.body.appendChild(script)
} 

// 动态添加样式
function loadStyles (url) {
  var link = document.createElement("link")
  link.rel = "stylesheet"
  link.type = "text/css"
  link.href = url
  var head = document.getElementsByTagName("head")[0]
  head.appendChild(link)
} 
```

# 使用 contains() 方法判断某个节点是否为另一个节点的后代
contains() 方法用于判断某个节点是否为另一个节点的后代，调用 contains() 方法的应该是祖先节点，也就是搜索开始的节点，这个方法接收一个参数，即需要检测的节点。
```js
console.log(document.documentElement.contains(document.body)) // true
```
这个例子检测了 \<body> 元素是不是 \<html> 元素的后代

# Element.getBoundingClientRect() 及 dataset 的使用

![](https://user-gold-cdn.xitu.io/2019/8/1/16c4bc24f27d5e6e?w=300&h=297&f=png&s=9043)

这是 [小册](https://github.com/InterviewMap/CS-Interview-Knowledge-Map) 上的一个例子，使用原生 JS 实现图片懒加载，需要了解这两个知识点

1.`Element.getBoundingClientRect()`方法返回元素的大小及其相对于视口的位置。具体解释及用法参考 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/getBoundingClientRect)


通过 `Element.getBoundingClientRect().top` 与
`window.innerHeight`（当前视窗的高度）比较就可以判断图片是否出现在可视区域。

注意这个 top 是相对于当前视窗的顶部的 top 值而不是一开始的顶部。

2.通过 Element.dataset 可以获取到为元素节点添加的 data-* 属性，我们可以通过这个属性来保存图片加载时的 url。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <style>
        * {
            padding: 0;
            margin: 0;
        } 
        .box-image {
          display: flex;
          flex-direction: column;
          align-items: center;
        }
        img {
            display: inline-block;
            height: 300px;
            margin-bottom: 20px;
        }
    </style>
</head>

<body>
    <div class="box-image">
        <img src="" class="image-item" lazyload="true" data-original="http://cmk1018.cn/wp-content/uploads/2019/04/6.jpg" alt="">
        <img src="" class="image-item" lazyload="true" data-original="http://cmk1018.cn/wp-content/uploads/2019/04/8.jpg" alt="">
        <img src="" class="image-item" lazyload="true" data-original="http://cmk1018.cn/wp-content/uploads/2019/04/aotu-10.jpg" alt="">
        <img src="" class="image-item" lazyload="true" data-original="http://cmk1018.cn/wp-content/uploads/2019/04/aotu-15.jpg" alt="">
        <img src="" class="image-item" lazyload="true" data-original="http://cmk1018.cn/wp-content/uploads/2019/05/%E7%BD%AA%E6%81%B6%E7%8E%8B%E5%86%A04.jpg" alt="">
        <img src="" class="image-item" lazyload="true" data-original="http://cmk1018.cn/wp-content/uploads/2019/05/%E7%BD%AA%E6%81%B6%E7%8E%8B%E5%86%A06.jpg" alt="">
        <img src="" class="image-item" lazyload="true" data-original="http://cmk1018.cn/wp-content/uploads/2019/04/9.jpg" alt="">
        <img src="" class="image-item" lazyload="true" data-original="http://cmk1018.cn/wp-content/uploads/2019/04/aotu-16.jpg" alt="">
    </div>
    <script>
        var viewHeight = document.documentElement.clientHeight;
        // 节流:加一个 300ms 的间隔执行
        function throttle(fn, wait) {
          let canRun = true
          return function (...args) {
            if (!canRun) return
            canRun = false 
            setTimeout(() => {
              fn.apply(this, args)
              canRun = true
            }, wait)
          }
        }
        function lazyload() {
          let imgs = document.querySelectorAll('img[data-original][lazyload]') // 获取文档中所有拥有 data-original lazyload 属性的<img>节点
          imgs.forEach(item => {
            if (item.dataset.original == '') {// HTMLElement.dataset 访问在 DOM 中的元素上设置的所有自定义数据属性(data-*)集。
              return
            }
            // 返回一个 DOMRect 对象，包含了一组用于描述边框的只读属性——left、top、right 和 bottom，
            // 单位为像素。除了 width 和 height 外的属性都是相对于视口的左上角位置而言的。
            let rect = item.getBoundingClientRect()
            // 其 top 值是相对于当前视窗的顶部而言的而不是绝对的顶部，所以 top 值 < window.innerHeight 的话图片就出现在底部了就需要加载
            if (rect.bottom >= 0 && rect.top < viewHeight) {
              let img = new Image()
              img.src = item.dataset.original
              // 图片加载完成触发 load 事件
              img.onload = function () {
                item.src = img.src
              }
              // 移除属性的话就不会重复加载了
              item.removeAttribute('data-original')
              item.removeAttribute('lazyload')
            }
          })
        }
        // 先调用一次加载最初显示在视窗中的图片
        lazyload();
        let throttle_lazyload = throttle(lazyload, 300)
        document.addEventListener('scroll', throttle_lazyload)
    </script>
</body>
</html>
```

# 如何渲染几万条数据且不卡住页面？
这也是小册上的，考察了利用文档碎片 (createDocumentFragment) 分批次插入节点
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>Document</title>
  </head>
  <body>
    <ul>
      控件
    </ul>
    <script>
      const total = 100000 // 10万条数据
      const once = 20      // 每轮插入的数据条目
      const loopCount = total / once // 渲染总次数
      let countOfRender = 0
      let ul = document.querySelector('ul')
      function add() {
        // 使用文档碎片优化性能
        const fragment = document.createDocumentFragment()
        for (let i = 0; i < once; i++) {
          const li = document.createElement('li')
          li.innerText = Math.floor(Math.random() * total)
          fragment.appendChild(li)
        }
        ul.appendChild(fragment)
        countOfRender+=1
        loop()
      }
      function loop() {
        if (countOfRender < loopCount) {
          window.requestAnimationFrame(add) // 使用 requestAnimationFrame 每隔 16ms(浏览器自己选择最佳时间)刷新一次
        }
      }
    </script>
  </body>
</html>
```


