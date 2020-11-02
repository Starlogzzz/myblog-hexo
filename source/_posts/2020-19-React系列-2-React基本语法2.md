---
title: React系列-2.React基本语法2
date: 2020-10-21 18:12:04
tags:
  - React
categories: 
  - React
cover: 
abbrlink: 202019
---

## 列表中key
key 帮助 React 识别哪些元素改变了，比如被添加或删除。因此你应当给数组中的每一个元素赋予一个确定的标识。
```jsx
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>
  <li key={number.toString()}>
    {number}
  </li>
);
```
> 通常我们从接口拿到的数据都是有一个独一无二的id的，但元素没有确定id的时候，万不得已你可以使用元素索引index作为 key。
> 如果列表项目的顺序可能会变化，我们不建议使用索引来用作key值，因为这样做会导致性能变差，还可能引起组件状态的问题。
> 如果你选择不指定显式的 key 值，那么 React 将默认使用索引用作为列表项目的 key 值。
## 事件总线
类似于Vue的时间总线，用来在全局进行简单的事件传递。
先 yarn add events
```jsx
import { EventEmitter } from "events"
const eventBus = new EventEmitter();

eventBus.emit("message", "啊哈哈哈哈", "嘻嘻嘻嘻") // 提交
eventBus.addListener("message", (ha, xi) => { // 监听
  console.log(ha, xi);
})
```
emit的第一个参数是type，接下来的参数是通过...args的方式引入的。
## ref
表示为对组件真正实例的引用，其实就是ReactDOM.render()返回的组件实例。ref可以挂载到组件上也可以是dom元素上。
+ 挂到组件(class声明的组件)上的ref表示对组件实例的引用。不能在函数式组件上使用 ref 属性，因为它们没有实例：
+ 挂载到dom元素上时表示具体的dom元素节点。

```jsx
import React, { PureComponent,createRef } from 'react'
export default class App extends PureComponent {
  constructor(props) {
    super(props)
    this.titleRef = createRef();
    this.titleEle = null;
  }
  render() {
    return (
      <div>
        {/* 方式一：字符串(不推荐) */}
        <h2 ref="dsm1">Hello dsm</h2>
        {/* 方式二：对象(推荐) */}
        <h2 ref={this.titleRef}>Hello dsm</h2>
        {/* 方式三：函数 */}
        <h2 ref={arg => {this.titleEle = arg}}>Hello dsm</h2>
        <button onClick={e => this.changeHtwo()}>修改</button>
      </div>
    )
  }
  changeHtwo() {
    this.refs.dsm1.innerHTML = "Hello 韩金龙";
    this.titleRef.current.innerHTML = "Hello 大司马";
    this.titleEle.innerHTML = "Hello 外币外币"
    console.log(this.titleEle);
  }
}
```
### ref转发
Ref 转发是一项将 ref 自动地通过组件传递到其一子组件的技巧。对于大多数应用中的组件来说，这通常不是必需的。
因为之前提到过，函数组件是没有实例的，但函数组件要是也想用ref要怎么做呢？答案是使用forwardRef。
引入``import React, { PureComponent, createRef, forwardRef } from 'react'``
```jsx
export default class App extends PureComponent {
  constructor(props) {
    super(props)
    this.AboutRef = createRef()
  }
  render() {
    return (
      <div>
        <Newabout ref={this.AboutRef}/>
      </div>
    )
  }
}

const Newabout = forwardRef(function(props, ref) {
  return <h2 ref={ref}>函数组件</h2>
})
```
这样ref就可以直接挂在子组件Newabout上了。
## 受控组件
在 HTML 中，表单元素（如input、 textarea 和 select）通常自己维护 state，并根据用户输入进行更新。而在 React 中，可变状态（mutable state）通常保存在组件的 state 属性中，并且只能通过使用 setState()来更新。
我们可以把两者结合起来，使 React 的 state 成为“唯一数据源”。渲染表单的 React 组件还控制着用户输入过程中表单发生的操作。被 React 以这种方式控制取值的表单输入元素就叫做“受控组件”。
```jsx
export default class App extends PureComponent {
  constructor(props) {
    super(props)
    this.state = {
      inputValue: ""
    }
  }
  render() {
    return (
      <div>
        <form onSubmit={e => this.handleSubmit(e)}>
          <label htmlFor="ainput">
            用户：
            {/* 受控组件 */}
            <input type="text" id="ainput" onChange={e => this.handleChange(e)} value={this.state.inputValue}/>
            <input type="submit"/>
          </label>
        </form>
      </div>
    )
  }
  handleSubmit(event) {
    event.preventDefault();
    console.log(this.state.inputValue);  
  }
  handleChange(event) {
    this.setState({
      inputValue: event.target.value
    })
  }
}
```
## 高阶组件
高阶组件（HOC）是 React 中用于复用组件逻辑的一种高级技巧。HOC 自身不是 React API 的一部分，它是一种基于 React 的组合特性而形成的设计模式。具体而言，高阶组件是参数为组件，返回值为新组件的函数。
```jsx
function enhanceComponent(WrappedComponent){
  return function NewComponent(props) {
    return <WrappedComponent {...props}/>
  }
}
```
这就是一个简单高阶组件，但这么看它其实没有任何作用，关于高阶组件的应用场景比如增强props，登录鉴权，劫持生命周期等。这里就不一一展开讨论。
## Portals
Portal 提供了一种将子节点渲染到存在于父组件以外的 DOM 节点的优秀的方案。
比如现在我们想写一个弹窗，点击某按钮就之后就显示这个组件，但我们的组件都会被渲染到``<div id="root"></div>``上的，这时我们就可以使用Protals来将他不置于这个div中。
```js
import { createPortal } from 'react-dom'; // 引入

class Father extends PureComponent {
  render() {
    return (
      <Son>
        <h2>这是测试文本</h2>
      </Son>
    )
  }
}
class Son extends PureComponent {
  render() {
    return createPortal(
      this.props.children,
      document.querySelector(".test")
    )
  }
}
```
之后我们还需要在index.html中创建一个id为test的div，这样我们这个h2就会被渲染到.test中而不是.root中，我们可以给.test添加样式来达到目的。
## 严格模式StrictMode
StrictMode 是一个用来突出显示应用程序中潜在问题的工具。与 Fragment 一样，StrictMode 不会渲染任何可见的 UI。它为其后代元素触发额外的检查和警告。只有在开发模式下才需要严格模式。
```jsx
<React.StrictMode>
  <div>
    <ComponentOne />
    <ComponentTwo />
  </div>
</React.StrictMode>
```
这样就启动了严格模式，内部的组件都会被“严格”的检查一下，包括他们的后代。
StrictMode主要用于：
+ 识别不安全的生命周期
+ 关于使用过时字符串 ref API 的警告
+ 关于使用废弃的 findDOMNode 方法的警告
+ 检测意外的副作用
+ 检测过时的 context API

## React中的css
React中写css一直是一个让人头疼的事情，至今为止也有很多解决方案，但也从来没有一种统一天下的方案，现在流行的包括：内联样式，普通css，module.css,css in js。这里我还是最喜欢使用css in js的方式，所以这里就只写css in js了。
这里使用styled-components来写css。yarn add styled-components
```js
import styled from "styled-components"
export const NewDiv = styled.div`
  color: red;
  .newdivh2{
    font-size: 60px;
  }
  .banner{
    background-color: blue;
  }
  span{
    font-size: 24px;
    color: #fff;
    &.active{  // 表示同时满足span和.active
      color: red;
    }
    &:hover{
      color: green;
    }
    &::after {
      content: "aaa";
    }
  }
`
export const TitleWrapper = styled.h2`
  text-decoration: underline;
  color: ${props => props.theme.themeColor};
  font-size: ${props => props.theme.fontSize};
`
```
需要使用样式的直接引入，像组件一样包裹一下。``<NewDiv></NewDiv>``内部就有这里定义的样式了，当然我们也可以像给组件传参数一样使用。








