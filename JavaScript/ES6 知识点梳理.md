# let 与 const 的使用            
⒈不存在变量提升 ☛ 变量要在声明之后再使用

⒉暂时性死区 ☛ let 和 const 形成封闭作用域，该作用域（代码块）内使用变量之前都必须先声明

⒊不允许重复声明 ☛ 不允许在相同作用域内重复声明同一变量

⒋块级作用域 ☛ ① 防止内层变量覆盖外层变量 ② 防止用于计数的循环变量泄露为全局变量

⒌const 保证的是变量指向的内存地址不得改动，可以为 const 声明的数组或对象添加元素 / 属性但是不能指向另一个地址

⒍let 和 const 声明的变量不会添加到 window 对象中（var 会），而会添加到一个 Script 作用域

例题：使用 var 声明的变量来控制循环会泄漏为全局变量，每一层循环新的 i 值都会覆盖旧的 i 值；换句话来讲，所有的 i 指向一个全局变量 i

```js
var a = []
for (var i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i)
  }
}
a[6]() // 10
```

使用 let，每一次循环的 i 其实都是一个新的变量；你可能会问，如果每一轮循环的变量 i 都是重新声明的，那它怎么知道上一轮循环的值，从而计算出本轮循环的值？

这是因为 JavaScript 引擎内部会记住上一轮循环的值，初始化本轮的变量 i 时，就在上一轮循环的基础上进行计算。

```js
var a = []
for (let i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i)
  }
}
a[6]() // 6
```
另外，for 循环还有一个特别之处，就是设置循环变量的那部分是一个父作用域，而循环体内部是一个单独的子作用域。
```js
for (let i = 0; i < 3; i++) {
  let i = 'abc'
  console.log(i)
}
// abc
// abc
// abc
```

# 解构赋值
## 1、数组的解构赋值
例子：`let [a, b, c] = [1, 2, 3]`
两边都是方括号，从左往右依次匹配赋值
如果解构不成功，变量的值为 undefined；可以有 “不完全解构”，例如下面这段代码：
```js
let [a, [b], d] = [1, [2, 3], 4]
a // 1
b // 2
d // 4
```

右边不一定必须是数组，只要某种数据结构具有 Iterator 接口，都可以采用数组形式的解构赋值。

变量可以有默认值：`let [foo = true] = [ ]`

## 2、对象的解构赋值
例子：`let {foo, bar} = {foo:"aaa", bar:"bbb"}`

两边都是花括号，与数组的解构赋值的不同：变量的赋值是无序的

两种写法

① 变量与属性同名
```js
let {bar, foo} = {foo:'aaa', bar:'bbb'}
```
② 变量名与属性名不一致，在变量前给出匹配的 “模式”，即 ":" 之前的不是要赋值的变量而是给出了之后的变量应该匹配哪个属性

```js
let obj = {first:'hello', last:'world'}
let {first: f, last: l} = obj
```
可以嵌套赋值，可以指定默认值
```js
let obj = {}
let arr = {}
({foo:obj.prop, bar:arr[0]} = {foo: 123, bar: true}) // 这里外面必须加个圆括号不然报错…
```

## 3、字符串的解构赋值
字符串被转换为一个类似数组的对象
```js
const[a,b,c,d,e] = 'hello'   // h e l l o
```
规则：只要等号右边的值不是对象或数组，就先将其转为对象。

## 4、函数参数的解构赋值
```js
function add([x, y]) {
  return x + y
}
console.log(add([1, 2]))
```

## 用途
1、提取 JSON 数据
```js
let jsonData = {
  id: 42,
  status: 'OK',
  data: [867, 5309]
}
let {id, status, data:number} = jsonData
console.log(id, status, number) // 42 'OK' [867, 5309]
```
2、从函数返回多个值
```js
function example1() {
  return [1, 2, 3]
}
let [a, b, c] = example1()

function example2() {
  return {
    foo: 1,
    bar: 2
  }
}
let {foo, bar} = example2()

console.log(a, b, c) // 1 2 3
console.log(foo, bar) // 1 2
```

3、函数参数的定义
解构赋值可以方便地将一组参数与变量名对应起来
```js
// 参数是一组有序的值
function f([x, y, z]) { ... }
f([1, 2, 3])

// 参数是一组无序的值
funtion f({x, y, z}) { ... }
f({z: 3, y: 2, x: 1})
```

4、输入模块的指定方法
加载模块时，往往需要指定输入的方法。解构赋值使得输入语句非常清晰
```js
const { SourceMapConsumer, SourceNode } = require("source-map")
```
上面的代码中，函数 add 的参数表面上是一个数组，但在传入参数的那一刻，数组参数就被解构成变量 x 和 y 。

# 函数的扩展
## 参数默认值
- ES6 允许函数的参数指定为默认值，参数默认值是惰性求值的，即每次调用函数都会重新计算
- 参数变量的声明是默认的，在函数体中不能用 let 或 const 再次声明
- 函数的 length 属性：返回 **没有指定默认值** 的参数个数
```js
console.log(function (a, b, c = 5) {}.length); // 2
```

>length 属性统计的是函数预期传入的参数个数，指定了默认值的参数以及  rest 参数都不会计入 length 属性

- 作用域问题：一旦设置了参数的默认值，函数进行声明初始化时，参数会形成一个单独的作用域。等到初始化结束，这个作用域就会消失。这种语法在不设置参数默认值时是不会出现的。
- 可以将参数默认值设为 undefined，表示这个参数是可省略的
- 定义了默认值的参数应该是函数的 **尾参数**，否则这个参数实际上是无法省略的

## rest 参数
 形式：（…变量名），用于获取函数的多余参数，这样就不需要 arguments 对象了。rest 参数搭配的变量是一个数组。
>[warning]rest 参数只能是最后一个参数，否则会报错
```js
// arguments 变量的写法
function sortNumbers() {
  return Array.prototype.slice.call(arguments).sort()
}

// rest 参数的写法
function sortNumbers = (...numbers) => numbers.sort()
```

## 箭头函数
ES6 允许使用箭头（=>）定义函数，箭头函数对于使用 function 关键字创建的函数有以下区别
- 箭头函数没有 arguments（建议使用更好的语法，剩余运算符替代）
- 箭头函数没有 prototype 属性，不能用作构造函数（不能用 new 关键字调用）
- 箭头函数没有自己 this，箭头函数的 this 始终等于它上层上下文中的 this
- 不可以使用 yield 命令，因此箭头函数不能用作 Generator 函数

基础语法：
```js
(参数1, 参数2, …, 参数N) => { 函数声明 }

// 相当于：(参数1, 参数2, …, 参数N) => { return 表达式; }
(参数1, 参数2, …, 参数N) => 表达式（单一）

// 当只有一个参数时，圆括号是可选的：
(单一参数) => {函数声明}
单一参数 => {函数声明}

// 没有参数的函数应该写成一对圆括号。
() => {函数声明}

// 加括号的函数体返回对象字面表达式：
参数 => ({foo: bar})

// 支持剩余参数和默认参数
(参数1, 参数2, ...rest) => {函数声明}
(参数1 = 默认值1,参数2, …, 参数N = 默认值N) => {函数声明}
```
```js
let controller = {
  a: 1,
  makeRequest: function () {
    // 这里的 this 使用默认绑定规则，绑定到全局对象上（非严格模式），具体见 this 章节

    setTimeout(function () {
      console.log(this.a) // undefined
    })
  }
}
controller.makeRequest()
// 使用箭头函数解决上面这个问题
let controller2 = {
  a: 1,
  makeRequest: function () {
    setTimeout(() => {
      console.log(this.a) // 1
    })
  }
}
controller2.makeRequest()
```

## 尾调用优化
函数调用自身称为递归，如果尾调用自身就称为尾递归。
递归非常耗费内存，因为需要同时保存成百上千个调用帧，而 ES6 的设计让尾递归只存在一个调用帧。
```js
// 非尾递归的 Fibonacci
function Fibonacci(n) {
  if (n <= 1) { return 1 }
  return Fibonacci(n - 1) + Fibonacci(n - 2)
}

Fibonacci(100) // 堆栈溢出

// 使用尾递归
function Fibonacci2(n, ac1 = 1, ac2 = 1) {
  if (n <= 1) { return ac2 }
  return Fibonacci2(n - 1, ac2, ac1 + ac2)
}
console.log(Fibonacci2(100))
```
尾递归阶乘:
```js
function factorial (n, acc = 1) {
  if (n === 1) return acc
  return factorial(n - 1, acc * n)
}
```

# 对象的扩展
比较重要的是 Object.assign() 方法的使用
`Object.assign(target, source1, source2)`：第一参数是目标对象，后面的参数都是源对象，该方法用于将源对象所有 **可枚举属性** 复制到目标对象
注意以下几点：
- 如果目标对象与源对象有同名属性，或多个源对象有同名属性，则后面的属性会覆盖前面的属性
- 复制的属性是有限制的，只复制源对象的自身属性（不复制继承属性），不复制不可枚举属性（enumerable: false）
- 实行的是浅复制，而不是深复制。即如果源对象某个属性的值是对象，那么目标对象复制得到的是这个对象的引用
- 如果同名属性是对象，Object.assign 的处理方法是替换而不是添加
```js
var target = { a: { b: 'c', d: 'e' } }
var source = { a: { b: 'hello' } }
Object.assign(target, source) // { a: { b: 'hello' } } 而不是 { a: { b: 'hello', d: 'e' } }
```
常见用途：
**为对象添加属性：**
```js
class Point {
  constructor(x, y) {
    Object.assign(this, {x, y}) // 将 x 属性和 y 属性添加到了 Point 类的对象实例中
  }
}
```
**为对象添加方法**
```js
Object.assign(SomeClass.prototype, {
  someMethod(arg1, arg2) {
    ...
  }
})
// 直接将方法添加到 SomeClass.prototype 中
```
**克隆对象**
```js
function clone(origin) {
  return Object.assign({}, origin)
}
// 将原始对象复制到一个空对象；如果想要保持继承链，可以采用以下代码
function clone(origin) {
  let originProto = Object.getPrototypeOf(origin)
  return Object.assign(Object.create(originProto), origin)
}
```
**合并多个对象**
```js
const merge = (target, ...sources) => Object.assign(target, ...sources)
// 如果希望合并后返回一个新对象，可以这么改写
const merge = (...sources) => Object.assign({}, ...sources)
```
**为属性指定默认值**
```js
const DEFAULTS = {
  logLevel: 0,
  outputFormat: 'html'
}

function processContent(options) {
  options = Object.assign({}, DEFAULTS, options)
  console.log(options)
}
// DEFAULT对象是默认值，options对象是用户提供的参数，如果两者有同名属性则options的属性值会覆盖DEFAULTS的属性值
// 这么使用的前提是DEFAULTS对象和options对象的所有属性的值只能是简单类型
```

# 扩展运算符的使用
这里总结下扩展运算符用的比较多的地方
1、将数组转换为用逗号分隔的参数序列
```js
const arr = []
arr.push(...[1, 2, 3, 4, 5])
console.log(arr) // [1, 2, 3, 4, 5]
```
2、某些场合可以替代函数的 apply 方法
```js
// ES5 的写法
Math.max.apply(null, [14, 3, 77])
// ES6 的写法
Math.max(...[14, 3, 77])
```
3、可用于将具有 iterator 接口的类似数组的对象转换为真正的数组
```js
let map = new Map([
  [1, 'one'],
  [2, 'two'],
  [3, 'three']
])

let arr = [...map.keys()] // [1, 2, 3]
```
4、用于对象的解构赋值：扩展运算符此时的作用相当于，将所有可遍历的、但尚未被读取的属性（键值对）分配到指定的对象上面，注意这是浅复制
```js
let { x, y, ...z } = { x: 1, y: 2, a: 3, b: 4 }
x // 1
y // 2
z // { a: 3, b: 4 }
```


# 异步编程
## Promise
Promise 简单来说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。从语法上来说，Promise 是一个对象，从它可以获取异步操作的消息。

Promise 的提出是为了解决传统异步编程的解决方案 - 回调函数和事件中回调地狱的问题。

比如下面这个串行读取文件的例子：
```js
fs.readFile(path1, function (err, data) {
  // read file1
  fs.readFile(path2, function (err, data) {
    // read file2
    fs.readFile(path3, function (err, data) {
      // read file3
      // 更多的回调......
    })
  })
})
```
下面我们用 Promise 来实现相同的效果。

首先需要将 readFile 方法封装为一个 Promise 对象
```js
function readFile_promise (path) {
  return new Promise((resolve, reject) => {
    fs.readFile(path, 'UTF-8', (err, data) => {
      if (data) {
        resolve(data)
      } else {
        reject(err)
      }
    })
  })
}
```
然后链式调用：
```js
// 使用 Promise 的链式调用
readFile_promise('foo.txt').then(value => {
  // ...
  console.log(value)
  return readFile_promise('bar.txt')
}).then(value => {
  // ...
  console.log(value)
  return readFile_promise('baz.txt')
}).then(value => {
  // ...
  console.log(value)
})
```
如果使用 async / await 语法则更直观了：
```js
async function readFile () {
  let result1 = await readFile_promise('foo.txt') // 返回的是该异步操作的结果
  let result2 = await readFile_promise('bar.txt')
  let result3 = await readFile_promise('baz.txt')
}
```
下面简单梳理下 Promise、Generator、async / await 的 API 和使用
1、三种状态：Pending（进行中）、Fulfilled（已成功）、Rejected（已失败）
2、构造函数：Promise()
```js
let promise = new Promise(function (resolve, reject) {
  // ... some code
  if (/* 异步操作成功 */) {
    resolve(value)
  } else {
    reject (error)
  }
})
```
Promise 构造函数接受一个函数作为参数，该函数的两个参数分别是 resolve 和 reject，其是两个函数，由 JavaScript 引擎提供，不需要自己部署
- resolve 函数将 Promise 对象的状态从 Pending 变为 Fulfilled （完成）
- reject 函数则将 Pending 变为 Rejected 状态

可以用 then 方法指定 Resolved 状态（Fulfilled）和 Rejected 状态的回调函数，通过参数接收 resolve 函数和 reject 函数传出的值

简单来说，是  Promise 的状态变为 Resolved 时会触发 then 方法绑定的回调函数（相当于事件监听）

3、`Promise.prototype.then()`

Promise 的实例的 then 方法是定义在原型对象 Promise.prototype 上的，then 方法的第一个参数是 Resolved 状态的回调函数，第二个参数（可选）是 Rejected 状态的回调函数，then 方法返回的是一个新的 Promise 实例，因此可以采用链式写法，如下
```js
getJSON('/posts.json').then(function(json) {
  return json.post
}).then(function(post) {
  // ...
})
```
前一个回调函数的返回结果会作为参数传递给下一个回调函数


4、`Promise.prototype.catch()`

其实这个方法是 .then(null, rejection) 的别名，使用这个方法可以更简洁地指定发生错误时的回调函数。

then 方法指定的回调函数如果在运行中抛出错误，也会被 catch 方法捕获
```js
let promise = new Promise((resolve, reject) => {
  throw new Error('test')
})
promise.catch(error => {
  console.log(error) // Error: test
})
```
如果 Promise 状态已经变成 Resolved，再抛出错误是无效的
```js
let promise = new Promise((resolve, reject) => {
  resolve('ok')
  throw new Error('test')
})

promise.then(value => {
  console.log(value) // ok
}).catch(error => {
  console.log(error)
})
```
Promise 对象的错误具有 “冒泡” 性质，会一直向后传递，直到被捕获为止。也就是说，错误总是会被下一个 catch 语句捕获
```js
getJSON('/post/1.json').then(post => {
  return getJSON(post.commentURL)
}).then(comments => {
  // some code
}).catch(error => {
  // 处理前面 3 个 Promise 产生的错误
})
```


5、`Promise.all()`

该方法接收一个数组作为参数，数组元素都为 Promise 对象（或者是具有 Iterator 接口，且返回的每个成员都是 Promise 实例的对象）
```js
var p = Promise.all([p1, p2, p3]
```
p 的状态由 p1 p2 p3 决定，有两种可能：
① 只有 p1 p2 p3 都变为 Fulfilled，p 才会变为 Fulfilled，此时p1 p2 p3 的返回值组成一个数组，传递给 p 的回调函数
② 只要 p1 p2 p3 中有一个变为 Rejected，p 就变为 Rejected，此时第一个被 Rejected 的实例的返回值会传递给 p 的回调函数


6、`Promise.race()`

与 all 类似，接受一个 Promise 对象组成的数组作为参数，只要其中有一个 Promise 对象率先变为 Resolved 状态那么 p 的状态就跟着改变，率先改变的 Promise 实例的返回值会被传递给 p 的回调函数

7、`Promise.resolve()`

将一个现有的对象转为 Promise 对象

8、`finally()`

不管状态如何改变最后都会执行 finally 指定的回调方法
## Generator
特征：function 命令与函数名之间有一个星号，函数体内部使用 yield 语句定义不同的内状态。

Generator 函数实际上是利用了遍历器对象（Iterator Object），当执行一个 Generator 函数时，会有一个指向内部状态的指针对象，调用遍历器对象的 next 方法会使指针移动到下一个状态，next（）方法返回一个对象，value 属性是当前 yield 语句的值， done 属性是一个布尔值表示遍历是否结束。

co 模块：co 模块用于 Generator 函数的自动执行
```js
var gen = function* () {
  var f1 = yield readFile('...')
  var f2 = yield readFile('...')
}
var co = require('co')
co(gen)
```
## async await
async await 其实就相当于 Generator 函数的语法糖

其相比于 Generator 的改进如下：

① 内置执行器，Generator 函数的执行必须依靠执行器，async 函数自带执行器，与普通函数一样调用即可

② 更好的语义，async 表示函数里有异步操作，await 表示紧跟在后面的表达式需要等待

③ 更广的适用性，await 命令后面可以是 Promise 对象和原始类型的值（如果不是 Promise 对象会被转换为一个 Promise 对象并立即 resolve）

④ 返回值是 Promise，async 函数返回一个 Promise 对象，可以用 then 方法指定下一步操作

# Proxy 与 Reflect
`var proxy = new Proxy(target, handler)`：生成一个 Proxy 实例，target 参数表示所要拦截的目标对象，handler 参数也是一个对象，用来定值拦截行为；
对于可以设置但没有设置拦截的操作，则直接落在目标对象上，按照原先的方式产生结果
 Proxy 支持的拦截操作：参数 propKey 即要操作的对象的属性名
- get(target, propKey, receiver)：拦截对象属性的读取
- set(target, propKey, value, receiver)：拦截对象属性的设置
- has(target, propKey)：拦截 propKey in proxy的操作，返回一个布尔值
- deleteProperty(target, propKey)：拦截 delete proxy[propKey] 的操作，返回一个布尔值
.......

感觉不好整理，用的也不多，就记一下与 defineProperty 的区别：当使用 defineProperty，我们修改原来的 obj 对象就可以触发拦截，而使用 proxy，就必须修改代理对象，即 Proxy 的实例才可以触发拦截。
[https://github.com/mqyqingfeng/Blog/issues/107](https://github.com/mqyqingfeng/Blog/issues/107)
```
// 利用 Proxy 拦截构造函数的执行方法来实现单例模式
function proxy(func) {
  let instance
  let handler = { // Proxy 构造函数的第二个参数是一个对象，定制拦截的行为
    construct(target, args) {
      if (!instance) {
        instance = Reflect.construct(func, args)
      }
      return instance
    }
  }
  return new Proxy(func, handler) // 拦截构造函数
}
```

## Reflect
Reflect 对象的设计目的：
- 将 Object 对象的一些明显属于语言内部的方法放到 Reflect 对象上，如Object.defineProperty
- 修改某些 Object 方法的返回结果，让其变得更为合理。比如 Object.defineProperty(obj, name, desc) 在无法定义属性时会抛出一个错误，而 Reflect.defineProperty(obj, name, desc) 则会返回 false
- 让 Object 操作都变成函数行为。如 name in obj 和 delete obj[name] 改为 Reflect.has(obj, name) 和 Reflect.deleteProperty(obj, name)
- Reflect 对象的方法与 Proxy 对象的方法一一对应，只要是 Proxy 对象的方法，就能在 Reflect 对象上找到对应的方法

```js
var obj = new Proxy({}, {
  get: function (target, key, receiver) {
    console.log(`getting ${key}`)
    return Reflect.get(target, key, receiver)
  },
  set: function (target, key, value, receiver) {
    console.log(`setting ${key}`)
    return Reflect.set(target, key, value, receiver)
  }
})

obj.count = 1 // setting count
console.log(++obj.count)
// getting count
// setting count
// 2
```

# iterator 和 for...of 循环
## 迭代器
所谓迭代器，其实就是一个具有 next() 方法的对象，每次调用 next() 都会返回一个结果对象，该结果对象有两个属性，value 表示当前的值，done 表示遍历是否结束。

我们直接用 ES5 的语法创建一个迭代器：
```js
function createIterator(items) {
    var i = 0;
    return {
        next: function () {
            var done = i >= item.length;
            var value = !done ? items[i++] : undefined;

            return {
                done: done,
                value: value
            };
        }
    };
}

// iterator 就是一个迭代器对象
var iterator = createIterator([1, 2, 3]);

console.log(iterator.next()); // { done: false, value: 1 }
console.log(iterator.next()); // { done: false, value: 2 }
console.log(iterator.next()); // { done: false, value: 3 }
console.log(iterator.next()); // { done: true, value: undefined }
```
## for of
除了迭代器之外，我们还需要一个可以遍历迭代器对象的方式，ES6 提供了 for of 语句，我们直接用 for of 遍历一下我们上节生成的遍历器对象试试：
```js
var iterator = createIterator([1, 2, 3]);

for (let value of iterator) {
    console.log(value);
}
```
结果报错 `TypeError: iterator is not iterable`，表明我们生成的 iterator 对象并不是 iterable(可遍历的)，那什么才是可遍历的呢？

其实一种数据结构只要部署了 Iterator 接口，我们就称这种数据结构是“可遍历的”（iterable）。

ES6 规定，默认的 Iterator 接口部署在数据结构的`Symbol.iterator`属性，或者说，一个数据结构只要具有`Symbol.iterator`属性，就可以认为是 "可遍历的"（iterable）。

举个例子：
```js
const obj = {
    value: 1
};

for (value of obj) {
    console.log(value);
}

// TypeError: iterator is not iterable
```
我们直接 for of 遍历一个对象，会报错，然而如果我们给该对象添加`Symbol.iterator`属性：
```js
const obj = {
    value: 1
};

obj[Symbol.iterator] = function () {
    return createIterator([1, 2, 3]);
};

for (value of obj) {
    console.log(value);
}

// 1
// 2
// 3
```
由此，我们也可以发现 for of 遍历的其实是对象的 Symbol.iterator 属性

## 默认有 iterator 接口的对象
* Array
* Map
* Set
* String
* TypedArray（类数组对象），如 arguments 对象，NodeList 对象
* Generator 对象


## for...of 与 for...in
- 对于数组的遍历,for ... in 会返回数组中所有可枚举的属性(包括原型链上可枚举的属性), for ... of 只返回数组的下标对应的属性值
- for...in 循环出的是 key，for...of 循环出的是 value，且 for...in 会遍历对象的整个原型链，for...of 只遍历当前对象
- for...of 不能循环普通的对象，需要通过和 Object.keys() 搭配使用，需要搭配具有 iterator 接口的对象使用，准确地说，for of 能遍历数组（的值）是因为数组默认有 iterator 接口
```js
let myArray = [3, 5, 7, 9]
myArray.name = '数组'
// for in for of 应用于数组中的区别
for(let index in myArray) {
  console.log(index) // 0 1 2 3 name
}

for(let value of myArray) {
  console.log(value) // 3 5 7 9
}
// 应用于对象中

let myObject = {
  property1: 'P1',
  property2: 'P2',
  property3: 'P3'
}

for(let value in myObject) {
  console.log(value) // property1 property2 property3
}

for(let value of myObject) {
  console.log(value) // myObject is not iterable
}

// How to make myObejct iterable? kind of complex
// Or we can use Object.keys()
for(let key of Object.keys(myObject)) {
  // 使用Object.keys()方法获取对象的key组成的数组
  console.log(key) // property1 property2 property3
}

function A() {
  this.property1 = 1
  this.property2 = 2
  this.property3 = 3
}
A.prototype.speak = function() {
  console.log('speak')
}
let B = new A()
for(let key in B) {
  console.log(key) // property1 property2 property3 speak 
}
```
