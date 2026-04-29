# React 父子组件 挂载/更新/卸载 执行顺序 面试完整版\.md

## 一、核心结论（面试背诵版）

### 1\. 挂载执行顺序

1. 父组件 函数体同步代码执行

2. 子组件 函数体同步代码执行

3. 子组件 useEffect 副作用执行

4. 父组件 useEffect 副作用执行

> 口诀：**先父后子渲染，先子后父副作用**
> 
> 

### 2\. 更新执行顺序

1. 父组件 函数体重新执行

2. 子组件 函数体重新执行

3. 子组件 useEffect 副作用执行

4. 父组件 useEffect 副作用执行

> 更新流程和挂载流程**顺序完全一致**
> 
> 

### 3\. 卸载执行顺序

1. 子组件 useEffect 清理函数先执行

2. 父组件 useEffect 清理函数后执行

> 口诀：**卸载先子后父**
> 
> 

---

## 二、原理一句话解释

函数组件执行分为两阶段：

1. **渲染阶段**：自上而下执行父子组件函数体同步代码

2. **提交阶段**：DOM 渲染完成后，**自下而上**执行 useEffect 副作用与清理函数

---

## 三、完整代码（拆分两个文件 \+ 详细注释）

### 1\. 父组件 Parent\.jsx

```jsx
import { useState, useEffect } from 'react';
import Child from './Child';

const Parent = () => {
  // 1. 渲染阶段：同步代码最先执行
  const [count, setCount] = useState(0);
  console.log('【1】父组件 - 函数体同步代码执行');

  // 副作用、清理函数
  useEffect(() => {
    console.log('【4】父组件 - useEffect 副作用执行');

    // 返回清理函数：组件卸载 / 依赖变化前执行
    return () => {
      console.log('父组件 - 卸载/更新 清理函数执行');
    };
  }, [count]);

  return (
    <div style={{ border: '2px solid #1890ff', padding: '20px', margin: '20px' }}>
      <h3>父组件 count：{count}</h3>
      {/* 点击触发父组件state更新，走更新生命周期 */}
      <button onClick={() => setCount(prev => prev + 1)}>
        触发父组件更新
      </button>

      {/* 嵌套子组件 */}
      <Child />
    </div>
  );
};

export default Parent;
```

### 2\. 子组件 Child\.jsx

```jsx
import { useEffect } from 'react';

const Child = () => {
  // 2. 渲染阶段：父执行完，再执行子组件同步代码
  console.log('【2】子组件 - 函数体同步代码执行');

  useEffect(() => {
    console.log('【3】子组件 - useEffect 副作用执行');

    // 子组件清理函数
    return () => {
      console.log('子组件 - 卸载/更新 清理函数执行');
    };
  }, []);

  return (
    <div style={{ border: '2px solid #f5222d', padding: '20px', marginTop: '10px' }}>
      <h4>子组件</h4>
    </div>
  );
};

export default Child;
```

### 3\. 入口使用

```jsx
import Parent from './Parent';

function App() {
  return (
    <div className="App">
      <Parent />
    </div>
  );
}

export default App;
```

---

## 四、控制台打印输出结果

### 1\. 首次挂载页面

```Plain Text
【1】父组件 - 函数体同步代码执行
【2】子组件 - 函数体同步代码执行
【3】子组件 - useEffect 副作用执行
【4】父组件 - useEffect 副作用执行
```

### 2\. 点击按钮触发更新

```Plain Text
【1】父组件 - 函数体同步代码执行
【2】子组件 - 函数体同步代码执行
子组件 - 卸载/更新 清理函数执行
父组件 - 卸载/更新 清理函数执行
【3】子组件 - useEffect 副作用执行
【4】父组件 - useEffect 副作用执行
```

### 3\. 组件卸载（路由离开 / 组件销毁）

```Plain Text
子组件 - 卸载/更新 清理函数执行
父组件 - 卸载/更新 清理函数执行
```

---

## 五、面试标准口述答案

1. 父子组件**挂载**：先从上到下执行父、子组件的函数体同步代码；再从下到上，先执行子组件 useEffect，再执行父组件 useEffect。

2. **更新**流程和挂载完全一样：先父后子执行渲染代码，再先子后父执行副作用。

3. **卸载**时先执行子组件的清理函数，再执行父组件清理函数。

4. 本质是 React 分**渲染阶段（自上而下）和提交副作用阶段（自下而上）**。

---

## 六、高频面试追问 \&amp; 答案

### 追问 1：为什么 useEffect 是子先执行、父后执行？

React 先完成所有子组件 DOM 挂载渲染，再通知父组件整个节点树挂载完成，所以副作用**自下而上**执行。

### 追问 2：依赖数组为空和有依赖，执行顺序变吗？

**顺序不变**，只是执行时机和次数变，父子执行先后顺序永远遵循：渲染自上而下，副作用自下而上。

---
