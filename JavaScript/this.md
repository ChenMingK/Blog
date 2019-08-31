参考资料：《你不知道的 JavaScript 上卷》

this 是一个很特别的关键字，被自动定义在所有函数的作用域中。

# this 的作用域
一种常见的误解是，this 指向函数的作用域。这个说法在某种情况下是正确的，但是在其他情况下却是错误的。

需要明确的是，this 在任何情况下都不指向函数的词法作用域。

在 JavaScript 内部，作用域确实和对象类似，可见的标识符都是它的属性。

但是作用域“对象”无法通过 JavaScript 代码访问，它存在于 JavaScript 引擎内部。

思考下面一段代码，它试图（但是没有成功）跨越边界，使用 this 来隐式调用函数的词法作用域：

```js
function foo () {
  var a = 2
  this.bar()
}

function bar () {
  console.log(this.a)
}

foo() 
```

首先，这段代码试图通过 this.bar() 来引用 bar() 函数，这是不可能成功的。调用 bar() 最自然的方法就是省略前面的 this，直接使用词法引用标识符。

此外，这段代码还试图使用 this 联通 foo() 和 bar() 的词法作用域，从而让 bar() 可以访问 foo() 作用域里的变量 a，这是不可能实现的，你不能使用 this 来引用一个词法作用域内部的东西。

# this 到底是什么
this 是在运行时绑定的，并不是在编写时绑定的，它的上下文取决于函数调用时的各种条件。this 的绑定和函数声明的位置没有任何关系，只取决于函数的调用方式。

当一个函数被调用时，会创建一个活动记录（有时也称为执行上下文），这个记录会包含函数在哪里被调用（调用栈）、函数的调用方法、传入的参数等信息。this 就是记录的其中一个属性，会在函数执行的过程中用到。

# 调用位置
在理解 this 的绑定过程之前，首先要理解调用栈（就是为了到达当前执行位置所调用的所有函数）和调用位置。
```js
function baz () {
  // 当前调用栈是：baz
  // 因此，当前调用位置是全局作用域

  console.log('baz')
  bar() // <-- bar 的调用位置
}

function bar () {
  // 当前调用栈是 baz -> bar
  // 因此，当前调用位置在 baz 中

  console.log('bar')
  foo() // <-- foo 的调用位置
}

function foo () {
  // 当前调用栈是 baz -> bar -> foo
  // 因此，当前调用位置在 bar 中
  console.log('foo')
}

baz() // <-- baz 的调用位置
```

# 绑定规则
最后，来看看在函数的执行过程中调用位置如何决定 this 的绑定对象。

你必须找到调用位置，然后判断需要应用下面四条规则中的哪一条，我们首先分别解释这四条规则，然后解释多条规则都可用时它们的优先级如何排列。
## 默认绑定
首先要介绍的是最常用的函数调用类型：独立函数调用。可以把这条规则看作是无法应用其他规则时的默认规则。

思考如下一段代码：
```js
function foo () {
  console.log(this.a)
}
var a = 2
foo () // 2
```
首先要注意的是，声明在全局作用域中的变量（如 var a = 2）就是全局对象的一个同名属性，它们本质上就是同一个东西，并不是通过复制得到的，就像一个硬币的两面一样。

接下来我们可以看到当调用 foo() 时，this.a 被解析成了全局变量 a，因为在本例中，函数调用时应用了 this 的 **默认绑定**，因此 this 指向全局对象。

我们怎么知道这里应用了 **默认绑定** 呢？可以通过分析调用位置来看看 foo() 是如何调用的。在代码中，foo() 是直接使用不带任何修饰的函数引用进行调用的，因此只能使用默认绑定，无法应用其他规则。
如果使用严格模式（strict mode），那么全局对象将无法使用默认绑定，因此 this 会绑定到 undefined

```js
function foo () {
  'use strict'

  console.log(this.a)
}

var a = 2

foo () // TypeError: Cannot read property 'a' of undefined
```

## 隐式绑定
```js
function foo () {
  console.log(this.a)
}

var obj = {
  a: 2,
  foo: foo
}

obj.foo() // 2
```
当函数引用有上下文对象时，隐式绑定规则会把函数调用中的 this 绑定到这个上下文对象。（函数作为对象的属性被调用）

需要注意的是，对象属性引用链中只有最后一层会影响调用位置，举例来说：
```js
function foo () {
  console.log(this.a)
}

var obj2 = {
  a: 42,
  foo: foo
}

var obj1 = {
  a: 2,
  obj2: obj2
}

obj1.obj2.foo() // 42
```

### 隐式丢失
一个最常见的 this 绑定问题就是被隐式绑定的函数会丢失绑定对象，那么它就会应用默认绑定，从而把 this 绑定到全局对象或 undefined 上。
```js
function foo () {
  console.log(this.a)
}

var obj = {
  a: 2,
  foo: foo
}

var bar = obj.foo // 函数别名

var a = 'oops, global' // a 是全局对象的属性

bar() // 'oops, global'
```
虽然 bar 是 obj.foo 的一个引用，但是实际上，它引用的是 foo 函数本身，因此此时的 bar() 其实是一个不带任何修饰的函数调用，因此应用了默认绑定。
一种更常见的情况发生在传入回调函数时：
```js
function foo () {
  console.log(this.a)
}

function doFoo (fn) {
  // fn 其实引用的是 foo
  fn () // <-- 调用位置!
}

var obj = {
  a: 2,
  foo: foo
}

var a = 'oops, global'

doFoo(obj.foo) // 'oops, global'
```
参数传递其实是一种隐式复制，因此我们传入函数时也会被隐式复制，所以结果和上一个例子一样。

如果把函数传入语言内置的函数而不是我们自己声明的函数时会发生什么？结果是一样的，没有区别：
```js
function foo () {
  console.log(this.a)
}

var obj = {
  a: 2,
  foo: foo
}

var a = 'oops, global'

setTimeout(obj.foo, 100) // 'oops, global'
```
JavaScript 环境内置的 setTimeout() 函数的实现和如下的伪代码类似：
```js
function setTimeout (fn, delay) {
  // 等待 delay 毫秒
  fn() // <-- 调用位置
}
```

## 显式绑定
调用一个函数时指定它的 this，所有的函数都具有 call、apply、bind 方法（这和原型有关）。
- call 方法在使用一个指定的 this 和若干个指定的参数值的前提下调用某个函数或方法
- apply 和 call 基本一样，只是参数变为数组了而已
- bind() 方法会创建一个新函数，当这个新函数被调用时，bind() 的第一个参数将作为它运行时的 this，传递参数时，可以在 bind 时传递部分参数，调用时又传递其余参数。一个绑定函数也能使用 new 操作符创建对象：这种行为就像把原函数当成构造器

需要解释的是 bind 创建的新函数作为构造函数时的行为：当 bind 返回的函数作为构造函数的时候，bind 时指定的 this 值会失效，但传入的参数依然生效，举例如下：
```js
var value = 2;

var foo = {
    value: 1
};

function bar(name, age) {
    this.habit = 'shopping';
    console.log(this.value);
    console.log(name);
    console.log(age);
}

bar.prototype.friend = 'kevin';

var bindFoo = bar.bind(foo, 'daisy');

var obj = new bindFoo('18'); // this 并没有绑定为 foo，失效了
// undefined
// daisy
// 18
console.log(obj.habit);
console.log(obj.friend);
// shopping
// kevin
```

## new 绑定
使用 new 来调用函数，或者说发生构造函数调用时，会自动执行以下操作：
1.  创建一个新对象，并将该对象的\_\_proto\_\_属性指向构造函数的 prototype 属性指向的对象（这一过程可以称为 [[ 原型 ]] 连接）
2.  将 this 绑定到这个新对象上
3.  执行构造函数中的代码（为这个新对象添加属性）
4.  返回这个新对象。（一般情况下，构造函数不返回值，但是用户可以选择主动返回对象，来覆盖正常的对象创建步骤）

## 判断 this
可以按照下面的顺序来进行判断（同时也表明了上述四条规则的优先级）：

1.函数是否在 new 中调用（new 绑定）？如果是的话 this 绑定的是新创建的对象

`var bar = new foo()`

2.函数是否通过 call、apply、bind 显式绑定调用？如果是的话，this 绑定的是指定的对象

`var bar = foo.call(obj2)`

3.函数是否在某个上下文对象中调用（隐式绑定）？如果是的话，this 绑定的是那个上下文对象

`var bar = obj.foo()`

4.如果都不是，使用默认绑定。在严格模式下会绑定到 undefined，否则绑定到全局对象。

`var bar = foo()`

# 其他注意事项
## 被忽略的 this
如果你把 null 或者 undefined 作为 this 的绑定对象传入 call、apply 或者 bind，这些值在调用时会被忽略，实际应用的是默认绑定规则：
```js
function foo () {
  console.log(this.a)
}

var a = 2
foo.call(null) // 2 浏览器环境
```
那么什么情况下会传入 null 呢？
一般是做函数绑定时如果不关心 this 的话，可以传入一个 null 作为占位值。
```js
function foo (a, b) {
  console.log(`a: ${a} b: ${b}`)
}

foo.apply(null, [2, 3])

var bar = foo.bind(null, 2)
bar(3)
```
## 箭头函数
箭头函数的`this`只取决于他外面的第一个不是箭头函数的函数的`this`
```js
function foo () {
  setTimeout(() => {
    // 这里的 this 在词法上继承自 foo()
    console.log(this.a)
  }, 100)
}

var obj = {
  a: 2
}

foo.call(obj) // 2
```

## 闭包中的 this
每个函数在被调用时都会自动取得两个特殊变量：this 和 arguments。内部函数在搜索这两个变量时，只会搜索到其活动对象为止，因此永远不可能直接访问外部函数中的这两个变量
（具体见作用域、作用域链、闭包章节）因此，下面的匿名函数是无法取得其包含作用域（或外部作用域）的 this 对象的。
``` js
var name = 'The Window'
var object = {
  name: 'My Object',
  getNameFunc: function () {
    return function () {
      return this.name
    }
  }
}

console.log(object.getNameFunc()()) // 'The Window' 浏览器环境 & 非严格模式 
```
