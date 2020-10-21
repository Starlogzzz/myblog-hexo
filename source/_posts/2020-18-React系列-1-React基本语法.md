---
title: React系列-1.React基本语法
date: 2020-10-21 14:27:25
tags:
  - React
categories: 
  - React
cover: 
abbrlink: 202018
---

## 前言
开学之后校园封闭管理，自己不能出去实习，在学校也没什么课，所以自己用了一个月的时间学习了一下React，这个系列可以说是我学完React之后的回顾，回忆并记录一下React相关的知识点。废话少说进入正题。

## 特点
React的特点如下：
+ 声明式：React 使创建交互式 UI 变得轻而易举。为你应用的每一个状态设计简洁的视图，当数据改变时 React 能有效地更新并正确地渲染组件。
+ 组件化：组件逻辑使用 JavaScript 编写而非模版，因此你可以轻松地在应用中传递数据，并使得状态与 DOM 分离。
+ 一次学习，随处编写：无论你现在正在使用什么技术栈，你都可以随时引入 React 来开发新特性，而不需要重写现有代码。React 还可以使用 Node 进行服务器渲染，或使用 React Native 开发原生移动应用。

## 依赖
React开发依赖三个库：react，react-dom，babel。
```html
  <script src="../react/react.development.js"></script>
  <script src="../react/react-dom.development.js"></script>
  <script src="../react/babel.min.js"></script>
```
当然你不想下载的话也可以引入他们的cdn。他们的作用分别为：
+ react：包含react所必须的核心代码
+ react-dom：react渲染在不同平台所需要的核心代码
+ babel：将jsx转换成React代码的工具

## 组件化
React的组件可以分为函数式组件和类组件，在hooks问世之前我们常常使用类组件，但类组件存在各种难以理解的问题，hooks出现之后使用函数式组件+hooks的开发方式大大减少了学习React的成本，但类组件也是一个很重要的知识点。
### 类组件
```jsx
class App extends React.Component {
  constructor(props){
    super(props);
    this.state = {
      message: "Hello React"
    }
  render(){
    return (
      <div>
        <h2>{this.state.message}</h2>
        <button onClick={this.btnClick.bind(this)}>改变文本</button>
      </div>
    )
  btnClick(){
    this.setState({
      message: "hello 修改咯"
    })
  }
}
```
这里定义了一个App组件(**注意：React中组件首字母必须大写，内部必须有一个render函数返回一个渲染的内容，若返回多组内容则需使用一个标签进行包裹，通常使用div，或者Fragement，<></>**)，点击button会改变h2的内容。
### 函数式组件
```jsx
function App() {
  function sayHello() {
    console.log("Hello")
  }
  return (
    <div>
      <button>打招呼</button>
    </div>
  )
}
```
函数式组件就是一个简单的函数，名字首字母同样要大写，与类组件不同的是它没有state，同时因为它是一个函数所以没有this。也没有生命周期。
### 生命周期
生命周期函数是类组件特有的，它可以在对应的时机被调用，常用的生命周期有：
```js
componentDidMount() { // 组件已经被渲染到DOM中后运行
  console.log("生命周期componentDidMount回调函数");
}
componentDidUpdate() { // 组件更新(重新渲染)后执行
  console.log("生命周期componentDidUpdate回调函数");
}
componentWillUnmount() { // 组件销毁前执行
  console.log("生命周期componentWillUnmount回调函数");
}
```
### 组件通信
#### 父传子
```jsx
  <ChildCpn name="韩金龙" age="18" />
```
子组件从props中拿到对应属性(通常使用解构直接拿出)``const {name, age} = props``。
#### 属性验证
先yarn add prop-types一下。
```jsx
import { PropTypes } from 'prop-types';

ChildCpn.propTypes = {
  name: PropTypes.string.isRequired, // isRequired:必须传入
  age: PropTypes.number,
  height: PropTypes.number,
  names: PropTypes.array
}
ChildCpn.defaultProps = { // 默认值
  name: "GZC",
  age: 20
}
```
组件*ChildCpn*验证父组件传来的值。
#### 子传父
其实和父传子差不多，但是父组件下方一个函数给子组件，子组件接受函数并调用函数传入一个参数，父组件就可以拿到子组件想给父组件传递的值，也就是函数的参数。比如：
```jsx
<TabControl title={this.title} changeName={index => this.changeName(index)}/>
```
给TabControl传递一个changeName的函数，子组件在合适的时机调用这个函数并把index作为参数传递过来就可以了。这里使用箭头函数来给changeName绑定this。
#### 跨组件通信
使用context进行跨组件通信。
```jsx
const MyContext = React.createContext({ // 定义初始值
  name: "gzc",
  age: 20
})
Profile.contextType = MyContext; // 声明Profile组件可以拿父组件传递过去的value
// 父组件: 这里把state传给子组件
export default class App extends Component {
  constructor(props) {
    super(props);
    this.state = {
      nickname: "许昊龙",
      level: 0
    }
  }
  render() {
    return (
      <div>
        <MyContext.Provider value={this.state}>
          <Profile/>
        </MyContext.Provider>
      </div>
    )
  }
}
// 类组件的子组件直接使用this.context.xxx就可以拿到对应的数据
// 函数组件的子组件
function Profile() {
  return (
    <MyContext.Consumer>
      {
        value => {
          return (
            <div>
              <h2>用户昵称: {value.nickname}</h2>
              <h2>用户等级: {value.level}</h2>
            </div>
          )
        }
      }
    </MyContext.Consumer>
  )
}
```
## State
不要直接修改 State, 这样代码不会重新渲染组件。
```jsx
// Wrong
this.state.comment = 'Hello';
// Correct
this.setState({
  comment: "Hello"
})
```
### state数据不可变
因为更改state会重新渲染页面，所以我们有时并不希望一些无关的state修改也会重新渲染页面，所以就出现了**shouldComponentUpdate**和**PureComponent**，关于他俩可以阅读 (官方文档)[https://zh-hans.reactjs.org/docs/optimizing-performance.html#shouldcomponentupdate-in-action] 这里不展开讨论，但它们两个都会对比新旧state的值从而判断要不要重新渲染，所以我们不能直接更改state中引用类型的值。
```jsx
addMember() {
  const peo = {name: "侯国玉", age: 20}
  const newFriends = [...this.state.friends];
  newFriends.push(peo)
  this.setState({
    friends: newFriends
  })
}
```
其实就是绕了个小弯。
### setState是异步的？
出于性能优化的考虑，多次更新state的话，会在一个时间点一次性更新state，而不是每一次都直接更新，所以setState是异步更新的
```jsx
changeMessage() {
  this.setState({
    message: "起飞"
  }, () => {
    console.log(this.state.message); // 新值："起飞"
  })
  console.log(this.state.message); // 原来的message的值
}
```
给一个button的onClick绑定这个方法，setState的第二个参数会在执行之后被回调，这里会发现message的值确实被修改了但直接打印的message却是之前的值，因为setState是异步的。

那setState就一直是异步的吗?并不是，这两种情况的setState都是同步的：
+ 将setState放入到定时器中
+ 使用原生dom调用setState
### setState的合并
```jsx
addCount() {
  // setState本身的合并(只有最后一个生效，会覆盖)
  this.setState({
    count: this.state.count + 1
  });
  this.setState({
    count: this.state.count + 2
  });
  this.setState({
    count: this.state.count + 3
  });

  // setState的数据的合并(全部生效)
  this.setState((prevState, props) => {
    return {
      count: prevState.count + 1
    }
  })
  this.setState((prevState, props) => {
    return {
      count: prevState.count + 1
    }
  })
  this.setState((prevState, props) => {
    return {
      count: prevState.count + 3
    }
  })
}
```





