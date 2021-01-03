---
title: 2021-1-vue源码之new Vue做了什么
date: 2021-01-01 10:13:30
tags:
  - vue源码
categories: 
  - vue源码
cover: https://s3.ax1x.com/2020/11/20/DQwbfs.md.jpg
abbrlink: 202101
---

# 前言
大家元旦快乐，转眼一看我已经一个月没写博客了，因为最近作为实习生入职了京东，没有大块的时间去做知识梳理，趁着这次假期，我来总结一下` new Vue `这个过程。

# _init
我们都知道` new Vue `的时候我们会去调用一个 `_init` 的方法，这个方法就是在` initMixin `中定义的，而` _init `方法主要就是去**合并选项，初始化生命周期，初始化事件，初始化渲染**等等，最后会去调用 `$mount` 我们也就先不仔细研究 `_init` 这个过程，接下来去看` $mount `的过程。

# 挂载$mount
接下来我们来分析一下挂载的这个流程，也就是 $mount

文件位置在 `src/platforms/web/entry-runtime-with-compiler` 
```js
// 保存原来的$mount
const mount = Vue.prototype.$mount
// 覆盖默认$mount
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && query(el)

  /* istanbul ignore if */
  if (el === document.body || el === document.documentElement) {
    process.env.NODE_ENV !== 'production' && warn(
      `Do not mount Vue to <html> or <body> - mount to normal elements instead.`
    )
    return this
  }

  // 解析option
  // render>template>el
  const options = this.$options
  // resolve template/el and convert to render function
  if (!options.render) {
    let template = options.template
    // 膜版解析
    if (template) {
      if (typeof template === 'string') {
        if (template.charAt(0) === '#') {
          template = idToTemplate(template)
          /* istanbul ignore if */
          if (process.env.NODE_ENV !== 'production' && !template) {
            warn(
              `Template element not found or is empty: ${options.template}`,
              this
            )
          }
        }
      } else if (template.nodeType) {
        template = template.innerHTML
      } else {
        if (process.env.NODE_ENV !== 'production') {
          warn('invalid template option:' + template, this)
        }
        return this
      }
    } else if (el) {
      template = getOuterHTML(el)
    }
    // 如果模板存在，执行编译
    if (template) {
      /* istanbul ignore if */
      if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
        mark('compile')
      }

      // 得到渲染函数
      const { render, staticRenderFns } = compileToFunctions(template, {
        outputSourceRange: process.env.NODE_ENV !== 'production',
        shouldDecodeNewlines,
        shouldDecodeNewlinesForHref,
        delimiters: options.delimiters,
        comments: options.comments
      }, this)
      options.render = render
      options.staticRenderFns = staticRenderFns

      /* istanbul ignore if */
      if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
        mark('compile end')
        measure(`vue ${this._name} compile`, 'compile', 'compile end')
      }
    }
  }
  // 执行挂载
  return mount.call(this, el, hydrating)
}
```
它内部去从 runtime/index.js 中拿到 $mount 并把他作为 mount 保存起来，然后自己定义一个新的 $mount 因为这和文件的作用有关，我们现在所在的是runtime with compiler，而 runtime/index 中的 $mount 是 runtimeOnly 所用的。
接下来我们会看到一些逻辑的判断，它会去查看我们的 el 是不是 body,html 这些标签，如果是就发出警告。
然后我们要去解析模板，优先级是按照 render > template > el来进行的，我们的 vue 最终只会根据 render 进行渲染，所以说 template 和 el 其实都会被转换成 render。
接着我们直接跳到最后 return 的部分，我们可以看到它调用了我们在 runtime/index 中定义的 mount 并返回结果。接下来我们来看一下它。

## runtime/index 中的 mount
```js
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && inBrowser ? query(el) : undefined
  // 初始化，将首次渲染结果替换el
  return mountComponent(this, el, hydrating)
}
```
我们就看最后 return 的 mountComponent 方法就可以了，它是定义在 `src/core/instance/lifecycle` 中的。

## mountComponent
经常犯的一个错误就是我们是用 runtimeOnly 的版本但没写 render 这就会发出警告，或者是我们用的是编译的版本，但将 template 转换成 render 的过程发生了错误，这里也会报错。
```js
export function mountComponent (
  vm: Component,
  el: ?Element,
  hydrating?: boolean
): Component {
  vm.$el = el
  // 如果没有正确的生成render的话
  if (!vm.$options.render) {
    vm.$options.render = createEmptyVNode
    if (process.env.NODE_ENV !== 'production') {
      /* istanbul ignore if */
      if ((vm.$options.template && vm.$options.template.charAt(0) !== '#') ||
        vm.$options.el || el) {
        warn(
          'You are using the runtime-only build of Vue where the template ' +
          'compiler is not available. Either pre-compile the templates into ' +
          'render functions, or use the compiler-included build.',
          vm
        )
      } else {
        warn(
          'Failed to mount component: template or render function not defined.',
          vm
        )
      }
    }
  }
  callHook(vm, 'beforeMount')

  let updateComponent
  /* istanbul ignore if */
  // 性能埋点
  if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
    updateComponent = () => {
      const name = vm._name
      const id = vm._uid
      const startTag = `vue-perf-start:${id}`
      const endTag = `vue-perf-end:${id}`

      mark(startTag)
      const vnode = vm._render()
      mark(endTag)
      measure(`vue ${name} render`, startTag, endTag)

      mark(startTag)
      vm._update(vnode, hydrating)
      mark(endTag)
      measure(`vue ${name} patch`, startTag, endTag)
    }
  } else {
    updateComponent = () => {
      // 参数1是vnode，参数2是和服务端渲染相关的东西
      vm._update(vm._render(), hydrating)
    }
  }

  // we set this to vm._watcher inside the watcher's constructor
  // since the watcher's initial patch may call $forceUpdate (e.g. inside child
  // component's mounted hook), which relies on vm._watcher being already defined
  // 渲染watcher
  new Watcher(vm, updateComponent, noop, {
    before () {
      if (vm._isMounted && !vm._isDestroyed) {
        callHook(vm, 'beforeUpdate')
      }
    }
  }, true /* isRenderWatcher */)
  hydrating = false

  // manually mounted instance, call mounted on self
  // mounted is called for render-created child components in its inserted hook
  if (vm.$vnode == null) {
    vm._isMounted = true
    callHook(vm, 'mounted')
  }
  return vm
}
```
再说一下 new Watcher 这个部分，这里是 new 一个渲染 Watcher 出来，可以看到它传入了一个 true 作为参数，我们再看一下 watcher 的定义，在 `observer/watcher` 中
```js
constructor (
    vm: Component,
    expOrFn: string | Function,
    cb: Function,
    options?: ?Object,
    isRenderWatcher?: boolean
  )
```
这里先只看它的 constructor ，可以看到第五个参数就是判断这个 watcher 是不是一个渲染 watcher 用的，我们传入 true。Watcher 在这里起到两个作用，一个是初始化的时候会执行回调函数，另一个是当 vm 实例中的监测的数据发生变化的时候执行回调函数。
其实在 Wathcer 内部会调用一个 get 方法，而 get 方法内部会去执行 `value = this.getter.call(vm, vm)`，这里就相当于调用我们在 new Watcher 中传入的 updateComponent ，也就是这个函数
```js
updateComponent = () => {
  // 参数1是vnode，参数2是和服务端渲染相关的东西
  vm._update(vm._render(), hydrating)
}
```
我们可以看到内部主要是执行了` _render和_update `这两个函数，这两个函数就是最后挂载到真实 DOM 上的逻辑。

## 总结
$mount 就是在没有定义 render 的情况下，先去把 el, template 转换为 render，然后就会调用 mountComponent 这个方法，这个方法就是定义了 updateComponent 这个函数，这个函数其实是一个渲染 watcher，因为实际上它进行了一次渲染，之后的更新我们也会执行渲染。

# _render
Vue 的 `_render` 方法是实例的一个私有方法，它用来把实例渲染成一个虚拟 Node。它的定义在 `src/core/instance/render.js` 文件中
```js
Vue.prototype._render = function (): VNode {
    const vm: Component = this
    const { render, _parentVnode } = vm.$options

    if (_parentVnode) {
      vm.$scopedSlots = normalizeScopedSlots(
        _parentVnode.data.scopedSlots,
        vm.$slots,
        vm.$scopedSlots
      )
    }

    // set parent vnode. this allows render functions to have access
    // to the data on the placeholder node.
    vm.$vnode = _parentVnode
    // render self
    let vnode
    try {
      // There's no need to maintain a stack because all render fns are called
      // separately from one another. Nested component's render fns are called
      // when parent component is patched.
      currentRenderingInstance = vm
      // 调用render
      vnode = render.call(vm._renderProxy, vm.$createElement)
    } catch (e) {
      handleError(e, vm, `render`)
      // return error render result,
      // or previous vnode to prevent render error causing blank component
      /* istanbul ignore else */
      if (process.env.NODE_ENV !== 'production' && vm.$options.renderError) {
        try {
          vnode = vm.$options.renderError.call(vm._renderProxy, vm.$createElement, e)
        } catch (e) {
          handleError(e, vm, `renderError`)
          vnode = vm._vnode
        }
      } else {
        vnode = vm._vnode
      }
    } finally {
      currentRenderingInstance = null
    }
    // if the returned array contains only a single node, allow it
    if (Array.isArray(vnode) && vnode.length === 1) {
      vnode = vnode[0]
    }
    // return empty vnode in case the render function errored out
    if (!(vnode instanceof VNode)) {
      if (process.env.NODE_ENV !== 'production' && Array.isArray(vnode)) {
        warn(
          'Multiple root nodes returned from render function. Render function ' +
          'should return a single root node.',
          vm
        )
      }
      vnode = createEmptyVNode()
    }
    // set parent
    vnode.parent = _parentVnode
    return vnode
  }
```
代码很长，最后返回 vnode，在其中我们会调用 render 方法生成 vnode， 接下来分析一下这个过程`  vnode = render.call(vm._renderProxy, vm.$createElement) `我们可以看到传入了两个参数，分别是当前上下文，和 $createElement

## _renderProxy
在` src/core/instance/init `中，我们其实做过一个判断
```js
    if (process.env.NODE_ENV !== 'production') {
      initProxy(vm)
    } else {
      vm._renderProxy = vm
    }
```
如果是生产环境，那么` _renderProxy `其实就是当前 vm，而在开发环境中，我们需要 initProxy 一下。

### initProxy
在` src/core/instance/proxy `中我们会做一个判断，看浏览器是否支持 proxy 这个语法，支持的话就 new Proxy，不支持直接返回 vm
```js
initProxy = function initProxy (vm) {
    // 浏览器支持proxy就new 一个
    if (hasProxy) {
      // determine which proxy handler to use
      const options = vm.$options
      const handlers = options.render && options.render._withStripped
        ? getHandler
        : hasHandler
      vm._renderProxy = new Proxy(vm, handlers)
    } else {
      vm._renderProxy = vm
    }
  }
```

## $createElement
在 initRender 中，我们定义了` vm.$createElement = (a, b, c, d) => createElement(vm, a, b, c, d, true) `

## createElement
我们可以看到在 $createElement 中我们调用了 createElement ，它的位置在 `src/core/vdom/create-element.js` 中
```js
// wrapper function for providing a more flexible interface
// without getting yelled at by flow
export function createElement (
  context: Component,
  tag: any,
  data: any,
  children: any,
  normalizationType: any,
  alwaysNormalize: boolean
): VNode | Array<VNode> {
  if (Array.isArray(data) || isPrimitive(data)) {
    normalizationType = children
    children = data
    data = undefined
  }
  if (isTrue(alwaysNormalize)) {
    normalizationType = ALWAYS_NORMALIZE
  }
  return _createElement(context, tag, data, children, normalizationType)
}
```
这里实际上就是对传入参数做一个调整，如果没传 data 的话，那么将 data 设置为 undefined 。
然后我们在第六个参数传入 true 的情况下，会给 normalizationType 赋值为 2 。
然后去调用 _createElement ，实际上这里就是做一个 _createElement 的扩展。

## _createElement
`_createElement` 方法有 5 个参数:
+ context 表示 VNode 的上下文环境，它是 Component 类型；
+ tag 表示标签，它可以是一个字符串，也可以是一个 Component；
+ data 表示 VNode 的数据，它是一个 VNodeData 类型，可以在 flow/vnode.js 中找到它的定义，这里先不展开说；
+ children 表示当前 VNode 的子节点，它是任意类型的，它接下来需要被规范为标准的 VNode 数组；
+ normalizationType 表示子节点规范的类型，类型不同规范的方法也就不一样，它主要是参考 render 函数是编译生成的还是用户手写的。

```js
export function _createElement (
  context: Component,
  tag?: string | Class<Component> | Function | Object,
  data?: VNodeData,
  children?: any,
  normalizationType?: number
): VNode | Array<VNode> {
  if (isDef(data) && isDef((data: any).__ob__)) {
    process.env.NODE_ENV !== 'production' && warn(
      `Avoid using observed data object as vnode data: ${JSON.stringify(data)}\n` +
      'Always create fresh vnode data objects in each render!',
      context
    )
    return createEmptyVNode() // 创建一个注释节点
  }
  // object syntax in v-bind
  if (isDef(data) && isDef(data.is)) {
    tag = data.is
  }
  if (!tag) {
    // in case of component :is set to falsy value
    return createEmptyVNode() 
  }
  // warn against non-primitive key
  if (process.env.NODE_ENV !== 'production' &&
    isDef(data) && isDef(data.key) && !isPrimitive(data.key)
  ) {
    if (!__WEEX__ || !('@binding' in data.key)) {
      warn(
        'Avoid using non-primitive value as key, ' +
        'use string/number value instead.',
        context
      )
    }
  }
  // support single function children as default scoped slot
  if (Array.isArray(children) &&
    typeof children[0] === 'function'
  ) {
    data = data || {}
    data.scopedSlots = { default: children[0] }
    children.length = 0
  }
  if (normalizationType === ALWAYS_NORMALIZE) {
    children = normalizeChildren(children)
  } else if (normalizationType === SIMPLE_NORMALIZE) {
    children = simpleNormalizeChildren(children)
  }

  // 根据tag类型做相应处理
  let vnode, ns
  if (typeof tag === 'string') {
    let Ctor
    ns = (context.$vnode && context.$vnode.ns) || config.getTagNamespace(tag)
    // 判断是否保留标签
    if (config.isReservedTag(tag)) {
      // platform built-in elements
      if (process.env.NODE_ENV !== 'production' && isDef(data) && isDef(data.nativeOn)) {
        warn(
          `The .native modifier for v-on is only valid on components but it was used on <${tag}>.`,
          context
        )
      }
      vnode = new VNode(
        config.parsePlatformTagName(tag), data, children,
        undefined, undefined, context
      )
    } else if ((!data || !data.pre) && isDef(Ctor = resolveAsset(context.$options, 'components', tag))) {
      // component
      // resolveAsset获取components中对应的组件构造函数
      // 自定义组件走这里
      vnode = createComponent(Ctor, data, context, children, tag)
    } else {
      // unknown or unlisted namespaced elements
      // check at runtime because it may get assigned a namespace when its
      // parent normalizes children
      vnode = new VNode(
        tag, data, children,
        undefined, undefined, context
      )
    }
  } else {
    // direct component options / constructor
    vnode = createComponent(tag, data, context, children)
  }
  if (Array.isArray(vnode)) {
    return vnode
  } else if (isDef(vnode)) {
    if (isDef(ns)) applyNS(vnode, ns)
    if (isDef(data)) registerDeepBindings(data)
    return vnode
  } else {
    return createEmptyVNode()
  }
}
```
这里主要看一下 children 的规范化这一部分
```js
  if (normalizationType === ALWAYS_NORMALIZE) {
    children = normalizeChildren(children)
  } else if (normalizationType === SIMPLE_NORMALIZE) {
    children = simpleNormalizeChildren(children)
  }
```
我们会走上面的 if 逻辑，去调用 normalizeChildren 返回children，接下来我们来看一下 normalizeChildren 函数。

+ simpleNormalizeChildren 方法调用场景是 render 函数是编译生成的。理论上编译生成的 children 都已经是 VNode 类型的，但这里有一个例外，就是 functional component 函数式组件返回的是一个数组而不是一个根节点，所以会通过 Array.prototype.concat 方法把整个 children 数组打平，让它的深度只有一层。
+ normalizeChildren 方法的调用场景有 2 种，一个场景是 render 函数是用户手写的，当 children 只有一个节点的时候，Vue.js 从接口层面允许用户把 children 写成基础类型用来创建单个简单的文本节点，这种情况会调用 createTextVNode 创建一个文本节点的 VNode；另一个场景是当编译 slot、v-for 的时候会产生嵌套数组的情况，会调用 normalizeArrayChildren 方法

## normalizeArrayChildren
```js
function normalizeArrayChildren (children: any, nestedIndex?: string): Array<VNode> {
  const res = []
  let i, c, lastIndex, last
  for (i = 0; i < children.length; i++) {
    c = children[i]
    if (isUndef(c) || typeof c === 'boolean') continue
    lastIndex = res.length - 1
    last = res[lastIndex]
    //  nested
    if (Array.isArray(c)) { // 是数组递归扁平化
      if (c.length > 0) {
        c = normalizeArrayChildren(c, `${nestedIndex || ''}_${i}`)
        // merge adjacent text nodes
        if (isTextNode(c[0]) && isTextNode(last)) {
          res[lastIndex] = createTextVNode(last.text + (c[0]: any).text)
          c.shift()
        }
        res.push.apply(res, c)
      }
    } else if (isPrimitive(c)) {
      if (isTextNode(last)) {
        // merge adjacent text nodes
        // this is necessary for SSR hydration because text nodes are
        // essentially merged when rendered to HTML strings
        res[lastIndex] = createTextVNode(last.text + c)
      } else if (c !== '') {
        // convert primitive to vnode
        res.push(createTextVNode(c))
      }
    } else {
      if (isTextNode(c) && isTextNode(last)) {
        // merge adjacent text nodes
        res[lastIndex] = createTextVNode(last.text + c.text)
      } else {
        // default key for nested array children (likely generated by v-for)
        if (isTrue(children._isVList) &&
          isDef(c.tag) &&
          isUndef(c.key) &&
          isDef(nestedIndex)) {
          c.key = `__vlist${nestedIndex}_${i}__`
        }
        res.push(c)
      }
    }
  }
  return res
}
```
这个函数接受两个参数，` children `表示要规范的子节点，` nestedIndex `表示嵌套的索引，因为单个 child 可能是一个数组类型。
` normalizeArrayChildren `的主要逻辑就是遍历 child，得到单个节点` c `，然后判断` C `的类型，如果是数组类型，则递归调用` normalizeArrayChildren `，如果是基本数据类型，则使用` createTextVNode `将其转换为 Vnode 类型。如果 children 是一个数组并且有嵌套，则根据 ` nextedIndex `去更新它的 key。如果存在两个连续的 text 节点，那么会把它们合并成一个 text 节点。
这一套组合拳之后` children `就变成了一个类型为 Vnode 的数组。

## createElement
接下来回到 createElement 中，我们刚才已经对 children 进行格式化判断了，然后我们要对 tag 类型做相应处理。
```js
// 根据tag类型做相应处理
  let vnode, ns
  if (typeof tag === 'string') {
    let Ctor
    ns = (context.$vnode && context.$vnode.ns) || config.getTagNamespace(tag)
    // 判断是否保留标签
    if (config.isReservedTag(tag)) {
      // platform built-in elements
      if (process.env.NODE_ENV !== 'production' && isDef(data) && isDef(data.nativeOn)) {
        warn(
          `The .native modifier for v-on is only valid on components but it was used on <${tag}>.`,
          context
        )
      }
      vnode = new VNode(
        config.parsePlatformTagName(tag), data, children,
        undefined, undefined, context
      )
    } else if ((!data || !data.pre) && isDef(Ctor = resolveAsset(context.$options, 'components', tag))) {
      // component
      // resolveAsset获取components中对应的组件构造函数
      // 自定义组件走这里
      vnode = createComponent(Ctor, data, context, children, tag)
    } else {
      // unknown or unlisted namespaced elements
      // check at runtime because it may get assigned a namespace when its
      // parent normalizes children
      vnode = new VNode(
        tag, data, children,
        undefined, undefined, context
      )
    }
  } else {
    // direct component options / constructor
    vnode = createComponent(tag, data, context, children)
  }
```
这里会根据情况生成哪个 vnode 从上到下分别是保留标签，自定义组件，不认识的节点就创建一个 vnode。
到此为止我们在 updateComponent 中的` vm._update(vm._render(), hydrating) `第一个参数传入的其实就是一个 vnode。

## 总结
vm._render 最终是通过执行 createElement 方法并返回的是 vnode

# _update
定义在 ` src/core/instance/lifecycle.js lifecycleMixin函数中 `看主要逻辑，内部主要是定义了 `_update`这个函数
```js
Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {
    const vm: Component = this
    const prevEl = vm.$el
    const prevVnode = vm._vnode
    const restoreActiveInstance = setActiveInstance(vm)
    vm._vnode = vnode
    // Vue.prototype.__patch__ is injected in entry points
    // based on the rendering backend used.
    if (!prevVnode) {
      // 渲染虚拟dom为真实dom（首次渲染） （参数1：真实dom， 参数2：虚拟dom）
      vm.$el = vm.__patch__(vm.$el, vnode, hydrating, false /* removeOnly */)
    } else {
      // 更新操作
      vm.$el = vm.__patch__(prevVnode, vnode)
    }
    restoreActiveInstance()
    // update __vue__ reference
    if (prevEl) {
      prevEl.__vue__ = null
    }
    if (vm.$el) {
      vm.$el.__vue__ = vm
    }
    // if parent is an HOC, update its $el as well
    if (vm.$vnode && vm.$parent && vm.$vnode === vm.$parent._vnode) {
      vm.$parent.$el = vm.$el
    }
    // updated hook is called by the scheduler to ensure that children are
    // updated in a parent's updated hook.
  }
```
在` if else `中我们得知首次渲染就走 if 如果是更新就会走 else ` vm.$el = vm.__patch__(vm.$el, vnode, hydrating, false /* removeOnly */) `
这段代码就是我们要研究的逻辑，但后两个参数都是不重要的，都是 false，我们只关注前两个参数，他们分别是** 真实 DOM ** 和 **虚拟 DOM **。我们接下来看一下 __patch__ 这个方法

位置： ` src/platforms/web/runtime/index.js `我们可以发现内部定义了` Vue.prototype.__patch__ = inBrowser ? patch : noop `它会判断一下我们是不是在浏览器中，因为在服务端比如 node.js 中我们是碰不到 dom 的，所以在服务端的 patch 是一个空函数。我们接下来去看一下 patch 方法。

位置: ` src/platforms/web/runtime/patch.js `
` export const patch: Function = createPatchFunction({ nodeOps, modules }) `

+ nodeOps: 一些操作 dom 的方法
+ modules: 一些模块

内部是引入了一下` createPatchFunction `函数然后返回了一个 patch 函数，那我们还得去看看这个 createPatchFunction 函数做了什么。

位置： ` src/core/vdom/patch.js `
首先，在文件中我们定义了一些钩子 ` const hooks = ['create', 'activate', 'update', 'remove', 'destroy'] `然后在 createPatchFunction 中
```js
export function createPatchFunction (backend) {
  let i, j
  const cbs = {}

  const { modules, nodeOps } = backend

  for (i = 0; i < hooks.length; ++i) {
    cbs[hooks[i]] = []
    for (j = 0; j < modules.length; ++j) {
      if (isDef(modules[j][hooks[i]])) {
        cbs[hooks[i]].push(modules[j][hooks[i]])
      }
    }
  }
}
```
先只看这一部分，我们可以得知，我们在运行 patch 函数的时候，执行钩子就会执行对应的模块函数，这里先不展开说，有个印象即可。然后这个函数中其实定义了许多辅助函数，这里先不提及，最后其实就是返回了一个 patch 函数，也就是说我们的 createPatchFunction 就返回了一个 patch
函数，也就是说我们在之前这个地方 ` Vue.prototype.__patch__ = inBrowser ? patch : noop `定义的 __patch__ 实际上就是在 createPatchFunction 中返回的这个 patch 那这样绕一个大圈有什么好处呢？其实这里用到了**函数柯理化**的思想。

我们再来从头顺一下，我们的 patch 首先就是 createPatchFunction 函数的执行结果。
` export const patch: Function = createPatchFunction({ nodeOps, modules }) `但这里我们传入参数传入的 nodeOps 很多操作是和平台相关的，因为 vue 现在是跨平台的，我们在 web 和 week 上操作 dom 的 api 是不一样的，这一部分是分离的。再看 modules ，其实在模块中，不同的生命周期做的事情也是不一样的。这样就不用我们每次执行 patch 的时候都用 if else 去判断平台。其实就是通过闭包，提前把所有参数传入了，用到了什么直接拿就可以了。
接下来就来看一下这个 patch。

## patch
在 patch 函数中，我们主要看 createElm 这个函数
```js
createElm(
  vnode,
  insertedVnodeQueue,
  // extremely rare edge case: do not insert if old element is in a
  // leaving transition. Only happens when combining transition +
  // keep-alive + HOCs. (#4590)
  oldElm._leaveCb ? null : parentElm,
  nodeOps.nextSibling(oldElm)
)
```
再看 createElm 这个函数，这里实际上就是使用原生的 document.createElement 创建了一个 DOM，然后判断是否有子节点再去递归，最后就会创建出一颗 DOM 树。然后挂载到页面上。

# 总结
[![sSAgRx.png](https://s3.ax1x.com/2021/01/02/sSAgRx.png)](https://imgchr.com/i/sSAgRx)
实际上 new Vue 这个过程还是很清晰的，但有一点值得注意，实际上 vue 是新创建一个 div 然后去替换我们写的那个 div 来完成的渲染页面

1. new Vue 的时候我们会内部调用 `_init` 方法，而` _init `内部会去做一些操作，比如**选项合并，初始化生命周期，初始化事件，初始化渲染，初始化data等等**，最后会去调用` $mount `挂载，而` $mount `是和版本相关的，有 runtime 版本， runtime with compiler 版本。
2. runtime with compiler 版本内部会去保存 runtime 版本的 `$mount` 然后内部生成 render 函数，如果我们写了 render 那么就直接用，如果没有的话去看 template 或者 el，根据 template 和 el 去生成 render 函数，也就是说我们的 vue 内部其实只认 render函数。最后去调用之前保存的 $mount 方法
3. ` $mount `内部会创建一个` 渲染 Watcher `，内部去调用一个` _update函数 `并传入一个 ` _render `其实就是 vnode。
4. ` _render `会去执行` createElement `
5. createElement 会去递归子节点生成一个 虚拟 dom 的数组，最后生成的就是虚拟 dom
6. ` _update `根据虚拟 dom 调用` __patch__ `将虚拟 dom 渲染为真实 dom，并且在页面中替换原有的 dom 元素，完成页面首次渲染。

