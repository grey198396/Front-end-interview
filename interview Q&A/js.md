<!-- TOC -->

- [js](#js)
    - [1. let, var, const 的区别](#1-let-var-const-的区别)
        - [2. typeof 和 instanceof 的区别](#2-typeof-和-instanceof-的区别)
        - [3. this 指向](#3-this-指向)
        - [4. 闭包](#4-闭包)
        - [5.for in, for of, forEach 用法和区别](#5for-in-for-of-foreach-用法和区别)
        - [6.浅拷贝和深拷贝](#6浅拷贝和深拷贝)
        - [7.原型](#7原型)
        - [8.继承](#8继承)
        - [9.AMD 和 CMD](#9amd-和-cmd)
        - [10.setTimeout, setInterval, requestAnimationFrame](#10settimeout-setinterval-requestanimationframe)
        - [11.宏任务/微任务](#11宏任务微任务)
        - [12.浏览器存储](#12浏览器存储)
        - [13.重绘和回流](#13重绘和回流)
        - [14.伪数组转真数组](#14伪数组转真数组)
        - [15.throttle 节流](#15throttle-节流)
        - [16.debounce 防抖](#16debounce-防抖)
        - [17.移动端 1px](#17移动端-1px)
        - [18.Object.seal() 和 Object.freeze()](#18objectseal-和-objectfreeze)
        - [19.class](#19class)
        - [20.Promise](#20promise)
        - [21.数组找最大值](#21数组找最大值)
        - [22.数组去重](#22数组去重)
        - [23.event loop](#23event-loop)
        - [24.路由、组件懒加载](#24路由组件懒加载)

<!-- /TOC -->
# js

## 1. let, var, const 的区别

```js
1.块级作用域 -- let const
    块级作用域解决了es5的两个问题：
    * 内层变量可能覆盖外层变量
    * 用来计数的循环变量泄漏为全局变量
2.变量提升 -- var
    即变量可以在声明前使用
3.给全局添加属性 -- var
    var声明的变量是全局变量，并且会将该变量添加为全局对象的属性
4.重复声明 -- var
    var可重复声明变量，且覆盖之前声明的变量
5.暂时性死区 -- let const
    变量在声明前，不可用，称之为暂时性死区
6.初始值设置 -- const
    const声明变量必须设置初始值，var和let可不设置
7.指针指向 -- const
    let创建的变量可以更改指针指向（重新赋值），const声明的变量不允许改变指针指向（const用来声明常量->引用数据类型可以重新赋值，因为引用的是地址）
```

### 2. typeof 和 instanceof 的区别

```js
typeof 可以用来判断原始类型，除了null
    在判断对象数据类型时只能准确判断函数，其他数据类型只会返回object
instanceof 通过原型链的方式判断数据类型
    可以正确判断对象数据类型，但无法准确判断原始数据类型
```

### 3. this 指向

```js
1. 箭头函数：this指向的是外层作用域this的值，且不会被改变（包裹它的第一个原始函数）
let obj = {
    a: function(){
        console.log(this)
    }
    b: () => {
        console.log(this)
    }
}
obj.a() // this指向的是obj （对象内部方法的this指向调用这些方法的对象，即最近的对象）
obj.b() // this指向的是window
//-------------------------------------------------------
2. 普通函数的this指向取决于函数如何被调用
   new => this指向构造函数下创建的实例
   没有new => 1.obj.func： this指向obj  //有调用者
             2. func： this 指向window（全局环境下的this，始终指向全局对象window）
             3. 异步调用： 指向window
//例1.
var length = 1
function fn(){
    console.log(this.length)  // this.length => lists.length
}
var lists = [fn, 2, 3, 4]
lists[0]() // fn() 有调用者，调用者是lists => 4

var f = list[0]
f() // 无调用者 =>  1

//例2.
var length = 1
function fn(){
    console.log(this.length)   // 1，5
}
var obj = {
    length:100,
    action:function(callback){
        callback()      // callback() => fn()
        arguments[0]()
        // 第0个参数 arguments[0] => fn
        // arguments[0]() => fn() => this.length =>arguments.length
    }
}
var arr = [1,2,3,4]
obj.action(fn,...arr)
//-------------------------------------------------------
3.改变this指向：call bind apply
    this指向第一个参数，如第一个参数是空则指向window
    // call和apply区别在于传参方式（call的性能更好一些）
    fn.call(a,b,c) // 第一个是this指向，第二三个是传给函数的实参
    fn.apply(a,[b,c]) // 第一个是this指向，第二三个实参是传入带下标的集合，数组或类数组

    // bind和call的区别在于立即执行还是等待执行,bind是创建一个新的函数，而call和apply是用来调用函数
    //bind()方法：改变fn中的this，fn并不执行
    fn.bind(obj, 1, 2); //不会有任何输出
    fn.bind(obj, 1, 2)();//调用后才会有输出
```

### 4. 闭包

```js
闭包指有权访问另一个函数中的变量的函数，可以通过在一个函数内部返回一个匿名函数来创建闭包。
（通过了解执行上下文，作用域，作用域链来了解闭包概念，执行上下文是动态的，但作用域链是静态的）
```

### 5.for in, for of, forEach 用法和区别

```js
// for in 用于遍历数组或对象
for (let i in obj) {
  console.log(i); //数组的下标或对象的key
  console.log(obj[i]); //值
}
// for of 用于遍历数组或字符串(ES6新增的方法)
for (let i of obj) {
  console.log(i); //值
}
// forEach用于遍历数组
forEach((item, i) => {
  console.log(item); //数组的每一项对象
  console.log(i); // 下标
});
```

### 6.浅拷贝和深拷贝

```javascript
// 浅拷贝  拷贝的是地址不是数据
var newObj = {};
for (let key in obj) {
  newObj[key] = obj[key];
}

// 深拷贝
function deepClone(obj) {
  let cloneObj = Array.isArray(obj) ? [] : {};
  if (obj && typeof obj == "object") {
    //先判断是否为对象
    for (var key in obj) {
      if (obj.hasOwnProperty(key)) {
        //hasOwnProperty 判断对象自身是否具有指定名称的属性
        if (obj[key] && typeof obj[key] == "object") {
          cloneObj[key] = deepClone(obj[key]); // 如果是引用类型，递归拷贝
        } else {
          cloneObj[key] = obj[key]; //基本类型，直接赋值
        }
      }
    }
  }
  return cloneObj;
}
```

### 7.原型

```js
/**
 * prototype: 原型 -> 函数的一个显式原型属性: 对象{}
 * __proto__: 原型链（链接点） -> 对象的一个隐式原型属性: 对象{}
 * 对象的__proto__ 保存着 该对象的构造函数的prototype
 */

function Obj(){
    this.a = 1
    this.c = 222
}
Obj.prototype.b = 2
Object.prototype.c = 3
let obj = new Obj()
console.log(obj.b)  // 222  //先找自身是否有这个属性，再往原型链__proto__上找
//原型链
/**
 * obj {
 *    a: 1,
 *    c: 222,
 *    __proto__: Obj.prototype = {
 *      b: 2,
 *      __proto__: Object.prototype = {
 *          c: 3
 *        // 最顶层的Object没有__proto__ 即 null
 *      }
 *   }
 * }
 *
 */

// hasOwnProperty 判断对象自身是否有指定名称的属性
 obj.hasOwnProperty('a') // true
 obj.hasOwnProperty('b') // false
 obj.hasOwnProperty('c') // true
 'a' in obj   // true
 'b' in obj   // true
 'c' in obj   // true

// Function和Object的特殊性
Obj.__proto__ === Function.prototype
Function.__proto__ === Function.prototype  // 底层既定的
typeof Object  // function
   Object.__proto__ === Function.prototype
=> Object.__proto__ === Function.__proto__
```

### 8.继承

```js
// 使用class实现继承
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  introduce() {}
}
class Student extends Person {
  constructor(name, age, sex) {
    super(name, age); // 属性通过父类.call(this,参数）或 super（）继承 super和class搭配使用
    this.sex = sex;
  }
  intro() {}
}
let stu = new Student("name", "age"); //实例化Student这个类
console.log(stu);
stu.introduce(); // 子类通过原型继承了父类的方法
stu.intro();
```

### 9.AMD 和 CMD

插件加载方式

AMD 依赖关系前置，在定义模块时就声明其依赖的模块

CMD 按需加载依赖就近，只有在用到某个模块时再 require

### 10.setTimeout, setInterval, requestAnimationFrame

```js
setTimeout延时器
// 只执行一次
setInterval定时器
// 定时不断调用函数
语法： 两者都需要两个参数(表达式, 时间)  //表达式可以是函数，也可以是字符串

setInterval的缺点：
1.无视代码错误，持续调用
2.无视网络延迟
3.不保证执行，如果调用的函数需要花很长时间，那调用就会被忽略

=> 不建议使用setInterval, 可以使用requestAnimationFrame执行需要异步执行的代码
```

### 11.宏任务/微任务

| 宏任务                | 微任务                         |
| :-------------------- | :----------------------------- |
| setTimeout            | queueMicrotask                 |
| setInterval           | MutationObserver               |
| MessageChannel        | Promise.[ then/catch/finally ] |
| I/O，事件队列         | process.nextTick               |
| setImmediate          |                                |
| script                |
| requestAnimationFrame |

```js
function func(num) {
  return function () {
    console.log(num);
  };
}
setTimeout(func(1));

async function async3() {
  await async4();
  console.log(8);
}

async function async4() {
  console.log(5);
}

async3();

function func2() {
  console.log(2);

  async function async1() {
    await async2();
    console.log(9);
  }

  async function async2() {
    console.log(5);
  }

  async1();

  setTimeout(func(4));
}

setTimeout(func2);
setTimeout(func(3));

new Promise((resolve) => {
  console.log("Promise");
  resolve();
})
  .then(() => console.log(6))
  .then(() => console.log(7));

console.log(0);
//5, promise, 0, 8, 6, 7, 1, 2, 5, 9, 3, 4
```

### 12.浏览器存储

```js
cookie  --  内存小4k，每次请求都会自动携带
localStorage  -- 除非清理，否则永久存在，5m，需手动设置到请求体
sessionStorage  -- 页面关闭就清理，5m，需手动设置到请求体
indexDB  --  除非清理，否则永久存在，大小根据本地硬件大小，需手动设置到请求体
```

### 13.重绘和回流

```js
// 重绘：
当节点需要改变外观但不改变布局时，称为重绘（eg.改变background-color...）
// 回流：
布局或几何属性发生改变时，称为回流（eg. 改变盒子大小、字体、定位、盒模型、添加删除样式/标签...）

回流必定发生重绘，但重绘不一定引发回流。回流的成本高于重绘，改变父节点的子节点很可能导致父节点的一系列回流

// 减少重绘与回流的css方案：
1. 使用transform 替代 top
2. 使用visibility 替代 display: none
3. 避免设置多层内联样式，css选择符从右往左匹配，避免节点层级过多
// 减少重绘与回流的js方案：
1.避免频繁操作样式，频繁操作DOM
2.避免频繁读取会引起回流和重绘的属性
```

### 14.伪数组转真数组

```js
Array.from()：将一个类数组对象或可遍历对象转换成一个真数组
//语法：
let arr = {
    '0': 'a',
    '1': 'b',
    '2': ['c', 'd', 'e']
    'length': 3
}
let newArr = Array.from(arr)
console.log(newArr)  // ['a', 'b', ['c', 'd', 'e']]
//基本要求：
1.具有length属性的对象，否则将转换成一个空数组
2.属性名必须为数值或字符串型的数字,否则将转换成一个值为undefined的数组

// Array.from还可以接受第二个参数，作用类似于数组的map方法，用来对每个元素进行处理，将处理后的值放入返回的数组
// Array.from(set, item => item + 1)
```

### 15.throttle 节流

```js
节流： 规定时间间隔内后续动作只执行一次
// 应用场景
1.高频事件，eg.快速点击，鼠标滑动，resize事件，scroll事件
2.下拉加载
3.视频播放记录时间
4.射击游戏的mousedown/keydown事件（单位时间只发射一颗子弹）
// 代码实现思路： setTimeout
function count(){
    console.log("计数")
}
// setTimeout写法
function throttle(fn, delay){
    let timer = null
    return function(){
        if(timer !== null) return
        timer = setTimeout(()=>{
            fn.apply(this, arguments)
            timer = null
        },delay)
    }
}
// 时间戳写法
function throttle(fn, delay){
    let pre = 0
    return function(){
        let new = new Date()
        if(new - pre > delay){
            fn.apply(this, arguments)
            pre = now
        }
    }
}
throttle(count, 2000)
```

### 16.debounce 防抖

```js
防抖：规定时间内，频繁触发事件，只执行最后一次
// 应用场景
1.搜索框搜索输入
2.文本编辑器实时保存
// 代码实现思路： setTimeout
function count(){
    console.log("计数")
}
function debounce(fn, delay){
    let timer   // 外部变量
    return function(){  // 闭包
        clearTimeout(timer) // 去掉之前的无效任务
        timer = setTimeout(()=>{    // 创建新任务
            // fn()  //直接执行函数可能引发作用域问题
            fn.apply(this, arguments)
        }, delay)
    }
}
debounce(count, 2000)
```

### 17.移动端 1px

实际开发过程中， 手机端 1px 会比实际看上去粗一点

```css
/* 最优解决方案： */
transform: scale(0.5);
```

### 18.Object.seal() 和 Object.freeze()

```js
Object.seal()  密封
// 封闭一个对象，阻止添加新属性，并将现有属性标为不可配置
Object.freeze()  冷冻
// 冻结一个对象，不可被修改、添加、删除，其原型也不能被修改
```

### 19.class

```js
//语法：
class Test(){
    static fn(){}  // 静态方法
    constructor(){}   // 构造函数
    fn2(){} // 方法
    set fn3(){}
    get fn3(){}
}
// class里面的静态方法只能用类调用，不能用实例调用
```

### 20.Promise

```js
异步编程;
```

### 21.数组找最大值

```js
Math.max(...arr);
```

### 22.数组去重

```js
new Set(...arr);
```

### 23.event loop

```js
事件循环机制event loop
浏览器的事件循环分为同步任务和异步任务，所有同步任务都在主线程上执行，形成一个函数调用栈（执行栈）；
异步任务则先放在任务队列(task queue)中，任务队列分为宏任务macro-task,微任务micro-task；
ps:执行微任务过程中，又产生了微任务，则会加入整个队列的队尾，在当前周期中执行。
```

### 24.路由、组件懒加载

```js
pros: 
1.首屏组件加载速度更快，解决白屏问题
2.按需加载

1.vue异步组件：
component:resolve => require(['组件路径],resolve)

2.ES6的import():
const 组件名=() => import('组件路径');

3.webpack的require.ensure()
```
