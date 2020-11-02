---
title: React系列-4.React-router
date: 2020-10-22 16:19:01
tags:
  - React
categories: 
  - React
cover: https://s1.ax1x.com/2020/10/22/BFmvLj.md.png
abbrlink: 202021
---

## 安装
同Vue Router一样React Router也是前端路由。
React Router的版本4开始，路由不再集中在一个包中进行管理了：
+ react-router是router的核心部分代码
+ react-router-dom是用于浏览器的
+ react-router-native是用于原生应用的

安装react-router-dom会自动帮助我们安装react-router的依赖，yarn add react-router-dom。

## 基本使用
react-router-dom给我们提供一些组件api，来实现某些功能。
```js
import {
  BrowserRouter,
  Route,
  NavLink,
  Switch
} from "react-router-dom"
```
直接从react-router-dom这个地方引入他们。

### BrowserRouter或HashRouter
+ Router中包含了对路径改变的监听，并且会将相应的路径传递给子组件
+ BrowserRouter使用history模式
+ HashRouter使用hash模式

```jsx App.js
import React from 'react';
import ReactDOM from 'react-dom';

import App from "./App"
import { BrowserRouter } from 'react-router-dom/cjs/react-router-dom.min';

ReactDOM.render(
  <BrowserRouter>
    <App />
  </BrowserRouter>,
  document.getElementById('root')
);
```
这样我们的App内部就可以使用history模式的react router了。要想使用hash模式替换成HashRouter即可。

### Link和NavLink
+ 通常路径的跳转是使用Link组件，最终会被渲染成a元素
+ NavLink是在Link基础之上增加了一些样式属性
+ to属性：Link中最重要的属性，用于设置跳转到的路径

```jsx
<NavLink exact to="/" activeClassName="active-link" activeStyle={{color: "red"}}>首页</NavLink>
<NavLink to="/about" activeClassName="active-link">关于</NavLink>
<NavLink to="/profile" activeClassName="active-link">我的</NavLink>
```
这里提供了三个NavLink，其实就相当于三个a标签，其他常用属性：
+ **exact**：表示精确匹配，比如/about也会被/来匹配到，有时会跳转不正确，所以要在首页或基础路径上添加exact属性。
+ **activeStyle**：可以让我们给NavLink添加活跃时的样式，但并不推荐。
+ **activeClassName**: 可以给NavLink添加一个活跃时的类名，活跃时就是当我们处在这个链接时的状态。我们可以添加css来设置活跃时下这个NavLink的样式。
默认的activeClassName：事实上在默认匹配成功时，NavLink就会添加上一个动态的active class。所以我们也可以直接编写样式。

### Route
+ Route用于路径的匹配
+ path属性：用于设置匹配到的路径
+ component属性：设置匹配到路径后，渲染的组件
+ exact：同上，只有精准匹配到完全一致的路径，才会渲染对应的组件

实际上Link和Route就相当于Vue的router-link和router-view。
```jsx
<Route exact path="/" component={Home}/>
<Route path="/about" component={About}/>
<Route path="/profile" component={Profile}/>
<Route path="/:id" component={User}/>
```
第四个的:id表示动态路由。

## Redirect
Redirect表示重定向，我们希望一些页面需要登录才能访问，这时就可以使用Redirect来将未登录的用户重定向到登录界面
```jsx 组件中render函数
render() {
  return this.state.isLogin? (
    <div>
      <h2>Hello Starlog</h2>
    </div>
  ):<Redirect to="/Login"/>
}
```
就比如我们在state中设置一个isLogin属性，如果是true的话就放行，是false的话就重定向到/Login。

## Switch
首先我们要知道不设置path属性时，这个组件会直接被渲染，或者说是所有路径都可以进行匹配。
```jsx
<Route path="/profile" component={Profile}/>
<Route path="/:id" component={User}/>
<Route component={NoMatch}/>
```
当我们访问/profile时实际上这三个路由都会被匹配上，因为这都满足他们的匹配条件，但我们并不希望这样，这时我们就可以使用Switch来进行精确匹配。
```jsx
<Switch>
  <Route path="/profile" component={Profile}/>
  <Route path="/:id" component={User}/>
  <Route component={NoMatch}/>
</Switch>
```
这样就会进行精确匹配了。

## 路由嵌套
直接在组件中正常使用就可以实现路由嵌套了，没什么好说的。

## 手动跳转
使用react router渲染出来的组件props中都有history这个对象，内部有一些可以跳转路由的方法供我们使用。
```jsx
<button onClick={e => this.JustJump()}>跳转</button>

JustJump() {
  console.log(this.props.history)
  this.props.history.push("/jump")
}
```
意思是这里我们定义一个button，手动调用一个方法JustJump，内部我们可以使用history下的一些方法进行跳转(这里使用的是push)，当然这样做的前提是这个组件是由react router渲染出来的，不然props下根本不会有history这个对象。

那如果我们希望在不被react router渲染的页面跳转呢？这时候就要用到withRouter这个高阶组件了。
```jsx
import { withRouter } from 'react-router-dom'
export default withRouter(App)
```
只需要导出组件的时候调用一下withRouter，并把要添加属性的组件传入就可以了。**注意：所使用的的组件在外部必须使用BrowserRouter或HashRouter标签包裹一下。**

## 参数传递
传递参数有三种方式：
+ 动态路由的方式； （不推荐）
+ search传递参数； （不推荐）
+ Link中to传入对象；（推荐）

### 动态路由
上面代码中也出现过动态路由，这里来解释一下，动态路由的概念指的是路由中的路径并不会固定。
+ 比如/detail的path对应一个组件Detail
+ 如果我们将path在Route匹配时写成/detail/:id，那么 /detail/abc、/detail/123都可以匹配到该Route，并且进行显示 
+ 这个匹配规则，我们就称之为动态路由
+ 通常情况下，使用动态路由可以为路由传递参数。

上面提到了使用react router渲染的页面的props中会有一个location对象，除此之外还有一个**match**对象，match对象其实就是存储一些有关url的参数，其中params就是我们使用动态路由传递的参数。

### search
```jsx
<NavLink to={`/detail2?name="Starlog"&age=21`} activeClassName="active-link">详情2</NavLink>
```
这里其实还是用props下面的一个对象，这个对象叫做**location**对象，其中**location.search**就是传递过来的参数。但我们还需要对其做一些处理，不太方便。

### Link中to传递
Link中的to可以传递一个对象过去，例如：
```jsx
<NavLink to={{
  pathname: "/detail3",
  search: "name=wuhu&age=21",
  state: info // 此处传入一个对象(info)为参数
}} activeClassName="active-link">详情3</NavLink>
```
参数解释：
+ pathname: A string representing(代表) the path to link to.
+ search: A string representation of query parameters.(在url后面拼接的参数)
+ hash: A hash to put in the URL, e.g. #a-hash.(hash类型的时候传入hash)
+ state: State to persist to the location.(保存一个状态到location)

到时候通过location就可以拿到对应参数传递的值了。

## react-router-config
我们还是希望有一种更简便的配置路由的方法，比如vue router中的配置。实际上我们可以使用react-router-config来像vue一样配置路由。yarn add react-router-config
```js router文件夹下index.js
const routes = [
  {
    path: "/",
    exact: true,
    component: Home
  },
  {
    path: "/about",
    component: About,
    routes: [
      {
        path: "/about",
        exact: true,
        component: AboutComponent
      },
      {
        path: "/about/money",
        component: AboutMoneyComponent
      },
      {
        path: "/about/future",
        component: AboutFutureComponent
      },
      {
        path: "/about/join",
        component: JoinToComponent
      }
    ]
  },
  {
    path: "/profile",
    component: Profile
  },
  {
    path: "/user",
    component: User
  }
]
export default routes
```
在页面中使用需要使用renderRoutes。
```jsx
import { renderRoutes } from 'react-router-config';
import routes from './router/index';
// 在原本<Switch></Switch>标签及内容的部分替换为
{renderRoutes(routes)}
```
通过renderRoutes的方式渲染的页面的props下会有一个**route**属性，route下会有一个routes属性，储存嵌套路由的地址。



