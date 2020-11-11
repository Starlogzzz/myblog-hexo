---
title: TypeScript学习笔记3-类与类型系统
date: 2020-11-10 10:10:21
tags:
  - ts
categories: 
  - ts
cover: https://s1.ax1x.com/2020/11/11/BXskvQ.jpg
abbrlink: 202026
---

# 类
面向对象编程中一个重要的核心就是：`类`，当我们使用面向对象的方式进行编程的时候，通常会首先去分析具体要实现的功能，把特性相似的抽象成一个一个的类，然后通过这些类实例化出来的具体对象来完成具体业务需求。这里主要讨论ts中的类。

## 类的组成
+ class
+ constructor
+ this
+ 成员属性，成员方法
+ 私有属性，私有方法

## 成员属性与方法定义
```ts
class Person { 
  name: string;
  age: number;
  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
}
```
使用` public `修饰符，比较方便。给构造函数参数添加修饰符来直接生成成员属性
```ts
class Person {
  constructor(public name: string, public age: number) { // 在类中定义成员属性
    /**
     * 当我们给构造函数参数设置了访问修饰符：public，那么ts会做如下两件事情
     *  - 给当前类添加同名的成员属性
     *  - 在类的实例化的时候，会把传入的参数值赋值给对应的成员属性
     */
  }
  getInf(title: string) {
    console.log(`传入参数title${title}，获取类中的成员属性${this.name}和${this.age}`) // 可以通过this来访问当前成员的属性
  }
}
let people = new Person("dsm", 31);
```

## 继承
在ts中我们也是通过` extends `来继承的。
- 如果子类没有重写构造函数，则会在默认的 `constructor` 中调用 `super()`
- 如果子类有自己的构造函数，则需要在子类构造函数中显示的调用父类构造函数 : `super(//参数)`，否则会报错
- 在子类构造函数中只有在 `super(//参数)` 之后才能访问 `this`
- 在子类中，可以通过 `super` 来访问父类的成员属性和方法
- 通过 `super` 访问父类的的同时，会自动绑定上下文对象为当前子类 `this`

```ts
class Teacher extends Person {
  constructor( // 定义子类中需要用的参数
    name: string,
    age: number,
    public job: string
  ) {
    super(name, age); // 调用父类构造函数
  }
  getInf(title: string) { // 重写父类中方法
    console.log(`我是子类Teacher中重写过的getIngo方法，${title}`)
  }
  getJob(job: string) { // 子类中的方法
    console.log(`我是一名${job}老师`)
  }
```

### 方法重写与重载
**重写：** 上面的` Teacher `中的` getInf `就重写了父类的对应方法。会覆盖原有继承过来的父类中的` getInf `。
**重载：** 定义多个函数，但参数个数，参数类型不同。 
我们先给` Person `类添加一个方法` doOverloaded `，继承之后重载这个函数
```ts Person中定义
doOverloaded(a: number, b: string) {
  console.log(`${a},${b},父类中的doOverloaded`)
}
```
```ts Teacher中重载
doOverloaded(a: number, b: string): void;
doOverloaded(a: number, b: string, c: boolean): void;
doOverloaded(a: number, b: string, c? :boolean) {
  super.doOverloaded(a, b); // 调用父类中的方法
  if(c) {
    console.log("重载的doOverloaded")
  }
}
```
这里我们重载了` doOverloaded `添加了第三个参数` c `，直接调用父类的方法，判断如果参数中有` c `那就再额外做一些操作即可。

# 修饰符
我们希望对类的成员进行一些限制，比如让外部无法获取或者修改类的成员。修饰符就是给类成员进行一个访问控制，保证数据的安全。
- readonly：只读
- public：公有，默认
- protected：受保护
- private：私有

## readonly
只读修饰符只能针对成员属性使用，且必须在声明时或构造函数里被初始化，类自身，子类，类外都可以读取这个值，但无法修改它。

## public
公有修饰符是类成员的默认修饰符，类自身，子类，类外都可以访问并修改它。

## protected
保护修饰符只能通过自身，子类访问并修改它。

## private
私有修饰符只能自身访问并修改它。

## 寄存器
有时候我们希望对类成员进行更加深入，或者说是更加细腻的操作，这时候就可以用到寄存器，而且定义寄存器之后我们也可以修改类中被` private `修饰的属性。类似于` defineProperty `中的` get `和` set `。

## 示例
```ts
class Person3 {
  constructor(
      readonly id: number, // 只读 子类可以访问，但是一旦确定不能修改
      public name: string,  // 公有 
      protected sex: string, // 保护 只能在类的内部或子类中才能访问和修改
      private _age: number // 私有 外部无法访问和修改
    ) {
  }
  method() {
    this.sex = "girl"; // 内部可以修改 protected
    this._age = 73; // 内部可以修改 private
  }
  set age(age: number) { // 内部有类似于defineProperty的方法，set和get设置和获取private属性，自己就是一个函数
    if(age >= 50) {
      this._age = 0;
    }
  }
  get age() {
    return this._age;
  }
}

class Plice extends Person3 {
  method2() {
    this.sex = "little girl";
    // this.age; // 只能在父类中访问和修改
  }
}

let pricident = new Person3(1, "trump", "man", 72);
// pricident.id = 2; // 无法修改只读
pricident.name = "little trump"; // 可以修改
// pricident.sex; // 只能在类的内部或子类中才能访问
// pricident._age; // 外部无法访问和修改

pricident.age = 7; 
```

# 静态成员
前面说的成员属性和方法可以说都是供实例进行使用的，那有没有什么是供我们的类自己使用的呢？它就是静态成员。
- 该成员属性或方法是类型的特征还是实例化对象的特征
- 如果一个成员方法中没有使用或依赖 `this` ，那么该方法就是静态的

```ts
class Person4 {
  static readonly TO_DO_LIST: Array<string> = ["吃饭", "睡觉", "学习"]; // 静态属性，在访问修饰符之前添加static，只能供本类使用

  constructor(
    public id: number,
    public name: string,
    private _age : number
  ) {

  }
  static info(message: string) { // 静态方法,只能供本类使用
    console.log("类Person4的静态方法info" + message);
  }
}

let user = new Person4(1, "dsm", 30);
Person4.TO_DO_LIST
Person4.info("Hello");
```
- 类的静态成员是属于类的，所以不能通过实例对象（包括 this）来进行访问，而是直接通过类名访问（不管是类内还是类外）
- 静态成员也可以通过访问修饰符进行修饰
- 静态成员属性一般约定（非规定）全大写

# 抽象类
有时候父类无法确定一个方法具体应该怎么实现，我们就可以把这个方法定义为抽象方法` abstract `，而抽象方法必须出现在抽象类中，所以定义类时也要添加一个` abstract `，并且抽象类不能使用 new 进行实例化，子类继承抽象类之后就必须实现抽象类中的所有抽象方法，否则子类也需添加` abstract `成为一个抽象类。
这里来模拟一下React的Component。
```ts
abstract class ReactComponent<T1, T2> { // 含有抽象方法的类一定是抽象类
  constructor(
    public state: T1, // 给state定义一个类型，方便子类创建时检查
    public props: T2, // 同上
  ) {
    this.props = props
  }
  abstract render(): string // render只能子类自己实现，所以标记为抽象方法没有实现，只定义
}

interface ReactState {
  num: number
}
interface ReactProps {
  val: string
}

class MyComponent extends ReactComponent<ReactState, ReactProps> {
  constructor(
    state: ReactState,
    props: ReactProps
  ) {
    super(state, props);
    this.state = state; // 这个组件直接在定义时传入state作为自己的state
  }
  render() { // 抽象类必须实现抽象方法
    this.props.val; // 有ReactProps的类型提示
    this.state.num; // 有ReactState的类型提示
    return ""
  }
}

let mycom = new MyComponent({ num: 10 }, { val: "20" }) // 分别传入组件的state和props
mycom.render();
```
使用抽象类之后，我们就约定了所有继承子类的所必须实现的方法，使类的设计更加的规范。

# 类与接口
在前面我们已经学习了接口的使用，通过接口，我们可以为对象定义一种结构和契约。我们还可以把接口与类进行结合，通过接口，让类去强制符合某种契约，从某个方面来说，当一个抽象类中只有抽象的时候，它就与接口没有太大区别了，这个时候，我们更推荐通过接口的方式来定义契约。
- 抽象类编译后还是会产生实体代码，而接口不会
- `TypeScript` 只支持单继承，即一个子类只能有一个父类，但是一个类可以实现过个接口
- 接口不能有实现，抽象类可以
- 一个类使用了一个接口，那么就必须实现接口中定义的东西
- 多接口可以连用，使用逗号` , `分隔
- 可以同时使用接口和继承
- 接口也可以继承

我们使用` implements `使用接口。
```ts
abstract class ReactComponent2<T1, T2> { 
  constructor(
    public state: T1, 
    public props: T2, 
  ) {
    this.props = props;
    this.state = state;
  }

  abstract render(): string 
}

interface ReactState {
  num: number
}
interface ReactProps {
  val: string
}

interface MessageLog { // 定义一个接口用来指定类中的logMessage函数
  logMessage(): string;
}

class MyComponent2 extends ReactComponent2<ReactState, ReactProps> implements MessageLog { // 类可以使用多个接口，用逗号分隔，同时类是单继承的
  constructor(
    state: ReactState,
    props: ReactProps
  ) {
    super(state, props);

    this.state = { 
      num: 20
    }
  }

  render() { 
    this.props.val;
    this.state.num;
    return "<MyComponent />"
  }
  logMessage() { // 类中必须实现logMessage函数
    return `输出${this.props}, ${this.state}`
  }
}

let mycom2 = new MyComponent2({ num: 10 }, { val: "20" })
mycom.render();

function getInfo(target: MyComponent2) {
  target.logMessage();
}

getInfo(mycom2)
```

# 类与对象类型
当我们在 TypeScript 定义一个类的时候，其实同时定义了两个不同的类型
- 构造函数类型（类类型）
- 对象类型

对象类型就是我们的 new 出来的实例类型，而类或者构造函数本身也是有类型的，那么这个类型就是构造函数类型。
```ts
class Person5 {
  static type: number;
  constructor( // 每一个类的类型就是这个构造函数的类型
    public name: string,
    public age: number,
  ) {

  }
}

interface TypeObj { // 定义一个供对象类型使用的接口
  name: string,
  age: number
}
interface TypeConstructor { // 定义一个供构造函数类型使用的接口
  new (name: string, age: number): Person5;
  type: number;
}

let child: TypeObj = new Person5("bzzb", 1); // child是Person5创建出来的对象，它的类型天生就是Person5

function fn1(arg: Person5) { // 传出参数为对象类型
  arg.age; // 实例拿属性
}

function fn2(arg: typeof Person5) { // 相当于Person5的构造函数类型
  new arg("starlog", 21)  
}
```
` typeof `拿到的就是构造函数本身的类型。

# 类型保护
根据判断逻辑的结果，缩⼩类型范围（有点类似断言），我们可以使用一些判断语句比如` if else `进行逻辑判断，还可以使用特定的⼀些关键字比如` typeof instanceof in `。
```ts
function fn(a: number | string) {
  // a.toFixed() // 报错，因为只有number属性上才有toFixed，而string上没有，而且ts无法判断a是什么类型。
  if(typeof a === "number") { // 指定a为number类型时
    a.toFixed(1) 
  } else { // a为string类型
    a.slice(1,2)
  }
}

function fn2(a: Array<number> | Object) {
  if(a instanceof Array) { // 当a是数组时
    a.push(1)
  } else { // a是对象时
    a.toString();
  }
}

interface item1 {
  name: string,
  age: number
}
interface item2 {
  job: string,
  height: number
}
function fn3(a: item1 | item2) {
  if("name" in a) { // a里有name
    a.age
  } else { // a里有job
    a.job
  }
}
```

我们还可以自己自定义一套保护规则，用来处理一些复杂情况。
```ts
// 自定义类型保护
function typeProduct(data: any): data is Element[]|NodeList { // data is Element[]|NodeList:类型谓词
  return data.forEach !== undefined; // data有forEach属性，那它就返回true
}

function fn4(ele: Element[]|NodeList|Element) {
  if(typeProduct(ele)) { // ele有forEach属性，返回true通过
    ele.forEach((ele: Element) => {
      ele.className = "Elements";
    })
  } else {
    ele.className = "Element"
  }
}
```
` data is Element[]|NodeList `是⼀种类型谓词，格式为：xx is XX ，返回这种类型的函数就可以被 TypeScript 识别为类型保护。

# 类型操作
类型数据只能作为类型来使用，⽽不能作为程序中的数据，类型数据只是用在编译检测阶段，而程序中数据用于程序执⾏阶段。ts中提供给我们一些方法来操作类型。

## typeof
` typeof `可以用于获取数据类型，主要看它把这个结果赋值给什么：
+ ` let x = typeof "xxx" ` x存储的是` "string" `这个值。
+ ` type x = typeof "xxx" ` x存储的是` string `这个类型。

```ts
let message = "Hello TS";

let mes = typeof message; // let 用来声明一个**变量**，是真真正正用得到的变量
type typemes = typeof message; // type 声明的是一种类型别名，只在程序检测过程中使用，此处typemes为string类型

let newMes = mes; // let创建的变量可以赋值
let newMes2: typemes = "XXXXX" // type创建的类型别名只能用来赋值类型
```

## keyof
keyof用来获取所有类型的` key `集合。
```ts
// keyof 获取类型所有key的集合
interface MyInterface {
  name: string,
  age: number
}
type keyOf = keyof MyInterface // type keyOf = "name" | "age"
```
我们还可以先拿到一个对象的类型，再获取它的` key `集合。
```ts
let p1 = {
  name: 'dsm',
  age: 32
}
type Person = typeof p1; // typeof拿到p1的类型Person
function getPersonVal(key: keyof Person) { // keyof拿到Person的key
  return p1[key];
}
```

## in
操作类型时，我们在内部可以使用` for in `进行遍历。
```ts
interface Person2 { // 定义一个接口
  name: string,
  age: number
}
type key = keyof Person2; // 拿到接口的key，设置为一个类型key
type newPerson2 = { // 创建一个新的类型别名，但把Person2的value全部该为string类型
  [k in key]: string; // ..in.. 拿到类型别名key中的类型，设置为string类型。
}
```
**注意：** ` in `之后的类型必须是` string number symbol`之一。

# 类型兼容
ts中的类型检测和java这种语言的类型检测其实不太一样，ts只检查结果，而不管定义的名字，比如我们定义两个类。
```ts
class Dog { // 创建一个狗类
  run: boolean;
  eat: string;
}
class Cat { // 创建一个猫类
  run: boolean;
  eat: string;
}
```
我们可以看到这两个类除了名字之外完全相同，这时我们定义一个只允许传入Cat类实例的函数。
```ts
function fn5(arg: Cat) { // 定义函数fn5，参数接收一只猫类的实例
  arg.eat
}
let dog1 = new Dog();
fn5(dog1) // 这里传入一个Dog的实例也不会报错，因为上面的Dog和Cat结构完全相同
```
可以发现fn5也是可以接受Dog类的实例的，因为Dog和Cat类的结构完全相同。那要是结构不同呢？我们给Dog类添加一个` wang `方法。
```ts
class Dog { // 创建一个狗类
  run: boolean;
  eat: string;
  wang() {
    console.log("汪！")
  }
}
```
现在的Dog类和Cat类结构就不一样了，但我们还是可以像刚才一样给fn5传入一个Dog类的实例，并不会有问题。因为我们的fn5需要传入一个Cat类的实例，但Cat类实例拥有的属性Dog类的实例都有。也就是说Dog类的实例是包含了Cat类的实例。所以说不会报错。
