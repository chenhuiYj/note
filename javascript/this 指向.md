关于 JavaScript 中函数里面的 this 指向，其实就是两句话可以概括
1.如果不是箭头函数，哪个对象调用函数，this指向哪个对象
2.如果是箭头函数，this指向外层第一个普通函数。如果没有被普通函数包裹，指向window

<img width="864" alt="WX20200528-175724@2x" src="https://user-images.githubusercontent.com/27403818/83127589-ef054180-a10c-11ea-9731-a94dc8fea041.png">

#### ①
这个很好懂，比如如下面代码，this 都是指向 window。普通函数直接调用，不管在什么作用域调用，都是 window
```
var fn1=function(){
    console.log(this) 
}
function fn2(){
    console.log(this) 
}
 fn1()  //window
 fn2()  //window
```

#### ②

apply, call, bind 三个函数都是改变 this 指向的，指向的都是第一个参数，如果第一个参数写 null 或者 undefined，this 指向 window。

```
let obj1={
    a:222
};
let obj2={
    a:111,
    fn:function(){
        console.log(this.a);
    }
}
obj2.fn.call(obj1) //222
obj2.fn.apply(obj1) //222
obj2.fn.bind(obj1)() //222
```

#### ③
如果是对象方法的调用，哪个对象调用函数，this 指向哪个对象
```
let obj={
    a(){
        console.log(this)
    },
    b:()=>{
        console.log(this)
    },
    c:{
       d(){
            console.log(this)
        } 
    }
}

obj.a() //obj
obj.b() //window
obj.c.d() //obj.c
```
obj.a() 和 obj.c.d() 相信都很好理解，至于 obj.b() 之所以指向 window , 是因为 obj.b 是一个箭头函数，而且 obj.b 没有被普通函数包裹，所以 obj.b 指向 window。