## 1.typeof

只能识别基础类型和引用类型，注意：null、 array、 DOM 的判断都显示 object
```javascript
console.log(typeof 123); // "number"
console.log(typeof "hello"); // "string"
console.log(typeof true); // "boolean"
console.log(typeof undefined); // "undefined"
console.log(typeof null); // "object"
console.log(typeof {}); // "object"
console.log(typeof []); // "object"
console.log(typeof function(){}); // "function"
console.log(typeof NaN); // "number"
console.log(typeof document.all); // "undefined"
console.log(typeof document.body); // "object"
```
## 2.constructor

指向创建该实例对象的构造函数，但 null 和 undefined 没有 constructor，以及 constructor 可以被改写，不太可靠
```javascript
const arr = [1, 2, 3];
console.log(arr.constructor === Array) // true

const obj = {name: "守候", age: 22};
console.log(obj.constructor === Object) // true


String.prototype.constructor = function fn() {
  return {};
}

// constructor 可以被改写

let Parent =function(){}
let Sou=new Parent()
Sou.constructor===Object //false
Sou.constructor===Parent //true
```
## 3.instanceof 

object instanceof Type ：判断 object 是不是 Type 的实例（检测构造函数的原型 Type.prototype 是否存在于实例对象 object 的原型链上）。但只可用来判断引用数据，number，string，boolean 等基本类型不是构造函数实例，不能用 instanceof 判断。
```javascript
let dataNumber=123
dataNumber instanceof Number //false

let dataStr=''
dataStr instanceof String //false

let dataArr=[]
dataArr instanceof Array //true

let dataObj={}
dataObj instanceof Object //true
```
## 4.Object.prototype.toString.call()

根据 Object.prototype.toString() 方法得到对象内部属性 [[Class]]，通过 call 或者 apply 改变 this 的指向，使 this 指向传进来的值，返回传进来的值的内部属性[[Class]]
```javascript
Object.prototype.toString.call({})              // '[object Object]'
Object.prototype.toString.call([])              // '[object Array]'
Object.prototype.toString.call(() => {})        // '[object Function]'
Object.prototype.toString.call(‘守候’)        // '[object String]'
Object.prototype.toString.call(1)               // '[object Number]'
Object.prototype.toString.call(NaN)             // '[object Number]', NaN 也是一种数字
Object.prototype.toString.call(true)            // '[object Boolean]'
Object.prototype.toString.call(Symbol())        // '[object Symbol]'
Object.prototype.toString.call(null)            // '[object Null]'
Object.prototype.toString.call(undefined)       // '[object Undefined]'
Object.prototype.toString.call(document.body)   //[object HTMLBodyElement]
Object.prototype.toString.call(document.all)     //[object HTMLAllCollection]
Object.prototype.toString.call(new Date())      // '[object Date]'
Object.prototype.toString.call(Math)            // '[object Math]'
Object.prototype.toString.call(window)          //[object Window]
Object.prototype.toString.call(new Set())       // '[object Set]'
Object.prototype.toString.call(new WeakSet())   // '[object WeakSet]'
Object.prototype.toString.call(new Map())       // '[object Map]'
Object.prototype.toString.call(new WeakMap())   // '[object WeakMap]'
function fn() {
    console.log(Object.prototype.toString.call(arguments))   //[object Arguments]
}
```
必须要 Object.prototype.toString，不能是其他的.prototype.toString，因为只有 Object.prototype.toString 会返回数据类型
```javascript
Function.prototype.toString.call(function(){return 1})   //'function(){return 1}'

Number.prototype.toString.call([])  //VM772:1 Uncaught TypeError: Number.prototype.toString requires that 'this' be a Number
```