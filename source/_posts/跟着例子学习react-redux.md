---
title: 跟着例子学习react-redux
date: 2018-09-16 23:30:42
tags: 
- react
---

# 跟着例子学习react-redux

本文通过一个简单的例子展开，一点点自己去实现一个redux+react-redux，让大家充分理解redux+react-redux出现的必要。

1. 这篇文章是第一次接触redux的我看过的，对于redux讲解得比较容易理解的
2. 其实redux的思想是一个管理状态的仓库，跟vuex相似


## 预备知识

在阅读本文之前，希望大家对以下知识点能提前有所了解：

1. 状态提升的概念
2. react高阶组件(函数)
3. es6基础
4. pure 组件(纯函数)
5. Dumb 组件

## React.js的context

> 此处介绍的特性，则类似vue组件中的provide、inject

这一节的内容其实是讲一个react当中一个你可能永远用不到的特性——context，但是它对你理解react-redux很有好处。那么context是干什么的呢？看下图：

![image](https://image-static.segmentfault.com/387/403/3874030634-5a671a9b94cdb_articlex)

假设现在这个组件树代表的应用是用户可以自主换主题色的，每个子组件会根据主题色的不同调整自己的字体颜色。
* “主题色”这个状态是所有组件共享的状态
* 根据状态提升中所提到的，需要把这个状态提升到根节点的 Index 上，然后把这个状态通过 props 一层层传递下去：

![image](https://image-static.segmentfault.com/369/330/3693305059-5a671b2d57224_articlex)

如果要改变主题色，在 Index 上可以直接通过 this.setState({ themeColor: 'red' }) 来进行。这样整颗组件树就会重新渲染，子组件也就可以根据重新传进来的 props.themeColor 来调整自己的颜色。

但这里的问题也是非常明显的，我们需要把 themeColor 这个状态一层层手动地从组件树顶层往下传，每层都需要写 props.themeColor。如果我们的组件树很层次很深的话，这样维护起来简直是灾难。

如果这颗组件树能够全局共享这个状态就好了，我们要的时候就去取这个状态，不用手动地传：

![image](https://image-static.segmentfault.com/492/913/492913461-5a672ebaf0e27_articlex)

就像这样，Index 把 state.themeColor 放到某个地方，这个地方是每个 Index 的子组件都可以访问到的。当某个子组件需要的时候就直接去那个地方拿就好了，而不需要一层层地通过 props 来获取。不管组件树的层次有多深，任何一个组件都可以直接到这个公共的地方提取 themeColor 状态。

React.js 的 context 就是这么一个东西，某个组件只要往自己的 context 里面放了某些状态，这个组件之下的所有子组件都直接访问这个状态而不需要通过中间组件的传递。一个组件的 context 只有它的子组件能够访问。

---

下面我们看看 React.js 的 context 代码怎么写，我们先把整体的组件树搭建起来。
用create-react-app创建工程：

```
create-react-app react-redux-demo1
```

现在我们修改 App，让它往自己的 context 里面放一个 themeColor：

```
import React, { Component } from 'react';
import PropTypes from 'prop-types';
import Header from './header';
import Main from './main';
import './App.css';

class App extends Component {
  static childContextTypes = {
    themeColor :PropTypes.string
  }
  constructor () {
    super()
    this.state = {
      themeColor : 'red'
    }
  }
  getChildContext () {
    return {
      themeColor : this.state.themeColor
    }
  }
  render () {
    return (
      <div>
        <Header />
        <Main />
      </div>
    )
  }
}

export default App;
```

构造函数里面的内容其实就很好理解，就是往 state 里面初始化一个 themeColor 状态。getChildContext 这个方法就是设置 context 的过程，它返回的对象就是 context（也就是上图中处于中间的方块），所有的子组件都可以访问到这个对象。我们用 this.state.themeColor 来设置了 context 里面的 themeColor。

接下来我们要看看子组件怎么获取这个状态，修改 App 的孙子组件 Title和Content：
```
-- title.js --

class Title extends Component {
  static contextTypes = {
    themeColor: PropTypes.string
  }

  render () {
    return (
      <h1 style={{ color: this.context.themeColor }}>React.js 小书标题</h1>
    )
  }
}
```

```
-- content.js --

import React, { Component } from 'react';
class Content extends Component {
    render () {
        return (
        <div>
            <h2>this is 内容</h2>
        </div>
        )
    }
}

export default Content;
```

一个组件可以通过 getChildContext 方法返回一个对象，这个对象就是子树的 context，提供 context 的组件必须提供 childContextTypes 作为 context 的声明和验证。

如果一个组件设置了 context，那么它的子组件都可以直接访问到里面的内容，它就像这个组件为根的子树的全局变量。任意深度的子组件都可以通过 contextTypes 来声明你想要的 context 里面的哪些状态，然后可以通过 this.context 访问到那些状态。

#### 利与弊

context 打破了组件和组件之间通过 props 传递数据的规范，极大地增强了组件之间的耦合性。而且，就如全局变量一样，context 里面的数据能被随意接触就能被随意修改，每个组件都能够改 context 里面的内容会导致程序的运行不可预料。

## 动手实现Redux

上节内容讲了React.js的content的特性，这个跟redux和react-redux什么关系呢？看下去就知道了，这边先卖个关子：）。Redux 和 React-redux 并不是同一个东西。Redux 是一种架构模式（Flux 架构的一种变种），它不关注你到底用什么库，你可以把它应用到 React 和 Vue，甚至跟 jQuery 结合都没有问题。而 React-redux 就是把 Redux 这种架构模式和 React.js 结合起来的一个库，就是 Redux 架构在 React.js 中的体现。

### “大张旗鼓”的修改共享状态

用 create-react-app 新建一个项目：react-redux-demo2：

```
create-react-app react-redux-demo2
```

修改 public/index.html 里面的 body 结构为：

```
<body>
    <div id='title'></div>
    <div id='content'></div>
</body>
```

删除 src/index.js 里面所有的代码，添加下面代码，代表我们应用的状态：

```
const appState = {
  title: {
    text: 'this is title',
    color: 'red',
  },
  content: {
    text: 'this is content',
    color: 'blue'
  }
}
```

我们新增几个渲染函数，它会把上面状态的数据渲染到页面上：

```
function renderApp (appState) {
  renderTitle(appState.title)
  renderContent(appState.content)
}

function renderTitle (title) {
  const titleDOM = document.getElementById('title')
  titleDOM.innerHTML = title.text
  titleDOM.style.color = title.color
}

function renderContent (content) {
  const contentDOM = document.getElementById('content')
  contentDOM.innerHTML = content.text
  contentDOM.style.color = content.color
}
renderApp(appState)
```

很简单，renderApp 会调用 rendeTitle 和 renderContent，而这两者会把 appState 里面的数据通过原始的 DOM 操作更新到页面上

![image](https://image-static.segmentfault.com/343/493/3434932651-5a682819003c7_articlex)

这是一个很简单的 App，但是它存在一个重大的隐患，我们渲染数据的时候，使用的是一个共享状态 appState，每个人都可以修改它。

**这里的矛盾就是：“模块（组件）之间需要共享数据”，和“数据可能被任意修改导致不可预料的结果”之间的矛盾。**

为了解决这个问题，我们可以学习 React.js 团队的做法，把事情搞复杂一些，提高数据修改的门槛：模块（组件）之间可以共享数据，也可以改数据。但是我们约定，这个数据并不能直接改，你只能执行某些我允许的某些修改，而且你修改的必须大张旗鼓地告诉我。

> 这里类似vuex的commit

我们定义一个函数，叫 dispatch，它专门负责数据的修改：

```
function dispatch (action) {
  switch (action.type) {
    case 'UPDATE_TITLE_TEXT':
      appState.title.text = action.text
      break
    case 'UPDATE_TITLE_COLOR':
      appState.title.color = action.color
      break
    default:
      break
  }
}
```

所有对数据的操作必须通过 dispatch 函数。它接受一个参数 action，这个 action 是一个普通的 JavaScript 对象，里面必须包含一个 type 字段来声明你到底想干什么。dispatch 在 swtich 里面会识别这个 type 字段，能够识别出来的操作才会执行对 appState 的修改。

任何的模块如果想要修改 appState.title.text，必须大张旗鼓地调用 dispatch：
```
dispatch({ type: 'UPDATE_TITLE_TEXT', text: 'this is dispatch' }) // 修改标题文本
dispatch({ type: 'UPDATE_TITLE_COLOR', color: 'blue' }) // 修改标题颜色
```

我们再也不用担心共享数据状态的修改的问题，我们只要把控了 dispatch，所有的对 appState 的修改就无所遁形，毕竟只有一根箭头指向 appState 了。

![image](https://image-static.segmentfault.com/437/167/437167931-5a6834213e323_articlex)

### 构建共享状态仓库

上面代码有个问题，就是每次dispatch修改数据的时候，其实只是数据发生了变化，如果我们不手动调用renderApp，页面不会发生变化。如何数据变化的时候程序能够智能一点地自动重新渲染数据，而不是手动调用？

往 dispatch里面加 renderApp 就好了，但是这样 createStore 就不够通用了。我们希望用一种通用的方式“监听”数据变化，然后重新渲染页面，这里要用到观察者模式。修改 createStore：

```
function createStore (state, stateChanger) {
  const listeners = []
  const subscribe = (listener) => listeners.push(listener)
  const getState = () => state
  const dispatch = (action) => {
    stateChanger(state, action)
    listeners.forEach((listener) => listener())
  }
  return { getState, dispatch, subscribe }
}
```

我们在 createStore 里面定义了一个数组 listeners，还有一个新的方法 subscribe，可以通过 store.subscribe(listener) 的方式给 subscribe 传入一个监听函数，这个函数会被 push 到数组当中。每当 dispatch 的时候，监听函数就会被调用，这样我们就可以在每当数据变化时候进行重新渲染：

```
const store = createStore(appState, stateChanger)
store.subscribe(() => renderApp(store.getState()))

renderApp(store.getState()) // 首次渲染页面
store.dispatch({ type: 'UPDATE_TITLE_TEXT', text: 'this is dispatch' }) // 修改标题文本
store.dispatch({ type: 'UPDATE_TITLE_COLOR', color: 'blue' }) // 修改标题颜色
// ...后面不管如何 store.dispatch，都不需要重新调用 renderApp
```

### 共享结构的对象来提高性能

其实我们之前的例子当中是有比较严重的性能问题的。我们在每个渲染函数的开头打一些 Log 看看：

```
function renderApp (appState) {
  console.log('render app...')
  renderTitle(appState.title)
  renderContent(appState.content)
}

function renderTitle (title) {
  console.log('render title...')
  const titleDOM = document.getElementById('title')
  titleDOM.innerHTML = title.text
  titleDOM.style.color = title.color
}

function renderContent (content) {
  console.log('render content...')
  const contentDOM = document.getElementById('content')
  contentDOM.innerHTML = content.text
  contentDOM.style.color = content.color
}
```

依旧执行一次初始化渲染，和两次更新，这里代码保持不变：

```
renderApp(store.getState()) // 首次渲染页面
store.dispatch({ type: 'UPDATE_TITLE_TEXT', text: 'this is dispatch' }) // 修改标题文本
store.dispatch({ type: 'UPDATE_TITLE_COLOR', color: 'blue' }) // 修改标题颜色
```

![image](https://image-static.segmentfault.com/217/791/2177914753-5a6837d4b6b0d_articlex)


可以看到问题就是，每当更新数据就重新渲染整个 App，但其实我们两次更新都没有动到 appState 里面的 content 字段的对象，而动的是 title 字段。其实并不需要重新 renderContent，它是一个多余的更新操作，现在我们需要优化它。

这里提出的解决方案是，在每个渲染函数执行渲染操作之前先做个判断，判断传入的新数据和旧的数据是不是相同，相同的话就不渲染了。

```
function renderApp (newAppState, oldAppState = {}) { // 防止 oldAppState 没有传入，所以加了默认参数 oldAppState = {}
    if (newAppState === oldAppState) return // 数据没有变化就不渲染了
      console.log('render app...')
  renderTitle(newAppState.title, oldAppState.title)
  renderContent(newAppState.content, oldAppState.content)
}

function renderTitle (newTitle, oldTitle = {}) {
  if (newTitle === oldTitle) return // 数据没有变化就不渲染了
  console.log('render title...')
  const titleDOM = document.getElementById('title')
  titleDOM.innerHTML = newTitle.text
  titleDOM.style.color = newTitle.color
}

function renderContent (newContent, oldContent = {}) {
  if (newContent === oldContent) return // 数据没有变化就不渲染了
  console.log('render content...')
  const contentDOM = document.getElementById('content')
  contentDOM.innerHTML = newContent.text
  contentDOM.style.color = newContent.color
}
```

然后我们用一个 oldState 变量保存旧的应用状态，在需要重新渲染的时候把新旧数据传进入去：

```
const store = createStore(appState, stateChanger)
let oldState = store.getState() // 缓存旧的 state
store.subscribe(() => {
  const newState = store.getState() // 数据可能变化，获取新的 state
  renderApp(newState, oldState) // 把新旧的 state 传进去渲染
  oldState = newState // 渲染完以后，新的 newState 变成了旧的 oldState，等待下一次数据变化重新渲染
})
...
function stateChanger (state, action) {
  switch (action.type) {
    case 'UPDATE_TITLE_TEXT':
      state.title.text = action.text
      break
    case 'UPDATE_TITLE_COLOR':
      state.title.color = action.color
      break
    default:
      break
  }
}
...
```

其实上面一顿操作根本达不到我们的预期的要求，你会发现还是渲染了content，这些引用指向的还是原来的对象，只是对象内的内容发生了改变。所以即使你在每个渲染函数开头加了那个判断又什么用？就像下面这段代码一样自欺欺人：

```
let people = {
    name:'ddvdd'
}
const oldPeople = people
people.name = 'yjy'
oldPeople !== people //false 其实两个引用指向的是同一个对象，我们却希望它们不同。
```

那怎么样才能达到我们要的要求呢？引入共享结构的对象概念：

```
const obj = { a: 1, b: 2}
const obj2 = { ...obj } // => { a: 1, b: 2 }
```

const obj2 = { ...obj } 其实就是新建一个对象 obj2，然后把 obj 所有的属性都复制到 obj2 里面，相当于对象的浅复制。上面的 obj 里面的内容和 obj2 是完全一样的，但是却是两个不同的对象。除了浅复制对象，还可以覆盖、拓展对象属性：

```
const obj = { a: 1, b: 2}
const obj2 = { ...obj, b: 3, c: 4} // => { a: 1, b: 3, c: 4 }，覆盖了 b，新增了 c
```

我们可以把这种特性应用在 appstate 的更新上，我们禁止直接修改原来的对象，一旦你要修改某些东西，你就得把修改路径上的所有对象复制一遍。我们修改 stateChanger，让它修改数据的时候，并不会直接修改原来的数据 state，而是产生上述的共享结构的对象：

```
function stateChanger (state, action) {
  switch (action.type) {
    case 'UPDATE_TITLE_TEXT':
      return { // 构建新的对象并且返回
        ...state,
        title: {
          ...state.title,
          text: action.text
        }
      }
    case 'UPDATE_TITLE_COLOR':
      return { // 构建新的对象并且返回
        ...state,
        title: {
          ...state.title,
          color: action.color
        }
      }
    default:
      return state // 没有修改，返回原来的对象
  }
}
```

因为 stateChanger 不会修改原来对象了，而是返回对象，所以我们需要修改一下 createStore。让它用每次 stateChanger(state, action) 的调用结果覆盖原来的 state：

```
function createStore (state, stateChanger) {
  const listeners = []
  const subscribe = (listener) => listeners.push(listener)
  const getState = () => state
  const dispatch = (action) => {
    state = stateChanger(state, action) // 覆盖原对象
    listeners.forEach((listener) => listener())
  }
  return { getState, dispatch, subscribe }
}
```

好了，我们在运行下看看结果是不是变成我们预期的那样了？

![image](https://image-static.segmentfault.com/149/440/1494404945-5a683dd527888_articlex)

### 我就喜欢叫它 “reducer”

经过了这么多节的优化，我们有了一个很通用的 createStore，主要传入appState、stateChanger就能使用。那么appState和stateChanger是否可以合并到一起去呢？显然可以：

```
function stateChanger (state, action) {
  if (!state) {
    return {
      title: {
        text: 'this is title',
        color: 'red',
      },
      content: {
        text: 'this is content',
        color: 'blue'
      }
    }
  }
  switch (action.type) {
    case 'UPDATE_TITLE_TEXT':
      return {
        ...state,
        title: {
          ...state.title,
          text: action.text
        }
      }
    case 'UPDATE_TITLE_COLOR':
      return {
        ...state,
        title: {
          ...state.title,
          color: action.color
        }
      }
    default:
      return state
  }
}
```

stateChanger 现在既充当了获取初始化数据的功能，也充当了生成更新数据的功能。如果有传入 state 就生成更新数据，否则就是初始化数据。这样我们可以优化 createStore 成一个参数，因为 state 和 stateChanger 合并到一起了：

```
function createStore (stateChanger) {
  let state = null
  const listeners = []
  const subscribe = (listener) => listeners.push(listener)
  const getState = () => state
  const dispatch = (action) => {
    state = stateChanger(state, action)
    listeners.forEach((listener) => listener())
  }
  dispatch({}) // 初始化 state
  return { getState, dispatch, subscribe }
}
```

createStore 内部的 state 不再通过参数传入，而是一个局部变量 let state = null。createStore 的最后会手动调用一次 dispatch({})，dispatch 内部会调用 stateChanger，这时候的 state 是 null，所以这次的 dispatch 其实就是初始化数据了。createStore 内部第一次的 dispatch 导致 state 初始化完成，后续外部的 dispatch 就是修改数据的行为了。

我们给 stateChanger 这个玩意起一个通用的名字：reducer，不要问为什么，它就是个名字而已，修改 createStore 的参数名字：
```
function createStore (reducer) {
  let state = null
  const listeners = []
  const subscribe = (listener) => listeners.push(listener)
  const getState = () => state
  const dispatch = (action) => {
    state = reducer(state, action)
    listeners.forEach((listener) => listener())
  }
  dispatch({}) // 初始化 state
  return { getState, dispatch, subscribe }
}
```

这是一个最终形态的 createStore，它接受的参数叫 reducer，reducer 是一个函数，细心的朋友会发现，它其实是一个纯函数（Pure Function）

## Redux在React当中的实践

...【后续过几天补充

> refer: https://segmentfault.com/a/1190000012976767