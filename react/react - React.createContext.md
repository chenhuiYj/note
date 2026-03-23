### React.createContext 详细解析
`React.createContext` 是 React 提供的用于**跨组件传递数据**的核心 API，解决了传统 props 逐级传递（"props 钻取"）的痛点，让数据可以在组件树中直接穿透多层组件传递，无需手动逐层传递。

#### 一、核心概念
1. **Context 是什么**：可以理解为 React 组件树中的一个"全局数据仓库"，能让任意层级的组件访问和修改其中的数据。
2. **createContext 作用**：创建一个 Context 对象，包含两个核心组件：
   - `Provider`：提供者，用于包裹需要共享数据的组件树，通过 `value` 属性传递数据（可以是任意类型：对象、函数、基本类型等）。
   - `Consumer`：消费者（已逐步被 `useContext` 替代），用于接收 Provider 传递的数据。
3. **默认值**：创建 Context 时可以传入默认值，仅在组件没有匹配到任何 Provider 时生效（注意：默认值不会因 Provider 的 `value` 为 `undefined` 而触发）。

---

#### 二、完整代码示例（基础用法）
下面以"用户信息共享"为例，展示 `createContext` 的完整使用流程：

```jsx
import React, { createContext, useContext, useState } from 'react';

// 1. 创建 Context 对象，可传入默认值（无 Provider 时生效）
const UserContext = createContext({
  name: '默认用户',
  age: 0,
  updateUser: () => {} // 也可传递修改数据的函数
});

// 2. 封装一个自定义 Hook（简化消费逻辑，推荐用法）
const useUserContext = () => {
  const context = useContext(UserContext);
  // 防止组件在 Context 外部使用时出错
  if (!context) {
    throw new Error('useUserContext 必须在 UserContext.Provider 内部使用');
  }
  return context;
};

// 子组件：深层嵌套，无需 props 传递直接获取用户信息
function ChildComponent() {
  // 3. 消费 Context 数据（推荐用 useContext Hook）
  const { name, age, updateUser } = useUserContext();

  return (
    <div style={{ margin: '20px', padding: '10px', border: '1px solid #eee' }}>
      <h3>子组件</h3>
      <p>用户名：{name}</p>
      <p>年龄：{age}</p>
      <button onClick={() => updateUser({ name: 'React 进阶', age: 30 })}>
        修改用户信息
      </button>
    </div>
  );
}

// 父组件：包裹 Provider，提供共享数据
function ParentComponent() {
  // 4. 定义需要共享的状态和方法
  const [user, setUser] = useState({
    name: 'React 新手',
    age: 20
  });

  // 修改用户信息的方法
  const updateUser = (newUser) => {
    setUser(newUser);
  };

  return (
    // 5. 使用 Provider 包裹组件树，通过 value 传递数据/方法
    <UserContext.Provider value={{ ...user, updateUser }}>
      <div style={{ padding: '20px', border: '1px solid #ccc' }}>
        <h2>父组件</h2>
        <ChildComponent />
      </div>
    </UserContext.Provider>
  );
}

// 根组件
function App() {
  return (
    <div>
      <h1>Context 示例</h1>
      <ParentComponent />
    </div>
  );
}

export default App;
```

#### 三、代码关键说明
1. **创建 Context**：
   ```jsx
   const UserContext = createContext(默认值);
   ```
   - 默认值仅在组件没有被 `UserContext.Provider` 包裹时生效（比如直接单独使用 `ChildComponent` 时）。
   - 示例中默认值包含 `name`、`age` 和空函数，保证消费时不会报错。

2. **Provider 提供者**：
   - 必须通过 `value` 属性传递数据，`value` 可以是任意类型（对象、函数、基本类型）。
   - 当 `value` 中的数据变化时，所有消费该 Context 的组件都会重新渲染。
   - 示例中传递了 `user` 状态和 `updateUser` 方法，让子组件既能读取也能修改数据。

3. **消费 Context（推荐用法）**：
   ```jsx
   const context = useContext(UserContext);
   ```
   - `useContext` 是 React Hooks，接收 Context 对象，返回 Provider 的 `value`。
   - 封装自定义 Hook `useUserContext` 是最佳实践：增加边界检查，避免组件在 Provider 外部使用时出现 `undefined` 错误。

4. **Consumer（旧用法，了解即可）**：
   早期没有 Hooks 时，需要用 `Consumer` 组件消费数据，语法较繁琐：
   ```jsx
   <UserContext.Consumer>
     {({ name, age }) => (
       <div>用户名：{name}，年龄：{age}</div>
     )}
   </UserContext.Consumer>
   ```

#### 四、进阶场景：多 Context 组合
如果需要共享多个独立的数据源（比如用户信息、主题设置），可以创建多个 Context 并嵌套 Provider：

```jsx
// 创建主题 Context
const ThemeContext = createContext({ theme: 'light', toggleTheme: () => {} });

function App() {
  const [theme, setTheme] = useState('light');
  const toggleTheme = () => setTheme(theme === 'light' ? 'dark' : 'light');

  return (
    <UserContext.Provider value={userData}>
      <ThemeContext.Provider value={{ theme, toggleTheme }}>
        <ParentComponent />
      </ThemeContext.Provider>
    </UserContext.Provider>
  );
}

// 子组件中同时消费两个 Context
function ChildComponent() {
  const { name } = useUserContext();
  const { theme } = useContext(ThemeContext);
  return <div style={{ background: theme === 'dark' ? '#333' : '#fff' }}>{name}</div>;
}
```

---

### 总结
1. **核心作用**：`React.createContext` 解决跨组件数据传递问题，替代 props 逐级传递，适用于全局/跨层级共享的数据（如用户状态、主题、语言等）。
2. **核心用法**：
   - `createContext` 创建 Context 对象 → `Provider` 提供数据 → `useContext` 消费数据。
   - 推荐封装自定义 Hook 消费 Context，增加边界检查，提升代码健壮性。
3. **注意事项**：
   - Provider 的 `value` 变化会触发所有消费组件重新渲染，避免传递频繁变化的对象（可通过 `useMemo` 优化）。
   - 每个 Context 独立，多数据源可创建多个 Context 嵌套使用。