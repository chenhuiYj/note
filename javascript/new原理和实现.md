
## 步骤
1.创建一个空对象

2.把对象的__proto__属性指向父类的 prototype，空对象将拥有父类的 prototype 的属性

3.执行构造函数，并且把 this 指向创建的对象，并且父类里面 this 的赋值（父类的属性）将赋值到新创建的对象。

4.返回对象
```javascript
function Base() {
    this.id = 'base';
}

Base.prototype.toString = function() {
    return this.id;
};

var obj = new Base();

//创建空对象{}，obj
//把 obj 的__proto__属性赋值到 Base 的 prototype 上面。
//执行 Base.call(obj,arguments),把 this 指向 obj，把 Base 的属性（this 的赋值）作用到 obj 上面
//返回对象

function base(name,age) {
    this.name=name
    this.age=age
}
base.prototype.sayName=function(){
    console.log(this.name)
}
function newFn(fn,...arg){
    let obj={}
    obj.__proto__=fn.prototype
    fn.call(obj,...arg)
    return obj
}

let obj1=newFn(base,'chen',31)
let obj2=newFn(base,'lu',33)
```