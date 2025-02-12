## 1.父组件向子组件通信
props 传递，利用 React 单向数据流的思想，通过 props 传递
```javascript
function Child(props){
  return (
    <div className="child">
     <input value={props.name} type="text"/>	
    </div>
  )
}
const Parent = <Child name="父组件调用子组件"/>
```

## 2.子组件向父组件通信
自定义事件
```javascript
import React from "react"
class Parent extends React.Component{
  constructor(){
    super()
    this.state={
      price:0
    }
  }
  getPrice(val){
    this.setState({
      price:val
    })
  }
  render(){
    return (<div>
      <span className="label">价格:</span>
      <span className="value">{this.state.price}</span>  
      <Child getPrice={this.getPrice.bind(this)}/>  
    </div>)	
  }
}
```

```javascript
class Child extends React.Component{
  getItemPrice(e){
    this.props.getPrice(e)
  }
  render(){
    return (
      <div>
        <button onClick={this.getItemPrice.bind(this)}>廓形大衣</button>
        <button onClick={this.getItemPrice.bind(this)>牛仔裤</button>	
      </div>	
    )
  }
}
```
事件冒泡

点击子组件的 button 按钮，事件会冒泡到父组件上
```javascript
const Child = () => {
    return <button>点击</button>;
};
```
```javascript
const Parent = () => {
    const sayName = name => {
        console.log(name);
    };
    return (
        <div onClick={() => sayName('lyllovelemon')}>
            <Child />
        </div>
    );
};

export default Parent;
```
Ref
```javascript
import React, { Component } from 'react';
 
class ParentComponent extends Component {
  constructor(props) {
    super(props);
    this.childRef = React.createRef();
  }
 
  sendMessageToChild = () => {
    this.childRef.current.receiveMessage('Hello from Parent');
  };
 
  render() {
    return (
      <div>
        <button onClick={this.sendMessageToChild}>Send Message</button>
        <ChildComponent ref={this.childRef} />
      </div>
    );
  }
}
```
```javascript
class ChildComponent extends Component {
  constructor(props) {
    super(props);
    this.state = { message: '' };
  }
 
  receiveMessage = (message) => {
    this.setState({ message });
  };
 
  render() {
    return <div>{this.state.message}</div>;
  }
}
 
export default ParentComponent;
```
## 3.兄弟组件通信
实际上就是通过父组件中转数据的，子组件 a 传递给父组件，父组件再传递给子组件 b
```javascript
import React from "react"
class Parent extends React.Component{
  constructor(props){
    super(props)
    this.state={
      count:0
    }
  }
  increment(){
    this.setState({
      count:this.state.count+1
    })
  }
  render(){
    return (
      <div>
        <ChildOne count={this.state.count} />
        <ChildTwo onClick={this.increment} />
      </div>
    )
  }
}
```
## 4.父组件向后代组件通信
Context
```javascript
import React from "react"
const PriceContext = React.createContext("price") 
export default class Parent extends React.Component{
  constructor(props){
    super(props)
  }
  render(){
    return (
      <PriceContext.Provider value={200}>
      </PriceContext>
    )
  }
}
class Child extends React.Component{
  ...
}
class SubChild extends React.Component{
  constructor(props){
    super(props)
  }
  render(){
    return (
      <PriceContext.Consumer>
        { price=> <div>price:{price}</div> }
      </PriceContext.Consumner>
    )
  }
}
```
## 5.无关组件通信
状态提升（Lifting State Up）：将共享状态提升到最近的公共祖先组件中，通过 props 传递给各子组件。

上下文（Context）API：React 提供的一种跨组件传递数据的方法。

发布订阅模式（PubSub）：通过第三方库实现非直接相关组件间的通信。

集中状态管理（如 Redux 或 MobX）：通过集中的 store 管理状态，并通过 props 或自定义 hooks 传递数据。