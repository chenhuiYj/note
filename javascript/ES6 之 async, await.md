本质上也是利用 promise，下面两段代码是等同的。
```javascript
async function test(){
	return ‘hello async’
}
//第二段代码
async function test(){
	return new Promise((resolve,reject)=>{
		resolve(‘hello async’)
	})
}
```
自然 async 也是前端微任务
```javascript
async function testAsync() {
    return "hello async";
}
let result = testAsync();
console.log(result)//Promise {<fulfilled>: 'hello async'}

async function testAsync1() {
    console.log("hello async");
}
let result1 = testAsync1();
console.log(result1);// Promise {<fulfilled>: undefined}
```
从结果中可以看到 async 函数返回的是一个 promise 对象，如果在函数中 return 一个直接量，async 会把这个直接量通过 Promise.resolve() 封装成 Promise 对象。如果 asyn 函数没有返回值，结果返回 Promise.resolve(undefined)

#### 相比于 Promise，它能更好地处理 then 链

下面实例：step1，step2，step3按顺序执行
```javascript
function takeLongTime(n) {
    return new Promise(resolve => {
        setTimeout(() => resolve(n + 200), n);
    });
}

function step1(n) {
    console.log(`step1 with ${n}`);
    return takeLongTime(n);
}

function step2(n) {
    console.log(`step2 with ${n}`);
    return takeLongTime(n);
}

function step3(n) {
    console.log(`step3 with ${n}`);
    return takeLongTime(n);
}
```
Promise 方式
```javascript
function doIt() {
    console.time("doIt");
    const time1 = 300;
    step1(time1)
        .then(time2 => step2(time2))
        .then(time3 => step3(time3))
        .then(result => {
            console.log(`result is ${result}`);
        });
}
doIt();
```
链式调用 then，使得代码难以阅读，维护。

如果用 async/await 来实现的话，会是这样：
```javascript
async function doIt() {
    console.time("doIt");
    const time1 = 300;
    const time2 = await step1(time1);
    const time3 = await step2(time2);
    const result = await step3(time3);
    console.log(`result is ${result}`);
}
doIt();
```
仍然是三个步骤，但每一个步骤都需要之前每个步骤的结果。Pomise 的实现看着很晕，传递参数太过麻烦。
```javascript
function doIt() {
    console.time("doIt");
    const time1 = 300;
    step1(time1)
        .then(time2 => {
            return step2(time1, time2)
                .then(time3 => [time1, time2, time3]);
        })
        .then(times => {
            const [time1, time2, time3] = times;
            return step3(time1, time2, time3);
        })
        .then(result => {
            console.log(`result is ${result}`);
        });
}
doIt();
```
用 async/await 来写
```javascript
async function doIt() {
    console.time("doIt");
    const time1 = 300;
    const time2 = await step1(time1);
    const time3 = await step2(time1, time2);
    const result = await step3(time1, time2, time3);
    console.log(`result is ${result}`);
}
doIt();
```