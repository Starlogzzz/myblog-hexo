---
title: Vue diff算法
date: 2020-08-30 14:13:49
tags: 
  - Vue
categories: 
  - Vue
cover: 
abbrlink: 202010
---

## 前言

最近一直在找实习工作，一直忙于面试相关的东西，所以好长时间没写文章了，但因为疫情的原因，学校要求开学必须返校o(╥﹏╥)o，拿到offer也没用啊，只能继续上学了(惨)，这次来总结一下Vue的diff算法

![dbg6Sg.png](https://s1.ax1x.com/2020/08/30/dbg6Sg.png)

**首先我们知道什么是虚拟dom?**
> 虚拟dom就是将真实的节点dom用js模拟出来。
比如:
```html
<div>
    <p>芜湖</p>
</div>
```
对应虚拟dom差不多就是
```js
Vnode = {
    tag: 'div',
    children: [
        { tag: 'p', text: '芜湖' }
    ]
};
```
虚拟dom的优点: 
+ 虚拟DOM轻量、快速：当它们发生变化时通过新旧虚拟DOM比对可以得到最小DOM操作量，从而提升性能。
diff算法就是用来比较新旧节点的区别而进行操作的。

什么是diff算法？
> 因为vue使用虚拟dom，而我们由虚拟dom计算出真实的dom所使用的的就是diff算法比较出最小操作

diff在哪，什么时候执行？
> 在patch中，存在新旧虚拟dom时才会执行

diff怎么执行？
> 深度优先，同级比较

## patch

![dbg6Sg.png](https://blog-10039692.file.myqcloud.com/1506309609752_5003_1506309612171.png)

这是一张网上很经典的图，表示了即仅在同级的vnode间做diff，递归地进行同级vnode的diff，最终实现整个DOM树的更新。

patchVnode
比较两个VNode，包括三种类型操作：属性更新、文本更新、子节点更新
具体规则如下：
1. 新老节点均有children子节点，则对子节点进行diff操作，调用updateChildren
2. 如果老节点没有子节点而新节点有子节点，先清空老节点的文本内容，然后为其新增子节点。
3. 当新节点没有子节点而老节点有子节点的时候，则移除该节点的所有子节点。
4. 当新老节点都无子节点的时候，只是文本的替换。
来看一下源码中这段操作
```js
    // 先查找新旧节点是否存在孩子
    const oldCh = oldVnode.children
    const ch = vnode.children
    // 属性更新
    if (isDef(data) && isPatchable(vnode)) {
      for (i = 0; i < cbs.update.length; ++i) cbs.update[i](oldVnode, vnode)
      if (isDef(i = data.hook) && isDef(i = i.update)) i(oldVnode, vnode)
    }
    // 判断是否是元素，(没有文本就是元素)
    if (isUndef(vnode.text)) {
      // 双方都有孩子
      if (isDef(oldCh) && isDef(ch)) {
        // 比孩子，reorder 下文中updateChildren调用位置
        if (oldCh !== ch) updateChildren(elm, oldCh, ch, insertedVnodeQueue, removeOnly)
      } else if (isDef(ch)) {
        // 新节点有孩子
        if (process.env.NODE_ENV !== 'production') {
          checkDuplicateKeys(ch)
        }
        // 清空老节点文本
        if (isDef(oldVnode.text)) nodeOps.setTextContent(elm, '')
        // 创建孩子并追加
        addVnodes(elm, null, ch, 0, ch.length - 1, insertedVnodeQueue)
      } else if (isDef(oldCh)) {
        // 老节点有孩子，删除即可
        removeVnodes(oldCh, 0, oldCh.length - 1)
      } else if (isDef(oldVnode.text)) {
        // 老节点存在文本，清空
        nodeOps.setTextContent(elm, '')
      }
    } else if (oldVnode.text !== vnode.text) {
      // 双方都是文本节点，更新文本
      nodeOps.setTextContent(elm, vnode.text)
    }
    if (isDef(data)) {
      if (isDef(i = data.hook) && isDef(i = i.postpatch)) i(oldVnode, vnode)
    }
  }
```
注意，这里的if，else if比较多，不要看错了if和else if的关系。

## updateChildren

updateChildren主要作用是用一种较高效的方式比对新旧两个VNode的children得出最小操作补丁。执行一个双循环是传统方式，vue中针对web场景特点做了特别的算法优化，我们看图说话：

![dbbeN4.png](https://s1.ax1x.com/2020/08/30/dbbeN4.png)

在新老两组VNode节点的左右头尾两侧都有一个变量标记，在遍历过程中这几个变量都会**向中间靠拢**。 当**oldStartIdx > oldEndIdx**或者**newStartIdx > newEndIdx**时结束循环。

首先，oldStartVnode、oldEndVnode与newStartVnode、newEndVnode**两两交叉比较**，共有4种比较方法。

+ 当 oldStartVnode和newStartVnode 或者 oldEndVnode和newEndVnode 满足sameVnode，直接将该VNode节点进行patchVnode即可，不需再遍历就完成了一次循环。如下图

![dbXLTK.png](https://s1.ax1x.com/2020/08/30/dbXLTK.png)

深度优先，同级比较。

+ 如果oldStartVnode与newEndVnode满足sameVnode。说明oldStartVnode已经跑到了oldEndVnode后面去了，进行patchVnode的同时还需要将真实DOM节点移动到oldEndVnode的后面。

![dbjt1J.png](https://s1.ax1x.com/2020/08/30/dbjt1J.png)

+ 如果oldEndVnode与newStartVnode满足sameVnode，说明oldEndVnode跑到了oldStartVnode的前面，进行patchVnode的同时要将oldEndVnode对应DOM移动到oldStartVnode对应DOM的前面。

![dbj076.png](https://s1.ax1x.com/2020/08/30/dbj076.png)

+ 如果以上情况均不符合，则在old VNode中找与newStartVnode满足sameVnode的vnodeToMove，若存在执行patchVnode，同时将vnodeToMove对应DOM移动到oldStartVnode对应的DOM的前面。

![dbjhHP.png](https://s1.ax1x.com/2020/08/30/dbjhHP.png)

+ 当然也有可能newStartVnode在old VNode节点中找不到一致的key，或者是即便key相同却不是sameVnode，这个时候会调用createElm创建一个新的DOM节点。

![dbvK8e.png](https://s1.ax1x.com/2020/08/30/dbvK8e.png)

+ 当结束时oldStartIdx > oldEndIdx，这个时候旧的VNode节点已经遍历完了，但是新的节点还没有。说明了新的VNode节点实际上比老的VNode节点多，需要将剩下的VNode对应的DOM插入到真实DOM中，此时调用addVnodes（批量调用createElm接口）。

![dbvyV0.png](https://s1.ax1x.com/2020/08/30/dbvyV0.png)

+ 但是，当结束时newStartIdx > newEndIdx时，说明新的VNode节点已经遍历完了，但是老的节点还有剩余，需要从文档中删 的节点删除。

![dbvfxJ.png](https://s1.ax1x.com/2020/08/30/dbvfxJ.png)

以上就是diff算法的机制，最后贴一下源码加深一下
```js
  // 重排算法
  function updateChildren (parentElm, oldCh, newCh, insertedVnodeQueue, removeOnly) {
    // 四个指针
    let oldStartIdx = 0
    let newStartIdx = 0
    let oldEndIdx = oldCh.length - 1
    let oldStartVnode = oldCh[0]

    // 四个节点
    let oldEndVnode = oldCh[oldEndIdx]
    let newEndIdx = newCh.length - 1
    let newStartVnode = newCh[0]
    let newEndVnode = newCh[newEndIdx]
    let oldKeyToIdx, idxInOld, vnodeToMove, refElm

    // removeOnly is a special flag used only by <transition-group>
    // to ensure removed elements stay in correct relative positions
    // during leaving transitions
    const canMove = !removeOnly

    if (process.env.NODE_ENV !== 'production') {
      checkDuplicateKeys(newCh)
    }
    // 循环条件：开始索引不大于结束索引
    while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
      // 头尾指针调整
      if (isUndef(oldStartVnode)) {
        oldStartVnode = oldCh[++oldStartIdx] // Vnode has been moved left
      } else if (isUndef(oldEndVnode)) {
        oldEndVnode = oldCh[--oldEndIdx]
        // 此处为头尾四种情况
      } else if (sameVnode(oldStartVnode, newStartVnode)) {
        // 两个开头相同
        patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
        // 索引向后移动一位
        oldStartVnode = oldCh[++oldStartIdx]
        newStartVnode = newCh[++newStartIdx]
      } else if (sameVnode(oldEndVnode, newEndVnode)) {
        // 老的开始等于新的结束，除了打补丁之外还要移动到队尾
        patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx)
        oldEndVnode = oldCh[--oldEndIdx]
        newEndVnode = newCh[--newEndIdx]
      } else if (sameVnode(oldStartVnode, newEndVnode)) { // Vnode moved right
        patchVnode(oldStartVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx)
        canMove && nodeOps.insertBefore(parentElm, oldStartVnode.elm, nodeOps.nextSibling(oldEndVnode.elm))
        oldStartVnode = oldCh[++oldStartIdx]
        newEndVnode = newCh[--newEndIdx]
      } else if (sameVnode(oldEndVnode, newStartVnode)) { // Vnode moved left
        patchVnode(oldEndVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
        canMove && nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm)
        oldEndVnode = oldCh[--oldEndIdx]
        newStartVnode = newCh[++newStartIdx]
      } else {
        // 四种猜想之后没有找到相同的，不得不开始循环查找
        if (isUndef(oldKeyToIdx)) oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx)
        // 查找在老的孩子数组中的索引
        idxInOld = isDef(newStartVnode.key)
          ? oldKeyToIdx[newStartVnode.key]
          : findIdxInOld(newStartVnode, oldCh, oldStartIdx, oldEndIdx)
        if (isUndef(idxInOld)) { // New element
          // 没找到则创建追加
          createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx)
        } else {
          // 找到移动到队首
          vnodeToMove = oldCh[idxInOld]
          if (sameVnode(vnodeToMove, newStartVnode)) {
            patchVnode(vnodeToMove, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
            oldCh[idxInOld] = undefined
            canMove && nodeOps.insertBefore(parentElm, vnodeToMove.elm, oldStartVnode.elm)
          } else {
            // same key but different element. treat as new element
            createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx)
          }
        }
        newStartVnode = newCh[++newStartIdx]
      }
    }
    // 整理工作，必定有数组还剩下的元素还未处理
    if (oldStartIdx > oldEndIdx) {
      refElm = isUndef(newCh[newEndIdx + 1]) ? null : newCh[newEndIdx + 1].elm
      addVnodes(parentElm, refElm, newCh, newStartIdx, newEndIdx, insertedVnodeQueue)
    } else if (newStartIdx > newEndIdx) {
      removeVnodes(oldCh, oldStartIdx, oldEndIdx)
    }
  }
```

具体每一步都做了什么可以看这篇文章
[深入 Vue2.x 的虚拟 DOM diff 原理]: https://cloud.tencent.com/developer/article/1006029

## key
补充一下key的作用，这个问题之前在面快手的时候被问到过，当时回答的不是很清晰，今天来彻底梳理一下key的作用。

> 官方文档对key的阐述：` key ` 的特殊 attribute 主要用在 Vue 的虚拟 DOM 算法，在新旧 nodes 对比时辨识 VNodes。如果不使用 key，Vue 会使用一种最大限度减少动态元素并且尽可能的尝试就地修改/复用相同类型元素的算法。而使用 key 时，它会基于 key 的变化重新排列元素顺序，并且会移除 key 不存在的元素。

大概意思差不多都能懂，但vue内部到底是怎么根据key来进行diff的呢？我们来对比一下不使用key和使用key时，vue的更新操作。

### 不使用key的情况
简单写了一个demo，定义一个数组` items = [1,2,3,4,5] `在mounted中设置一个定时器，一秒之后在2的后面添加一个6
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <div id="app">
    <p v-for="item in items">{{item}}</p>
  </div>
  <script src="vue.js"></script>
  <script>
    let app = new Vue({
      el: "#app",
      data() {
        return {
          items: [1,2,3,4,5]
        }
      },
      mounted() {
        setTimeout(() => {
          this.items.splice(2, 0, 6)
        }, 1000)
      }
    })
  </script>
</body>
</html>
```
我们需要打断点进行调试，在这个demo中，我们可以打断点时搜索：` while (old ` 找到updateChildren中比较元素那一行，在那一行上打一个条件断点：` oldStartVnode.tag === "p" `，因为我们只关心p标签是怎么进行diff的，而不关心内部的值是怎么更新的，所以添加了这个条件。

[![DNESW8.png](https://s3.ax1x.com/2020/11/24/DNESW8.png)](https://imgchr.com/i/DNESW8)
运行到这一行就会进入比较新旧的首节点，你也可以在右侧查看这两个节点的具体信息，但实际上我们知道它们是相同节点，并没有发生变化。接下来我们点进去看一下sameVnode函数。[![DNVVnH.png](https://s3.ax1x.com/2020/11/24/DNVVnH.png)](https://imgchr.com/i/DNVVnH)你可以看到，我们上来就会比较两个节点的key是否相等，不相等的话直接返回false，但因为我们并没有添加key，所以这两个key都是` undefined `它们是相同的，所以进入下一个比较，比较一下它们的tag，再比较一下它们是不是注释这种。

因为第一个p标签并没有发生变化，所以它并不需要被更新，接下来去比较下一个p标签，直到比较到第三个p标签，也就是值为3的那个，但在第三个p标签比较sameVnode时，它们还是相同标签，所以会强行去更新内容。[![DNYXi8.png](https://s3.ax1x.com/2020/11/24/DNYXi8.png)](https://imgchr.com/i/DNYXi8)可以看到，3被强行更新成了6。但这并不合适，因为接下来所有元素都需要这样被强行更新，4更新成3[![DNtFoV.png](https://s3.ax1x.com/2020/11/24/DNtFoV.png)](https://imgchr.com/i/DNtFoV)然后5更新成4，最后在末尾添加一个5这样，也就是说如果我们不使用key的话，仅仅插入一个元素，需要这个元素之后的**所有元素都进行一次更新**，这显然会消耗很多的性能。

### 使用key
使用key的话就不同了，我们比较这两个新旧节点
> 1,2,3,4,5
  1,2,6,3,4,5

因为前两次比较都是相同的，所以直接来看重点第三次
> 3,4,5
  6,3,4,5

这次首先比较首节点，也就是3和6，通过key我们可以得知他们不相同，所以我们会跳去比较尾结点，他们都是5，相同。下一次就变成了比较
> 3,4
  6,3,4

之后还会比较尾结点，直到剩余一个6。最后我们计算出6所在的位置，直接插入一个6即可。[![DNUZr9.png](https://s3.ax1x.com/2020/11/24/DNUZr9.png)](https://imgchr.com/i/DNUZr9)

所以从实践中我们可以得出，使用key之后我们仅仅执行了一次插入操作，这比不使用key时操作简便很多，但我们这个例子很简单，要是某个页面有1000个元素，在第二个元素之后插入一个元素，不使用key的话需要更新998次！！而使用key仅需插入一次。

