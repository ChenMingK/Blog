# 实现 call、apply、bind
## 实现 call
首先要明白 call 是干什么的：call 方法在使用一个指定的 this 和若干个指定的参数值的前提下调用某个函数或方法

① 如何改变 this 指向？
- 将函数设置为对象的属性
- 执行该函数
- 删除该函数

delete 运算符可以删除对象的属性，但是不能删除一个变量或者函数

② 如何把参数传给要执行的函数并使其执行，且传入的参数个数是不确定的？
- 运用 rest 运算符，或者使用 arguments 对象

``` js
Function.prototype.call2 = function (context, ...args) {
  // 首先要获取调用 call 的函数，用 this 可以获取
  context.fn = this
  context.fn(...args)
  delete context.fn // delete 删除对象的属性
}

let foo = {
  value: 1
}

function bar(name, age) {
  console.log(name)
  console.log(age)
  console.log(this.value)
}

bar.call2(foo, 'kevin', 18) // kevin 18 1
```
## 实现 apply
和 call 基本一样，只是参数变为数组了而已
``` js
Function.prototype.apply2 = function (context, arr) {
  // 首先要获取调用 call 的函数，用 this 可以获取
  context.fn = this
  context.fn(...arr)
  delete context.fn // delete 删除对象的属性
}

let foo = {
  value: 1
}

function bar(name, age) {
  console.log(name)
  console.log(age)
  console.log(this.value)
}

bar.apply2(foo, ['kevin', 18]) // kevin 18 1
```
## 实现 bind
首先也是搞明白 bind 的作用是什么：bind() 方法会创建一个新函数，当这个新函数被调用时，bind() 的第一个参数将作为它运行时的 this，传递参数时，
可以在 bind 时传递部分参数，调用时又传递其余参数


完整的 bind 还考虑了：一个绑定函数也能使用 new 操作符创建对象：这种行为就像把原函数当成构造器。
提供的 this 值被忽略，同时调用时的参数被提供给模拟函数。


下面的代码只实现 bind 的部分功能：
``` js
Function.prototype.myBind = function (context, ...args1) {
  if (typeof this !== 'function') { // 只允许函数调用 bind 方法
    throw new Error('Error')
  }
  let self = this
  let firstArgs = args1
  
  return function (...args2) {
    return self.apply(context, firstArgs.concat(args2))
  }
}

```
更完整的 bind 实现参考：[https://github.com/mqyqingfeng/Blog/issues/12](https://github.com/mqyqingfeng/Blog/issues/12)
# Promise 封装原生 AJAX
``` js
let AJAX = function (url) {
  let promise = new Promise(function (resolve, reject) {
    let XHR = new XMLHttpRequest()
    XHR.open('get', url, false) // false：异步 true：同步
    XHR.onreadystatechange = handler
    XHR.send()

    function handler() {
      if (XHR.readyState !== 4) {
        return 
      }
      if (XHR.status === 200) {
        resolve(XHR.response)
      } else {
        reject(new Error(XHR.statusText))
      }
    }
  })
  return promise
}
```
XMLHttpRequest 的一些属性和方法可参考：[AJAX 整理](https://www.kancloud.cn/chenmk/web-knowledges/1106858)

# 函数节流、防抖 + immediate版
参考 [https://github.com/mqyqingfeng/Blog/issues/22](https://github.com/mqyqingfeng/Blog/issues/22)

防抖和节流的作用都是防止函数多次调用，区别在于，假设一个用户一直触发这个函数，且每次触发函数的时间间隔小于 wait，
防抖的情况下只会调用一次，而节流的情况会每隔一定时间调用函数
``` js
// 防抖
function debounce(fn, wait) {
  let timer = 0
  return function (...args) {
    if (timer) clearTimeout(timer) // 清除并重新设置定时任务
    timer = setTimeout(() => {
      fn.apply(this, args)
    }, wait)
  }
}
```
但是需要注意的是，因为需要防抖一般是浏览器事件，那么我们就需要确保使用 debounce 处理过的事件处理程序不能出现一些意外的错误，比如 this 指向和 event 对象
```js
// 解决 this 和 event 不正确的问题
function debounce(func, wait) {
    var timeout;

    return function () {
        var context = this;
        var args = arguments;

        clearTimeout(timeout)
        timeout = setTimeout(function(){
            func.apply(context, args)
        }, wait);
    }
}
```
接下来再考虑一个立即执行的需求：我不希望非要等到事件停止触发后才执行，我希望立刻执行函数，然后等到停止触发 n 秒后，才可以重新触发执行
```js
// 加上 immediate 参数表示是否需要立即执行
function debounce(func, wait, immediate) {

    var timeout;

    return function () {
        var context = this;
        var args = arguments;

        if (timeout) clearTimeout(timeout);
        if (immediate) {
            // 如果已经执行过，不再执行
            var callNow = !timeout;
            timeout = setTimeout(function(){
                timeout = null; // 考虑到不再触发之后的情况，如果不置 timeout 为null 就无法再实现下次的立即执行!
            }, wait)
            if (callNow) func.apply(context, args)
        }
        else {
            timeout = setTimeout(function(){
                func.apply(context, args)
            }, wait);
        }
    }
}
```
节流的简单实现，对于一些特殊的需求可以去看冴羽大大的博客。
``` js
// 节流，不会清除定时任务，而是延缓其执行
function throttle(fn, interval) {
  let canRun = true
  return function (...args) {
    if (!canRun) return // 如果 interval 时间内再次触发会在此处阻拦
    canRun = false
    setTimeout(() => {
      canRun = true
      fn.apply(this, args) // this 指向这个节流函数的调用者
    }, interval)
  }
}
```
# 函数柯里化
在计算机科学中，柯里化（Currying）是把接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数，并且返回接受余下的参数且返回结果的新函数的技术。
**函数柯里化的主要作用和特点就是参数复用、提前返回和延迟执行。**


简单来说，思路就是参数不够的话就存储参数并返回一个函数准备接收新的参数，参数够了的话就执行。
``` js
function curry(fn, args1 = []) { // 默认值设为[]，防止访问args1.length报错
  return fn.length === args1.length ? fn.apply(null, args1) : function (...args2) {
    return curry(fn, args1.concat(args2))
  }
}

// fn(a,b,c,d)=>fn(a)(b)(c)(d)
// fn(a,b,c,d)=>fn(a, b)(c)(d)
function add(a, b, c, d) {
  console.log(a + b + c + d)
}

let curryAdd = curry(add)
curryAdd(1, 2)(3)(4) // 10
curryAdd(1)(2)(3)(4)
curryAdd(1, 2, 3, 4)
```
另一种常见的问题
```js
/* 
    实现一个add方法，使计算结果能够满足如下预期：
    add(1)(2)(3) = 6;
    add(1, 2, 3)(4) = 10;
    add(1)(2)(3)(4)(5) = 15; 
*/

function add() {
    // 第一次执行时，定义一个数组专门用来存储所有的参数
    var _args = Array.prototype.slice.call(arguments);

    // 在内部声明一个函数，利用闭包的特性保存 _args 并收集所有的参数值
    var _adder = function() {
        _args.push(...arguments);
        return _adder;
    };

    // 利用 toString 隐式转换的特性，当最后执行时隐式转换，并计算最终的值返回
    _adder.toString = function () {
        return _args.reduce(function (a, b) {
            return a + b;
        });
    }
    return _adder;
}

add(1)(2)(3)                // 6
add(1, 2, 3)(4)             // 10
add(1)(2)(3)(4)(5)          // 15
add(2, 6)(1)                // 9
```
# 深浅拷贝
## 深拷贝
`JSON.parse(JSON.stringnify(object))`的不足：
*   会忽略 undefined
*   会忽略 symbol
*   不能序列化函数
*   不能解决循环引用的对象

自己实现一个深拷贝
``` js
function deepCopy(obj) {
  let newObj
  if (typeof obj === 'object') { // 数组或对象 typeof 会返回'object'
    newObj = Array.isArray(obj) ? [] : {}
    for(let key in obj) {
      if (obj.hasOwnProperty(key)) {
        // 如果对象的属性仍然为对象/数组则递归
        newObj[key] = typeof obj[key] === 'object' ? deepCopy(obj[key]) : obj[key]
      }
    }
  } else {
    newObj = obj // 简单数据类型直接复制
  }
  return newObj
}
```
测试一下
``` js
let a = {
  age: undefined,
  sex: Symbol('male'),
  jobs: function () {},
  name: 'cmk'
}

let b = JSON.parse(JSON.stringify(a))
console.log(b) // { name: 'cmk' }
let c = deepCopy(a)
console.log(c) // { age: undefined, sex: Symbol(male), jobs: [Function: jobs], name: 'cmk' }
```
# 字符串相关
## 1.字符串转驼峰
``` js
// 字符串转驼峰，注意第一个字符不大写，如border-bottom-color -> borderBottomColor
function Change(str) {
  let arr = str.split('-') // 字符串分割为数组
  // map返回由新的项组成的数组
  arr = arr.map((item, index) => {
    return index === 0 ? item : item.charAt(0).toUpperCase() + item.substring(1) // 第一个字符转大写与之后的字符合并
  })
  return arr.join("") // 数组合并为字符串
}
console.log(Change('border-bottom-color'))
```
## 2.查找字符串中出现次数最多的字符
用 map 试一下,map['a'] 存储字符 a 出现的次数
``` js
// sdddrtkjsfkkkasjdddj 出现的最多的字符是 d
function mostChar(str) {
  let map = new Map()
  let max = 0
  let maxChar
  for (let i = 0; i < str.length; i++) {
    let item = str[i]
    // console.log(item)
    if (!map.has(item)) {
      map.set(item, 0)
    } else {
      map.set(item, map.get(item) + 1) // 这里的 +1 不能直接操作......
      if (map.get(item) > max) {
        max = map.get(item)
        maxChar = item
      }
    }
  }
  return maxChar
}
```
用对象的属性来存储试一下，maxNumber  存储最大次数，maxChar存储对应的字符
``` js
// 查找字符串中出现次数最多的字符的次数
// sdddrtkjsfkkkasjdddj 出现的最多的字符是 d
function findMostChars(str) {
  let maxNumber = 0, maxChar
  let obj = {}
  for (let i = 0; i < str.length; i++) {
    const char = str[i]
    // console.log(obj[char]) if not exist -> undefined
    if (!obj[char]) {
      obj[char] = 1
    } else {
      obj[char]++
      if (obj[char] > maxNumber) {
        maxChar = char
        maxNumber = obj[char]
      }
    }
  }
  return maxChar
}
```
# 数组相关
## 1.数组扁平化 5种方法
``` js
const arr = [1, [2, 3, [4, 5]]]
function flattern(arr) {
  return arr.reduce((preValue, currValue, currIndex, array) => {
    return preValue.concat(Array.isArray(currValue) ? flattern(currValue) : currValue)
  }, []) // []作为第一个preValue
}

// toString & split
function flattern2(arr) {
  return arr.toString().split(',').map(item => {
    return Number(item) // split分隔后数组元素为字符串形式, 需要转换为数值形式
  })
}
PS：该方法及下一种方法都有一个问题，即[1, '1', 2, '2']的结果是[1, 1, 2, 2]，我们改变了原数据（类型），这是不应该的
// join & split
function flattern3(arr) {
  return arr.join(',').split(',').map(item => { // join方法和toString方法效果一样?
    return parseInt(item)
  })
}

// concat递归
function flattern4(arr) {
  let res = []
  arr.map(item => {
    if (Array.isArray(item)) {
      res = res.concat(flattern(item))
    } else {
      res.push(item)
    }
  })
  return res
}

// 扩展运算符
function flattern5(arr) {
  while (arr.some(item => Array.isArray(item))) { // 如果数组元素中有数组
    arr = [].concat(...arr) // [].concat(...[1, 2, 3, [4, 5]]) 扩展运算符可以将二维数组变为一维的
  }
  return arr
}
```
如果只是将二维数组扁平化为一维数组的话也可以这样写
``` javaScript
function flattern(arr) {
  return Array.prototype.concat.apply([], arr)
}
```

## 2.数组乱序
思路：从最后一个数开始，每次将当前遍历位置的数与其之前的随即一个数交换（random index实现），然后将这个位置视为已排序的，依次往前重复执行该过程。
注意 Math.random() 的范围是 [0, 1)

``` js
let x = [1, 2, 3, 4, 5];
function shuffle(arr) {
  var length = arr.length,
    randomIndex,
    temp;
  while (length) {
    randomIndex = Math.floor(Math.random() * (length--));
    temp = arr[randomIndex];
    arr[randomIndex] = arr[length];
    arr[length] = temp
  }
  return arr;
}
console.log(shuffle(x))
```
## 3.数组去重
参考：[https://github.com/mqyqingfeng/Blog/issues/27](https://github.com/mqyqingfeng/Blog/issues/27)
尝试写一个名为 unique 的工具函数，我们根据一个参数 isSorted 判断传入的数组是否是已排序的，如果为 true，我们就判断相邻元素是否相同，如果为 false，我们就使用 indexOf 进行判断
```js
var array1 = [1, 2, '1', 2, 1];
var array2 = [1, 1, '1', 2, 2];

// unique 工具函数用于数组去重
function unique(array, isSorted) {
    var res = [];
    var seen = [];

    for (var i = 0, len = array.length; i < len; i++) {
        var value = array[i];
        if (isSorted) { // 如果数组是排序过的，判断当前元素与前一个元素是否相等
            if (!i || seen !== value) {
                res.push(value)
            }
            seen = value;
        }
        else if (res.indexOf(value) === -1) { // 如果数组是未排序的，使用 indexOf 方法
            res.push(value);
        }        
    }
    return res;
}

console.log(unique(array1)); // [1, 2, "1"]
console.log(unique(array2, true)); // [1, "1", 2]
```

在数组中元素有一些特殊类型如对象、NaN 的情况下，不同的数组去重方法可能会有不同的返回值
```js
// demo1
var arr = [1, 2, NaN];
arr.indexOf(NaN); // -1
```

indexOf 底层还是使用 === 进行判断，因为 NaN === NaN 的结果为 false，所以使用 indexOf 查找不到 NaN 元素

```js
// demo2
function unique(array) {
   return Array.from(new Set(array));
}
console.log(unique([NaN, NaN])) // [NaN]
```

Set 认为尽管 NaN === NaN 为 false，但是这两个元素是重复的。

## 4.用 reduce 实现 map 方法
```js
// map(function(item, index, arr){}) 
// 对数组的每个元素执行相应的操作，返回一个新数组，其由由操作后的元素组成；不会修改原数组

// reduce(function(prev, curr, index, arr){}, optional)
// 参数依次为：前一个值，当前值，项的索引，数组对象；
// 函数返回的任何值都会作为第一个参数自动传给下一项，首次执行的prev就是数组第一个元素，curr是数组的第二个元素；
// 还可以接受一个参数作为归并的初始值，如果传入了这个参数，首次执行的prev就是这个值，curr是数组的第一个元素
Array.prototype.map = function (callback) {
  let arr = this // this->调用该方法的数组
  return arr.reduce((prev, curr, index, arr ) => {
    prev.push(callback(curr, index, arr))
    return prev
  }, []) // 最后返回的是prev,传入[]则第一轮prev = [], curr= 数组第1个元素
}

let m = [1, 2, 3, 4, 5].map((item, index, arr) => {
  return item * item
})
console.log(m) // [1, 4, 9, 16, 25]
```
