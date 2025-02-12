在没有 hooks 之前，函数组件不能拥有自身的状态和生命周期，也就是无状态组件。但是在有了 hooks 之后，函数组件逐渐成为主流，可以在绝大多数情况下满足开发的需求。
## 1.Class 组件的问题

### 1-1.this 指向
```javascript
class ExampleOfClass extends Component {
  constructor(props) {
    super(props)
    this.state = {
      count: 1
    }
  }
  handleClick=function {
    console.log(this.state.count)//报错，因为在 onClick 中调用 this.handleClick 函数，就是普	通调用，this 指向 undefined。
	//可以 onClick={this.handleClick.bind(this)}，
	//或者 onClick={()=>{this.handleClick.bind(this)}}
	//或者在 constructor 里面加上 this.handleClick= this.handleClick.bind(this);
	//或者把 handleClick 定义成箭头函数 handleClick=()=>{}
  }
  render() {
    const { count } = this.state
    return (
      <div>
        <p>you click { count }</p>
        <button onClick={this.handleClick}>点击</button>
      </div>
    )
  }
}
```
另一个 this 问题
```javascript
import React from "react"

const ProfileFunction: React.FC<{goods:string}> = (props) => {
    const showMessage = () => {
        alert(`你下单的是“${props.goods}”！` )
    }
    const handleClick = () => {
        setTimeout(showMessage, 3 * 1000)
    }
    return (
        <button onClick={handleClick}>购买</button>
    )
}

class ProfileClass extends React.Component<
    { goods: string },  // props 类型
    {}  // state 类型
> {
    showMessage = () => {
        alert(`你下单的是“${this.props.goods}”！` )
    }
    handleClick = () => {
        setTimeout(this.showMessage, 3 * 1000)
    }
    render() {
        return <button onClick={this.handleClick}>购买</button>
    }
}


export default class App extends React.Component {
    state = {class ProfileClass extends React.Component<
    { goods: string },  // props 类型
    {}  // state 类型
> {
    showMessage = () => {
        alert(`你下单的是“${this.props.goods}”！` )
    }
    handleClick = () => {
        setTimeout(this.showMessage, 3 * 1000)
    }
    render() {
        return <button onClick={this.handleClick}>购买</button>
    }
}

        goods: '苹果',
    };
    render() {
        return (
            <>
                <label>
                    请选择：
                    <select
                        value={this.state.goods}
                        onChange={e => this.setState({ goods: e.target.value })}
                    >
                        <option value="苹果">苹果</option>
                        <option value="香蕉">香蕉</option>
                        <option value="西瓜">西瓜</option>
                    </select>
                </label>
                <h1>{this.state.goods}</h1>
                <p>
                    <ProfileFunction goods={this.state.goods} />
                    <b> (function)</b>
                </p>
                <p>
                    <ProfileClass goods={this.state.goods} />
                    <b> (class)</b>
                </p>
            </>
        )
    }
}
```
在线案例：https://codesandbox.io/p/sandbox/func-vs-class-tw7rr0

函数组件：当用户选择苹果，点击购买后，再切换浏览香蕉，提示信息反馈用户下单的是苹果。

类 组 件 ：当用户选择苹果，点击购买后，再切换浏览西瓜，提示信息反馈用户下单的是西瓜！

出现的原因是因为函数组件中，参数 props 本身是不可变的，函数组件中的 showMessage 在3秒延迟后拿到的仍然是原来的 props.goods。

但是类组件中实例的 this 是可变的，类组件中的 showMessage 在3秒延迟后去拿 this.props.goods 时，由于 this 发生了变化，所以造成取到的值不是原来的值。

要解决这个问题，就得调用时候先保存 props 的值。
```javascript
handleClick = () => {
    const {user} = this.props;
    setTimeout(() => this.showMessage(user), 3000);
};
```
### 1-2.类组件代码量比函数组件多：

这个从上面的案例中可见 ProfileFunction 代码比 ProfileClass 少，同样功能的函数组件代码量比类组件少一些。

### 1-3.类组件过于臃肿不易拆分：

类组件是面向对象编程思维方式，函数组件是面向过程编程思维方式。React 的设计思路更推崇组合，而不是继承。在类组件中大量使用继承会造成组件过重，功能难以拆分。

## 2.函数组件的问题
生命周期

### 挂载阶段：getDerviedStateFromProps VS 无

该方法用于在 props 被传入后根据 props 更新 state。

函数组件中也可以写代码根据 props 更新 state，但这样做会造成重复渲染。如果遇到需要根据 props 更新 state 的情况，应该考虑做状态提升。如果你发现在某个组件中必须要根据 props 更新 state 又无法做状态提升，那么该组件应该写成类式组件，而不是函数式组件。

### 挂载阶段：componentDidMount VS useEffect
该方法用于在组件挂载以后执行副作用操作，如发起网络请求、设置计时器、创建订阅等。

函数组件有 useEffect。

### render：
在类组件的 render 方法中返回要渲染的内容。render 里不能有副作用和 setState！

函数组件的 return 和类组件 render 方法的 return 效果一致。

### 更新阶段：getDerviedStateFromProps VS 无

同挂载阶段的同名方法一样。

### 更新阶段：shouldComponentUpdate VS memo、useMemo、useCallback

该方法返回 true 表示需要更新、返回 false 表示无需更新。可在此添加判断条件做性能优化，另外 PureComponent 实现原理也相同。

函数组件对应的 hooks 有很多，常用的有 memo、useMemo、useCallback，同样可以做性能优化。

### render：
同挂载阶段一样。

### 更新阶段：getSnapshotBeforeUpdate VS 无

该方法在最近一次渲染输出(提交到 DOM 节点)之前调用。它使得组件能在发生更改之前从 DOM 中捕获一些信息(如滚动位置等)。此生命周期方法的任何返回值将作为参数传递给 componentDidUpdate()。此用法并不常见，但它可能出现在 UI 处理中，如以特殊方式处理滚动位置的聊天线程等。

函数组件无该方法对应的 hooks。

### 更新阶段：componentDidUpdate VS 无

组件更新后会立即调用该方法，首次渲染不会调用。当组件更新后，可以在此处对 DOM 进行操作。注意：在该方法中慎用 setState，如果要用必须将其包裹在条件语句里。

函数组件无该方法对应的 hooks，因为 React 本身设计是减少直接操作 DOM，在 React 中除了 useRef 外直接操作 DOM 的场景很少，函数组件没有该方法对应的 hooks 不算什么问题。

### 卸载阶段：componentWillUnmount VS useEffect

该方法会在组件卸载及销毁之前直接调用。在此方法中执行必要的清理操作，例如：清除计时器、取消网络请求或清除订阅等。

函数组件有 useEffect。

### 其他，错误边界：componentDidCatch、static getDerivedStateFromError VS 无

在类组件中定义了 static getDerivedStateFromError 或 componentDidCatch 这两个生命周期方法中的任意一个或两个时，那么它就变成一个错误边界。当抛出错误后，请使用 static getDerivedStateFromError 渲染备用 UI，使用 componentDidCatch 打印错误信息。

函数组件无错误边界对应的 hooks

#### 最后，函数组件和类组件各有优势。类组件功能最为完备和强大，某些特殊用途(如错误边界)组件只能写成类式组件。函数组件没有 this 困扰且代码简洁，大部分的普通组件都可以写成函数组件。