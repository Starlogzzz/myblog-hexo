---
title: React系列-3.Redux
date: 2020-10-22 08:58:05
tags:
  - React
categories: 
  - React
cover: https://s1.ax1x.com/2020/10/22/BiXq2T.jpg
abbrlink: 202020
---

## 介绍
目前我们使用js开发的程序，需要管理的状态越来越多了，就比如我做的网易云音乐web端项目，它在播放功能上需要管理数个状态来完成不同的功能，而且管理起来相当麻烦，因为它们之间会有依赖，一个状态改变会影响其他状态的值。并且我们无法得知什么时候改变了状态，是谁改变了状态，所以我们需要一个管理状态的工具来帮助我们完成这些我们很难完成的事，这就是Redux的目的。
安装redux: yarn add redux

## 三大原则
Redux 可以用这三个基本原则来描述：
1. 单一数据源：整个应用的 state 被储存在一棵 object tree 中，并且这个 object tree 只存在于唯一一个 store 中。
2. State 是只读的：唯一改变 state 的方法就是触发 action，action 是一个用于描述已发生事件的普通对象。
3. 使用纯函数来执行修改：为了描述 action 如何改变 state tree ，你需要编写 reducers。

## 核心概念
我们通常会创建一个store文件夹用来写Redux相关代码，如果一个应用中需要管理的状态太多，我们可以拆分reducer，接下来我们会谈到这一点。正常情况下我们的store下会有四个文件分别为：
+ actionCreators.js
+ constans.js
+ index.js
+ reducer.js

接下来就来介绍他们的作用。

### Action
Redux要求我们通过action来更新数据：
+ 所有数据的变化，必须通过派发（dispatch）action来更新
+ action是一个普通的JavaScript对象，用来描述这次更新的type和content
一般来说你会通过 store.dispatch() 将 action 传到 store。

```js actionCreater.js
import {
  ADD_NUMBER,
  SUB_NUMBER,
  INCREMENT,
  DECREMENT,
} from './constants.js';

export const addAction = num => ({
  type: ADD_NUMBER,
  num
});
export const subAction = num => ({
  type: SUB_NUMBER,
  num
});
export const incAction = () => ({
  type: INCREMENT
});
export const decAction = () => ({
  type: DECREMENT
});
```
这就是四个不同的action，起名时通常也给它们的结尾加上action表示他是一个action，对于type而言正常要求传入一个字符串来记录一下这个action的操作是什么或者描述一下它做了什么，这里我们通常习惯创建一个constans来保存type，以免我们写错type导致无法正常运行。
```js constans.js
export const ADD_NUMBER = "ADD_NUMBER";
export const SUB_NUMBER = "SUB_NUMBER";
export const INCREMENT = "INCREMENT";
export const DECREMENT = "DECREMENT";
```
### Reducer
Reducers 指定了应用状态的变化如何响应 actions 并发送到 store 的，记住 **actions 只是描述了有事情发生了这一事实，并没有描述应用如何更新 state**。
+ reducer是一个纯函数
+ reducer做的事情就是将传入的state和action结合起来生成一个新的state

```js reducer.js
import {
  ADD_NUMBER,
  SUB_NUMBER,
  INCREMENT,
  DECREMENT,
} from './constants.js';
const defaultState = { // 定义state初始值
  counter: 0
}
function reducer(state = defaultState, action) {
  switch (action.type) { // 注意：不要直接修改state
    case ADD_NUMBER:
      return { ...state, counter: state.counter + action.num };
    case SUB_NUMBER:
      return { ...state, counter: state.counter - action.num };
    case INCREMENT:
      return { ...state, counter: state.counter + 1 };
    case DECREMENT:
      return { ...state, counter: state.counter - 1 };
    default: // 在default情况下返回旧的state
      return state;
  }
}
export default reducer;
```
我们同样需要引入constants，reducer就是一个纯函数，接收旧的state和action，并返回新的state。reducer只要传入的参数相同，返回计算得到的下一个state就一定相同，单纯执行计算。
#### reducer拆分
上面也提到了我们会在某些情况下拆分reducer。
```js reducer.js
function reducer(state = {}, action) {
  return {
    counterInfo: countReducer(state.counterInfo, action),
    homeInfo: messageReducer(state.homeInfo, action)
  }
}
```
这里我们定义了一个reducer，它其实是countReducer和messageReducer的结合。我们再分别定义这两个reducer就可以了，这样我们就拆分了reducer，可以在小的reducer中管理不同类型或者是不同使用场景的state。
### Store
Store是用来把 action 和 reducer 联系到一起的对象，和vuex不同的是，应用中所有的 state 都以一个对象树的形式储存在一个单一的 store 中。当需要拆分数据处理逻辑时，你应该使用 reducer 组合 而不是创建多个 store。
```js index.js
import { createStore } from 'redux';
import reducer from "./reducer"
const store = createStore(reducer); // createStore接收两个参数，第一个参数是reducer，第二个参数是与中间件相关，接下来我们再讨论中间件，这里先略过
export default store;
```

## React中使用Redux
这里需要再强调一下：Redux和React之间没有关系。我们可以在任何地方使用Redux，包括Vue和Angular。但最好还是在React中使用Redux。yarn add react-redux。
我们需要知道应该怎么在页面中dispatch一个action来改变状态，react-redux可以帮助我们建立redux和组件的桥梁。
```js index.js
import React from 'react';
import ReactDOM from 'react-dom';
import App from "./App"

import {Provider} from "react-redux"
import store from "./store"

ReactDOM.render(
  <Provider store={store}>
  <App />
  </Provider>,
  document.getElementById('root'));
```
我们使用Provider包裹想用redux的组件，并把store传入其中，这样内部的组件就可以使用redux了。
接下来展示如何在组件中使用redux。
```jsx Home.js
import React, { PureComponent } from 'react';
import { connect } from 'react-redux'; // 引入connect
import axios from 'axios';
import {
  incAction,
  addAction,
  changeBannerAction,
  changeRecommendAction
} from '../store/actionCreators'
class Home extends PureComponent {
  constructor(props) {
    super(props)
  }
  componentDidMount() { // 发送网络请求，获取数据
    axios({
      url: "XXXXXXXXX",
    }).then(res => {
      const data = res.data.data
      this.props.changeBanners(data.banner.list) // 调用方法去dispatch action改变state
      this.props.changeRecommends(data.recommend.list) // 调用方法去dispatch action改变state
    })
  }
  render() {
    return (
      <div>
        <h1>Home</h1>
        <h2>当前计数: {this.props.counter}</h2>
        <button onClick={e => this.props.increment()}>+1</button> 
        <button onClick={e => this.props.addNumber(5)}>+5</button>
      </div>
    )
  }
}

const mapStateToProps = state => ({ // 返回一个对象
  counter: state.counter,
})

const mapDispatchToProps = dispatch => ({
  increment() {
    dispatch(incAction());
  },
  addNumber(num) {
    dispatch(addAction(num));
  },
  changeBanners(banners) {
    dispatch(changeBannerAction(banners))
  },
  changeRecommends(recommends) {
    dispatch(changeRecommendAction(recommends))
  }
})

export default connect(mapStateToProps, mapDispatchToProps)(Home); // 把state和dispatch传给Home
```
此处只是简单的请求到数据并dispatch，并没有在页面中展示（map的话代码太长了）。*connect*实际上是一个纯函数，返回值是一个函数，它内部会帮我们使用生命周期函数来订阅store和取消订阅，两个参数分别是两个函数，*connect*内部会向返回的函数内部调用这两个函数并传入store.getState()和store.dispatch。在返回的函数中，再把需要使用redux的组件传入（这里是Home）就可以在Home组件的props中拿到state和dispatch了。

## 中间件redux-thunk
首先我们要知道：**默认情况下的dispatch(action)，action需要是一个JavaScript的对象。**
实际开发中我们需要发送网络请求来获取数据，那么在redux中该怎么操作呢？实际上我们可以在dispatch一个action之前做这件事情，然后再去reducer中根据网络请求获取的数据去修改state，中间件就可以帮我们dispatch一个函数，用这个函数先来发送网络请求，拿到数据后再去dispatch一个真正的action。这样其实我们不仅可以发送网络请求，还可以做一些判断或者是扩展一些自己的代码。
### 使用方法
1. yarn add redux-thunk
2. 在创建store时传入应用了middleware的enhance函数
```js index.js
import {createStore,applyMiddleware,compose} from 'redux'; // 1.
import thunkMiddleware from "redux-thunk"; // 2.

import reducer from './reducer.js'; 

const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ || compose;
// redux-thunk传入一个中间件作为参数
const storeEnhancer = applyMiddleware(thunkMiddleware) // 3.
const store = createStore(reducer, composeEnhancers(storeEnhancer)); // 4.
export default store;
```
在index.js中实际上只比不使用中间件的代码多了这四步，这里顺便展示了如何开始React-devtools这个帮我们开发的工具。
3. 定义返回一个函数的action
前面也说了，我们需要dispatch一个函数，该函数在dispatch之后会被执行。
```js actionCreaters.js
export const getHomeMultidataAction = dispatch => {
  axios({ // 先发送网络请求
    url: "http://123.207.32.32:8000/home/multidata",
  }).then(res => {
    const data = res.data.data
    dispatch(changeBannerAction(data.banner.list)) // 再dispatch
    dispatch(changeRecommendAction(data.recommend.list))
  })
}
```
组件中使用的时候直接dispatch这个函数就可以了，挺简单的，没啥好说的。

## 总结
其实刚开始学习redux的时候我是一脸蒙蔽的，这东西对于我来说有点太原生了，也可能是之前使用Vuex给我惯出毛病了o(╥﹏╥)o，但看不懂的地方多看几遍，还是可以看懂并理解的。有了hooks之后，redux的使用相对来说也简单了许多，但该学还是要学的，不能偷懒。





