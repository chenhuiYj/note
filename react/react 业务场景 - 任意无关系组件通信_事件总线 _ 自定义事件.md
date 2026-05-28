## 场景说明（工作真实业务）

### 业务场景

**两个完全没有关系的组件（跨页面、跨层级、无父子 / 兄弟关系），需要互相通信、触发对方更新。**

典型例子：

1. **头部 Header 组件** 点击「切换主题」

2. **底部 Footer 组件** 要同步变颜色

3. **内容区 Content 组件** 也要同步变样式

这三个组件**完全无嵌套、无关系、无公共父组件**，不能用 props、不能用状态提升，必须用：

## 解决方案：事件总线（Event Bus）

- 自定义事件监听 \+ 触发

- 任意组件都能**发射事件**

- 任意组件都能**监听事件**

- 完全解耦，无关系组件也能通信

---

# 完整代码（可直接运行）

## 1\. 创建事件总线工具：eventBus\.js

```jsx
// 事件总线：实现任意无关系组件通信
class EventBus {
  constructor() {
    // 存储所有事件与回调
    this.events = {};
  }

  // 订阅事件（监听）
  on(eventName, callback) {
    if (!this.events[eventName]) {
      this.events[eventName] = [];
    }
    this.events[eventName].push(callback);
  }

  // 发布事件（触发）
  emit(eventName, data) {
    if (!this.events[eventName]) return;
    this.events[eventName].forEach((cb) => cb(data));
  }

  // 取消订阅
  off(eventName, callback) {
    if (!this.events[eventName]) return;
    this.events[eventName] = this.events[eventName].filter(
      (cb) => cb !== callback
    );
  }
}

// 单例模式，全局唯一
export default new EventBus();
```

---

## 2\. 组件 A：Header\.jsx（发送事件）

**作用：点击按钮 → 发布事件（切换主题）**

```jsx
import { useState } from 'react';
import eventBus from './eventBus';

const Header = () => {
  const [theme, setTheme] = useState('light');

  // 点击切换主题 → 发射事件
  const changeTheme = () => {
    const newTheme = theme === 'light' ? 'dark' : 'light';
    setTheme(newTheme);

    // 关键：向全局发布事件，携带数据
    eventBus.emit('themeChange', newTheme);
  };

  return (
    <div style={{ padding: 20, background: '#eee' }}>
      <h3>Header 组件（无关系组件 A）</h3>
      <button onClick={changeTheme}>点击切换主题</button>
    </div>
  );
};

export default Header;
```

---

## 3\. 组件 B：Footer\.jsx（接收事件）

**作用：监听事件 → 自动更新主题**

```jsx
import { useState, useEffect } from 'react';
import eventBus from './eventBus';

const Footer = () => {
  const [theme, setTheme] = useState('light');

  useEffect(() => {
    // 关键：监听事件
    const handleThemeChange = (newTheme) => {
      setTheme(newTheme);
    };

    // 订阅事件
    eventBus.on('themeChange', handleThemeChange);

    // 组件卸载时取消订阅（防止内存泄漏）
    return () => {
      eventBus.off('themeChange', handleThemeChange);
    };
  }, []);

  return (
    <div
      style={{
        padding: 20,
        background: theme === 'dark' ? '#333' : '#fff',
        color: theme === 'dark' ? '#fff' : '#000',
      }}
    >
      <h3>Footer 组件（无关系组件 B）</h3>
      <p>当前主题：{theme}</p>
    </div>
  );
};

export default Footer;
```

---

## 4\. 组件 C：Content\.jsx（也能接收）

```jsx
import { useState, useEffect } from 'react';
import eventBus from './eventBus';

const Content = () => {
  const [theme, setTheme] = useState('light');

  useEffect(() => {
    const handleThemeChange = (newTheme) => {
      setTheme(newTheme);
    };
    eventBus.on('themeChange', handleThemeChange);
    return () => {
      eventBus.off('themeChange', handleThemeChange);
    };
  }, []);

  return (
    <div
      style={{
        padding: 40,
        background: theme === 'dark' ? '#555' : '#f7f7f7',
      }}
    >
      <h3>Content 组件（无关系组件 C）</h3>
    </div>
  );
};

export default Content;
```

---

## 5\. 入口组件 App\.jsx（三个组件完全无关系）

```jsx
import Header from './Header';
import Content from './Content';
import Footer from './Footer';

function App() {
  return (
    <div>
      {/* 三个组件完全独立、无嵌套、无关系 */}
      <Header />
      <Content />
      <Footer />
    </div>
  );
}

export default App;
```

---

# 执行流程（面试必说）

1. **Header 点击按钮**

2. 调用 `eventBus\.emit\(\&\#39;themeChange\&\#39;, 新主题\)`

3. **Footer、Content 都监听了 themeChange**

4. 所有监听组件**自动收到数据并更新**

---

# 核心特点（面试必背）

1. **组件完全无关系也能通信**

2. 一对多、多对多通信都支持

3. 解耦强，不依赖父子 / 兄弟关系

4. 适用于：**全局主题、全局消息、跨页面触发、无关系组件联动**

---

# 面试官最爱问

### 问：EventBus 有什么缺点？

答：

- 事件名容易冲突

- 难以追踪数据流

- 大型项目不推荐，适合简单项目

### 问：大型项目无关系组件通信用什么？

答：用全局状态管理（Zustand、Redux、Pinia），不用 EventBus。

---
