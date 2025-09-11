# Redux 详细用法与配置项说明

## 1. Redux 核心概念

Redux 是一个可预测的状态管理库，基于三个核心原则：
- **单一数据源**：整个应用的状态存储在一个 store 中
- **状态只读**：唯一改变状态的方法是触发 action
- **纯函数修改**：使用纯函数 reducer 来描述状态如何变化

## 2. 基础配置与用法

### 2.1 安装依赖
```bash
npm install redux react-redux
```

### 2.2 创建 Action Types
```js
// src/actions/types.js
export const INCREMENT = 'INCREMENT';
export const DECREMENT = 'DECREMENT';
export const ADD_TODO = 'ADD_TODO';
export const TOGGLE_TODO = 'TOGGLE_TODO';
export const SET_FILTER = 'SET_FILTER';
```

### 2.3 创建 Action Creators
```js
// src/actions/index.js
import { INCREMENT, DECREMENT, ADD_TODO, TOGGLE_TODO, SET_FILTER } from './types';

// 同步 Action Creators
export const increment = () => ({
  type: INCREMENT
});

export const decrement = () => ({
  type: DECREMENT
});

export const addTodo = (text) => ({
  type: ADD_TODO,
  payload: {
    id: Date.now(),
    text,
    completed: false
  }
});

export const toggleTodo = (id) => ({
  type: TOGGLE_TODO,
  payload: { id }
});

export const setFilter = (filter) => ({
  type: SET_FILTER,
  payload: { filter }
});

// 异步 Action Creator (使用 redux-thunk)
export const fetchTodos = () => {
  return async (dispatch, getState) => {
    try {
      dispatch({ type: 'FETCH_TODOS_REQUEST' });
      const response = await fetch('/api/todos');
      const todos = await response.json();
      dispatch({ type: 'FETCH_TODOS_SUCCESS', payload: todos });
    } catch (error) {
      dispatch({ type: 'FETCH_TODOS_FAILURE', payload: error.message });
    }
  };
};
```

### 2.4 创建 Reducers
```js
// src/reducers/counter.js
import { INCREMENT, DECREMENT } from '../actions/types';

const initialState = {
  count: 0
};

const counterReducer = (state = initialState, action) => {
  switch (action.type) {
    case INCREMENT:
      return {
        ...state,
        count: state.count + 1
      };
    case DECREMENT:
      return {
        ...state,
        count: state.count - 1
      };
    default:
      return state;
  }
};

export default counterReducer;
```

```js
// src/reducers/todos.js
import { ADD_TODO, TOGGLE_TODO, SET_FILTER } from '../actions/types';

const initialState = {
  items: [],
  filter: 'ALL' // ALL, ACTIVE, COMPLETED
};

const todosReducer = (state = initialState, action) => {
  switch (action.type) {
    case ADD_TODO:
      return {
        ...state,
        items: [...state.items, action.payload]
      };
    case TOGGLE_TODO:
      return {
        ...state,
        items: state.items.map(todo =>
          todo.id === action.payload.id
            ? { ...todo, completed: !todo.completed }
            : todo
        )
      };
    case SET_FILTER:
      return {
        ...state,
        filter: action.payload.filter
      };
    default:
      return state;
  }
};

export default todosReducer;
```

### 2.5 合并 Reducers
```js
// src/reducers/index.js
import { combineReducers } from 'redux';
import counterReducer from './counter';
import todosReducer from './todos';

const rootReducer = combineReducers({
  counter: counterReducer,
  todos: todosReducer
});

export default rootReducer;
```

### 2.6 创建 Store
```js
// src/store/index.js
import { createStore, applyMiddleware, compose } from 'redux';
import thunk from 'redux-thunk';
import rootReducer from '../reducers';

// Redux DevTools 配置
const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ || compose;

const store = createStore(
  rootReducer,
  composeEnhancers(
    applyMiddleware(thunk)
  )
);

export default store;
```

### 2.7 配置 Provider
```js
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
);
```

## 3. 组件中使用 Redux

### 3.1 使用 connect (Class 组件)
```js
// src/components/Counter.js
import React, { Component } from 'react';
import { connect } from 'react-redux';
import { increment, decrement } from '../actions';

class Counter extends Component {
  render() {
    const { count, increment, decrement } = this.props;
    
    return (
      <div>
        <h2>计数: {count}</h2>
        <button onClick={increment}>+</button>
        <button onClick={decrement}>-</button>
      </div>
    );
  }
}

// mapStateToProps: 将 state 映射为 props
const mapStateToProps = (state) => ({
  count: state.counter.count
});

// mapDispatchToProps: 将 dispatch 映射为 props
const mapDispatchToProps = (dispatch) => ({
  increment: () => dispatch(increment()),
  decrement: () => dispatch(decrement())
});

export default connect(mapStateToProps, mapDispatchToProps)(Counter);
```

### 3.2 使用 Hooks (函数组件)
```js
// src/components/TodoList.js
import React from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { addTodo, toggleTodo, setFilter } from '../actions';

const TodoList = () => {
  // useSelector: 从 store 中选择数据
  const { items, filter } = useSelector(state => state.todos);
  const dispatch = useDispatch();

  // 根据过滤条件显示待办事项
  const filteredTodos = items.filter(todo => {
    switch (filter) {
      case 'ACTIVE':
        return !todo.completed;
      case 'COMPLETED':
        return todo.completed;
      default:
        return true;
    }
  });

  const handleAddTodo = (text) => {
    dispatch(addTodo(text));
  };

  const handleToggleTodo = (id) => {
    dispatch(toggleTodo(id));
  };

  const handleSetFilter = (filter) => {
    dispatch(setFilter(filter));
  };

  return (
    <div>
      <div>
        <input 
          type="text" 
          onKeyPress={(e) => {
            if (e.key === 'Enter') {
              handleAddTodo(e.target.value);
              e.target.value = '';
            }
          }}
          placeholder="添加待办事项"
        />
      </div>
      
      <div>
        <button onClick={() => handleSetFilter('ALL')}>全部</button>
        <button onClick={() => handleSetFilter('ACTIVE')}>未完成</button>
        <button onClick={() => handleSetFilter('COMPLETED')}>已完成</button>
      </div>

      <ul>
        {filteredTodos.map(todo => (
          <li 
            key={todo.id}
            onClick={() => handleToggleTodo(todo.id)}
            style={{ 
              textDecoration: todo.completed ? 'line-through' : 'none',
              cursor: 'pointer'
            }}
          >
            {todo.text}
          </li>
        ))}
      </ul>
    </div>
  );
};

export default TodoList;
```

## 4. 配置项详细说明

### 4.1 createStore 配置项
```js
const store = createStore(
  reducer,           // 必需：根 reducer 函数
  preloadedState,    // 可选：初始状态
  enhancer          // 可选：store 增强器
);
```

**参数说明：**
- `reducer`: 根 reducer 函数，处理所有 action
- `preloadedState`: 初始状态，用于服务端渲染或状态持久化
- `enhancer`: 增强器，如 `applyMiddleware`、`compose`

### 4.2 combineReducers 配置项
```js
const rootReducer = combineReducers({
  counter: counterReducer,  // 键名对应 state 中的属性名
  todos: todosReducer,      // 值是对应的 reducer 函数
  user: userReducer
});
```

**配置说明：**
- 键名会成为 state 中的属性名
- 每个 reducer 只管理对应的 state 部分
- 自动处理 action 分发到对应的 reducer

### 4.3 applyMiddleware 配置项
```js
import { applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import logger from 'redux-logger';

const middleware = applyMiddleware(
  thunk,    // 处理异步 action
  logger    // 打印 action 和 state 变化
);
```

**常用中间件：**
- `redux-thunk`: 处理异步 action
- `redux-logger`: 开发环境日志
- `redux-saga`: 处理复杂异步流程
- `redux-persist`: 状态持久化

### 4.4 connect 配置项
```js
connect(
  mapStateToProps,    // 可选：将 state 映射为 props
  mapDispatchToProps, // 可选：将 dispatch 映射为 props
  mergeProps,         // 可选：合并 props 的函数
  options            // 可选：连接选项
)(Component)
```

**参数详解：**

#### mapStateToProps
```js
// 函数形式
const mapStateToProps = (state, ownProps) => ({
  count: state.counter.count,
  todos: state.todos.items
});

// 返回 null 表示不订阅 store 更新
const mapStateToProps = () => null;
```

#### mapDispatchToProps
```js
// 对象形式（推荐）
const mapDispatchToProps = {
  increment,
  decrement,
  addTodo
};

// 函数形式
const mapDispatchToProps = (dispatch, ownProps) => ({
  increment: () => dispatch(increment()),
  decrement: () => dispatch(decrement())
});
```

### 4.5 useSelector 配置项
```js
// 基础用法
const count = useSelector(state => state.counter.count);

// 使用比较函数优化性能
const todos = useSelector(
  state => state.todos.items,
  (left, right) => left.length === right.length
);

// 多个选择器
const { count, todos } = useSelector(state => ({
  count: state.counter.count,
  todos: state.todos.items
}));
```

### 4.6 useDispatch 配置项
```js
const dispatch = useDispatch();

// 直接使用
dispatch(increment());

// 使用 useCallback 优化性能
const handleIncrement = useCallback(() => {
  dispatch(increment());
}, [dispatch]);
```

## 5. 最佳实践

### 5.1 文件结构
```
src/
├── actions/
│   ├── index.js
│   └── types.js
├── reducers/
│   ├── index.js
│   ├── counter.js
│   └── todos.js
├── store/
│   └── index.js
└── components/
    ├── Counter.js
    └── TodoList.js
```

### 5.2 命名规范
- Action Types: `UPPER_SNAKE_CASE`
- Action Creators: `camelCase`
- Reducers: `camelCase + Reducer`
- State 属性: `camelCase`

### 5.3 性能优化
```js
// 使用 React.memo 避免不必要的重渲染
const TodoItem = React.memo(({ todo, onToggle }) => {
  return (
    <li onClick={() => onToggle(todo.id)}>
      {todo.text}
    </li>
  );
});

// 使用 useCallback 缓存函数
const handleToggle = useCallback((id) => {
  dispatch(toggleTodo(id));
}, [dispatch]);
```

## 6. 调试工具

### 6.1 Redux DevTools
```js
// 安装浏览器扩展后自动启用
const store = createStore(
  rootReducer,
  window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__()
);
```

### 6.2 中间件调试
```js
import { createLogger } from 'redux-logger';

const logger = createLogger({
  collapsed: true,
  diff: true
});

const store = createStore(
  rootReducer,
  applyMiddleware(logger)
);
```

## 7. 总结

Redux 的核心配置项包括：
- **createStore**: 创建 store，配置 reducer、初始状态、中间件
- **combineReducers**: 合并多个 reducer
- **applyMiddleware**: 应用中间件
- **connect**: 连接组件与 store（Class 组件）
- **useSelector/useDispatch**: Hooks 方式连接 store（函数组件）

每个配置项都有其特定作用，合理配置可以实现高效的状态管理和组件通信。
