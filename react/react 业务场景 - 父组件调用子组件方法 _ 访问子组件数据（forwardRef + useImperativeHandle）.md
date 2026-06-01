# React 场景题：父组件调用子组件方法 / 访问子组件数据

## （forwardRef \+ useImperativeHandle 完整版）

---

# 一、场景说明（工作真实需求）

## 业务场景

父组件里有一个按钮，**点击这个按钮，要主动触发子组件内部的方法**（比如：表单提交、重置、刷新、关闭弹窗、执行动画等）。

### 典型例子：

1. 父组件：页面按钮

2. 子组件：内部封装了表单（Form）

3. 需求：**点击父组件按钮 → 触发子组件里的表单提交 / 重置**

### 为什么不能直接调用？

- 子组件的方法是**内部私有**的

- 父组件不能直接拿到子组件实例

- 必须用 React 提供的 **ref 转发 \+ 暴露方法** 方案

---

# 二、核心 API 作用（必须背）

1. **`forwardRef`**
让子组件能够接收、转发父组件传过来的 `ref`

2. **`useImperativeHandle`**
让子组件**主动暴露**指定方法 / 数据给父组件
→ 父组件就能通过 `ref.current.xxx()` 调用子组件方法

---

# 三、完整代码（拆分 3 个文件，可直接运行）

## 1\. 子组件：ChildForm\.jsx（内部有方法，暴露给父组件）

```jsx
import { forwardRef, useImperativeHandle, useState } from 'react';

// forwardRef 让子组件接收 ref
const ChildForm = forwardRef((props, ref) => {
  const [formData, setFormData] = useState('表单内容');

  // 子组件内部方法：提交表单
  const submitForm = () => {
    alert('子组件表单提交：' + formData);
  };

  // 子组件内部方法：重置表单
  const resetForm = () => {
    setFormData('');
    alert('子组件表单已重置');
  };

  // 关键：向父组件暴露方法
  useImperativeHandle(ref, () => ({
    // 暴露给父组件调用的方法
    submitForm,
    resetForm,
    // 也可以暴露数据
    formData,
  }));

  return (
    <div style={{ border: '1px solid #aaa', padding: 20, margin: 10 }}>
      <h3>子组件 Form</h3>
      <input
        value={formData}
        onChange={(e) => setFormData(e.target.value)}
        placeholder="输入表单内容"
      />
    </div>
  );
});

export default ChildForm;
```

---

## 2\. 父组件：Parent\.jsx（调用子组件方法）

```jsx
import { useRef } from 'react';
import ChildForm from './ChildForm';

const Parent = () => {
  // 创建 ref，用于绑定子组件
  const childRef = useRef(null);

  // 父组件按钮：调用子组件的提交方法
  const handleChildSubmit = () => {
    // 调用子组件暴露的 submitForm
    childRef.current.submitForm();
  };

  // 父组件按钮：调用子组件的重置方法
  const handleChildReset = () => {
    childRef.current.resetForm();
  };

  // 获取子组件内部数据
  const getChildData = () => {
    alert('子组件数据：' + childRef.current.formData);
  };

  return (
    <div style={{ padding: 20 }}>
      <h2>父组件</h2>

      {/* 父组件按钮：控制子组件 */}
      <button onClick={handleChildSubmit}>点击调用子组件提交</button>
      <button onClick={handleChildReset} style={{ marginLeft: 10 }}>
        点击调用子组件重置
      </button>
      <button onClick={getChildData} style={{ marginLeft: 10 }}>
        获取子组件数据
      </button>

      <hr style={{ margin: '20px 0' }} />

      {/* 子组件绑定 ref */}
      <ChildForm ref={childRef} />
    </div>
  );
};

export default Parent;
```

---

## 3\. 入口 App\.jsx

```jsx
import Parent from './Parent';

function App() {
  return <Parent />;
}

export default App;
```

---

# 四、执行流程（面试必说）

1. 父组件创建 `useRef`

2. 父组件把 `ref` 传给子组件

3. 子组件用 `forwardRef` 接收这个 `ref`

4. 子组件用 `useImperativeHandle` 把**内部方法暴露**给 ref

5. 父组件通过 `ref.current.xxx()` **主动调用子组件方法**

---

# 五、高频业务使用场景（工作必用）

1. 父组件按钮 → 触发子组件表单提交

2. 父组件 → 关闭子组件弹窗

3. 父组件 → 让子组件刷新列表

4. 父组件 → 获取子组件内部表单数据

5. 父组件 → 控制子组件动画 / 滚动

6. 富文本、上传组件、地图组件… 都要用这种方式

---

# 六、面试官标准答案（必背）

### 问：forwardRef 和 useImperativeHandle 作用是什么？

**答：**

- `forwardRef` 用来让子组件接收父组件传递的 `ref`。

- `useImperativeHandle` 让子组件**自定义暴露**方法 / 属性给父组件，避免父组件直接操作整个 DOM。

- 最终实现：**父组件调用子组件内部方法**。

### 问：什么时候用？

**答：**
需要**父组件主动控制子组件行为**时使用，比如：表单提交、弹窗关闭、列表刷新、获取子组件内部状态。

---
