promise 的构造函数是同步执行，promise.then 中的函数是异步执行（微任务）。
使用 async 定义的函数，当它被调用时，它返回的其实是一个 Promise 对象。
而 await 又做了什么处理呢？从字面意思上看 await 就是等待，await 等待的是一个表达式，这个表达式的返回值可以是一个 promise 对象也可以是其他值。
```javascript
async function async1() {
    console.log("async1 start");//2
    await async2();
    console.log("async1 end");//6
}
async function async2() {
    console.log( 'async2');//3
}
console.log("script start");//1
setTimeout(function () {
    console.log("settimeout");//8
},0);
async1();
new Promise(function (resolve) {
    console.log("promise1");//4
    resolve();
}).then(function () {
    console.log("promise2");//7
});
console.log('script end');//5
```
![](https://image-static.segmentfault.com/283/402/2834025787-b4ef99682683a78e_fix732)
图片来源：https://segmentfault.com/a/1190000038328662

1.setTimeout: setTimeout 的回调函数放到宏任务队列里，等到执行栈清空以后执行;

2.Promise: Promise 本身是同步的立即执行函数，当执行 resolve 或者 reject 的时候，此时是异步操作。换句话说，当一个 Promise 对象的状态变为 fulfilled（已完成）或 rejected（已拒绝）时，相应的回调函数会被放入到任务队列中等待执行。这个过程如下：

    2-1.当你调用一个返回 Promise 的异步函数时，该函数会立即返回一个未解决的 Promise 对象。

    2-2.一旦异步操作完成，异步函数内部会通过调用 resolve 或 reject 来解决或拒绝这个 Promise。

    2-3.当 Promise 的状态变化时（即调用 resolve 或 reject），如果你已经使用.then 或.catch 为这个 Promise 设置了回调函数，这些回调会被放入到任务队列中等待主线程空闲时执行。

3.async,await 函数返回一个 Promise 对象，当函数执行的时候，一旦遇到 await 就 会先返回，等到触发的异步操作完成，再执行函数体内后面的语句。可以理解为， 是让出了线程，跳出了 async 函数体。