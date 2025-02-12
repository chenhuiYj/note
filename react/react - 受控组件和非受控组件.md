受控组件：指其输入值由 React 组件的状态（state）进行控制的组件。在受控组件中，表单元素的值由组件的状态驱动，任何对输入值的更改都需要通过状态更新来实现。
```javascript
class ControlledInput extends React.Component {
  constructor(props) {
    super(props);
    this.state = { value: '' };
    this.handleChange = this.handleChange.bind(this);
  }
 
  handleChange(event) {
    this.setState({ value: event.target.value });
  }
 
  render() {
    return (
      <input
        value={this.state.value}
        onChange={this.handleChange}
        type="text"
      />
    );
  }
}
```
Input 的 value 完全由 state 控制，value={this.state.value}。

##### 优点
1.所有表单数据都存储在组件的状态中，易于管理和维护。

2.开发者可以轻松地验证输入、清空表单等操作，便于实现复杂的表单逻辑。

3.实时反馈，增强用户体验

##### 缺点
1.性能开销，对于大量表单元素的复杂组件，频繁更新状态可能导致性能问题。

2.需要编写额外的事件处理逻辑，可能使代码变得冗长。

非受控组件：指其输入值不由 React 组件的状态控制，而是通过直接操作 DOM 元素来获取输入值。通常来说，非受控组件使用 React 的 ref 来访问 DOM 元素并获取其值。
```javascript
import react {useRef} from ‘react’

function UnControlledForm (){
	const inputRef=useRef(null)

	const handleSubmit(e){
		e.preventDefault()
		console.log(inputRef.current.value)
	}

	return (
		<form onSubmit={handleSubmit}>
			<input type=”text” ref={inputRef}/>
			<button type=”submit” >submit</button>
		</form>
		)
}
```
##### 优点

1.不需要额外的状态管理，代码相对简洁，实现较为简单

2.在某些情况下，不需要实时更新状态，使用非受控组件能够减少渲染次数。

##### 缺点

1.由于数据不在组件的状态中，跟踪输入变化变得困难。

2.无法实现即时验证或反馈，用户体验可能受到影响。

3.非受控组件使用了直接操作 DOM 的方式，违背了 React 的声明式编程理念。

当表单简单并且不需要实时验证的时候，可以使用非受控组件实现。但大多数情况下，建议还是使用受控组件

![](https://img2020.cnblogs.com/blog/1161361/202108/1161361-20210806155147147-148149795.png)
图片来源：https://www.cnblogs.com/houxianzhou/p/15108948.html