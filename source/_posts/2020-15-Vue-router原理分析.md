---
title: Vue-router原理分析
date: 2020-09-13 14:51:27
tags:
  - Vue
categories: 
  - Vue
cover: 
abbrlink: 202015
---

## 知识点
Vue Router 是 Vue.js 官方的路由管理器。它和 Vue.js 的核心深度集成，让构建单页面应用变得易如反掌。

使用步骤:
+ 安装vue-router
> npm install vue-router

+ 使用vue-router插件，router目录下index.js
```js
import Router from 'vue-router'
Vue.use(Router)
```

+ 创建router实例
```js
export default new Router({...})
```

+ 在根组件上添加该实例，main.js
```js
import router from './router'
new Vue({
  router,
}).$mount("#app");
```

+ 添加路由视图与导航，App.vue
```html
<router-link to="/">Home</router-link>
<router-link to="/about">About</router-link>
<router-view></router-view>
```

## 源码实现
### 需求分析
+ 作为一个插件存在：实现VueRouter类和install方法(use的时候会调用这个`VueRouter.install(Vue)`)
+ 实现两个全局组件：router-view用于显示匹配组件内容，router-link用于跳转
+ 监控url变化：监听hashchange或popstate事件
+ 响应最新url：创建一个响应式的属性current，当它改变时获取对应组件并显示

这里只是简单的实现以下vue-router的基本功能。

### 创建svue-router.js

```js
let _Vue; // 引用构造函数，VueRouter中要使用
// 保存选项
class VueRouter {
  constructor(options) {
    this.$options = options;
  }
}
// 插件：实现install方法，注册$router
VueRouter.install = function (Vue) {
  // 引用构造函数，VueRouter中要使用
  _Vue = Vue;
  // 1.挂载$router
  _Vue.mixin({
    beforeCreate() {
      // 只有根组件拥有router选项 
      if (this.$options.router) {
        // vm.$router 
        Vue.prototype.$router = this.$options.router;
      }
    }
  });
};
export default VueRouter;
```
> 为什么要用混入方式写？主要原因是use代码在前，Router实例创建在后，而install逻辑又需要用到该实例

### 实现router-link和router-view
在之前的install方法中加入
```js
// 定义两个全局组件router-link,router-view
Vue.component('router-link', {
  // template: '<a>'
  props: {
    // 接受传进来的to里的数据
    to: {
      type: String,
      require: true
    },
  },
  // 渲染成a标签
  render(h) {
    // <router-link to="/about">
    // <a href="#/about">xxx</a>
    // return <a href={'#'+this.to}>{this.$slots.default}</a>
    return h('a', {
      attrs: {
        href: '#' + this.to
      }
    }, this.$slots.default)
  }
})
Vue.component('router-view', {
  render(h) {
    // 找到当前url对应的组件
    const {routeMap, current} = this.$router
    const component = routeMap[current] ? routeMap[current].component : null
    // 渲染传入组件
    return h(component)
  }
})
```
对于router-view组件内部实现现在不需要看懂，接下来会说这部分，先跳过它。

### 监控URL变化
定义响应式的current属性，监听hashchange事件。在*class VueRouter*的*constructor*内部添加
```js
// 需要定义一个响应式的current属性
const initial = window.location.hash.slice(1) || '/'
_Vue.util.defineReactive(this, 'current', initial)

// 监控url变化，因为回调函数调用this会变化，所以使用bind固定一下
window.addEventListener('hashchange', this.onHashChange.bind(this))
window.addEventListener('load', this.onHashChange.bind(this))

// 获取目标URL
onHashChange() {
  // 只要#后面部分
  this.current = window.location.hash.slice(1)
  console.log(this.current);
}
```
这里我们使用vue内部的defineReactive给current添加响应式，这样我们的current改变就可以直接作出响应了。

### 处理路由表
我们不想每次在路由发生变化的时候遍历$options，所以这里创建一个路由表，只遍历一次即可。
还是在*class VueRouter*的*constructor*内部添加
```js
// 缓存path和route映射关系(路由映射表)
this.routeMap = {}
this.$options.routes.forEach(
  route => {
    this.routeMap[route.path] = route
  })
```

### 相应最新URL
现在我们可以继续完成router-view组件了。使用vue的响应式可以在url改变时界面重新渲染。
```js
Vue.component('router-view', {
  render(h) {
    // 找到当前url对应的组件
    const {routeMap, current} = this.$router
    const component = routeMap[current] ? routeMap[current].component : null
    // 渲染传入组件
    return h(component)
  }
})
```

### 完整代码
`svue-router.js`
```js
// 实现一个插件
// 返回一个函数
// 或者返回一个对象，他有一个install方法
let _Vue

class VueRouter {
  // 选项：routes - 路由表
  constructor(options) {
    this.$options = options

    // 缓存path和route映射关系(路由映射表)
    this.routeMap = {}
    this.$options.routes.forEach(
      route => {
        this.routeMap[route.path] = route
      })
    // console.log(route);
    
    // 需要定义一个响应式的current属性
    const initial = window.location.hash.slice(1) || '/'
    _Vue.util.defineReactive(this, 'current', initial)
    

    // 监控url变化
    window.addEventListener('hashchange', this.onHashChange.bind(this))
    window.addEventListener('load', this.onHashChange.bind(this))
  }

  onHashChange() {
    // 只要#后面部分
    this.current = window.location.hash.slice(1)
  }
}

VueRouter.install = function(Vue) {
  // 引用Vue构造函数，在上面VueRouter中使用
  _Vue = Vue

  // 1.挂载$router
  Vue.mixin({
    beforeCreate() {
      // 此处this指的是组件实例
      if (this.$options.router) {
        Vue.prototype.$router = this.$options.router
      }
    }
  })

  // 2.定义两个全局组件router-link,router-view
  Vue.component('router-link', {
    // template: '<a>'
    props: {
      to: {
        type: String,
        require: true
      },
    },
    render(h) {
      // <router-link to="/about">
      // <a href="#/about">xxx</a>
      // return <a href={'#'+this.to}>{this.$slots.default}</a>
      return h('a', {
        attrs: {
          href: '#' + this.to
        }
      }, this.$slots.default)
    }
  })
  Vue.component('router-view', {
    render(h) {
      // 找到当前url对应的组件
      const {routeMap, current} = this.$router
      const component = routeMap[current] ? routeMap[current].component : null
      // 渲染传入组件
      return h(component)
    }
  })
}
export default VueRouter
```
