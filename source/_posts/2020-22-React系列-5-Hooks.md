---
title: React系列-5.Hooks
date: 2020-10-23 09:29:27
tags:
  - React
categories: 
  - React
cover: https://s1.ax1x.com/2020/10/23/Bk4dz9.png
abbrlink: 202022
---

## 前言
这应该React基础系列最后一部分内容了，写完这篇也就暂时告别React了，也算是基本掌握了React的使用方法了。
Hook是React 16.8的新增特性，它可以让我们在不编写class的情况下使用state以及其他的React特性（比如生命周期）。
首先我们要明白函数式组件和class组件的区别，class组件有自己的state，有生命周期，这些都是函数式组件没有的。
而且class组件本身要是很复杂的话，比如componentDidMount中需要发送网络请求或监听某事件，再在componentWillUnmount中移除，这样的代码看起来是很难受的，也很难去拆分。同时class组件内部的this也很难理解，时不时就会出现问题。而且类似于使用Provider、Consumer来共享一些状态，但是多次使用Consumer时，我们的代码就会存在很多嵌套，变得很复杂。

Hooks的出现就解决了这些问题，它可以让我们在使用函数组件的同时使用state等class组件的特性，可以说是取其精华去其糟粕。Hook只能在函数组件中使用，不能在类组件，或者函数组件之外的地方使用，接下来就开始一一列举常用的hook。

Hook 就是 JavaScript 函数，这个函数可以帮助你 钩入（hook into） React State以及生命周期等特性。

**但是使用它们会有两个额外的规则：** 
+ 只能在函数最外层调用 Hook。不要在循环、条件判断或者子函数中调用。
+ 只能在 React 的函数组件中调用 Hook。不要在其他 JavaScript 函数中调用。

## useState
顾名思义，就是让我们可以使用state，直接上代码
```jsx
import React, { useState } from 'react'
export default function MultiHookState() {
  const [name, setName] = useState("芜湖大司马")
  const [age, setAge] = useState(18);
  const [friend, setFriend] = useState(["PDD","Wh1t3zz"])
  const [students, setStudents] = useState([
    { id: 110, name: "badwoman", age: 20 },
    { id: 111, name: "sabsertion", age: 30},
    { id: 112, name: "shoemaker", age: 22 }
  ])
  return (
    <div>
      <h2>当前名称：{name}</h2>
      <h2>当前年龄：{age}</h2>
      <ul>
      {
        friend.map((item,index) => {
          return <li key={index}>{item}</li>
        })
      }
      </ul>
    </div>
  )
}
```
useState是一个数组，我们可以通过数组的解构，来完成赋值会非常方便。数组内第一个值为当前状态的值（第一调用为初始化值），第二个参数为改变 state 用的函数， useState 参数为初始化值，如果不设置为undefined。它与 class 里面的 this.state 提供的功能完全相同。一般来说，在函数退出后变量就会消失，而 state 中的变量会被 React 保留。
setState的时候会再次执行const [state, setState] = useState()，只不过数据发生了变化，该变量与先前的变量无关。所以const的值才会“改变”。

## useEffect
useEffect 可以让你来完成一些类似于class中生命周期的功能。
+ 通过useEffect的Hook，可以告诉React需要在渲染后执行某些操作；
+ useEffect要求我们传入一个回调函数，在React执行完更新DOM操作之后，就会回调这个函数；
  + 参数一：执行的回调函数；
  + 参数二：该useEffect在哪些state发生变化时，才重新执行（受谁的影响，相当于componentDidupdate），如果一个函数我们不希望依赖任何的内容时，也可以传入一个空的数组(相当于componentDidMount),
+ 默认情况下，无论是第一次渲染之后，还是每次更新之后，都会执行这个回调函数；

```jsx
import React, { useState, useEffect } from 'react'

export default function HooksChangeTitle() {
  const [counter, setCounter] = useState(0)
  useEffect(() => { // 因为改变state会重新渲染页面，所以会执行操作修改title
    document.title = counter;
  }, [counter]) // counter变化才会重新执行
  useEffect(() => { // 渲染页面之后立即订阅
    console.log("订阅一些事件");

    return () => { // 更新之后回调这个函数取消订阅
      console.log("取消订阅");
    }
  }, [])
  return (
    <div>
      <h2>当前计数：{counter}</h2>
      <button onClick={e => setCounter(counter + 1)}>+1</button>
    </div>
  )
}
```
这个demo会在计数+1时修改title为计数的值，使用useState和useEffect可以轻松的完成。而且我们还模拟了一下之前我们在生命周期函数中完成的订阅和取消订阅。

## useContext
还是看名字就知道是干嘛的，useContext是让我们使用Context的，以前我们在组件中使用Context很复杂，类组件的话需要 类名.contextType = MyContext来使用Context。函数式组件需要我们使用MyContext.Consumer包裹内容。用起来还是有些复杂的。而且函数式组件多个Context共享时的方式会存在大量的嵌套，难上加难。
useContext允许我们通过Hook来直接获取某个Context的值。
```jsx App.js
export const PersonContext = createContext();
export const ThemeContext = createContext();

<PersonContext.Provider value={{name: "wuhudsm", age: 32}}>
  <ThemeContext.Provider value={{color: "red", fontSize: "30px"}}>
    <ContextHook/>
  </ThemeContext.Provider>
</PersonContext.Provider>
```
```jsx ContextHook.js
import React, { useContext } from 'react'
import {
  PersonContext,
  ThemeContext
} from "../App"
export default function ContextHook() {
  const user = useContext(PersonContext) // 直接拿到Context
  const theme = useContext(ThemeContext)
  return (
    <div>
      <h2>ContextHook</h2>
    </div>
  )
}
```

## useReducer
useState 的替代方案。它接收一个形如 (state, action) => newState 的 reducer，并返回当前的 state 以及与其配套的 dispatch 方法。（如果你熟悉 Redux 的话，就已经知道它如何工作了。） state 逻辑较复杂且包含多个子值，或者下一个 state 依赖于之前的 state 等。并且，使用 useReducer 还能给那些会触发深更新的组件做性能优化，因为你可以向子组件传递 dispatch 而不是回调函数 。
```jsx
import React, { useReducer } from 'react'

export default function Home() {
  function reducer(state, action) {
    switch(action.type) {
      case "increment":
        return {...state, counter: state.counter + 1}
      case "decrement":
        return {...state, counter: state.counter - 1}
      default:
        return state;
    }
  }
  const [state, dispatch] = useReducer(reducer, {counter: 0}); // 第二个参数给state设置初始值

  return (
    <div>
      <h2>当前计数：{state.counter}</h2>
      <button onClick={e => dispatch({type : "increment"})}>+1</button>
      <button onClick={e => dispatch({type : "decrement"})}>-1</button>
    </div>
  )
}
```

## useCallback
把内联回调函数及依赖项数组作为参数传入 useCallback，它将返回该回调函数的**记忆值**，该回调函数仅在某个依赖项改变时才会更新。当你把回调函数传递给经过优化的并使用引用相等性去避免非必要渲染（例如 shouldComponentUpdate，memo）的子组件时，它将非常有用。
使用场景: **在将一个组件中的函数, 传递给子元素进行回调使用时, 使用useCallback对函数进行处理。**
```jsx
import React, {useState, useCallback, memo} from 'react';

const ZCButton = memo((props) => { // 子元素回调使用父组件中的函数increment
  console.log("HYButton重新渲染: " + props.title);
  return <button onClick={props.increment}>HYButton +1</button>
});

export default function CallbackHookDemo02() {
  console.log("CallbackHookDemo02重新渲染");
  const [count, setCount] = useState(0);
  const [show, setShow] = useState(true);
  const increment1 = () => {  // 会重新渲染
    console.log("执行increment1函数");
    setCount(count + 1);
  }
  const increment2 = useCallback(() => { // 不会重新渲染
    console.log("执行increment2函数");
    setCount(count + 1);
  }, [count]);
  return (
    <div>
      <h2>CallbackHookDemo01: {count}</h2>
      <ZCButton title="btn1" increment={increment1}/>
      <ZCButton title="btn2" increment={increment2}/>
      <button onClick={e => setShow(!show)}>show切换</button> 
    </div>
  )
}
```
点击show切换按钮之后只有increment1会被重新渲染，increment2因为使用了useCallback保留了上次的返回值，memo进行浅层比较时发现没有变化所以不会重新渲染。

## useMemo
返回一个**记忆值**，把“创建”函数和依赖项数组作为参数传入 useMemo，它仅会在某个依赖项改变时才重新计算记忆值。这种优化有助于避免在每次渲染时都进行高开销的计算。如果没有提供依赖项数组，useMemo 在每次渲染时都会计算新的值。
```jsx
import React, { memo, useState } from 'react'

const Info = memo(function(props) {
  console.log("Info重新渲染")
  console.log(props)
  return <h2>名字：{props.info.name}</h2>
})
export default memo(function TestuseMemo() {
  const [show, setShow] = useState(true);
  console.log("TestuseMemo重新渲染")

  const name = {name: "starlog"}; // 因为show切换之后页面重新渲染，name也会被重新定义，所以给Info传递的name始终不相同，memo进行浅层比较时就会重新渲染Info
  return (
    <div>
      <Info info={name}/> 
      <button onClick={e => setShow(!show)}>show切换</button>
    </div>
  )
})
```
这里即使show的切换和Info组件一点关系没有并且我们使用memo包裹了Info，但show的改变还是会重新渲染Info，使用useMemo就可以解决这个问题。仅需将name使用useMemo定义一下即可。
```jsx
const name = useMemo(() => {
  return {name: "starlog"}
}, []);
```
name谁也不依赖，页面重新渲染的时候它不会重新被定义，所以Info处的memo就会浅层比较不进行重新渲染。
与useCallback不同的是，useCallback对返回值只能是函数，只对函数进行优化，而useMemo对返回值进行优化，返回值可以是任何东西，我们可以用useMemo实现useCallback。

## useRef
useRef返回一个ref对象，返回的ref对象在组件的整个生命周期保持不变。
最常用的useRef两种用法：
1. 引入DOM（或者组件，但是需要是class组件）元素； 
2. 保存一个数据，这个对象在整个生命周期中可以保存不变；
```jsx 简化代码
import React, { useRef, PureComponent, forwardRef } from 'react'

class RefCpn1 extends PureComponent {
  render() {
    return <h2>RefCpn1</h2>
  }
}
const RefCpn2 = forwardRef(function(props, ref) {
  return <h2 ref={ref}>RefCpn2</h2>
})

// 组件内部
const inputRef = useRef();
const refCpn1 = useRef();
const refCpn2 = useRef();

<input ref={inputRef} type="text"/>
<RefCpn1 ref={refCpn1}/>
<RefCpn2 ref={refCpn2}/>
```
第一种用法挺简单的，但注意函数式组件要使用forwardRef包裹一下。
```jsx
import React, { useRef, useState, useEffect } from 'react'

export default function RefHookDemo02() {
  const [count, setCount] = useState(0)
  const numRef = useRef(count);

  useEffect(() => {
    numRef.current = count;
  }, [count])
  return (
    <div>
      <h2>count上一次的值：{numRef.current}</h2>
      <h2>count这一次的值：{count}</h2>
      <button onClick={e => setCount(count+10)}>+10</button>
    </div>
  )
}
```
第二种用法其实也没啥好说的，就是这个使用useRef保存的值在组件整个生命周期中是不会变化的。

## useImperativeHandle
回忆一下刚才提及的useRef和forwardRef的使用，我们可以得知，父组件其实是拿到了整个绑定ref的子组件的实例，但我们可能仅仅是希望暴露给父组件一些特定的操作(比如input的focus)，父组件其实可以肆意修改我们的子组件，这样的设计是有问题的。
通过useImperativeHandle的Hook，将传入的ref和useImperativeHandle第二个参数返回的对象绑定到了一起，所以在父组件中，使用 inputRef.current时，实际上使用的是返回的对象，比如我调用了 focus函数，甚至可以调用 printHello函数。
[![BEiDrn.png](https://s1.ax1x.com/2020/10/23/BEiDrn.png)](https://imgchr.com/i/BEiDrn)
有点乱，但应该可以看懂。

## useLayoutEffect
useLayoutEffect看起来和useEffect非常的相似，事实上他们也只有一点区别而已，useEffect会在渲染的内容更新到DOM上后执行，不会阻塞DOM的更新，而useLayoutEffect会在渲染的内容更新到DOM上之前执行，会阻塞DOM的更新，如果我们希望在某些操作发生之后再更新DOM，那么应该将这个操作放到useLayoutEffect。
```jsx
useLayoutEffect(()=>{
  if(count === 0){
    setCount(Math.random())
  }
},[count])
```
用的并不多，知道即可。

## 自定义Hook
这里不在做讲解，直接看代码就能看懂
(案例)[https://github.com/Starlogzzz/learn-react/tree/master/13_learn_hooks/src/11-%E8%87%AA%E5%AE%9A%E4%B9%89hook]
(自定义的hook)[https://github.com/Starlogzzz/learn-react/tree/master/13_learn_hooks/src/hooks]

## react-redux中hooks
### useDispatch
```jsx
import { useDispatch } from 'react-redux';
const dispatch = useDispatch();
useEffect(() => {
  // test
  dispatch(getNewAlbumAction(10)) // 直接可以使用dispatch
}, [dispatch]);
```

### useSelector
判断是否重新渲染时使用==对返回的对象进行比较。所以每次对比都不相同，要传入shallowEqual进行浅层比较。
```jsx
const { newAlbums } = useSelector(state => ({
  newAlbums: state.getIn(["recommend", "newAlbums"]) // 这里使用了immutableJS
}), shallowEqual); // 进行浅层比较
```
可以直接拿到我们的state，再也不用像以前那样折磨的写redux了。