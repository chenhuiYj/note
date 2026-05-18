# 解决 Props Drilling 的 React Context 方案

# 场景：祖孙多层级传值繁琐（Props 层层透传）

## 业务实际场景

项目结构：`App\(祖组件\)` → `Father\(父组件\)` → `Son\(孙组件\)`
需求：**祖组件存放全局用户信息、主题色**，孙组件直接使用，不想经过中间父组件逐层传递 props，解决 `Props Drilling` 属性钻取问题。

## 技术方案

`React\.createContext` 创建全局上下文 \+ `Context\.Provider` 顶层注入数据 \+ 后代组件 `useContext` 直接取值，**中间组件无需接收任何参数**。

---

# 完整项目代码（拆分 3 个组件 \+ 可直接运行）

## 1\. 新建上下文文件 `UserContext\.js`

```jsx
import { createContext } from 'react';

// 1. 创建上下文，设置默认值
const UserContext = createContext({
  username: '游客',
  theme: 'light',
  changeTheme: () => {}
});

export default UserContext;
```

## 2\. 祖组件 App\.jsx（数据提供方 Provider）

```jsx
import { useState } from 'react';
import UserContext from './UserContext';
import Father from './Father';

const App = () => {
  // 全局共享状态
  const [username, setUsername] = useState('前端开发者');
  const [theme, setTheme] = useState('light');

  // 全局修改主题方法，跨层传给子孙组件
  const changeTheme = () => {
    setTheme(theme === 'light' ? 'dark' : 'light');
  };

  // 统一要向下穿透的数据和方法
  const contextValue = {
    username,
    theme,
    changeTheme
  };

  return (
    {/* 顶层包裹，所有后代组件都能消费 */}
    <UserContext.Provider value={contextValue}>
      <div style={{
        padding: 20,
        background: theme === 'dark' ? '#333' : '#fff',
        color: theme === 'dark' ? '#fff' : '#000'
      }}>
        <h2>祖组件 App</h2>
        <button onClick={changeTheme}>切换全局主题</button>
        <hr />
        {/* 中间父组件，无需传任何props */}
        <Father />
      </div>
    </UserContext.Provider>
  );
};

export default App;
```

## 3\. 中间层父组件 Father\.jsx（中转站，无需接收 / 转发参数）

```jsx
import Son from './Son';

const Father = () => {
  console.log('中间父组件，不接收任何props');
  return (
    <div style={{ border: '1px solid #ccc', padding: 15, margin: 10 }}>
      <h3>中间父组件 Father</h3>
      {/* 直接渲染孙组件，无任何传参 */}
      <Son />
    </div>
  );
};

export default Father;
```

## 4\. 孙组件 Son\.jsx（跨层直接消费数据）

```jsx
import { useContext } from 'react';
import UserContext from './UserContext';

const Son = () => {
  // 直接从上下文取出全局数据与方法
  const { username, theme, changeTheme } = useContext(UserContext);

  return (
    <div style={{
      border: '1px solid skyblue',
      padding: 15,
      margin: 10,
      background: theme === 'dark' ? '#555' : '#f0f7ff'
    }}>
      <h4>孙组件 Son</h4>
      <p>当前登录用户：{username}</p>
      <p>当前全局主题：{theme === 'light' ? '浅色模式' : '深色模式'}</p>
      <button onClick={changeTheme}>孙组件内切换主题</button>
    </div>
  );
};

export default Son;
```

---

# 执行效果

1. 祖组件定义**用户信息、主题状态、修改主题方法**

2. 用 `UserContext\.Provider` 包裹所有后代，注入数据

3. 中间 `Father` 组件**零 props**，不用转发任何数据

4. 最底层孙组件直接 `useContext` 获取数据 \+ 调用方法

5. 无论祖组件、孙组件点击切换主题，**全局所有消费组件同步更新**

---

# 面试核心要点背诵

1. **解决问题**：解决多层嵌套组件层层传递 props 造成的属性钻取（Props Drilling）。

2. **使用流程**

    - `createContext\(\)` 创建上下文容器

    - 顶层组件用 `XXX\.Provider` 传入共享数据

    - 任意后代组件使用 `useContext\(上下文实例\)` 直接取值

3. **适用场景**

    - 全局通用数据：用户信息、登录态、主题、语言、全局配置

    - 多层嵌套组件跨层级通信

4. **缺点 \&amp; 优化**

    - 上下文数据一变，所有消费组件都会重渲染

    - 复杂全局状态优先用 Zustand / Redux，简单跨层用 Context 足够

5. **和状态提升区别**

    - 状态提升：只适合**兄弟 / 浅层父子**

    - Context：适合**深层嵌套、全局跨层级**

---

# 面试官追问标准答案

### 1\. useContext 穿透数据，组件会不必要重渲染吗？

会，只要 Provider 的 `value` 整体引用变化，所有使用 `useContext` 的组件都会重新渲染；可以拆分多个 Context、配合 useMemo 缓存 value 优化。

### 2\. Context 和 Redux 分别怎么选？

- 简单跨层级共享、少量全局字段 → 用 Context

- 数据量大、修改逻辑复杂、多组件频繁读写 → 用状态管理库

> （注：文档部分内容可能由 AI 生成）
