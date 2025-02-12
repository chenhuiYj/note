## 1.原型链继承

定义：重写一个某个构造函数的 prototype，使得 prototype 指向父类的一个实例。通过原型链串联起来。
```javascript
function Animal(){
    this.like=['sleep','play','eat']
}

function dog(){
    this.name='dog'
    this.friends={
        old:['zr','rt'],
        new:['by']
    }
}

dog.prototype=new Animal()

let dogF=new dog()

let dogB=new dog()

dogF.name='F'
dogF.friends.old.push('yx')

dogB.name='B'
dogB.friends.old.push('zd')

console.log(dogF)//{name:'F',friends:{old:['zr','rt','yx'],new:['by']}}
console.log(dogB)//{name:'F',friends:{old:['zr','rt','zd'],new:['by']}}
console.log(new dog())//{name:'F',friends:{old:['zr','rt'],new:['by']}}


dogF.like.push('run')
console.log(dogF.like)//['sleep', 'play', 'eat', 'run']
console.log(dogB.like)//['sleep', 'play', 'eat', 'run']
console.log(new dog().like)//['sleep', 'play', 'eat', 'run']
```
因为 dogF 没有 like 属性，就会通过__proto__向上一级（dog.prototype）查找，dog.prototype，而 dog.prototype 指向 new Animal()，所以实际上，修改了 dog.prototype，而 like 又是一个引用类型对象，dogF 和 dogB 保存的实际上是 like 的一个指针。执行 dogF.like.push('run')

修改了 like 属性后，就会修改 dog.prototype 上面的值。所以 dogB.like，new dog()的值都改变了。

```javascript
dogF.gender='1'
dogB.gender='2'
console.log(dogF.gender)//1
console.log(dogB.gender)//2
console.log(new dog().gender)//0
```
因为 dogF 和 dogB 没有 gender 属性，就会通过__proto__向上一级（dog.prototype）查找，而 gender 只是一个基本类型的数据，dogF 和 dogB 保存的就是原来的值，所以修改 dogF 和 dogB 的 gender 属性不会影响原来的值。

### 优点：
实现相对简单
子类实例可以直接访问到父类实例或父类原型上的属性方法
### 缺点：
实例无法向父类的父类传递参数。（dogF 无法向 Animal 传递参数）
父类所有的引用类型属性都会被实例出来的对象共享，所以修改一个实例对象的引用类型属性，会导致所有实例对象受到影响（执行 dogF.like.push('run')后，new dog()的值都会被影响）

## 2.盗用构造函数

定义：在子类构造函数中，调用父类构造函数方法，但通过 call 或者 apply 方法改变了父类构造函数内 this 的指向，使得子类实例出来的对象，自身拥有来自父类构造函数的方法跟属性，且分别独立，互不影响。
```javascript
function Animal(name,friends){
    this.name=name
    this.friends=friends
    this.likes=['eat','sleep']
}
Animal.prototype.play='玩'


function dog(name,friends,gender){
    Animal.call(this,name,friends)
    this.gender=gender
}

let dogF=new dog('dogF',['zr','rt'],'0')
let dogB=new dog('dogB',['zr','rt'],'1')

dogF.friends.push('yx')
dogB.friends.push('zd')

console.log(dogF.friends)// ['zr', 'rt', 'yx']
console.log(dogB.friends)//['zr', 'rt', 'zd']
```
修改实例 dogF 不会影响到实例 dogB（因为实例出来的对象，所有的属性、方法都是添加到实例对象自身，而不是添加到实例对象的原型上，它们是完全独立，指向的都是不同的地址）
```javascript
console.log(dogF.play)//undefined
console.log(dogB.play)//undefined
```
子类的原型 dog.prototype 与父类原型 Animal.prototype 没有打通。因为 dog.prototype.__proto__直接指向了 Object.prototype，如果打通了的话，应该是 dog.prototype.__proto__指向 Animal.prototype，这也是为什么实例 dog 没有继承父类 play 属性的原因，因为访问不到。

### 优点

实例化时，可以传参

子类通过 call 或 apply 方法，将父类里的所有属性、方法复制到实例对象的自身，而不是共享原型链上同一个属性，所以修改一个实例对象的引用类型属性时，不会导致所有实例对象受到影响

### 缺点

法继承父类原型上的属性与方法

## 3.组合继承

定义：组合继承顾名思义就是，利用原型链继承跟借用构造函数继承相结合，而创造出来的一种新的继承方式。

利用原型链继承，实现实例对父类原型（Animal.protoytype）上的方法与属性继承；利用借用构造函数继承，实现实例对父类构造函数（function Animal() {}）里方法与性的继承，并且解决原型链继承的缺陷。
```javascript
function Animal(gender,likes){
    this.gender=gender
    this.like=likes
    this.homes=['bj']
}

Animal.prototype.run = function() {
  console.log('跑步');
}

Animal.prototype.runArea = ['a','b']

function dog(gender,likes){
//第二次调用 Animal
    Animal.call(this,gender,likes)
    this.name='dog'
}

//第一次调用 Animal
dog.prototype=new Animal()

let dogF=new dog('0',['eat'])

let dogB=new dog('1',['sleep'])


dogF.like.push('play')
dogF.homes.push('gz')
dogF.runArea.push('c')

dogB.like.push('run')
dogB.homes.push('sh')
dogB.runArea.push('d')

console.log(dogF)//{gender: "0",homes: ['bj', 'gz'],like: ['eat', 'play'],name: "dog"}
console.log(dogB)//{gender: "1",homes: ['bj', 'sh'],like: ['sleep', 'run'],name: "dog"}
console.log(new dog('2',['dream']))//{gender: "2",homes: ['bj'],like: ['dream'],name: "dog"}

console.log(dogF.runArea)// ['a', 'b', 'c', 'd']
console.log(dogB.runArea)// ['a', 'b', 'c', 'd']
console.log(new dog().runArea)// ['a', 'b', 'c', 'd']
console.log(new Animal().runArea)// ['a', 'b', 'c', 'd']
//修改的实际是 Animal.prototype.runArea
```

### 优点：

利用原型链继承，将实例 cat、子类 Cat、父类 Animal 三者的原型链串联起来，让实例对象继承父类原型 Animal.prototype 的方法与属性

利用借用构造函数继承，将父类构造函数 function Animal() {}的属性、方法添加到实例自身上，解决原型链继承，实例修改引用类型属性时对后续实例影响问题

利用构造函数继承，实例化对象时，可传参

### 缺点：

两次调用父类构造函数 function Animal() {}（第一次在 new Animal()时候调用，第二次在子类 dog 构造函数内调用）

Animal 实例自身拥有的属性，子类 dog.prototype 里也会有，造成不必要的浪费（因为 dog.prototype 被重写为 new Animal()了，new Animal()是父类的一个实例，也有 home、gender、like 属性）

## 4.原型继承

定义：原型式继承的的实现方法与普通继承的实现方法不同，原型式继承并没有使用严格意义上的构造函数，而是借助原型可以基于已有的对象创建新对象，同时还不必因此创建自定义类型。

```javascript
function objectCopy(obj) {
  function Fun() { };
  Fun.prototype = obj;
  return new Fun()
}

let person = {
  name: "yhd",
  age: 18,
  friends: ["jack", "tom", "rose"],
  sayName:function() {
    console.log(this.name);
  }
}

let person1 = objectCopy(person);
person1.name = "wxb";
person1.friends.push("lily");
person1.sayName(); // wxb

let person2 = objectCopy(person);
person2.name = "gsr";
person2.friends.push("kobe");
person2.sayName(); // "gsr"

console.log(person.friends); // ["jack", "tom", "rose", "lily", "kobe"]
```

对比原型链继承方式，其实这两种方式差不多，所以它要跟原型链继承存在一样的缺点，但是实现起来比原型式继承更加简洁方便一些。如果我们只是想让一个对象跟另一个对象保持类似，原型式继承可能更加舒服，因为它不需要像原型链继承那样大费周章。

### 优点：

实现比原型链继承更简洁（不需要写什么构造函数了，也不需要写子类 dog，直接父类继承 Animal）

子类实例可以访问到父类的属性方法

### 缺点：

父类所有的引用类型属性都会被实例出来的对象共享，所以修改一个实例对象的引用类型属性，会导致所有实例对象受到影响

实例化时，不能传参数

## 5.寄生式继承

定义：在原型式继承基础上进行一个小封装，增强了一下实例出来的对象

```javascript
function objectCopy(obj) {
    function foo() {}
    foo.prototype=obj
    return new foo()
}

function createObject(obj1) {
    let clone = objectCopy(obj1)
	//增强了一下实例出来的对象
    clone.getName=function () {
        console.log('say what')
    }
    return clone
}

var person={
    name:'john',
    friends:['lily','mike']
}

var person1=createObject(person)
var person2=createObject(person)

person1.friends.push('jordan')
person2.friends.push('nowiziki')

console.log(person1)
console.log(person2)
```

### 优点：

实现比原型链继承更简洁
子类实例可以访问到父类的属性方法

### 缺点：

1.父类所有的引用类型属性都会被实例出来的对象共享，所以修改一个实例对象的引用类型属性，会导致所有实例对象受到影响

2.实例化时，不能传参数

3.寄生式继承优缺点跟原型式继承一样，但最重要的是它提供了一个类似工厂的思想，是对原型式继承的一个封装。前面我们说到组合继承还是会有一些缺陷，通过原型式继承跟寄生式继承，我们可以利用这两个继承的思想，来解决组合继承的缺陷，它就是寄生组合式继承。

## 6.寄生组合继承
```javascript
function Car(obj){
    this.color=obj.color
    this.price=obj.price
    this.model=obj.model
    this.drive=function(){
        console.log('开车')
    }
}

function NEV(obj) {
    Car.call(this,obj)
    this.isNEV=true;
}

function ICEV(obj) {
    Car.call(this,obj)
    this.isNEV=false;
    this.engine=obj.engine
}

function cloneObject(obj) {
    function fn() {}
    fn.prototype=obj
    return new fn()
}

function inherit(SubFn,Fn) {
    let prototype = cloneObject(Fn.prototype)
    prototype.constructor=SubFn
    SubFn.prototype=prototype
}

inherit(NEV,Car)
inherit(ICEV,Car)

NEV.prototype.showType=function () {
    console.log('这是新能源车')
}

ICEV.prototype.showEngine=function () {
    console.log('引擎型号：'+this.engine)
}

let modelY=new NEV({color:'黑色',price:700000,model:"model Y"})
let bmw=new NEV({color:'银色',price:700000,model:"730", engine:"I3"})
```

### 优点
实现比较理想，只调用一次 Car
### 缺点
实现较为复杂

## 7.class
```javascript
class Car {
    constructor(obj){
        this.color=obj.color
        this.price=obj.price
    }

    drive(){
        console.log('开车')
    }
}

class NEV extends Car {
    constructor(obj){
        super(obj)
        this.models=['x','y','t']
        this.isNEV=true
    }
}

let modelY=new NEV({color:"白色",price:400000})
let modelX=new NEV({color:"银色",price:400000})
```

### 优点
写法简洁
### 缺点
写法有缺陷，无法完成实现 es5的功能

