# 1.重建二叉树
[牛客网链接](https://www.nowcoder.com/practice/8a19cbe657394eeaac2f6ea9b0f6fcf6?tpId=13&tqId=11157&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

题目描述：输入某二叉树的前序遍历和中序遍历的结果，请重建出该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。例如输入前序遍历序列 {1,2,4,7,3,5,6,8} 和中序遍历序列 {4,7,2,1,5,3,8,6}，则重建二叉树并返回。

解题思路：
前序序列第一个数将中序序列划分为根和左右子树，递归地利用这个规律来构建二叉树
```js
function TreeNode(x) {
    this.val = x;
    this.left = null;
    this.right = null;
}

function reConstructBinaryTree(pre, vin)
{
    // write code here
    if (pre.length === 0 || vin.length === 0) return null // slice划分时越界会返回[]
    let root = new TreeNode(pre[0])
    let index = vin.indexOf(pre[0])
    root.left = reConstructBinaryTree(pre.slice(1, index+1), vin.slice(0, index))
    root.right = reConstructBinaryTree(pre.slice(index + 1), vin.slice(index+1))
    return root
}
```
# 2.树的子结构
[牛客网链接](https://www.nowcoder.com/practice/6e196c44c7004d15b1610b9afca8bd88?tpId=13&tqId=11170&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

题目描述：输入两棵二叉树 A，B，判断 B 是不是 A 的子结构。（ps：我们约定空树不是任意一个树的子结构）

解题思路：递归：首先判断根结点的值是否相等，如果相等则进一步判断左右结点是否相等，否则判断 A 的左子树是否包含B（true 的话则会返回，false 则会继续判断右边），A 的右子树是否包含 B，另外注意 null 是非 null 树的子结构
```js
function isSubTree(pA, pB) {
    if (pB === null) return true // 空的话肯定是A的子树，注意优先判断pB
    if (pA === null) return false // 判断B是否为A的子树，A为空的话肯定不是了
    if (pA.val === pB.val) {
        return isSubTree(pA.left, pB.left) && isSubTree(pA.right, pB.right) // 短路特性,同时为true才为true
    } else {
        return false
    }
}
function HasSubtree(pRoot1, pRoot2)
{
    // write code here
    if (pRoot1 === null || pRoot2 === null) return false
    return isSubTree(pRoot1, pRoot2) || HasSubtree(pRoot1.left, pRoot2) 
        || HasSubtree(pRoot1.right, pRoot2)
}
```
# 3.二叉树的镜像
[牛客网链接](https://www.nowcoder.com/practice/564f4c26aa584921bc75623e48ca3011?tpId=13&tqId=11171&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

题目描述：操作给定的二叉树，将其变换为源二叉树的镜像。

解题思路：递归：先交换最下层的结点的左右子树，再交换上层的左右子树
```js
function Mirror(root)
{
    // write code here
    if (root === null) return null
    if (root.left !== null) {
        Mirror(root.left)
    }
    if (root.right !== null) {
        Mirror(root.right)
    }
    if (root.left === null && root.right === null) return
    let tmp = root.left
    root.left = root.right
    root.right = tmp
    return root
}
```
# 4.从上往下打印二叉树
[牛客网链接](https://www.nowcoder.com/practice/7fe2212963db4790b57431d9ed259701?tpId=13&tqId=11175&tPage=2&rp=2&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

题目描述：从上往下打印出二叉树的每个节点，同层节点从左至右打印。

解题思路：使用队列，先放入根结点，再放入左右子结点，广搜
```js
function PrintFromTopToBottom(root)
{
    // write code here
    let queue = []
    const res = []
    if (root === null) return res
    queue.push(root) // 队列->层次遍历
    while(queue.length !== 0) {
        let top = queue.shift()
        res.push(top.val)
        if (top.left !== null) {
            queue.push(top.left)
        }
        if (top.right !== null) {
            queue.push(top.right)
        }
    }
    return res
}
```
# 5.二叉搜索树（BST）的后序遍历序列
[牛客网链接](https://www.nowcoder.com/practice/a861533d45854474ac791d90e447bafd?tpId=13&tqId=11176&tPage=2&rp=2&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

题目描述：输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历的结果。如果是则输出Yes,否则输出No。假设输入的数组的任意两个数字都互不相同。
```js
function VerifySquenceOfBST(sequence)
{
    // write code here
    // BST特性：左子结点比根结点小，右子结点大于根结点
    let size = sequence.length
    if (size === 0) return false
    let i = 0
    while (--size) { // "根"总为数组最后一个元素
        while (sequence[i++] < sequence[size]); // 左子树都应该小于根元素
        while (sequence[i++] > sequence[size]); // 右子树都应该大于根元素
        if (i < size) return false // 理论上上面两个循环都应该到达最后一个元素
        i = 0
    }
    return true
}
```
解题思路：非递归方法 -> 看注释
# 6.二叉树中和为某一个值的路径
[牛客网链接](https://www.nowcoder.com/practice/b736e784e3e34731af99065031301bca?tpId=13&tqId=11177&tPage=2&rp=2&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

题目描述：输入一颗二叉树的跟节点和一个整数，打印出二叉树中结点值的和为输入整数的所有路径。路径定义为从树的根结点开始往下一直到叶结点所经过的结点形成一条路径。(注意: 在返回值的list中，数组长度大的数组靠前)

解题思路：dfs的思想，关键是路径的保存；注意题目中“路径”指的是到达叶结点的，中间结点不用判断，整体思路：如果为叶结点则判断累计的和是否与指定的数相等，如果相等，则添加当前路径；dfs的时候注意返回的时候路径弹出当前结点。
```js
var path;
var stack;
function FindPath(root, expectNumber)
{
    // write code here
    if(root == null){
        return [];
    }
    path= [];
    stack = [];
    cal(root,expectNumber);
    return path;
     
}
 
function cal(root,expectNumber){
    stack.push(root.val);
    if(root.val == expectNumber && root.left==null && root.right==null){
        path.push(stack.slice());
    }
    else{
        if(root.left!=null){
            cal(root.left,expectNumber-root.val);
        }
        if(root.right!=null){
            cal(root.right,expectNumber-root.val);
        }
    }
    stack.pop();
}
```
# 7.二叉树的深度
题目描述：输入一棵二叉树，求该树的深度。从根结点到叶结点依次经过的结点（含根、叶结点）形成树的一条路径，最长路径的长度为树的深度。
```js
var maxDepth = function (root) {
    if (root === undefined || root === null) {
        return 0
    }
    return Math.max(maxDepth(root.left),maxDepth(root.right)) + 1
}
```

# 8.平衡二叉树
[牛客网链接](https://www.nowcoder.com/practice/8b3b95850edb4115918ecebdf1b4d222?tpId=13&tqId=11192&tPage=2&rp=2&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

题目描述：输入一棵二叉树，判断该二叉树是否是平衡二叉树。

平衡二叉搜索树（Self-balancing binary search tree）又被称为AVL树（有别于AVL算法），且具有以下性质：它是一 棵空树或它的左右两个子树的高度差的绝对值不超过1，并且左右两个子树都是一棵平衡二叉树。平衡二叉树的常用实现方法有红黑树、AVL、替罪羊树、Treap、伸展树等。 最小二叉平衡树的节点总数的公式如下 F(n)=F(n-1)+F(n-2)+1 这个类似于一个递归的数列，可以参考Fibonacci(斐波那契)数列，1是根节点，F(n-1)是左子树的节点数量，F(n-2)是右子树的节点数量。
```js
// 平衡二叉树：左右子树高度差不超过1，且左右子树也是平衡二叉树
function IsBalanced_Solution(pRoot)
{
    // write code here
  if (pRoot === null) return true
  let leftTreeDeep = getDeep(pRoot.left)
  let rightTreeDeep = getDeep(pRoot.right)
  if (Math.abs(leftTreeDeep - rightTreeDeep) > 1) {
    return false
  }
  return IsBalanced_Solution(pRoot.left) && IsBalanced_Solution(pRoot.right)
}
// 获取树的高度
function getDeep(root) {
  if (root === null) return 0
  let left = getDeep(root.left)
  let right = getDeep(root.right)
  return left > right ? left + 1 : right + 1
}
```
# 9.对称的二叉树
[牛客网链接](https://www.nowcoder.com/practice/ff05d44dfdb04e1d83bdbdab320efbcb?tpId=13&tqId=11211&tPage=3&rp=3&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

题目描述：请实现一个函数，用来判断一颗二叉树是不是对称的。注意，如果一个二叉树同此二叉树的镜像是同样的，定义其为对称的。

解题思路：如果二叉树是对称的，左子树的左子树和右子树的右子树相同，左子树的右子树和右子树的左子树相同即可
```js
function isSymmetrical(pRoot)
{
   if(pRoot==null){
       return true;
   }
    return jury(pRoot,pRoot);
}
function jury(root1,root2){
    if(root1==null && root2==null){
        return true;
    }
    if((root1==null && root2!=null)||(root1!=null && root2==null)){
        return false;
    }
    if(root1.val!==root2.val){
        return false;
    }
    return(jury(root1.left,root2.right)&&jury(root2.left,root1.right));
}
```
# 10.按之字形顺序打印二叉树
[牛客网链接](https://www.nowcoder.com/practice/91b69814117f4e8097390d107d2efbe0?tpId=13&tqId=11212&tPage=3&rp=3&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

题目描述：请实现一个函数按照之字形打印二叉树，即第一行按照从左到右的顺序打印，第二层按照从右至左的顺序打印，第三行按照从左到右的顺序打印，其他行以此类推。
```js
function Print(pRoot)
{
  // write code here
  // 奇数行从左往右，偶数行从右往左
  const res = [], queue = []
  let level = 1
  if (pRoot === null) return res
  queue.push(pRoot)
  while (queue.length !== 0) { // 每轮弹出一层的结点并放入下一层的
    let low = 0, high = queue.length // 确定每层的结点数
    const line = []
    while (low++ < high) {
      let tmp = queue.shift()
      line.push(tmp.val)
      if (tmp.left !== null) { queue.push(tmp.left) }
      if (tmp.right !== null) { queue.push(tmp.right) }
    }
    if (level % 2 === 0) {
      res.push(line.reverse())
    } else {
      res.push(line)
    }
    level++
  }
  return res
}
```
