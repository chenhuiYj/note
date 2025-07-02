useState 是 React 中的一个 Hook，它允许在函数组件中管理状态。useState 返回一个状态值和一个更新该状态的函数，这使得函数组件能够持有和更新状态，而无需转换为类组件。
```js
import React, { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}

export default Counter;
```

### 1.useState 的原理与实现
```js
// React 内部 useState 实现（简化版本）
function useState(initialState) {
  // 获取当前 Hook 索引的 Fiber 节点上的状态
  const hook = getHook(); 
  
  if (!hook) {
    // 如果是第一次渲染，初始化 Hook，并设置初始状态
    hook = {
      state: typeof initialState === 'function' ? initialState() : initialState,
      queue: [],
    };
    mountHook(hook);
  }
  
  // 返回状态和用于更新状态的函数
  const setState = (action) => {
    // 将新状态入队
    const newState = typeof action === 'function' ? action(hook.state) : action;
    hook.state = newState;
    // 触发组件的重新渲染
    scheduleUpdate();
  };

  return [hook.state, setState];
}
```

getHook()：React 通过 getHook 函数根据当前组件的 Fiber 获取对应的 Hook。React 会以调用顺序追踪 Hook，所以 useState 调用顺序必须一致。

setState(action)：setState 函数通过入队一个新的状态更新来触发重新渲染。

### 2.useState 的执行顺序和调用规则

首先，React 依赖 Hook 的调用顺序，而不是变量名。

每次组件渲染，React 都会按照 Hook 的执行顺序，在内部记录每一个 Hook 对应的状态，就像维护一个数组一样，只要调用顺序不变，React 就能正确管理状态。

```js
hookStates = [
  useState(0),   // index 0
  useState('')   // index 1
];
```

React 提供了两条核心规则来避免这类问题：

只能在最顶层调用 Hook：不要在条件、循环、嵌套函数中调用 Hook；
只能在 React 函数组件或自定义 Hook 中调用 Hook。

这些规则的目标就是保证 Hook 的调用顺序在每次渲染中始终一致。

```js
function MyComponent({ isLoggedIn }) {
  if (isLoggedIn) {
    useState(1); // ❌ 错误用法
  }
  useEffect(() => {
    // ...
  }, []);
}
```
第一次渲染时，isLoggedIn 是 true，useState(1) 被执行，React 记录它为第一个 Hook；

第二次渲染时，如果 isLoggedIn 是 false，useState(1) 没执行，useEffect 成了第一个 Hook。

所以在 React 中，useState 的调用顺序必须保持一致，因为 React 依赖 Hook 的调用顺序来将状态和组件实例关联起来。这就是为什么不能在条件语句或循环中调用 Hook。

### 3. setState 的批处理机制
React 中的状态更新并不会立即发生。为了优化性能，React 会将多个状态更新“批处理”，即将多次 setState 调用合并为一次重新渲染。只有当 React 确定所有更新都准备好后，才会触发一次组件更新。

例如，以下代码：
```js
function Counter() {
  const [count, setCount] = useState(0);

  const incrementTwice = () => {
    setCount(count + 1);
    setCount(count + 1);
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={incrementTwice}>Increment Twice</button>
    </div>
  );
}
```
虽然 setCount 调用了两次，但 React 只会触发一次重新渲染。上述的例子，实际等价于如下：

上述的例子，实际等价于如下：

```js
Object.assign(
  previousState,
  {index: state.count+ 1},
  {index: state.count+ 1},
  ...
)
```

为了在这种情况下正确更新状态，React 将会批量更新状态队列，并在下一个渲染周期中应用最终的状态结果。

解决方案：函数式更新

为了解决上述 setState 不会立即更新的问题，React 提供了 函数式更新 的形式，它能够确保每次更新都基于最新的状态值。
```js
function Counter() {
  const [count, setCount] = useState(0);

  const incrementTwice = () => {
    setCount(prevCount => prevCount + 1);
    setCount(prevCount => prevCount + 1);
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={incrementTwice}>Increment Twice</button>
    </div>
  );
}
```
在这个例子中，setCount 的回调函数能够确保每次更新都是基于最新的状态值进行的，这样状态就能正确递增两次。

### 4.更新机制

setState的更新类型分成：异步更新和同步更新

在组件生命周期或React合成事件中，setState是异步
```js
const [count, setCount] = useState(0);
setCount(1)
console.log(count); // 0
```
从上面可以看到，最终打印结果为0，并不能在执行完setCount之后立马拿到最新的count

如果想要立刻获取更新后的值，在第二个参数的回调中更新后会执行
```js
setCount(1,()=>{
    console.log(count); // 1
})

```

在setTimeout或者原生dom事件中，setState是同步
```js
const [count, setCount] = useState(0);
setTimeout(() => {
    setCount(1)
    console.log(count); // 1
}, 0);
```

```js
const [count, setCount] = useState(0);
const btnEl = document.getElementById("btn");
btnEl.addEventListener('click', () => {
    setCount(1);
    console.log(count); // 1
})
```
