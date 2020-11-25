---
title: 手写集合
sidebar: auto
tags:
  - js
  - 基础知识
  - 手写
categories:
  - js
  - 基础知识
  - 手写
abbrlink: 8109
cover: ../img/2.png
date: 2020-06-23 00:00:00
---
## 改变this指向方法
## 1.call

```js
    (function(proto){
        function call(context = window,...args){
            context.$fn = this;
            let result = context.$fn(args);
            delete context.$fn;

            return result;
        }
        proto.call = call;
    })(Function.prototype)
```

向Function.prototype上添加call方法，call方法接受两个参数，第一个参数默认为window，第二个参数是需要传给调用call方法的参数。

```js
    context.$fn = this;
    let result = context.$fn(args);
    delete context.$fn;
    return result;
```
这里使用``对象.函数()``这种方法改变this指向，这样this就会指向这个“对象”，调用$fn把返回结果给return出去，再把添加的$fn删除。

## 2.apply

```js
    ~function(proto){
            function apply(context = window,args){
                context.$fn = this;
                let result = context.$fn(...args);
                delete context.$fn;

                return result;
            }
            proto.apply = apply;
        }(Function.prototype)
```

原理等同于call,只不过传的第二个参数为数组

## 3.bind

```js
    (function(proto){
        function bind(context = window,...args){
            return (...amArg)=>{
                this.call(context,...args.concat(amArg));
            };
        }
        proto.bind = bind;
    })(Function.prototype)
```

由于bind只会返回一个函数,所以使用箭头函数方便获取调用bind的对象,内部用call改变他的this指向

# 实现深拷贝
```js
function deepclone(obj, hash = new WeakMap()){
  if(obj == null) return null;
  if(typeof obj !== "object") return obj
  if(obj instanceof RegExp) return new RegExp(obj)
  if(obj instanceof Date) return new Date(obj)
  if (hash.get(obj)) return hash.get(obj);
  let clone = new obj.constructor()
  hash.set(obj, clone);
  Object.keys(obj).forEach(key =>{
    clone[key] = deepclone(obj[key], hash);
  })
  return clone
}
```

# 数组扁平化与数组去重
## 数组扁平化
使用some
```js
function flat(arr) {
  while(arr.some(item => Array.isArray(item))) {
    arr = [].concat(...arr)
  }
  return arr;
}
```

使用reduce
```js
function flat(arr) {
  return arr.reduce((a, b) => {
    return a.concat(Array.isArray(b) ? flat(b) : b)
  }, [])
}
```

遍历
```js
function flat(arr) {
  let newArr = [];
  for(let i=0; i<arr.length; i++) {
    if(Array.isArray(arr[i])) {
      newArr = newArr.concat(flat(arr[i]))
    } else {
      newArr.push(arr[i])
    }
  }
}
```

## 数组去重
使用includes
```js
function distinct(arr) {
  let newArr = [];
  for(let i=0; i<arr.length; i++) {
    if(!newArr.includes(arr[i])) {
      newArr.push(arr[i])
    }
  }
  return newArr;
}
```

使用map
```js
function distinct(arr) {
  let newArr = [];
  let map = new Map();
  for(let i=0; i<arr.length; i++) {
    if(!map.has(arr[i])) {
      newArr.push(arr[i]);
      map.set(arr[i], 0)
    }
  }
  return newArr;
}
```

使用filter
```js
function distinct(arr) {
  return arr.filter((item, index, arr) => {
    return arr.indexOf(item, 0) === index;
  })
}
```

