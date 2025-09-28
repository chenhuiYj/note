优势

1、localStorage 存储空间比较大，相比 cookie 的 4K，localStorage 可以存储 5M 大小的数据，也可以节约带宽

2、localStorage 没有时间限制，数据一直有效，只能手动删除

劣势

 1、浏览器兼容性问题，并且在 IE8 以上的 IE 版本才支持 localStorage 这个属性。

 2、目前所有的浏览器中都会把 localStorage 的值类型限定为 string 类型，这个在对我们日常比较常见的JSON对象类型需要一些转换

 3、localStorage 在浏览器的隐私模式下面是不可读取的

 4、如果存储 localStorage 过多的话会消耗内存空间，会导致页面变卡。

 5、localStorage不能被爬虫抓取到
 
> localStorage 与 sessionStorage 的唯一一点区别就是 localStorage 属于永久性存储，而 sessionStorage 属于当会话结束的时候，sessionStorage 中的键值对会被清空

使用

使用 localStorage 的时候，需要判断当前浏览器是否支持 localStorage
```js
if(! window.localStorage){
    alert("浏览器不支持localstorage");
    return false;
}else{
    //逻辑代码
}
```
```js
if(!window.localStorage){
    alert("浏览器不支持localstorage");
}else{
    //写入a字段
    localStorage["a"]=1;
    //写入b字段
    localStorage.b=2;
    //写入c字段
    localstorage.setItem("c",3);
    console.log(typeof localstorage["a"]); //String
    console.log(typeof localstorage["b"]);//String
    console.log(typeof localstorage["c"]);//String
    // 取值方式1
    console.log(localstorage.a);
    // 取值方式2
    console.log(localstorage["b"]);
    // 取值方式3
    console.log(localstorage.getItem("c"));
}
```
删除
```js
//将localStorage的所有内容删除
localStorage.clear();
//删除某个key的值
localStorage.setItem("a",1);
localStorage.removeItem("a");
```