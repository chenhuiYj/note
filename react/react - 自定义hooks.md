# React 自定义 Hooks 完整示例
---

## 一、useFetch —— 封装网络请求逻辑

统一管理接口请求的 data（数据）、loading（加载状态）、error（错误状态），支持立即请求/手动触发请求，适配 GET/POST 等各类请求方式。

```jsx
import { useState, useEffect, useCallback } from 'react';

/**
 * 自定义Hook：封装网络请求逻辑
 * @param {string} url - 请求地址（必传）
 * @param {Object} options - 请求配置（method、headers、body等，可选）
 * @param {boolean} immediate - 是否立即执行请求（默认true，可选）
 * @returns {Object} - { data: 接口返回数据, loading: 加载状态, error: 错误信息, refetch: 手动触发请求函数 }
 */
function useFetch(url, options = {}, immediate = true) {
  // 定义请求相关状态
  const [data, setData] = useState(null); // 接口返回数据
  const [loading, setLoading] = useState(false); // 加载中状态
  const [error, setError] = useState(null); // 错误信息

  // 封装核心请求逻辑，用useCallback缓存函数，避免组件重渲染时重复创建
  const fetchData = useCallback(async () => {
    // 发起请求前重置状态
    setLoading(true);
    setError(null);

    try {
      // 发送请求，合并默认配置与自定义配置
      const response = await fetch(url, {
        method: 'GET', // 默认请求方式为GET
        headers: {
          'Content-Type': 'application/json', // 默认请求头
          ...options.headers, // 合并用户自定义请求头
        },
        ...options, // 合并其他请求配置（如body、mode、credentials等）
      });

      // 校验请求是否成功（状态码200-299为成功）
      if (!response.ok) {
        throw new Error(`请求失败：${response.status} ${response.statusText}`);
      }

      // 解析响应数据（根据接口返回格式调整，可改为response.text()等）
      const result = await response.json();
      setData(result); // 存储接口返回数据
    } catch (err) {
      // 捕获请求过程中的异常，更新错误状态
      setError(err.message || '请求发生未知错误');
    } finally {
      // 无论请求成功/失败，都结束加载状态
      setLoading(false);
    }
  }, [url, options]); // 依赖url和options，仅当两者变化时重新生成fetchData

  // 立即执行请求（当immediate为true且url存在时）
  useEffect(() => {
    if (immediate && url) {
      fetchData();
    }
  }, [url, options, immediate, fetchData]); // 依赖变化时重新判断是否执行请求

  // 返回状态和手动触发请求的函数，供组件使用
  return { data, loading, error, refetch: fetchData };
}

// ============== useFetch 使用示例 ==============
function FetchExample() {
  // 调用自定义Hook，请求示例接口（JSONPlaceholder 测试接口）
  const { data, loading, error, refetch } = useFetch(
    'https://jsonplaceholder.typicode.com/todos/1', // 请求地址
    { method: 'GET' }, // 请求配置（这里用GET，POST可传body）
    true // 立即执行请求
  );

  // 加载中状态展示
  if (loading) return <div style={{ padding: 20 }}>加载中...</div>;
  // 错误状态展示
  if (error) return <div style={{ padding: 20, color: 'red' }}>错误：{error}</div>;

  // 成功状态展示数据
  return (
    <div style={{ padding: 20, border: '1px solid #eee', borderRadius: 8, margin: 10 0 }}>
      <h3 style={{ margin: 0 0 15px 0 }}>useFetch 示例 - 任务信息</h3>
      {data && (
        <div>
          <p><strong>任务ID：</strong>{data.id}</p>
          <p><strong>任务标题：</strong>{data.title}</p>
          <p><strong>完成状态：</strong>{data.completed ? '✅ 已完成' : '❌ 未完成'}</p>
        </div>
      )}
      <button 
        onClick={refetch} 
        style={{ marginTop: 15, padding: 6 12, cursor: 'pointer' }}
      >
        重新加载数据
      </button>
    </div>
  );
}
```

---

## 二、useLocalStorage —— 本地存储状态持久化

实现 React 状态与 localStorage 双向同步，刷新页面数据不丢失，支持直接像 useState 一样使用，自动处理 JSON 序列化/反序列化。

```jsx
import { useState, useCallback } from 'react';

/**
 * 自定义Hook：封装localStorage操作，同步React状态与本地存储
 * @param {string} key - 本地存储的key（必传，用于标识存储内容）
 * @param {any} initialValue - 初始值（本地存储无数据时使用，可选）
 * @returns {[any, Function]} - [存储的值, 更新值的函数]（与useState返回格式一致）
 */
function useLocalStorage(key, initialValue) {
  // 初始化状态：优先从localStorage读取，无数据则使用初始值
  const [storedValue, setStoredValue] = useState(() => {
    try {
      // 从localStorage获取指定key的值
      const item = window.localStorage.getItem(key);
      // 解析JSON（若item为null，说明无存储数据，返回初始值）
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      // 捕获异常（如隐私模式下localStorage不可用）
      console.error('读取localStorage失败：', error);
      return initialValue;
    }
  });

  // 封装更新函数：同时更新React状态和localStorage
  const setValue = useCallback(
    (value) => {
      try {
        // 支持传入函数（类似setState），接收当前值计算新值
        const valueToStore = value instanceof Function ? value(storedValue) : value;
        // 更新React状态
        setStoredValue(valueToStore);
        // 更新localStorage（转为JSON字符串存储，避免类型丢失）
        window.localStorage.setItem(key, JSON.stringify(valueToStore));
      } catch (error) {
        console.error('写入localStorage失败：', error);
      }
    },
    [key, storedValue] // 依赖key和当前存储值，确保更新逻辑正确
  );

  // 返回值和更新函数，用法与useState完全一致
  return [storedValue, setValue];
}

// ============== useLocalStorage 使用示例 ==============
function LocalStorageExample() {
  // 调用自定义Hook，绑定本地存储key为"username"，初始值为空字符串
  const [username, setUsername] = useLocalStorage('username', '');
  // 示例：存储对象类型数据
  const [userInfo, setUserInfo] = useLocalStorage('userInfo', { age: 0, gender: '' });

  return (
    <div style={{ padding: 20, border: '1px solid #eee', borderRadius: 8, margin: 10 0 }}>
      <h3 style={{ margin: 0 0 15px 0 }}>useLocalStorage 示例 - 本地存储</h3>
      
      <div style={{ marginBottom: 20 }}>
        <label style={{ marginRight: 10 }}>用户名：</label>
        <input
          type="text"
          value={username}
          onChange={(e) => setUsername(e.target.value)}
          placeholder="请输入用户名"
          style={{ padding: 6, width: 200 }}
        />
        <p style={{ marginTop: 5 }}>当前存储的用户名：{username || '未输入'}</p>
      </div>

      <div>
        <label style={{ marginRight: 10 }}>用户年龄：</label>
        <input
          type="number"
          value={userInfo.age}
          onChange={(e) => setUserInfo({ ...userInfo, age: Number(e.target.value) })}
          style={{ padding: 6, width: 200 }}
        />
        <p style={{ marginTop: 5 }}>当前存储的用户信息：{JSON.stringify(userInfo)}</p>
      </div>

      <button 
        onClick={() => {
          setUsername('');
          setUserInfo({ age: 0, gender: '' });
        }} 
        style={{ marginTop: 15, padding: 6 12, cursor: 'pointer' }}
      >
        清空本地存储
      </button>
      <p style={{ marginTop: 10, color: '#666', fontSize: 12 }}>（可刷新页面验证数据是否持久化）</p>
    </div>
  );
}
```

---

## 三、useDebounce —— 防抖Hook（高频触发优化）

用于处理输入框搜索、窗口resize、滚动等高频触发场景，延迟执行函数，减少资源消耗。包含两个版本：防抖值（useDebounce）、防抖函数（useDebounceFn）。

```jsx
import { useState, useEffect, useCallback } from 'react';

/**
 * 自定义Hook：防抖值（适用于输入框等场景，返回防抖后的值）
 * @param {any} value - 需要防抖的值（必传）
 * @param {number} delay - 防抖延迟时间（毫秒，默认500ms，可选）
 * @returns {any} - 防抖后的值
 */
function useDebounce(value, delay = 500) {
  // 定义防抖后的值的状态
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    // 设置定时器：延迟delay毫秒后更新防抖值
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    // 清理函数：当value/delay变化或组件卸载时，清除定时器（避免内存泄漏）
    return () => {
      clearTimeout(timer);
    };
  }, [value, delay]); // 仅当value或delay变化时，重新设置定时器

  return debouncedValue;
}

/**
 * 自定义Hook：防抖函数（适用于点击、提交等事件，返回防抖后的函数）
 * @param {Function} fn - 需要防抖的函数（必传）
 * @param {number} delay - 防抖延迟时间（毫秒，默认500ms，可选）
 * @returns {Function} - 防抖后的函数
 */
function useDebounceFn(fn, delay = 500) {
  const debouncedFn = useCallback(
    (...args) => {
      // 用定时器存储标识，每次调用前清除上一次的定时器
      let timer;
      clearTimeout(timer);
      timer = setTimeout(() => {
        fn(...args); // 延迟后执行原函数，并传递参数
      }, delay);
    },
    [fn, delay] // 仅当原函数或延迟时间变化时，重新生成防抖函数
  );

  return debouncedFn;
}

// ============== useDebounce 使用示例 ==============
function DebounceExample() {
  // 原始输入值（实时变化）
  const [inputValue, setInputValue] = useState('');
  // 防抖后的值（延迟500ms更新）
  const debouncedValue = useDebounce(inputValue, 500);

  // 模拟搜索请求（使用防抖函数）
  const handleSearch = useDebounceFn((value) => {
    console.log('发起搜索请求：', value);
    // 实际场景中可在这里调用接口
  }, 500);

  return (
    <div style={{ padding: 20, border: '1px solid #eee', borderRadius: 8, margin: 10 0 }}>
      <h3 style={{ margin: 0 0 15px 0 }}>useDebounce 示例 - 输入防抖</h3>
      
      <input
        type="text"
        value={inputValue}
        onChange={(e) => setInputValue(e.target.value)}
        placeholder="输入内容测试防抖（如搜索关键词）"
        style={{ padding: 6, width: 300 }}
      />
      
      <div style={{ marginTop: 15 }}>
        <p><strong>实时输入值：</strong>{inputValue}</p>
        <p><strong>防抖后的值（500ms后更新）：</strong>{debouncedValue}</p>
      </div>
      
      <button
        onClick={() => handleSearch(debouncedValue)}
        style={{ marginTop: 15, padding: 6 12, cursor: 'pointer' }}
        disabled={!debouncedValue} // 无内容时禁用按钮
      >
        搜索（防抖触发）
      </button>
      <p style={{ marginTop: 10, color: '#666', fontSize: 12 }}>
        （快速输入时，搜索函数仅在停止输入500ms后执行）
      </p>
    </div>
  );
}
```

---

## 四、完整入口组件（整合所有示例）

将上述所有示例组件整合到一个入口组件中，可直接复制到 React 项目中运行。

```jsx
import React from 'react';
import ReactDOM from 'react-dom/client';

// 引入上述所有自定义Hook和示例组件
// （实际项目中可拆分到单独文件，如 hooks/useFetch.js、components/FetchExample.js）

function App() {
  return (
    <div style={{ padding: 30, maxWidth: 800, margin: 0 auto }}>
      <h2 style={{ textAlign: 'center', marginBottom: 30 }}>React 自定义 Hooks 示例合集</h2>
      
      <FetchExample />
      <LocalStorageExample />
      <DebounceExample />
    </div>
  );
}

// 渲染到DOM（需在public/index.html中创建id为root的容器）
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<App />);

export default App;
```

---

## 五、自定义 Hooks 核心规则

1. 命名规范：必须以 `use` 开头（如 useFetch、useDebounce），React 才能识别其为 Hook，避免与普通函数混淆。

2. 调用规则：只能在 React 函数组件、自定义 Hooks 的顶层调用，不能在条件语句、循环、嵌套函数中调用（确保 Hooks 执行顺序一致）。

3. 复用逻辑：专注于复用**状态逻辑**（如请求状态、存储逻辑、防抖逻辑），不负责 UI 渲染，UI 渲染由使用 Hook 的组件负责。

4. 依赖管理：使用 useEffect、useCallback 等 Hooks 时，务必正确设置依赖数组，避免无效重渲染或逻辑错误。

---

## 六、扩展说明

- 所有代码均基于 React 18+ 编写，兼容最新版本 React，无需额外依赖。

- 可根据实际业务需求调整 Hook 逻辑（如 useFetch 支持请求取消、useLocalStorage 改为 sessionStorage 等）。

- 建议将每个自定义 Hook 拆分到单独文件（如 `src/hooks/useFetch.js`），便于项目维护。

---