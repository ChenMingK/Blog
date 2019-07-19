# 栈
栈是一种特殊的受限线性表。
栈是限定只能在表的一端进行插入或删除操作的线性表。特性：LIFO


用  JavaScript 来模拟一个栈：
``` js
class Stack {
  constructor () {
    this.stack = []
  }

  push (item) {
    this.stack.push(item)
  }

  pop () {
    this.stack.pop()
  }

  peek () {
    return this.stack[this.getCount() - 1]
  }

  getCount () {
    return this.stack.length
  }

  empty () {
    return this.getCount() === 0
  }
}
```

## 应用
使用栈进行括号匹配 [LeetCode](https://leetcode.com/problems/valid-parentheses/submissions/1)
``` javaScript
/**
 * @param {string} s
 * @return {boolean}
 */
var isValid = function(s) {
    let map = {
        '(': -1,
        ')': 1,
        '[': -2,
        ']': 2,
        '{': -3,
        '}': 3
    }
    let stack = []
    for (let i = 0; i < s.length; i++) {
        if (map[s[i]] < 0) {
            stack.push(map[s[i]])
        } else {
            if (map[s[i]] + stack.pop() !== 0) return false
        }
    }
    return stack.length === 0 ? true : false
};
```

# 队列
队列是一种特殊的受限线性表。
队列是限定只能在表的一端进行插入操作，在另一端进行删除操作的线性表。特性：FIFO


用 JavaScript 模拟一个单链队列
``` js
class Queue {
  constructor () {
    this.queue = []
  }

  enQueue (item) {
    return this.queue.push(item)
  }

  deQueue() {
    this.queue.shift()
  }

  getHeader () {
    return this.queue[0]
  }

  getLength () {
    return this.queue.length
  }

  isEmpty () {
    return this.getLength() === 0
  }
}
```
因为单链队列在出队操作的时候需要 O(n) 的时间复杂度，所以引入了循环队列。循环队列的出队操作平均是 O(1) 的时间复杂度。
``` javaScript
class SqQueue {
  constructor(length) {
    this.queue = new Array(length + 1)
    // 队头
    this.first = 0
    // 队尾
    this.last = 0
    // 当前队列大小
    this.size = 0
  }
  enQueue(item) {
    // 判断队尾 + 1 是否为队头
    // 如果是就代表需要扩容数组
    // % this.queue.length 是为了防止数组越界
    if (this.first === (this.last + 1) % this.queue.length) {
      this.resize(this.getLength() * 2 + 1)
    }
    this.queue[this.last] = item
    this.size++
    this.last = (this.last + 1) % this.queue.length
  }
  deQueue() {
    if (this.isEmpty()) {
      throw Error('Queue is empty')
    }
    let r = this.queue[this.first]
    this.queue[this.first] = null
    this.first = (this.first + 1) % this.queue.length
    this.size--
    // 判断当前队列大小是否过小
    // 为了保证不浪费空间，在队列空间等于总长度四分之一时
    // 且不为 2 时缩小总长度为当前的一半
    if (this.size === this.getLength() / 4 && this.getLength() / 2 !== 0) {
      this.resize(this.getLength() / 2)
    }
    return r
  }
  getHeader() {
    if (this.isEmpty()) {
      throw Error('Queue is empty')
    }
    return this.queue[this.first]
  }
  getLength() {
    return this.queue.length - 1
  }
  isEmpty() {
    return this.first === this.last
  }
  resize(length) {
    let q = new Array(length)
    for (let i = 0; i < length; i++) {
      q[i] = this.queue[(i + this.first) % this.queue.length]
    }
    this.queue = q
    this.first = 0
    this.last = this.size
  }
}
```
# 链表
链表是一个线性结构，同时也是一个天然的递归结构。链表结构可以充分利用计算机内存空间，实现灵活的内存动态管理。


但是链表失去了数组随机读取的优点，同时链表由于增加了结点的指针域，空间开销比较大。

用 JavaScript 模拟一个单向链表
``` js
class Node {
  constructor(v, next) {
    this.value = v
    this.next = next
  }
}
class LinkList {
  constructor() {
    // 链表长度
    this.size = 0
    // 虚拟头部
    this.dummyNode = new Node(null, null)
  }
  find(header, index, currentIndex) {
    if (index === currentIndex) return header
    return this.find(header.next, index, currentIndex + 1)
  }
  addNode(v, index) {
    this.checkIndex(index)
    // 当往链表末尾插入时，prev.next 为空
    // 其他情况时，因为要插入节点，所以插入的节点
    // 的 next 应该是 prev.next
    // 然后设置 prev.next 为插入的节点
    let prev = this.find(this.dummyNode, index, 0)
    prev.next = new Node(v, prev.next)
    this.size++
    return prev.next
  }
  insertNode(v, index) {
    return this.addNode(v, index)
  }
  addToFirst(v) {
    return this.addNode(v, 0)
  }
  addToLast(v) {
    return this.addNode(v, this.size)
  }
  removeNode(index, isLast) {
    this.checkIndex(index)
    index = isLast ? index - 1 : index
    let prev = this.find(this.dummyNode, index, 0)
    let node = prev.next
    prev.next = node.next
    node.next = null
    this.size--
    return node
  }
  removeFirstNode() {
    return this.removeNode(0)
  }
  removeLastNode() {
    return this.removeNode(this.size, true)
  }
  checkIndex(index) {
    if (index < 0 || index > this.size) throw Error('Index error')
  }
  getNode(index) {
    this.checkIndex(index)
    if (this.isEmpty()) return
    return this.find(this.dummyNode, index, 0).next
  }
  isEmpty() {
    return this.size === 0
  }
  getSize() {
    return this.size
  }
}
```
## 常见问题
1.反转单向链表


``` js
/**
 * Definition for singly-linked list.
 * function ListNode(val) {
 *     this.val = val;
 *     this.next = null;
 * }
 */
/**
 * @param {ListNode} head
 * @return {ListNode}
 */
var reverseList = function(head) {
  let prev = null
  let curr = head
  /*
    ①判断当前节点是否为空
    ②不为空就获取当前节点的下一节点
    ③然后把当前节点的next设为上一个节点
    ④然后把pre设为curr， curr设为next
  */
  while (curr !== null) {
      let nextTemp = curr.next
      curr.next = prev
      prev = curr
      curr = nextTemp
  }
  return prev
};
```


2.如何判断一个链表是否存在环？

- hashmap 存储节点 ID
- 快慢指针

[参考答案](https://www.cnblogs.com/qingyunzong/p/9143321.html)

# 树
## 二叉树
树拥有很多种结构，二叉树是树中最常用的结构，同时也是一个天然的递归结构。

二叉树拥有一个根节点，每个节点至多拥有两个子节点，分别为：左节点和右节点。树的最底部节点称之为叶节点。

满二叉树(full binary tree)：每一个结点或者是一个分支结点，并恰好有两个非空子结点；或者是叶结点。

完全二叉树(complete binary tree)：从根结点起每层从左到右填充。一棵高度为 d 的完全二叉树除了 d-1 层外，
每一层都是满的。底层叶结点集中在左边的若干位置上。

## 递归方式遍历二叉树
``` js
function preorderTraversal (root) {
  if (root !== null) {
    visit(root)
    preorderTraversal(root.left)
    preorderTraversal(root.right)
  }
}

function middleTraversal (root) {
  if (root !== null) {
    middleTraversal(root.left)
    visit(root)
    middleTraversal(root.right)
  }
}

function postTraversal (root) {
  if (!root) {
    postTraversal(root.left)
    postTraversal(root.right)
    visit(root)
  }
}
```

## 非递归方式遍历二叉树
``` js
// 用 stack 模拟递归，用一个 result 数组存储结点顺序
function preorderTraversal (root) {
  if (root === null) return 
  const result = [], stack = []
  stack.push(root)
  while (stack.length !== 0) {
    let p = stack.pop()
    result.push(p.val)
    // 先放右边再放左边，因为栈的特性，后进先出，我们先拿左边的就相当于先访问左子树了
    if (p.right !== null) {
      stack.push(p.right)
    }
    if (p.left !== null) {
      stack.push(p.left)
    }
  }
  return result
}

// 中序遍历：左根右，先把根和左边的结点全部存入栈，再处理右边的，右边子树也做相同处理
function middleTraversal (root) {
  if (root === null) return
  const result = [], stack = []

  while (true) {
    while (root !== null) {
      stack.push(root)
      root = root.left
    }
    // 终止条件：最后树遍历完了就终止了
    if (stack.length === 0) {
      break
    }
    let p = stack.pop()
    result.push(p.val)
    root = p.right
  }

  return result
}

// LRD
function postTraversal (root) {
  if (root === null) return
  const result = [], stack = []
  stack.push(root)
  
  while (stack.length !== 0) {
    let p = stack.pop()
    result.push(p.val)
    if (p.left !== null) {
      stack.push(p.left)
    }
    if (p.right !== null) {
      stack.push(p.right)
    }
  }

  return result.reverse() // 反转过来恰好是后序遍历......
}
```
