---
title: '模拟call,apply,bind'
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

## 1. call

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

向Function.prototype上添加call方法,call方法接受两个参数,第一个参数默认为window,第二个参数是需要传给调用call方法的参数

```js
    context.$fn = this;
    let result = context.$fn(args);
    delete context.$fn;
    return result;
```

把this指向调用call的对象,调用$fn把返回结果给return出去，再把添加的$fn删除

## 2. apply

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

## 3. bind

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
