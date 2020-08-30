---
title: Vue diff算法
date: 2020-08-30 14:13:49
tags: 
  - Vue
categories: 
  - vue
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