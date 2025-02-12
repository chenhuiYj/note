Redux 是一个用于 JavaScript 应用的状态管理库，它通常与 React 一起使用，但也可以用于其他 JavaScript 应用框架。下面是一个 Redux 的学习指南，包括核心概念、基础教程、使用方法和最佳实践。

## 1.概念

‌Store‌：Store 是应用唯一的数据源。它保存了应用的全部状态，提供了获取当前状态、添加监听、分发动作（dispatch）等方法。

‌Action‌：Action 是一个描述发生了什么的普通对象。Action 描述了 State 如何发生变化。通常，一个 Action 对象会包含一个 type 属性来表示动作的类型。

‌Reducer‌：Reducer 是一个纯函数，用来描述应用的状态是如何随着时间推移而改变的。Reducer 接受当前应用的状态和一个动作（action），并返回一个新的状态。

‌Dispatch‌：Dispatch 是将一个 Action 发送到 Store 的过程。这是触发状态更新的唯一方式。

‌Selector‌：Selector 是用于读取 Store 中状态数据的函数。它帮助我们从 Store 中提取出需要的数据，并进行必要的计算。


## 2.例子
```javascript
// src/actions/todoActions.js
export const ADD_TODO = 'ADD_TODO';
export const REMOVE_TODO = 'REMOVE_TODO';

export const addTodo = (todo) => ({
  type: ADD_TODO,
  payload: todo,
});

export const removeTodo = (id) => ({
  type: REMOVE_TODO,
  payload: id,
});

// src/reducers/todoReducer.js
import { ADD_TODO, REMOVE_TODO } from '../actions/todoActions';

const initialState = {
  todos: [],
};

const todoReducer = (state = initialState, action) => {
  switch (action.type) {
    case ADD_TODO:
      return {
        ...state,
        todos: [...state.todos, { id: Date.now(), ...action.payload }],
      };
    case REMOVE_TODO:
      return {
        ...state,
        todos: state.todos.filter((todo) => todo.id !== action.payload),
      };
    default:
      return state;
  }
};

export default todoReducer;
```

```javascript
// src/store/index.js
import { createStore } from 'redux';
import todoReducer from '../reducers/todoReducer';

const store = createStore(todoReducer);

export default store;
```

```javascript
// src/index.js
import React from 'react';
import ReactDOM from 'react-dom';
import { Provider } from 'react-redux';
import store from './store';
import App from './App';

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
```

```javascript
// src/App.js
import React, { useState } from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { addTodo, removeTodo } from './actions/todoActions';

const App = () => {
  const todos = useSelector((state) => state.todos);
  const dispatch = useDispatch();

  const [input, setInput] = useState('');

  const handleAddTodo = () => {
    if (input.trim()) {
      dispatch(addTodo({ text: input }));
      setInput('');
    }
  };

  return (
    <div>
      <h1>Todo List</h1>
      <input
        type="text"
        value={input}
        onChange={(e) => setInput(e.target.value)}
        onKeyPress={(e) => {
          if (e.key === 'Enter') {
            handleAddTodo();
          }
        }}
      />
      <button onClick={handleAddTodo}>Add Todo</button>
      <ul>
        {todos.map((todo) => (
          <li key={todo.id}>
            {todo.text}
            <button onClick={() => dispatch(removeTodo(todo.id))}>Remove</button>
          </li>
        ))}
      </ul>
    </div>
  );
};

export default App;
```