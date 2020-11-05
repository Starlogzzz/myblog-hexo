---
title: TypeScript学习笔记1.初识类型
date: 2020-11-05 16:53:31
tags:
  - ts
categories: 
  - ts
cover: https://s1.ax1x.com/2020/11/05/BRcVrd.jpg
abbrlink: 202024
---

# `TypeScript`学习笔记 1.初识类型
## 安装 `TypeScript` 
通过 `NPM` 包管理工具安装 `TypeScript` 编译器

```shell
npm i -g typescript
```

安装完成以后，我们可以通过命令 `tsc` 来调用编译器

```shell
# 查看当前 tsc 编译器版本
tsc -v
```

## 编写代码
`TypeScript` 代码

```typescript
// ./src/helloTS.ts
let str: string = 'Hello TypeScript';
```

## 编译执行
`TypeScript` 编写的程序并不能直接通过浏览器运行，我们需要先通过 `TypeScript` 编译器把 `TypeScript` 代码编译成 `JavaScript` 代码。

使用我们安装的 `TypeScript` 编译器 `tsc` 对 `.ts` 文件进行编译

```shell
tsc ./src/helloTS.ts
```

默认情况下会在当前文件所在目录下生成同名的 `js` 文件

## 编译选项
### --outDir
指定编译文件输出目录

```shell
tsc --outDir ../dist ./helloTS.ts
```
输出到dist目录中

### --target
指定编译的代码版本目标，默认为 `ES3`

```shell
tsc --outDir ../dist --target es6 ./helloTS.ts
```

### --watch
在监听模式下运行，当文件发生改变的时候自动编译

```shell
tsc --outDir ../dist --target es6 --watch ./helloTS.ts
```

我们可以使用编译配置文件：`tsconfig.json`，我们可以把上面的编译选项保存到这个配置文件中。

## 编译配置文件
我们可以把编译的一些选项保存在一个指定的 `json` 文件中，默认情况下 `tsc` 命令运行的时候会自动去加载运行命令所在的目录下的 `tsconfig.json` 文件，配置文件格式如下

```json
{
	"compilerOptions": {
		"outDir": "./dist",
		"target": "es6",
    "watch": true,
	},
  // ** : 所有目录（包括子目录）
  // * : 所有文件，也可以指定类型 *.ts
  "include": ["./src/**/*"]
}
```

有了单独的配置文件，我们就可以直接运行

```shell
tsc
```

### 指定加载的配置文件
使用 `--project` 或 `-p` 指定配置文件目录，会默认加载该目录下的 `tsconfig.json` 文件

```shell
tsc -p ./configs
```

## 类型标注
类型标注就是给代码中的数据添加一个类型说明，当给一个数据添加一个类型标注以后，这个数据无法存储与标注类型不符合的数据类型。

`TypeScript` 的类型标注，我们可以分为

- 基础的简单的类型标注
- 高级的深入的类型标注

## 基础的简单的类型标注
- 基础类型
- 空和未定义类型
- 对象类型
- 数组类型
- 元组类型
- 枚举类型
- 无值类型
- Never类型
- 任意类型
- 未知类型

### 基础类型
基础类型包含：<u>string</u>，<u>number</u>，<u>boolean</u>

标注语法

```typescript
let title: string = 'Hello';
let n: number = 20;
let isShow: boolean = true;
```

### 空和未定义类型
因为在基础数据类型中， `null` 和 `undefined` 这两个类型的数据只有一个值，就是它们本身，所以我们给一个变量标注这两个类型后，那么改变量就无法被修改了。

```ts
let a: null;
a = 1; // 报错
```

默认情况下 `null` 和 `undefined` 是所以类型的子类型，所以你可以把他俩赋值给其他类型的变量。

```ts
let a: string;
a = null; // 不报错
```

如果一个变量声明但未赋值，那么这个变量的值为` undefined `，但是如果它还没有标注类型的话，那么默认类型为 `any` 。

```ts
let a; // any
```

**注意：**因为` null `和` undefined `都是其他类型的子类型，所以默认情况下有一些问题。

将原本定义好数据类型的变量赋值null，但这个变量还可以使用原来数据类型的方法，比如定义一个 `a:number` ，`a=null` 之后`a.toFixed()` 仍可执行。

解决：`tsconfig.json`中`strictNullChecks` 配置为 `true`，开启对null的严格检查。

### 对象类型
#### 内置对象类型
和基本数据类型类似。

```ts
let a: object = {}; // 不推荐使用
let arr: Array<number> = [1,2,3];
let d1: Date = new Date();
```

#### 自定义对象类型
大部分时候我们是使用自定义对象类型的。

- 字面量标注
- 接口
- 定义 类或者构造函数

1. 字面量标注
```ts
let obj: { name:string, age:number } = { // 字面量标注，不常用
  name : "dsm",
  age: 31
}
```
2. 接口
```ts
interface Person {
  name: string,
  age: number
}
let obj: Person = { // 使用接口，推荐使用
  name: "dsm",
  age: 31
}
```
3. 定义类或构造函数
```ts
class Person {
  constructor(public name: string, public age: number) {
 
  }
}
let obj: Person = { // 类与构造函数，不是很推荐
  name: "dsm",
  age: 31
}
```
#### 包装对象
js中的` String `，` Number `，` Boolean `就是所谓的包装对象。
简单说就是这三大写的和小写的不是一个东西，ts中也一样。
```ts
let lettleString: string;
lettleString = "10";
lettleString = new String(10); // 报错，String涵盖string，而string不涵盖String。

let bigString: String;
bigString = "10";
bigString = new String(10); // 没问题
```

#### 数组类型

ts中，数组中元素类型必须一致，所以要给数组标注数据类型。有两种方式：
1. 使用泛型
2. 简单标注

```ts
let arr: Array<number> = [1, 2, 3]; // 使用泛型
arr.push(9);
// arr.push("0") // 不可以添加其他类型的数据
let arr1: string[] = ["1", "2", "3"]; // 简单标注
```

#### 元组类型
元组类似于数组，但它可以指定多种数据类型，但内容顺序必须与指定的一致，并且新加入的数据必须是设定的数据类型之一。

```ts
let data1:[string, number] = ["1", 2]; 
// let data2:[string, number] = [1, "2"]; // 报错，顺序不可替换
data1.push("3")
data1.push(4)

// data1.push(null) // 报错，因为上面开启了对null的严格检查
// data1.push(undefined)
```

#### 枚举类型
枚举的作用组织收集一组关联数据的方式，通过枚举我们可以给一组有关联意义的数据赋予一些友好的名字。推荐名称全部大写表示常量。枚举分为数字枚举和字符串枚举，若上项为数字枚举可以直接声明而不赋值，默认值为上一项值+1。
```ts
enum HTTP_CODE {
  OK = 200,
  NOT_FOUND = 404,
  METHOD_NOT_ALLOWED // 405
}
HTTP_CODE.OK; // 200
// HTTP_CODE.OK = 0; // error
HTTP_CODE["OK"]; // 200

enum PATH_FIND {
  ASSETS = "./src/assets",
  COMPONENTS = "./src/compoents",
  // COMMON, // 不可以直接像数字一样直接定义，字符串枚举需要赋值
  INDEX = 0,
  INDEX_NUM // 上一个值是数字可以定义但不赋值，默认值为上一值+1
}
PATH_FIND.INDEX_NUM // 1
```

#### 无值类型
表示没有任何数据类型，一般用于表示无返回值。
```ts
function fn(): void {
  // 返回值void，或者说无返回值，还可以返回null，undefined
  // return undefined;
  // return null; // 开启null严格检查后会报错
}
```

#### Never类型
表示一个函数永远不会` return `，定义` never `，比如抛出错误。
```ts
function fn1(): never {
  throw new Error("报错")
}
```

#### 任意类型
设置为随意类型，其实就相当于不用ts声明，像原生js那样，什么类型都可以接收。
+ 一个变量声明但未赋值时默认为` any `类型。
+ 任何类型都可以赋值给` any `类型。
+ ` any `类型可以赋值给任意类型
+ ` any `类型有任意属性和方法

```ts
let a: any;
a = null;
a.b;

let b: number = 10;
b = a; // 赋一个null
b.toFixed(); // 会报错

function fn3(a: any): number {
  a.indexOf("a", 1)
  return 1
}
```
其实看起来很方便，但我们声明any类型就相当于没用ts，那就没意义了，所以不太推荐频繁使用。
**当指定 `noImplicitAny` 配置为 `true`，当函数参数出现**隐含**的 `any` 类型时报错**

#### 未知类型
未知类型` unknow `只能赋值给 ` unknow `和` any `。并且没用任何属性和方法。
```ts
function fn4(): unknown { // 返回值类型未知
  return 1
}

let t: unknown;
// let x: number = t; // 报错，不能将unknown赋值给其他属性
// t.a // unknown类型没有任何属性和方法
t = 10;
```

#### 函数类型
对函数进行类型标注。主要是对参数和返回值。
```ts
// 可以给函数的参数或者返回值添加类型
function fn6(a: string, b: number): number {
  return 1
}

// fn6(1,2) // 报错，严格遵守
```


