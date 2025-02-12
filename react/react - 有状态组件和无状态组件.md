## 有状态组件
有状态组件也称容器组件，是能够在其生命周期内维护和管理自身状态的组件。状态（state）是组件内部的私有数据，它可以随时间变化，并且当状态改变时，组件会重新渲染以反映新的状态。

有状态组件通常被用于处理复杂的逻辑和数据操作，例如从后端获取数据、处理表单等。当状态发生变化时，有状态组件会重新渲染。

```javascript
Class 组件的有状态组件

import React from 'react';
 
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0,
    };
  }
 
  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Increment
        </button>
      </div>
    );
  }
}
```
函数组件的有状态组件
```javascript
import React,{useState} from 'react';

let Counter =function(){
	let [count, setCount]=useState(0)
	return (
      <div>
        <p>Count: {count}</p>
        <button onClick={setCount(preValue=>preValue+1)}>
          Increment
        </button>
      </div>
);
}
```
## 无状态组件
也称为展示组件，是指没有自己的状态和生命周期方法的组件。主要负责接收 props，并根据这些 props 来渲染视图。无状态组件通常被用于展示静态内容，不包含复杂的业务逻辑。由于无状态组件没有内部状态，所以渲染开销较小，性能较好。

```javascript
import React from 'react';

const Counter = ({ count}) => (
  <p>count</p>
);
```
有状态组件和无状态组件的区别

‌状态管理‌：有状态组件可以存储和更新状态，而无状态组件则完全依赖于外部传入的 props。

‌组件类型‌：有状态组件通常是类组件，而无状态组件通常是函数组件。

‌复杂性和性能‌：由于有状态组件涉及状态管理和可能的生命周期方法，它们通常比无状态组件更复杂。无状态组件由于没有状态管理和生命周期方法，通常更加简单且性能更高。

‌复用性‌：无状态组件由于不依赖于内部状态，因此更容易复用。

在实际开发中如何选择使用有状态组件或无状态组件？

‌无状态组件‌：适用于展示型组件，这些组件只负责渲染数据而不涉及复杂的状态逻辑。它们易于测试、复用和维护。例如，列表项、卡片、按钮；或者在需要高性能的场景下，如动画、滚动列表等。

‌有状态组件‌：适用于需要管理内部状态或处理复杂逻辑的组件。例如，用户输入表单、数据加载和更新等场景。