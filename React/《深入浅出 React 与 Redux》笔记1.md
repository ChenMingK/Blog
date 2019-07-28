# 第 1 章
## 1.JSX 的 onClick 事件处理方式和 HTML 的 onclick 有很大的不同：
在 HTML 中直接使用 onclick 存在的问题：

- onclick 添加的事件处理函数是在全局环境下执行的，这污染了全局环境，很容易产生意料不到的后果。

- 给很多 DOM 元素添加 onclick 事件，可能会影响网页的性能，毕竟，网页的事件处理函数越多，性能就会越低。

- 对于使用 onclick 的 DOM 元素，如果要动态地从 DOM 树中删掉的话，需要把对应的事件处理器注销，如果忘了注销，就可能造成内存泄漏，这样的 bug 很难被发现

上述问题在 JSX 中都不存在

- onClick 挂载的每个函数，都可以控制在组件范围内，不会污染全局空间

- 我们在 JSX 中看到一个组件使用了 onClick，但并没有产生直接使用 onclick 的HTML，而是使用了事件委托（event delegation）的方式处理点击事件。
无论有多少个 onClick 出现，其实最后都只在 DOM 树上添加了一个事件处理函数，挂在最顶层的 DOM 节点上。
所有的点击事件都被这个事件处理函数捕获，然后根据具体组件分配给特定函数，使用事件委托的性能当然要比为每个 onClick 都挂载一个事件处理函数要高，

- 因为 React 控制了组件的生命周期，在 unmount 的时候自然能够清除相关的所有事件处理函数，内存泄漏也不再是一个问题。

## 2.script 脚本中的 eject(弹射) 命令

执行 `npm run eject` 就是把潜藏在 react-scripts 中的一系列技术栈配置都 “弹射” 到应用的顶层，然后我们就可以研究这些配置细节，而且可以更灵活地定制应用的配置。

如 config 目录下的 webpack.config.dev.js 文件，定制 npm start 所做的构造过程

## 3.*UI=render(data)*

React 的理念可归结为这一公式。

用户看到的界面（UI），应该是一个函数（在这里叫 render）的执行结果，只接受数据（data）作为参数。

这个函数是一个纯函数，即没有任何副作用，输出完全依赖于输入的函数，两次函数调用如果输入相同，得到的结果也绝对相同。

如此一来，最终的用户界面，在 render 函数确定的情况下完全取决于输入数据。

对于开发者来说，重要的是区分开哪些属于 data，哪些属于 render，想要更新用户界面，要做的就是更新 data，用户界面自然会作出响应。所以 React 实践的也是“响应式编程”（Reactive Programming）的思想，React 的名字由此而来。

## 4.Virutal DOM 的简单描述

DOM 树是对 HTML 的抽象，Virtual DOM 是对 DOM 树的抽象。

Virtual DOM 不会触及浏览器的部分，只是存在于 JavaScript 空间的树形结构，每次自上而下渲染 React 组件时，会对比这一次产生的 Virtual DOM 和上一次渲染的 Virtual DOM，对比就会发现差别，然后修改真正的 DOM 树时就只需要触及差别中的部分就行

# 第 2 章 设计高质量的 React 组件

## 1.组件的划分通则

组件的划分要满足高内聚（High Cohesion）和低耦合（Low Coupling）的原则

**高内聚** 指的是把逻辑紧密相关的内容放在一个组件中：传统上，内容由 HTML 表示，交互行为放在 JavaScript 代码文件中，样式放在 CSS 文件中定义，这虽然满足一个工程模块的需要，却要放在三个不同的文件中。

React 中，展示内容的 JSX、定义行为的 JavaScript、甚至定义样式的 CSS，都可以放在一个 JavaScript 文件中，所以 React 天生具有高内聚的特点

**低耦合** 指不同组件之间的依赖关系要尽量弱化，也就是每个组件要尽量独立。

保持整个系统的低耦合度，需要对系统中的功能由充分的认识，然后根据功能点划分模块，让不同的组件去实现不同的功能

## 2.React 中的数据

React 组件的数据分为两种，porp 和 state，prop 或者 state 改变都可能引发组件的重新渲染，那么设计一个组件的时候，什么时候用 porp，什么时候用 state 呢？

prop 是组件的对外接口，state 是组件的内部状态，对外用 prop，内部用 state

prop 的类型不限于纯数据，也可以是函数，函数类型的 prop 等于让父组件给了子组件一个回调函数



关于 propTypes 的使用需要注意一些问题：定义类的 propTypes 属性，无疑是要占用一些代码空间，而且类型检查也是要消耗 CPU 计算资源的。

其次，在线上环境做 propTypes 类型检查没有什么帮助，即在开发过程中为了避免犯错我们才使用 propTypes，发布产品代码时，可以用一种自动的方式将 propTypes 去掉。

`babel-react-optimize` 具有这个功能，可以通过过 npm 安装，但是确保只在发布产品代码的时候使用它。

state 需要在构造函数中初始化（通过对 state 的赋值），组件的 state 必须是一个 JavaScript 对象。

通过 `this.state` 读取当前组件的 state，通过 `this.setState()` 更新 state

**prop 和 state 的对比**

* prop 用于定义外部接口，state 用于记录内部的状态
* prop 的赋值在外部世界使用组件时，state 的赋值在组件内部
* 组件不应该改变 prop 的值，而 state 的存在目的就是让组件来改变的。虽然 React 并没有办法阻止你去修改传入的 props 对象，但是你仍然不该跨越这条红线，否则最后可能出现不可预料的 bug

# 第 3 章：从 Flux 到 Redux

## Flux 到 Redux

<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/diveintoreact1.png" width=50%/>

对于 MVC 框架，为了让数据流可控，Controller 应该是中心，当 View 要传递消息给 Model 时，应该调用 Controller 的方法，同样，当 Model 要更新 View 时，也应该通过 Controller 引发新的渲染


<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/diveintoreact2.png" width=50%/>

一个 Flux 应用包含四个部分：

* Dispatcher：处理动作分发，维持 Store 之间的依赖关系
* Store：负责存储数据和处理数据相关逻辑
* Action：驱动 Dispatcher 的 JavaScript 对象
* View：视图部分，负责显示用户界面

MVC 与 Flux 的对比：在 MVC 框架中，系统能够提供什么样的服务，通过 Controller 暴露函数来实现。每增加一个功能，Controller 往往就需要增加一个函数；在 Flux 的世界里，新增加功能并不需要 Dispatcher 增加新的函数，要做的就是增加一种新的 Action 类型， Dispatcher 的对外接口并不用改变。
当需要扩充应用所能处理的”请求“时，MVC 方法就需要增加新的 Controller，而对于 Flux 则只是增加新的 Action

Flux 的优点就在于其 “单向数据流” 的管理方式，这种 “限制” 禁绝了数据流混乱的可能
其不足之处在于：

* Store 之间的依赖关系
* 难以进行服务器端渲染
* Store 混杂了逻辑和状态



## Redux
<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/diveintoreact3.png" />

Flux 的基本原则是“单向数据流”，Redux 在此基础上强调三个基本原则：

* 唯一数据源（Single Source of Truth）
* 保持状态只读（State is read-only）
* 数据改变只能通过纯函数完成（Changes are made with pure functions）

1.唯一数据源：应用的状态数据应该只存储在唯一的一个 Store 上，避免了状态数据分散在多个 Store 中造成的数据冗余

2.保持状态只读：要修改 Store 的状态，必须要通过派发一个 action 对象来完成

3.数据改变只能通过纯函数完成：这里所说的纯函数就是 Reducer，在 Redux 中，每个 reducer 的函数签名如下：

`reducer(state, action)`

第一个参数 state 是当前的状态，第二个参数 action 是接收到的 action 对象，而 reducer 函数要做的事情，就是根据 state 和 action 的值产生一个新的对象返回。注意 reducer 必须是纯函数，即函数的返回结果完全由参数 state 和 action 决定，而且不产生任何副作用，也不能修改参数 state 和 action 对象

## 容器组件与傻瓜组件

在 Redex 框架下，一个 React 组件基本上就是要完成以下两个功能：

* 和 Redux Store 打交道，读取 Store 的状态，用于初始化组件的状态，同时还要监听 Store 的状态改变；当 Store 状态发生变化时，需要更新组件状态，从而驱动组件重新渲染；当需要更新 Store 状态时，就要派发 action 对象
* 根据当前 prop 和 state，渲染出用户界面



如果 React 组件都是要包办上面所说的两个任务，似乎做的事情稍微多了点。

所以我们可以考虑拆分为两个组件，分别承担一个任务，然后把两个组件嵌套起来，完成原本一个组件完成的所有任务。

这样的关系里，两个组件是父子组件的关系。负责与 Redux Store 打交道的组件，处于外层，称为容器组件（Container Component）；只负责渲染界面的组件，处于内层，叫做展示组件（Presentational Component）；

外层的容器组件也叫作聪明组件（Smart Componnet），内层的展示组件又叫做傻瓜组件（Dumb Component）

<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/diveintoreact4.png" width=50%/>


状态全部交由容器组件打理，展示组件只需要根据 props 来渲染结果，不需要 state；

这只是设计 React组件的一种模式，和 Redux 没有直接关系，另外，没有 state 只有一个 render 方法，所有数据都来自于 props 的组件称为 **无状态组件**。


无状态组件可以写成一个函数，不再需要用对象表示，如下所示：
```js
// 解构赋值
function Counter ({caption, onIncrement, onDecrement, value}) {
  return (
    <div>
      <button style={buttonStyle} onClick={onIncrement}>+</button>
      <button style={buttonStyle} onClick={onDecrement}>-</button>
      <span>{caption} count: {value}</span>
    </div>
  );
}

// 不使用解构赋值
function Counter (props) {
  const {caption, onIcrement, onDecrement, value} = props
  // ...
}
```

## 组件 Context

不使用 react-redux 之前，每个组件都需要直接导入 Redux Store

`import store from '../Store.js'`

虽然 Redux 应用全局就一个 Store，但是这样的直接导入依然有问题（很麻烦）

为了解决这个问题，那就只能让上层组件把 Store 传递下来，首先想到的是用 props，但是层层传递的方法显然是不可行的。

设想在一个嵌套多层的组件结构中，只有最里层的组件才需要使用 store，但是为了把 store 从最外层传递到最里层，就要求中间所有的组件都需要增加对这个 store prop 的支持，即使根本不使用它。

React 提供了一个叫 Context 的功能，能完美地解决这个问题。

所谓 Context，就是“上下文环境”，让一个树状组件上所有组件都能访问一个共同的对象。

<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/diveintoreact5.png" width=50%/>


其实 react-redux 这个库就是帮我们完成了 Context 的功能。

这里分析下它使用的这条语句：
```js
export default connect(mapStateToProps, mapDispatchToProps)(Counter)
```
connect 是 react-redux 提供的一个方法，这个方法接收两个参数，执行结果依然是一个函数，所以才可以在后面又加一个圆括号（柯里化？），把 connect 函数执行的结果立即执行。
这里有两次函数的执行，第一次是 connect 函数的执行，第二次是把 connect 函数返回的函数再次执行，最后产生的就是容器组件。
connect 函数具体做了哪些事呢？

* 把 Store 上的状态转化为内层傻瓜组件的 prop
* 把内层傻瓜组件中的用户动作转化为派送给 Store 的动作

这两个工作一个是内层傻瓜对象的输入，一个是内层傻瓜对象的输出。

mapStateToProps（命名是业界习惯）就是把 Store 上的状态转化为内层组件的 props，建立映射关系；

mapDispatchToProps 把内层傻瓜组件暴露出来的函数类型的 prop 关联上 dispatch 函数的调用。

```js
// 第二个参数是直接传递给外层容器组件的 props
function mapDispatchToProps(dispatch, ownProps) {
  return {
    onIncrement: () => {
      dispatch(Actions.increment(ownProps.caption));
    },
    onDecrement: () => {
      dispatch(Actions.decrement(ownProps.caption));
    }
  }
}
```

# 第四章 模块化 React 和 Redux 应用

## 代码文件的组织方式

1.按角色组织

```shell
reducers/
    todoReducer.js
    filterReducer.js
actions/
    todoActions.js
    filterAction.js
components/
    todoList.js
    todoItem.js
    filter.js
containers/
    todoListContainer.js
    todoItemContainer.js
    filterContainer.js
```
- reducer 目录包含所有 Redux 的 reducer
- actions 目录包含所有 action 构造函数
- components 目录包含所有的傻瓜组件
- containers 目录包含所有的容器组件

2.按功能组织
```shell
todoList/
    action.js
    actionType.js
    index.js
    reducer.js
    views/
        component.js
        container.js
filter/
    action.js
    actionType.js
    index.js
    reducer.js
    views/
        component.js
        container.js
```
- actionType.js 定义 action 类型
- action.js 定义 action 构造函数，决定了这个功能模块可以接受的动作
- reducer.js 定义这个功能模块如何响应 action.js 中定义的动作
- views 目录包含这个功能模块中所有的 React 组件，包括傻瓜组件和容器组件
- index.js 把所有的角色导入，然后统一导出

>个人感觉不太能够接受这两种组织文件的方式......

# 第 5 章 React 组件的性能优化
## 引入性能检测工具 React Pref
`npm install react-addons-perf -D`：开发环境下性能检测（哪些组件造成了无意义的渲染）

`npm install redux-immutable-state-invariant -D`：开发环境下检测 reducer 是否为纯函数，不是则报错

Store.js 文件
```js
import {createStore, combineReducers, applyMiddleware, compose} from 'redux';

import {reducer as todoReducer} from './todos';
import {reducer as filterReducer} from './filter';

// 下面三行代码!
import Perf from 'react-addons-perf'
const win = window;
win.Perf = Perf

const reducer = combineReducers({
  todos: todoReducer,
  filter: filterReducer
});

const middlewares = []; // 考虑到将来的扩展，使用数组变量 middlewares 来存储所有的中间件，之后的中间件直接 push
if (process.env.NODE_ENV !== 'production') {
  // 使用 require 是因为 import 语句不能存在于条件语句中
  middlewares.push(require('redux-immutable-state-invariant')()); // react-immutable-state-invariant 中间件只在开发环境下有意义，用于检查 reducer 是否为纯函数
}

// Redux 提供 compose 函数把多个 Store Enhancer 组合在一起
const storeEnhancers = compose(
  applyMiddleware(...middlewares),
  (win && win.devToolsExtension) ? win.devToolsExtension() : (f) => f, // Redux Devtools 开发者工具
);

export default createStore(reducer, {}, storeEnhancers); // Store Enhancers 能够让 createStore 函数产生的 Store 对象具有更多的功能
```
## 单个组件的性能优化
更改 shouldComponentUpdate 函数的默认实现，根据每个 React 组件的内在逻辑定制其行为
```js
shouldComponentUpdate(nextProps, nextState) {
  // 假设影响渲染内容的 prop 只有 completed 和 text，只需要确保
  // 这两个 prop 没有变化，函数就可以返回 false
  return (nextProps.completed !== this.props.completed) ||
    (nextProps.text !== this.props.text)
}
```
## 多个组件的性能优化（这部分主要介绍了虚拟 DOM 的原理）
用户操作引发界面的更新并不会让 React 生成的虚拟 DOM 推倒重来，React 在更新阶段巧妙地对比原有的 Virtual DOM 和新生成的 Virtual DOM，找出两者的不同之处，根据不同来修改 DOM 树，这样就只需要做最小的必要改动。

这个“找不同”的过程，就叫做 Reconciliation（调和）。

React 对比两个 Virtual DOM 的树形结构时，从根节点开始递归往下比对，在树形结构上，每个节点都可以看作这个节点以下部分子树的根节点。所以这个比对算法可以从 Virtual DOM上任何一个节点开始执行。

React 首先检查两个树形结构的根节点的类型是否相同，根据相同或者不同有不同处理方式：

**1.节点类型不同的情况**

直接扔掉原来的，构建新的 DOM 树，原有的树形结构上的 React 组件会经历“卸载”的生命周期，而取而代之的组件会经历“装载”的生命周期
```html
<div>
    <Todos />
</div>

// 我们想要更新成这样
<span>
    <Todos />
</span>
```
比如上面的代码在比较时，根节点类型不一样，一切推倒重来，重新构建一个 span 节点及其子节点

作为开发者，需要避免这种浪费的情景出现（div 和 span 的子节点是一样的）

**2.节点类型相同的情况**

如果两个树形结构的根节点类型相同，React 就认为原来的根节点只需要更新过程，不会将其卸载，也不会引发根节点的重新装载

这里有必要区分一下节点的类型：一类是 DOM 元素类型，一类是 React 组件；对于 DOM 元素类型， React 会保留节点对应的 DOM 元素，只对树形结构根节点上的属性和内容做一下比对，然后只更新修改的部分
```html
<div style={{color: 'red', fontSize: 15}} className="welcome">
  Hello World
</div>

// 改变之后的 JSX，React 可以对比发现内容和属性的变化，只修改这些变化的部分
<div style={{color: 'green', fontSize: 15}} className="farewell">
  Hello World
</div>
```
如果属性结构的根节点是 React 组件类型，React 能做的就是根据新节点的 props 取更新原来根节点的组件实例，即按顺序触发下列函数（旧生命周期）
- shouldComponentUpdate
- componentWillReceiveProps
- componentWillUpdate
- render
- componentDidUpdate

处理完根节点的对比之后，会对根节点的每个子节点重复一样的动作。

**3.多个子组件的情况**

当一个组件包含多个子组件的情况：
```html
<ul>
  <TodoItem text="First" completed={false} />
  <TodoItem text="Second" completed={false} />
</ul>

// 更新为
<ul>
  <TodoItem text="Zero" completed={false} />
  <TodoItem text="First" completed={false} />
  <TodoItem text="Second" completed={false} />
</ul>
```
直观上看，只需要创建一个新组件，更新之前的两个组件；但是实际情况并不是这样的，React 并没有找出两个序列的精确差别，而是直接挨个比较每个子组件。

在上面的新的 TodoItem 实例插入在第一位的例子中，React 会首先认为把 text 为 First 的 TodoItem 组件实例的 text 改成了 Zero，text 为 Second 的 TodoItem 组件实例的 text 改成了 First，在最后面多出了一个 TodoItem 组件实例。

这样的操作的后果就是，现存的两个实例的 text 属性被改变了，强迫它们完成了一个更新过程，创造出来的新的 TodoItem 实例用来显示 Second。

我们可以看到，理想情况下只需要增加一个 TodoItem 组件，但实际上其还强制引发了其他组件实例的更新。

假设有 100 个组件实例，那么就会引发 100 次更新，这明显是一个浪费；所以就需要开发人员在写代码的时候提供一点小小的帮助，这就是接下来要讲的 key 的作用

> 如果 React 采用的是先找出两个序列的差异的算法，时间是 O(N^2)，这不适合一个对性能要求很高的场景

## key 的作用
```html
<ul>
  <TodoItem key={1} text="First" completed={false} />
  <TodoItem key={2} text="Second" completed={false} />
</ul>

// 新增一个 TodoItem 实例
<ul>
  <TodoItem key={0} text="Zero" completed={false} />
  <TodoItem key={1} text="First" completed={false} />
  <TodoItem key={2} text="Second" completed={false} />
</ul>
```
React 根据 key 值，就可以知道现在的第二个和第三个 TodoItem 实例其实就是之前的第一个和第二个实例，所以 React 就会把新创建的 TodoItem 实例插在第一位，

对于原有的两个 TodoItem 实例只用原有的 props 来启动更新过程，这样 shouldComponentUpdate 就会发生作用，避免无谓的更新操作；

了解了这些之后，我们就知道 key 值应该是 **唯一** 且 **稳定不变的**

比如用数组下标值作为 key 就是一个典型的错误，看起来 key 值是唯一的，但是却不是稳定不变的

比如：[a, b, c] 值与下标的对应关系：a: 0 b:1 c:2

删除a -> [b, c] 值与下标的对应关系 b:0 c:1 

无法用 key 值来确定比对关系（新的 b 应该与旧的 b 比，如果按 key 值则是与 a 比）

<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/diveintoreact6.png" />


> 需要注意，虽然 key 是一个 prop，但是接受 key 的组件并不能读取到 key 的值，因为 key 和 ref 是 React 保留的两个特殊 prop，并没有预期让组件直接访问
