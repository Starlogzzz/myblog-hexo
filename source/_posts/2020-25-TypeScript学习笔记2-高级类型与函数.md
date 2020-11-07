---
title: TypeScript学习笔记2-高级类型与函数
date: 2020-11-07 12:50:58
tags:
  - ts
categories: 
  - ts
cover: https://s1.ax1x.com/2020/11/05/BRcVrd.jpg
abbrlink: 202025
---

# 前言
本篇主要记录接口，高级属性，以及ts中的函数。

# 接口
上一节我们提到了自定义对象类型可以使用接口，接下来详细解释一下接口这个概念。接口是对复杂的对象类型进行标注的一种方式。简单定义一个接口
```ts
interface Person {
  name: string;
  readonly age: number; // 只读属性，定义后无法被修改
  sex?: string; // sex还有可能是undefined值
  [key: string]: string | number | undefined; // 要把上面的类型覆盖，不然会报错
  [key: number]: number // 数字类型必须是字符串类型的子类型或者相同类型
}

let people: Person = { // 使用接口进行对象的类型标注
  name: "dsm",
  age: 32,
  sex: "boy",
  job: "gamer" // 通过接口定义的任意类型添加
}
```
接下来来解释一下各自的含义。

## 可选属性
上面的代码中` sex `就是一个可选属性，我们在属性前面加上一个` ? `表示它是一个可选属性，使用接口创建对象时不写sex也不会报错。

## 只读属性
` readonly `表示这个属性是一个只读属性，它无法被修改，比如上面的` age `就是只读属性。
```ts
people.age = 40; // 报错
```

## 任意属性
我们有时希望给接口拓展一些属性，可以通过任意属性来实现。设置一个索引类型。

### 数字索引
` [key: number] `就是一个数字索引，它的索引只可以是number类型。
```ts
people.100 = 100; // 添加属性
```

### 字符串索引
` [key: string] `表示字符串索引，它的索引只可以是string类型。
```ts
people.level = 10; // 添加属性
```
注意，字符串索引的任意属性在可选属性下使用会有一些问题
```ts
  sex?: string; // 可能是string或者undefined
  [key: string]: string | number | undefined; // 要把上面的类型覆盖，不然会报错
  [key: number]: number // 数字类型必须是字符串类型的子类型或者相同类型
```
因为这样定义的话，下面的任意属性实际上覆盖了上面的可选属性，但sex可以是undefined，所以可选属性的范围要完全包裹任意属性。同时，数字索引是字符串索引的子类型，所以我们必须还有覆盖数字索引的类型，它最后就变成了整个样子。再举一个栗子。
```ts
interface P1 {
    [prop1: string]: string;
    [prop2: number]: number;	// 错误
}
interface P2 {
    [prop1: string]: Object;
    [prop2: number]: Date;	// 正确
}
```

## 使用接口描述函数
我们还可以使用接口来描述一个函数。
```ts
// 接口描述函数
interface MyFunc {
  (a: number, b: number): number
}

// 使用接口MyFunc定义函数addCount
let addCount: MyFunc = function (a: number, b: number): number {
  return a + b;
}

function goAdd(callback: MyFunc) { // 也可以使用接口给回调函数添加类型
  let x = callback(10, 20);
}
goAdd(function(a: number, b: number) {
  return a + b;
})
```

## 接口合并
如果我们定义了多个同名的接口，并不会下面的接口覆盖上面的，而是合并成一个接口。
- 如果合并的接口存在同名的非函数成员，则必须保证他们类型一致，否则编译报错
- 接口中的同名函数则是采用重载

```ts
// 接口的合并，两者会合并成一个接口
interface Inter1 {
  item1: number;
  item3: boolean;
  fn(params: string): void; // 和下面那个fn是重载函数
}

interface Inter1 {
  item2: string;
  item3: boolean;
  fn(params: number): void;
}

let item: Inter1 = {
  item1: 10,
  item2: "20",
  item3: true,
  fn: function(item: any): any {
    
  }
}
```

# 高级类型
上一篇主要讲了ts的基本类型，现在来讲一下高级类型。

## 联合类型
联合类型也可以称为多选类型，当我们希望标注一个变量为多个类型之一时可以选择联合类型标注。（类似于或）
```ts
function fn1(ele: Element, key: string, val: string | number) { // val就是联合类型

}
// tsconfig中配置lib后需添加dom库
let box = document.querySelector("div");
if (box) {
  fn1(box, "color", "red")
}
```

## 交叉类型
交叉类型也可以称为合并类型，可以把多种类型合并到一起成为一种新的类型。（类似于并且）
```ts
// 交叉类型也可以称为合并类型，可以把多种类型合并到一起成为一种新的类型，并且的关系
interface item1 {
  name: string
}
interface item2 {
  age: number
}

// 指定target为es5时不能使用assign等es6方法，需要在lib中引入对应的库来扩展
let obj: item1 & item2 = Object.assign({}, {name: "dsm"}, {age: 31});
obj.name;
console.log(obj.name)
obj.age;
console.log(obj.age)
// 自动编译成es5，无需lib
let arr = [...[1,2,3]];
// target为es5时，需要lib中添加es6
Promise.resolve();
```

## 字面量类型
我们可以使用字面量类型来标注固定值。
```ts
// 直接指定sex为两种值之一
function fn(name: string, sex: "man" | "woman") {}
fn("Starlog", "man");
fn("Minra", "woman");
// fn("QJJ", "i dont konw") // 报错
```

## 类型别名
我们可以创建一个类型的别名用来储存一个类型，这样可以方便我们复用它。
```ts
type callback = (name: string) => number; // 函数类型别名
type choose = "A" | "B" | "C" | "D"; // 字面量类型别名
let fn2: callback = function(a) { // 使用类型别名
  return 100;
}

let fn3: callback = function(a) { // 使用类型别名
  return 100;
}
```

### interface和type的区别
**interface**

- 只能描述 `object`/`class`/`function` 的类型
- 同名 `interface` 自动合并，利于扩展

**type**

- 不能重名
- 能描述所有数据

## 类型推导
创建变量时直接赋值，ts会根据赋值类型直接添加类型，这就是类型推导。
```ts
let str = "wuhuqifei" // 自动推导类型为string
let num = 100; // 自动推导类型为number
// str = 10; // 报错
// num = "?" // 报错

function fn4(a = 1) { // 函数参数推到为number
  return "100"  // 返回值推导为string
}
```

## 类型断言
有的时候我们可能想再精细一点，比如给一个` img `标注一个` Element `类型，但它实际是一张图片，这时我们就无法使用` src `属性了，因为` src `并不是所以` Element ` 共有的属性。这时我们就可以使用类型断言。
```ts
let img = document.querySelector("img"); // 默认拿到img为Element类型

img.src = "XXXXXXXX" // 报错，默认识别img为Element，但src属性并不是Element公有的

(<HTMLImageElement>img).src // 添加断言，方式一
(img as HTMLImageElement).src // 添加断言，方式二
```
这样` img `就被我们指定成了` HTMLImageElement `，这样就可以拿到` src `属性了。
断言只是一种预判，并不会数据本身产生实际的作用，即：类似转换，但并非真的转换了。

# 函数类型标注
ts中函数类型可以标注参数和返回值。当然我们有很多种标注的方式，上面也都提到过了。
```ts
function fn1(a: number): string {
  return "";
}

let fn2: (a: number) => string = function(a) {
  return "";
}

type useType = (a: number) => string;
let fn3: useType = function(a) {
  return "";
}

interface useInter {
  (a: number): string;
}
let fn4: useInter = function(a) {
  return "";
}
```

## 可选参数
和接口的可选属性类型，我们可以自己决定是否传入该参数。
```ts
function fn5(a: number, b?:string): string { // b可传可不传
  return "";
}

fn5(10);
fn5(10, "20");
```

## 默认参数
ES6中我们可以直接以给参数赋值的方式定义参数默认值，ts中我们也可以设置默认参数。并且有默认值的参数也是可选的，设置了默认值的参数可以根据值自动推导类型。
```ts
function fn6(a:number, b: "chooseA" | "chooseB" = "chooseA") { // 第二个参数只能选择chooseA或者chooseB，默认选择chooseA
}

fn6(10, "chooseA")
fn6(10, "chooseB")
fn6(10); // 使用默认值chooseA
// fn6(10, "chooseC") // 报错
```

## 剩余参数
ES6中我们可以给函数参数定义剩余参数，而在ts中也是可以的，但这个剩余参数是一个数组，就要我们赋一个类型过去。
```ts
interface MyObj {
  [key: string]: any
}

function fn7(firstItem: MyObj, ...others:Array<MyObj>) { // 剩余参数others中只存MyObj类型的数据。
  return Object.assign(firstItem, ...others)
}

let newObj = fn7({ num: 1}, { num: 2 }, { num: 3 });
```

## 函数中this标注
自从ES6引入了箭头函数，我们考虑this指向时就需要两者分开考虑了。

### 普通函数
普通函数的this指向是不固定的，所以它被标注为` any `，但我们可以在函数的第一个参数位（它不占据实际参数位置）上显式的标注 `this` 的类型。
```ts
interface ObjType {
  name: string,
  fn: (a: number) => number
}

let obj: ObjType = {
  name: "dsm",
  fn: function(this:ObjType ,a) {
    // this // 不设置this的类型，此处this为any
    // this // 设置之后类型为ObjType
    return 10;
  }
}
```

### 箭头函数
箭头函数的特点之一就是它的` this `在定义时已经确定指向，它的` this `取决于它所在作用域的` this `。
```ts
interface NewMyObj {
  name: string,
  fn: (a: string) => void;
}

let newobj:NewMyObj = {
  name: "测试箭头函数中this",
  fn: function(this: NewMyObj, a) {
    return () => {
      this // 指向MyObj
    }
  }
} 
```

## 函数重载
因为之前学过JAVA，所以对于重载这个概念并不陌生。
```ts
function processDiv(el: HTMLDivElement, attr: "box-sizing", value: "content-box", width: string): void;
function processDiv(el: HTMLDivElement, attr: "box-sizing", value: "border-box",  width: string): void;
function processDiv(el: HTMLDivElement, attr?: any, value?: any): void {
  //
}

let div = document.querySelector("div");
div && processDiv(div, "box-sizing", "content-box", "15px");
div && processDiv(div, "box-sizing", "border-box", "20px");
```
这里我们设置了两种processDiv，它需要我们在第三个processDiv中实现我们要的功能，而上面两个只是定义并不实现，用` ; `分隔。函数重载使用时参数必须一一对应。