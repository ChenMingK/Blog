# 数组
- isArray()：Array.isArray(value) 用于检测变量是否为数组类型
- toString()：把数组转换为字符串，并返回结果，每一项以逗号分隔
- push() & pop()：push() 方法用于数组末尾添加项，pop() 方法弹出数组末尾项并返回该项
- shift() & unshift()：移除数组中的第一个项并返回该项 / 数组最前端添加项
- reverse()：反转数组顺序，会改变原数组并返回反转后的数组
- sort()：排序数组，默认为升序；可接受一个自定义的比较函数作为参数
- concat()：基于当前数组的所有项创建一个新数组，该方法会先创建一个当前数组的副本，然后将接收到的参数添加到这个副本的末尾，最后返回新构建的数组
- slice()：基于当前数组中的一或多个项创建一个新数组
- splice()：删除数组中的项或往数组中插入项
- indexOf() & lastIndexOf()：接收两个参数，要查找的项和可选的表示查找起点位置的索引
- 迭代方法：**迭代方法都不会修改原数组**。每个方法都接收两个参数：要在每一项上运行的函数和 （可选的）运行该函数的作用域对象——影响 this 的值。传入这些方法中的函数会接收三个参数：数组项的值、该项在数组中的位置和数组对象本身。
    - every()：对数组中的每一项运行给定函数，如果该函数对每一项都返回 true，则返回 true
    - some()：对数组中的每一项运行给定函数，如果该函数对任一项返回 true，则返回 true
    - filter()：对数组中的每一项运行给定函数，返回该函数会返回 true 的项组成的数组
    - forEach()：对数组中的每一项运行给定函数。这个方法没有返回值
    - map()：对数组中的每一项运行给定函数，返回每次函数调用的结果组成的数组
- reduce & reduceRight()：归并方法，迭代数组的所有项，然后得到一个最终返回的值 
- join(seperator)：把数组中的所有元素放入一个字符串。元素是通过指定的分隔符进行分隔的。seperator 是可选参数，默认使用','分隔
*****


下面通过一些示例来加深对一些稍微复杂的函数的理解：


**1.sort()**：

`values.sort(compare)`，比较函数（compare）接收两个参数，如果第一个参数应该位于第二个之前则返回一个负数，如果两个参数相等则返回 0，如果第一个参数应该位于第二个之后则返回一个正数
```js
function compare (value1, value2) {
    if(value1 < value2){
        return -1
    } else if (value1 > value2) {
        return 1
    } else {
        return 0
    }
}
```
对于数值类型或者其 valueOf() 方法会返回数值类型的对象类型，可以使用一个更简单的比较函数。这个函数只要用第二个值减第一个值即可
```js
function compare (value1, value2) { return value2 - value1 }
```
由于比较函数通过返回一个小于零、等于零或大于零的值来影响排序结果，因此减法操作就可以适当地处理所有这些情况

**2.concat() 与 slice() 与 splice()**


在没有给 `concat()`方法传递参数的情况下，它只是复制当前数组并返回副本。如果传递给 concat() 方法的是一或多个数组，则该方法会将这些数组中的每一项都添加到结果数组中。如果传递的值不是数组，这些值就会被简单地添加到结果数组的末尾。
```js
var colors = ["red", "green", "blue"]
var colors2 = colors.concat("yellow", ["black", "brown"])
alert(colors) // red,green,blue 
alert(colors2) // red,green,blue,yellow,black,brown
```
`slice()`方法可以接受一或两个参数，即要返回项的起始和结束位置。在只有一个参数的情况下，slice() 方法返回从该参数指定位置开始到当前数组末尾的所有项。如果有两个参数，该方法返回起始和结束位置之间的项，但不包括结束位置的项。注意，slice() 方法不会影响原始数组。
```js
var colors = ["red", "green", "blue", "yellow", "purple"]
var colors2 = colors.slice(1)
var colors3 = colors.slice(1,4)
alert(colors2) // green,blue,yellow,purple
alert(colors3) // green,blue,yellow
```
`splice()`：
- 删除：可以删除任意数量的项，只需指定2个参数：要删除的第一项的位置和要删除的项数。例如，`splice(0,2)`会删除数组中的前两项。
- 插入：可以向指定位置插入任意数量的项，只需提供 3 个参数：起始位置、0（要删除的项数）和要插入的项。如果要插入多个项，可以再传入第四、第五，以至任意多个项。例如`splice(2,0,"red","green")`会从当前数组的位置 2 开始插入字符串"red"和"green"。
- 替换：可以向指定位置插入任意数量的项，且同时删除任意数量的项，只需指定 3 个参数：起始位置、要删除的项数和要插入的任意数量的项。插入的项数不必与删除的项数相等。例如， `splice (2,1,"red","green")`会删除当前数组位置 2 的项，然后再从位置 2 开始插入字符串 "red"和"green"。（先删除再插入）
- 返回值：splice() 方法始终都会返回一个数组，该数组中包含从原始数组中删除的项（如果没有删除任何项，则返回一个空数组）

**3.迭代方法的使用**


```js
var numbers = [1,2,3,4,5,4,3,2,1]
var filterResult = numbers.filter(function (item, index, array) {
 return (item > 2)
})
alert(filterResult) // [3,4,5,4,3]
```
来看一道常见的面试题：`['1', '2', '3'].map(parseInt)`答案是多少?


答案 ：[1, NaN, NaN]


解析：parseInt 接收两个参数 (sting, radix)，其中 radix 代表进制。省略 radix 或 radix = 0，则数字将以十进制解析
因此，map 遍历 ['1', '2', '3']，相应 parseInt 接收参数如下
```js
parseInt('1', 0)  // 1
parseInt('2', 1)  // NaN
parseInt('3', 2)  // NaN
```
通过下面一些示例你可以更清晰地理解 parseInt() 方法：
```js
parseInt("10")		    //返回 10
parseInt("19",10)		//返回 19 (10+9)
parseInt("11",2)		//返回 3 (2+1)
parseInt("17",8)		//返回 15 (8+7)
parseInt("1f",16)		//返回 31 (16+15)
```
**4.归并方法的使用**


```js
reduce(function (prev, curr, index, arr) {}, optional)
```
- 参数依次为：前一个值，当前值，项的索引，数组对象。
- 函数返回的任何值都会作为第一个参数自动传给下一项，首次执行时 prev 就是数组第一个元素，curr 是数组的第二个元素。
- 还可以接受一个参数作为归并的初始值，如果传入了这个参数，首次执行的 prev 就是这个值，curr 是数组的第一个元素

通过这个常见面试题可以加深我们对 reduce 的理解：用数组的 reduce 实现 map 方法
```js
// map(function (item, index, arr) {}) 
// 对数组的每个元素执行相应的操作，返回一个新数组，其由由操作后的元素组成；不会修改原数组

Array.prototype.map = function (callback) {
  let arr = this // this->调用该方法的数组
  return arr.reduce((prev, curr, index, arr ) => {
    prev.push(callback(curr, index, arr)) // 数组中每一项执行传入的回调函数
    return prev
  }, []) // 最后返回的是prev,传入[]则第一轮prev = [], curr= 数组第1个元素
}

let m = [1, 2, 3, 4, 5].map((item, index, arr) => {
  return item * item
})
console.log(m) // [1, 4, 9, 16, 25]
```

**5.检测数组的方式**


```js
const myArray = [1, 3, 5, 7, 9]
const myValue = 1
// 1.使用 instanceof, 判断该对象的原型链上是否有 Array.prototype
console.log(myArray instanceof Array) // true
console.log(myValue instanceof Array) // false
 
// 2.使用 constructor, 注意实例本身是没有 constructor 属性的，
//   会从原型链上读取到 Array.prototype (如果是数组)
console.log(myArray.constructor === Array) // true
console.log(myValue.constructor === Array) // false
 
// 3.使用 Object.prototype.toString.call(arr) === '[object Array]'
// 即改变 this 指向, 调用 Object 原型上的 toString 方法
function myIsArray(arr) {
  return Object.prototype.toString.call(arr)
}
let obj = {}
let fn = function() {}
console.log(myIsArray(myArray)) // [object Array]
console.log(myIsArray(myValue)) // [object Number]
console.log(myIsArray(obj))     // [object Object]
console.log(myIsArray(fn))      // [object Function]
 
// 4.ES5 定义了 Array.isArray
console.log(Array.isArray(myArray)) // true
```
## ES6新增
- Array.from()：该方法用于将两类对象转换为真正的数组：类似数组的对象（array-like object）和可遍历（iterable）对象（包括 Set 和 Map）
```js
// NodeList 对象
let ps = document.querySelectorAll('p')
Array.from(ps).forEach(function (p) { // 只有转换为真正的数组才能使用数组方法
  console.log(p)
})

// arguments对象
function foo () {
  var args = Array.from(arguments)
  // ...
}
```
- Array.of()：该方法用于将一组值转换为数组，总是返回参数值组成的数组，如果没有参数则返回一个空数组
```js
// Array.of 弥补构造函数因参数个数不同而造成的的行为差异
Array.of(3, 11, 8) // [3, 11, 8]
Array.of(3) // [3]
Array.of(3).length // 1

Array() // []
Array(3) // [, , ,]
Array(3, 11, 8) // [3, 11, 8]
```

- copyWithin(target, start, end)：在当前数组内部将指定位置的成员复制到其他位置（会覆盖原有成员），然后返回当前数组。这个方法会修改当前数组
    - target（必选）：从该位置开始替换数据
    - start（可选）：从该位置开始读取数据，默认为 0；如果为负数，表示倒数
    - end（可选）：到该位置前停止读取数据，默认等于数组长度
```js
[1, 2, 3, 4, 5].copyWithin(0, 3) // [4, 5, 3, 4, 5]
```
- find() 和 findIndex()：find 方法用于找出第一个符合条件的数组成员，参数是一个回调函数，所有数组成员依次执行该回调函数，直到找出第一个返回值为 true 的成员，然后返回该成员，如果没有符合条件的成员，则返回 undefined
- findIndex() 与 find() 类似，返回的是符合条件的成员的位置，如果都不符合，返回 -1
```js
// 找出数组中第一个大于 9 的成员
// 回调函数可接受 3 个参数：当前的值，当前的位置，原数组
[1, 5, 10, 15].find(function (value, index, arr) {
  return value > 9
}) // 10

[1, 5, 10, 15].findIndex(function (value, index, arr) {
  return value > 9
}) // 2

// 这两个方法可以发现 NaN，弥补了数组的 IndexOf 方法的不足
[NaN].indexOf(NaN) // -1
[NaN].findIndex(y => Object.is(NaN, y)) // 0
```
- fill()：fill 方法使用给定值填充一个数组
```js
['a',  'b', 'c'].fill(7) // 7 7 7

// 用于初始化空数组
new Array(3).fill(7) // 7 7 7

// 还可以接受第二个和第三个参数，用于指定填充的起始位置和结束位置
['a', 'b', 'c'].fill(7, 1, 2) // ['a', 7, 'c']
```
- includes()：返回一个布尔值，表示某个数组是否包含给定的值
```js
// 第二个参数表示搜索的起始位置，默认为0；如果为负数表示倒数的位置；如果大于数组长度，则会重置为0
[1, 2, 3].includes(3, 3) // false
[1, 2, 3].includes(3, -1) // true
```
# 字符串
- concat()：拼接字符串
- slice(start, end) & substr(start, length) & substring(start, stop)：功能类似，记 substr 和substring 即可，根据是按长度来截取还是按指定下标来进行选择；不指定第二个参数就是截取从 start 位置到末尾的所有字符串
- indexOf(value ,index) & lastIndexOf()：从一个字符串中搜索给定的子字符串，然后返子字符串的位置（如果没有找到该子字符串，则返回 -1）区别是从前还是从后开始
- trim()：这个方法会创建一个字符串的副本，删除前置及后缀的所有空格，然后返回结果；有些浏览器还支持非标准的 trimLeft() 和 trimRgiht() 方法
- toLowerCase() & toLocaleLowerCase() & toUpperCase() & toLocaleUpperCase()：大小写转换，Lcalel 是针对特定地区的实现。一般来说，在不知道自己的代码将在哪种语言环境中运行的情况下，还是使用针对地区的方法更稳妥一些
    - 返回一个转换后的字符串（不影响原字符串），toLowerCase() 是转换为小写，toUpperCase()是转换为大写
- split(separator, howmany)：基于指定的分隔符将一个字符串分割成多个子字符串，并将结果放在一个数组中
- match()：根据是否全局搜索，返回值会不同
- search()：返回字符串中第一个匹配项的索引；如果没有找到匹配项，则返回 -1。而且，search() 方法始终是从字符串开头向后查找模式。
- replace()：用于在字符串中用一些字符替换另一些字符，或替换一个与正则表达式匹配的子串。

较难理解的主要是最后三个模式匹配方法，在左侧目录的“正则表达式”部分中有介绍，这里不再赘述。
## ES6新增
- codePointAt()：对于 2 字节存储的常规字符而言其返回结果与 charCodeAt() 相同，区别是其该方法能正确处理 4 字节存储的字符，返回这个字符的码点
- fromCodePoint()：可识别大于 0xFFFF 的字符，弥补了 String.fromCharCode 方法的不足，作用上与 codePointAt 正好相反
- at()：能正确返回码点大于 oxFFFF 的字符的位置
- 遍历器接口：ES6 为字符串添加了遍历器接口，使得字符串可以由 for...of 循环遍历
```js
for (let codePoint of 'foo') {
    console.log(codePoint)
}
// "f"
// "o"
// "o"
```
```js
// 用 1-9a-zA-Z 生成 5 位随机数
let arr = []
for (let i = 1; i <= 9; i++) {
  arr.push(i)
}
for (let i = 97; i <= 122; i++) { // a 的 ASCII 码为 97，z 为 122 -> 'a'.codePointAt(0)
  arr.push(String.fromCodePoint(i)) // 字符转换成码点 + 1 后再转换为字符
}

for (let i = 65; i <= 90; i++) { // A 的 ASCII 码为 65，Z 为90
  arr.push(String.fromCodePoint(i))
}

let randomString = ''
for (let i = 0; i < 5; i++) {
  let randomIndex = Math.floor(arr.length * Math.random())
  randomString += arr[randomIndex]
}
console.log(randomString)
```
- includes()：返回布尔值，表示是否找到了参数字符串
- startsWith()：返回布尔值，表示参数字符串是否在源字符串的头部
- endsWith()：返回布尔值，表示参数字符串是否在源字符串的尾部
```js
var s = 'Hello world'
// 都支持第二个参数，表示开始搜索的位置；endsWith 的行为与其他两个不同，
// 它针对前 n 个字符，而其他两个方法针对从第 n 个位置到字符串结束之间的字符
s.startsWith('world', 6) // true
s.endsWith('Hello', 5) // true
s.includes('Hello', 6) // false
```

- repeat()：返回一个新字符串，表示将原字符串重复 n 次。`'x'.repeat(3) // "xxx"`
- padStart() & padEnd()：字符串补全长度，padStart 用于头部补全，padEnd 用于尾部补全；如果某个字符串不够指定长度，会在头部或尾部补全
```js
'x'.padStart(5, 'ab') // 'ababx'
'x'.padEnd(4, 'ab') // 'abax'

'x'.padEnd(5, 'ab') // 'xabab'
'x'.padEnd(4, 'ab') // 'xaba

// 如果省略第二个参数，则用空格来补全
'x'.padEnd(4) // 'x   '

// 可以这么用
// 为数值补全位数
'12'.padStart(10, '0') // "0000000012"
// 提示字符串格式
'12'.padStart(10, 'YYYY-MM-DD') // "YYYY-MM-12"
'09-12'.padStart(10, 'YYYY-MM-DD') // "YYYY-09-12"
```
- 模板字符串
    - 使用模板字符串表示多行字符串，所有的空格和缩进都会被保留在输出中
    - 在模板字符串中嵌入变量，需要将变量名写在 `${} `中
    - `${fn()}`大括号内可以放入任意的 JavaScript 表达式，可以进行运算，以及引用对象属性，可以调用函数
# Map
|  Map方法   |  描述   |
| :---- | :---- |
|  Map()   |  构造函数   |
|  set(key, value)   |  设置 key 所对应的键值，然后返回整个 Map 结构。<br>如果 key 已经有值，则键值被更新，否则新生成该键   |
|  get(key)   |   读取 key 对应的键值，若未找到 key 则返回 undefined  |
|  has(key)   |   返回一个布尔值，表示某个键是否存在于 Map 结构中  |
|   delete(key)  |  删除某个键，返回 true；删除失败返回 false   |
|   clear()  |  清除所有成员，没有返回值   |
|   keys()  |    返回键名的遍历器 |
|  values()   |  返回键值的遍历器   |
|  entries()  |   返回键值对的遍历器 |
|  forEach()  | 使用回调函数遍历每个成员，用法：map.forEach(function (value, key, map){ })   |

# Set
Set 内部判断两个值是否相同使用的算法为“Same-value equality”，它类似于精确相等运算符（===），
主要的区别是 NaN 等于自身，而精确相等运算符认为 NaN 不等于自身。


|   Set方法  |  描述   |
| :---- | :---- |
|   Set()  | 构造函数，可以接受一个数组，或者具有 iterable 接口的其他数据结构作为参数    |
|   add(value) |  添加某个值，返回 Set 结构本身  |
|  delete(value)  |  删除某个值，返回布尔值，表示是否删除成功  |
|  has(value)  |  返回布尔值，表示参数是否为 Set 的成员  |
|  clear()  | 清除所有成员，没有返回值   |
|  keys()  |  返回键名的遍历器，用法：`for (let item of set.keys())` |
|   values() | 返回键值的遍历器   |
|  entries()  | 返回键值对的遍历器   |
|   forEach() | 使用回调函数遍历每个成员   |

用途：
1.去除数组的重复元素
```js
function dedupe (array){
    return Array.from(new Set(array))
}
console.log(dedupe([1, 1, 2, 3]))  // [1, 2, 3]
```
2.扩展运算符（...）内部使用 for...of 循环，所以也可以用于 Set 结构，扩展运算符和 Set结构相结合也可以去除数组的重复成员
```js
let arr = [3, 5, 2, 2, 5, 5]
let unique = [...new Set(arr)]
console.log(unique)  // [3, 5, 2]
```
3.利用 Set 实现并集（Union）、交集（Intersect）、差集（Difference）
```js
let a = new Set([1, 2, 3])
let b = new Set([4, 3, 2])
 
// 并集
let union = new Set([...a, ...b])
console.log(union)  // Set {1, 2, 3, 4}
 
// 交集
let intersect = new Set([...a].filter(x => b.has(x)))
console.log(intersect); // Set {2, 3}
 
// 差集
let difference = new Set([...a].filter(x => !b.has(x)))
console.log(difference)  // Set {1}

```
