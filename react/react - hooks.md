## 1. react-hooks 解决的问题
函数组件中不能拥有自己的状态（state）。在 hooks 之前函数组件是无状态的，都是通过 props 来获取父组件的状态，但是 hooks 提供了 useState 来维护函数组件内部的状态。

函数组件中不能监听组件的生命周期。useEffect 聚合了多个生命周期函数。

class 组件中生命周期较为复杂（在15版本到16版本的变化大）。

class 组件逻辑难以复用（HOC，render props）。

## 2. hooks 对比 class 的好处

### 2-1.hooks 写法比 class 简洁

### 2-2.业务代码更加聚合

使用 class 组件经常会出现一个功能出现在两个生命周期函数内的情况，这样分开写有时候可能会忘记。比如：
```javascript
let timer = null
componentDidMount() {
    timer = setInterval(() => {
        // ...
    }, 1000)
}
// ...
componentWillUnmount() {
    if (timer) clearInterval(timer)
}
```
由于添加定时器和清除定时器是在两个不同的生命周期函数，中间可能会有很多其他的业务代码，所以可能会忘记清除定时器，如果在组件卸载时没有添加清楚定时器的函数就可能会造成内存泄漏、网络一直请求等问题。
但是使用 hooks 可以让代码更加的集中，方便我们管理，也不容易忘记
```javascript
useEffect(() => {
    let timer = setInterval(() => {
        // ...
    }, 1000)
    return () => {
        if (timer) clearInterval(timer)
    }
}, [/*...*/])
```
### 2-3.逻辑复用方便

class 组件的逻辑复用通常用 render props 以及 HOC 两种方式。react hooks 提供了自定义 hooks 来复用逻辑。

下面以获取鼠标在页面的位置的逻辑复用为例：

class 组件 render props 方式复用
```javascript
import React, { Component } from 'react'

class MousePosition extends Component {
  constructor(props) {
    super(props)
    this.state = {
      x: 0,
      y: 0
    }
  }

  handleMouseMove = (e) => {
    const { clientX, clientY } = e
    this.setState({
      x: clientX,
      y: clientY
    })
  }

  componentDidMount() {
    document.addEventListener('mousemove', this.handleMouseMove)
  }

  componentWillUnmount() {
    document.removeEventListener('mousemove', this.handleMouseMove)
  }

  render() {
    const { children } = this.props
    const { x, y } = this.state
    return(
      <div>
        {
          children({x, y})
        }
      </div>
    )
  }

}
```
使用
```javascript
class Index extends Component {
  constructor(props) {
    super(props)
  }

  render() {
    return (
      <MousePosition>
        {
          ({x, y}) => {
            return (
              <div>
                <p>x:{x}, y: {y}</p>
              </div>
            )
          }
        }
      </MousePosition>
    )
  }
}

export default Index
```
自定义 hooks 方式复用
```javascript
import React, { useEffect, useState } from 'react'

function usePosition() {
  const [x, setX] = useState(0)
  const [y, setY] = useState(0)

  const handleMouseMove = (e) => {
    const { clientX, clientY } = e
    setX(clientX)
    setY(clientY)
  } 

  useEffect(() => {
    document.addEventListener('mousemove', handleMouseMove)
    return () => {
      document.removeEventListener('mousemove', handleMouseMove)
    }
  })
  return [
    {x, y}
  ]
}
```
使用
```javascript
function Index() {
  const [position] = usePosition()
  return(
    <div>
      <p>x:{position.x},y:{position.y}</p>
    </div>
  )
}

export default Index
```
可以很明显的看出使用 hooks 对逻辑复用更加的方便，使用的时候逻辑也更加清晰。

## 3. 常见 hook
### 3-1.useState
```javascript
const [value, setValue] = useState(0)
```
这种语法方式是 ES6的数组结构，数组的第一个值是声明的状态，第二个值是状态的改变函数。
### 3-2.useEffect
```javascript
useEffect(() => {
    //handler function...
    
    return () => {
        // clean side effect
    }
}, [//dep...])
```
useEffect 接收一个回调函数以及依赖项，当依赖项发生变化时才会执行里面的回调函数。useEffect 类似于 class 组件 didMount、didUpdate、willUnmount 的生命周期函数。

1.useEffect 是异步的在组件渲染完成后才会执行

2.useEffect 的回调函数只能返回一个清除副作用的处理函数或者不返回

3.如果 useEffect 传入的依赖项是空数组那么 useEffect 内部的函数只会执行一次

### 3-3.useMemo
useMemo 返回一个记忆化的值。它接收一个创建函数和一个依赖项数组，并返回该创建函数的结果，但仅在其依赖项发生变化时才会重新计算这个结果。这对于避免在每次渲染时都执行昂贵的计算特别有用。

使用场景：当有一个计算结果，并且这个计算结果依赖于某些值，但你希望避免在每次渲染时都重新计算这个结果时，可以使用 useMemo。
```javascript
import React, { useState, useMemo } from 'react';

function ExpensiveCalculationComponent() {
  const [count, setCount] = React.useState(0);
  const [dependency, setDependency] = React.useState('initial');

  // 使用 useMemo 来记忆化计算结果
  const computedValue = React.useMemo(() => {
    console.log('Computing value...');
    return count*2
  }, [count, dependency]); // 依赖项数组，当这些值变化时，才会重新进入并计算 computedValue，所以每次点击 Increment Count，count 变化了，每次都会进入。但是进入 Change Dependency，只有第一次会进入并计算，因为后面 dependency 都是'initial changed'

  return (
    <div>
      <p>Count: {count}</p>
      <p>Computed Value: {computedValue}</p>
      <button onClick={() => setCount(count + 1)}>Increment Count</button>
      <button onClick={() => setDependency('initial changed')}>Change Dependency</button>
    </div>
  );
}
```
### 3-4.useCallback

useCallback 的作用与 useMemo 相同，但它是专门为函数构建的。我们直接给它一个函数，它记住那个函数，在渲染之间进行线程处理。换句话说，这两个表达有相同的效果：
```javascript
React.useCallback(function helloWorld(){}, []);
// 在功能上等价于
React.useMemo(() => function helloWorld(){}, []);
```
useCallback 是语法糖。useCallback 是 useMemo 的语法糖，是「useMemo」的返回值为函数时的特殊情况，缓存的是函数的引用。如果组件中有一个函数，使用 useCallback，可以实现只有在依赖的数据发生变化的时候，函数才会重新渲染。

useCallback 返回一个记忆化的回调函数，该回调函数在依赖项（数组中的值）发生变化时才会更新。这对于将回调函数传递给子组件以优化性能特别有用，因为它可以防止子组件因为父组件的重新渲染而不必要地重新渲染。

使用‌场景‌：
当你有一个传递给子组件的回调函数，并且你希望避免在每次父组件渲染时都重新创建这个函数时，可以使用 useCallback。
```javascript
function ParentComponent() {
  const [count, setCount] = useState(0);

  // 使用 useCallback 来记忆化回调函数
  const handleIncrement = useCallback(() => {
    setCount(count + 1);
  }, [count]); // 注意：这里的依赖项数组包含了 count，这意味着当 count 变化时，handleIncrement 也会更新，子组件就会更新。

  return (
    <ChildComponent onIncrement={handleIncrement} />
  );
}

const ChildComponent = React.memo(({ onIncrement }) => {
  console.log('ChildComponent rendered');

  return (
    <button onClick={onIncrement}>Increment</button>
  );
});
```
这个例子中，handleIncrement 函数被 useCallback 记忆化。当 ParentComponent 渲染时，只要 count 没有变化，handleIncrement 就不会重新创建。这有助于避免 ChildComponent 因为接收到新的 onIncrement 回调函数而不必要地重新渲染。

### 3-5.useRef
useRef 类似于 react.createRef。
```javascript
const node = useRef(null)
<input ref={node} />
```
这样可以通过 node.current 属性访问到该 DOM 元素。
需要注意的是 useRef 创建的对象在组件的整个生命周期内保持不变，也就是说每次重新渲染函数组件时，返回的 ref 对象都是同一个（使用 React.createRef ，每次重新渲染组件都会重新创建 ref）。
### 3-6.useReducer
useReducer 类似于 redux 中的 reducer。
```javascript
const [state, dispatch] = useReducer(reducer, initstate)
```
useReducer 传入一个计算函数和初始化 state，类似于 redux。通过返回的 state 我们可以访问状态，通过 dispatch 可以对状态作修改。
```javascript
const initstate = 0;
function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {number: state.number + 1};
    case 'decrement':
      return {number: state.number - 1};
    default:
      throw new Error();
  }
}
function Counter(){
    const [state, dispatch] = useReducer(reducer, initstate);
    return (
        <>
          Count: {state.number}
          <button onClick={() => dispatch({type: 'increment'})}>+</button>
          <button onClick={() => dispatch({type: 'decrement'})}>-</button>
        </>
    )
}
```
### 3-7.useContext
通过 useContext 我们可以更加方便的获取上层组件提供的 context。

父组件
```javascript
import React, { createContext, Children } from 'react'
import Child from './child'

export const MyContext = createContext()

export default function Parent() {

  return (
    <div>
      <p>Parent</p>
      <MyContext.Provider value={{name: 'cc', age: 21}}>
        <Child />
      </MyContext.Provider>
    </div>
  )
}
```
子组件
```javascript
import React, { useContext } from 'react'
import { MyContext } from './parent'

export default function Parent() {
  const data = useContext(MyContext) // 获取父组件提供的 context
  console.log(data)
  return (
    <div>
      <p>Child</p>
    </div>
  )
}
```
父组件创建并导出 context：export const MyContext = createContext()

父组件使用 provider 和 value 提供值：<MyContext.provide value={{name: 'cc', age: 22}} />

子组件导入父组件的 context：import { MyContext } from './parent'

获取父组件提供的值：const data = useContext(MyContext)

不过在多数情况下我们都不建议使用 context，因为会增加组件的耦合性。

### 3-8.useLayoutEffect
useEffect 在全部渲染完毕后才会执行；

useLayoutEffect 会在 浏览器 layout 之后，painting 之前执行，并且会柱塞 DOM；可以使用它来读取 DOM 布局并同步触发重渲染。
```javascript
export default function LayoutEffect() {
  const [color, setColor] = useState('red')
  useLayoutEffect(() => {
      alert(color) // 会阻塞 DOM 的渲染
  });
  useEffect(() => {
      alert(color) // 不会阻塞
  })
  return (
      <>
        <div id="myDiv" style={{ background: color }}>颜色</div>
        <button onClick={() => setColor('red')}>红</button>
        <button onClick={() => setColor('yellow')}>黄</button>
      </>
  )
}
```
上面的例子中 useLayoutEffect 会在 painting 之前执行，useEffect 在 painting 之后执行。