Promise 是异步编程的一种解决方案，比传统的解决方案——回调函数和事件——更合理和更强大。所谓 Promise，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。从语法上说，Promise 是一个对象，从它可以获取异步操作的消息。

promise 的构造函数是同步执行，promise.then 中的函数是异步执行（微任务）。

## 1.优略势

试想一下，如果有以下的一个场景，一个人先起床，起床了就去刷牙吃饭。如果觉得太困了就继续睡觉。代码大概如下
```javascript
getUp({
    success: function(res) {
        brushTeeth({
            success: function(res) {
                eat()
            }
        })
    },
    fail: function(err) {
        sleep()
    }
})
```
现实开发中，业务代码肯定比 getUp，brushTeeth，eat 复杂很多很多。这就可能会导致回调地狱的出现，代码的可维护性也很差。而 promise 的出现就解决了这个问题。
```javascript
let result = true

let getUp = new Promise((resolve, reject) => {
    if (result) {
        resolve('起床成功')
    } else {
        reject('太困了，继续睡觉')
    }
})

getUp.then(res=>{
    return brushTeeth()
})
.then(res=>{
    return eat()
})
.catch(err=>sleep())
```
Promise 的 then 的链式调用就很好的避免了回调地狱，以及增加了代码的可读性。

## 2.状态：

promise 拥有 pending-进行中，fulfilled-已完成，rejected-已错误三种状态。状态不可逆，一个 promise，一旦从 pending 进入 fulfilled 或者 rejected，那么就不能再回到 pending。
```javascript
let result = true
let list = [1, 2, 3]

function getData() {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            if (result) {
                resolve(list)
            } else {
                reject('请求失败')
            }
        }, 1000)
    })
}

let request1 = getData()

console.log(request1) // Promise {<pending>}

// 过了一秒后，再打印
// 由于 result 为 true,进入 if，执行 resolve，Promise 状态变成 fulfilled
console.log(request1)
// Promise {<fulfilled>: Array(3)}
// PromiseResult: [1,2,3]

// 重新创建一个 request2，并且 result 改成 false
result = false
let request2 = getData()
console.log(request2) // Promise {<pending>}

// 过了一秒后，再打印
// 由于 result 为 false,进入 else，执行 reject，Promise 状态变成 rejected
console.log(request2)
// Promise {<rejected>: '请求失败'}
// PromiseResult: "请求失败"
```

状态无法改变，但值一直都在。在.then 里面 return 一个值，那么下一个.then 还可以接收到。
```javascript
let promise1 = new Promise((resolve, reject) => {
    resolve(123)
}).then(res=>{
    console.log(res)
    return res+1
}).then(res=>{
    console.log(res)
    return res+1
}).then(res=>{
    console.log(res)
    return res+1
})

// 123
// 124
// 125

promise1 // Promise {<fulfilled>: 126}
promise1.then(res => console.log(res)) // 126

promise1.then(res => {
    console.log(res)
    return res + 1
})

// 127

promise1 // Promise {<fulfilled>: 126}

promise1 = promise1.then(res => {
    console.log(res) // 127
    return res + 1
})

promise1 // Promise {<fulfilled>: 127}

promise1 = promise1
    .then(res => {
        console.log(res) // 127
        return res + 1
    })
    .then(res => {
        console.log(res) // 128
        return res + 1
    })
    .then(res => {
        console.log(res) // 129
    })
// 没有继续 return，所以后面显示 undefined
promise1 // Promise {<fulfilled>: undefined}
```

## 3.其他 Promise 方法

**Promise.all([promise1,promise2,promise3])**: 当所有 promise 都返回 resolve 后，Promise.all 才会返回 resolve，任意一个或者几个返回 reject，Promise.all 就会返回 reject。

**Promise.race([promise1,promise2,promise3])**: 看哪个 promise 返回得更快（状态改变得更快），以第一次返回/改变状态的 promise 为结果。如果第一个返回的 promise 返回 fulfilled，那么 Promise.race 就是 fulfilled。如果第一个返回的 promise 返回 rejected，那么 Promise.race 就是 rejected

**Promise.allSettled([promise1,promise2,promise3])**: 与 Promise.all 类似，但 Promise.allSettled 不会因为某一个 reject 而结束，而是等所有的 promise 都返回之后，才结束，不管每个 promise 是 fulfilled 还是 reject。

**Promise.any([promise1,promise2,promise3])**: 只要 promise 有一个变成 fulfilled 状态，包装实例就会变成 fulfilled 状态；如果所有参数实例都变成 rejected 状态，包装实例就会变成 rejected 状态。

**Promise.resolve()**: 将现有对象转为 Promise 对象
```javascript
Promise.resolve('foo')
// 等价于
new Promise(resolve => resolve('foo'))
```

1. 如果参数是 Promise 实例，那么 Promise.resolve 将不做任何修改、原封不动地返回这个实例。

2. 参数是一个 thenable 对象。
```javascript
let thenable = {
    then: function(resolve, reject) {
        resolve(42)
    }
}
```
Promise.resolve()方法会将这个对象转为 Promise 对象，然后就立即执行 thenable 对象的 then()方法
```javascript
let thenable = {
    then: function(resolve, reject) {
        resolve(42)
    }
}

let p1 = Promise.resolve(thenable)
p1.then(function(value) {
    console.log(value) // 42
})
```

3. 如果参数是一个原始值，或者是一个不具有 then()方法的对象，则 Promise.resolve()方法返回一个新的 Promise 对象，状态为 resolved。
```javascript
const p = Promise.resolve('Hello')

p.then(function(s) {
    console.log(s)
})
// Hello
```

4. 不带有任何参数 Promise.resolve()直接返回一个 resolved 状态的 Promise 对象。
```javascript
const p = Promise.resolve()

p.then(function() {
    // ...
})
```

**Promise.reject()**: 方法也会返回一个新的 Promise 实例，该实例的状态为 rejected。
```javascript
const p = Promise.reject('出错了')
// 等同于
const p = new Promise((resolve, reject) => reject('出错了'))

p.then(null, function(s) {
    console.log(s)
})
// 出错了
```

## 4.实际应用示例

```javascript
// 模拟异步操作
function fetchUserData(userId) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            if (userId > 0) {
                resolve({ id: userId, name: '张三', age: 25 })
            } else {
                reject('用户ID无效')
            }
        }, 1000)
    })
}

function fetchUserPosts(userId) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve([
                { id: 1, title: '文章1', content: '内容1' },
                { id: 2, title: '文章2', content: '内容2' }
            ])
        }, 800)
    })
}

// 使用 Promise.all 并行获取数据
async function getUserInfoAndPosts(userId) {
    try {
        const [userData, posts] = await Promise.all([
            fetchUserData(userId),
            fetchUserPosts(userId)
        ])
        
        console.log('用户信息:', userData)
        console.log('用户文章:', posts)
        
        return { userData, posts }
    } catch (error) {
        console.error('获取数据失败:', error)
        throw error
    }
}

// 使用 Promise.race 设置超时
function fetchWithTimeout(url, timeout = 5000) {
    const fetchPromise = fetch(url)
    const timeoutPromise = new Promise((_, reject) => {
        setTimeout(() => reject(new Error('请求超时')), timeout)
    })
    
    return Promise.race([fetchPromise, timeoutPromise])
}

// 使用 Promise.allSettled 处理多个请求
async function fetchMultipleUsers(userIds) {
    const promises = userIds.map(id => fetchUserData(id))
    const results = await Promise.allSettled(promises)
    
    const successful = results.filter(result => result.status === 'fulfilled')
    const failed = results.filter(result => result.status === 'rejected')
    
    console.log('成功的请求:', successful.length)
    console.log('失败的请求:', failed.length)
    
    return results
}
```