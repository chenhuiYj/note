## 1.概念
mobx 通过透明的函数响应式编程(transparently applying functional reactive programming - TFRP)使得状态管理变得简单和可扩展。
## 2.举例
```javascript
// src/stores/todoStore.js
import { useLocalObservable } from 'mobx-react-lite';

const useTodoStore = () => {
  const todos = useLocalObservable(() => ({
    items: [],

    addTodo(text) {
      if (text.trim()) {
        this.items.push({
          id: Date.now(),
          text,
          completed: false,
        });
      }
    },

    removeTodo(id) {
      this.items = this.items.filter(todo => todo.id !== id);
    },

    toggleTodo(id) {
      const todo = this.items.find(todo => todo.id === id);
      if (todo) {
        todo.completed = !todo.completed;
      }
    },
  }));

  return todos;
};

export default useTodoStore;
```

```javascript
// src/components/TodoList.js
import React, { useState } from 'react';
import useTodoStore from '../stores/todoStore';

const TodoList = () => {
  const todos = useTodoStore();
  const [input, setInput] = useState('');

  const handleAddTodo = () => {
    if (input.trim()) {
      todos.addTodo(input);
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
        {todos.items.map(todo => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => todos.toggleTodo(todo.id)}
            />
            {todo.text}
            <button onClick={() => todos.removeTodo(todo.id)}>Remove</button>
          </li>
        ))}
      </ul>
    </div>
  );
};

export default TodoList;
```
```javascript
// src/App.js
import React from 'react';
import TodoList from './components/TodoList';

const App = () => {
  return (
    <div>
      <TodoList />
    </div>
  );
};

export default App;
```