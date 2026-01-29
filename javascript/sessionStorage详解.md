# sessionStorage 详解：为什么不同 Tab 页不共享？

## 📚 目录

1. [sessionStorage 的设计原理](#sessionstorage-的设计原理)
2. [为什么不同标签页不共享](#为什么不同标签页不共享)
3. [sessionStorage vs localStorage](#sessionstorage-vs-localstorage)
4. [实际示例](#实际示例)
5. [如果需要跨标签页共享数据](#如果需要跨标签页共享数据)
6. [sessionStorage 的使用场景](#sessionstorage-的使用场景)
7. [总结](#总结)

---

## sessionStorage 的设计原理

### 1. 作用域限制

`sessionStorage` 的作用域是**单个浏览器标签页（Tab）**，而不是整个浏览器会话。每个标签页都有自己独立的 `sessionStorage` 空间。

### 2. 为什么这样设计？

- ✅ **隔离性**：不同标签页的数据互不干扰
- ✅ **安全性**：避免标签页间数据泄露
- ✅ **生命周期**：标签页关闭时自动清理

---

## 为什么不同标签页不共享

### 核心原因

`sessionStorage` 的设计目标就是**为每个标签页提供独立的存储空间**，这是浏览器的标准行为，不是 bug。

### 技术实现

每个浏览器标签页都有自己独立的：
- JavaScript 执行上下文
- DOM 环境
- Storage 存储空间（sessionStorage）

即使打开同一个 URL，不同的标签页也会创建完全独立的 `sessionStorage` 实例。

---

## sessionStorage vs localStorage

| 特性 | sessionStorage | localStorage |
|------|----------------|---------------|
| **作用域** | 单个标签页 | 同源的所有标签页 |
| **生命周期** | 标签页关闭时清除 | 除非手动清除，否则永久保存 |
| **跨标签页共享** | ❌ 不共享 | ✅ 共享 |
| **存储大小** | 通常 5-10MB | 通常 5-10MB |
| **触发 storage 事件** | ❌ 不触发（同标签页内） | ✅ 触发（其他标签页） |
| **数据持久化** | 临时存储 | 永久存储 |
| **使用场景** | 临时数据、表单数据 | 用户偏好、长期数据 |

### 详细对比

#### 1. 作用域对比

```javascript
// sessionStorage - 单个标签页
// 标签页 A
sessionStorage.setItem('key', 'value-A');
console.log(sessionStorage.getItem('key')); // 'value-A'

// 标签页 B（新打开的标签页）
console.log(sessionStorage.getItem('key')); // null（无法访问）

// localStorage - 所有同源标签页
// 标签页 A
localStorage.setItem('key', 'value-A');
console.log(localStorage.getItem('key')); // 'value-A'

// 标签页 B（可以访问）
console.log(localStorage.getItem('key')); // 'value-A'（可以访问）
```

#### 2. 生命周期对比

```javascript
// sessionStorage - 标签页关闭时自动清除
sessionStorage.setItem('temp', 'data');
// 关闭标签页后，数据自动清除

// localStorage - 需要手动清除
localStorage.setItem('persistent', 'data');
// 关闭标签页后，数据仍然保留
// 需要手动清除：localStorage.removeItem('persistent')
```

#### 3. storage 事件对比

```javascript
// localStorage - 可以监听其他标签页的变化
window.addEventListener('storage', (e) => {
  console.log('其他标签页修改了 localStorage:', e.key, e.newValue);
});

// sessionStorage - 不会触发 storage 事件（即使是同标签页内）
// 因为 sessionStorage 不支持跨标签页通信
```

---

## 实际示例

### 示例 1: 验证 sessionStorage 不共享

```html
<!-- 标签页 A: index.html -->
<!DOCTYPE html>
<html>
<head>
  <title>标签页 A</title>
</head>
<body>
  <button onclick="setData()">设置数据</button>
  <button onclick="getData()">获取数据</button>
  <div id="result"></div>

  <script>
    function setData() {
      sessionStorage.setItem('test', '来自标签页 A');
      document.getElementById('result').textContent = '已设置: 来自标签页 A';
    }

    function getData() {
      const value = sessionStorage.getItem('test');
      document.getElementById('result').textContent = '获取到: ' + (value || 'null');
    }
  </script>
</body>
</html>
```

```html
<!-- 标签页 B: index2.html -->
<!DOCTYPE html>
<html>
<head>
  <title>标签页 B</title>
</head>
<body>
  <button onclick="getData()">获取数据</button>
  <div id="result"></div>

  <script>
    function getData() {
      // 即使标签页 A 设置了数据，这里也获取不到
      const value = sessionStorage.getItem('test');
      document.getElementById('result').textContent = '获取到: ' + (value || 'null（无法访问标签页 A 的数据）');
    }
  </script>
</body>
</html>
```

### 示例 2: localStorage 可以共享

```javascript
// 标签页 A
localStorage.setItem('shared', '这是共享数据');
console.log('标签页 A 设置:', localStorage.getItem('shared'));

// 标签页 B（监听变化）
window.addEventListener('storage', (e) => {
  if (e.key === 'shared') {
    console.log('标签页 B 收到更新:', e.newValue);
  }
});

// 标签页 B 也可以直接读取
console.log('标签页 B 读取:', localStorage.getItem('shared'));
```

---

## 如果需要跨标签页共享数据

### 方案一：使用 localStorage（推荐，兼容性好）

#### 优点
- ✅ 所有现代浏览器都支持
- ✅ 简单易用
- ✅ 可以监听其他标签页的变化（通过 `storage` 事件）

#### 缺点
- ⚠️ 数据会持久化（除非手动清除）
- ⚠️ 需要手动管理数据生命周期

#### 实现示例

```javascript
// 标签页 A - 发送数据
function sendDataToOtherTabs(data) {
  localStorage.setItem('shared-data', JSON.stringify({
    data: data,
    timestamp: Date.now(),
  }));
}

// 标签页 B - 接收数据
window.addEventListener('storage', (e) => {
  if (e.key === 'shared-data' && e.newValue) {
    try {
      const sharedData = JSON.parse(e.newValue);
      console.log('收到其他标签页的数据:', sharedData);
      // 处理数据...
    } catch (error) {
      console.error('解析数据失败:', error);
    }
  }
});

// 标签页 B - 也可以直接读取（同步读取）
function getSharedData() {
  const stored = localStorage.getItem('shared-data');
  if (stored) {
    return JSON.parse(stored);
  }
  return null;
}
```

#### 完整工具函数

```javascript
/**
 * 跨标签页数据共享工具（基于 localStorage）
 */
class CrossTabStorage {
  constructor(key) {
    this.key = key;
    this.listeners = new Set();
  }

  /**
   * 设置数据（会通知其他标签页）
   */
  set(data) {
    const payload = {
      data: data,
      timestamp: Date.now(),
    };
    localStorage.setItem(this.key, JSON.stringify(payload));
  }

  /**
   * 获取数据
   */
  get() {
    const stored = localStorage.getItem(this.key);
    if (stored) {
      try {
        const payload = JSON.parse(stored);
        return payload.data;
      } catch (error) {
        console.error('解析数据失败:', error);
        return null;
      }
    }
    return null;
  }

  /**
   * 监听数据变化
   */
  onUpdate(callback) {
    const handler = (e) => {
      if (e.key === this.key && e.newValue) {
        try {
          const payload = JSON.parse(e.newValue);
          callback(payload.data, payload.timestamp);
        } catch (error) {
          console.error('解析数据失败:', error);
        }
      }
    };

    window.addEventListener('storage', handler);
    this.listeners.add({ callback, handler });

    // 返回取消监听的函数
    return () => {
      window.removeEventListener('storage', handler);
      this.listeners.delete({ callback, handler });
    };
  }

  /**
   * 清除数据
   */
  clear() {
    localStorage.removeItem(this.key);
  }

  /**
   * 清理所有监听器
   */
  destroy() {
    this.listeners.forEach(({ handler }) => {
      window.removeEventListener('storage', handler);
    });
    this.listeners.clear();
  }
}

// 使用示例
const sharedStorage = new CrossTabStorage('my-shared-data');

// 设置数据
sharedStorage.set({ count: 10, message: 'Hello' });

// 监听变化
const unsubscribe = sharedStorage.onUpdate((data) => {
  console.log('收到更新:', data);
});

// 获取数据
const currentData = sharedStorage.get();
console.log('当前数据:', currentData);

// 清理
// unsubscribe();
// sharedStorage.destroy();
```

### 方案二：使用 Broadcast Channel API（更现代，性能更好）

#### 优点
- ✅ 性能更好（直接内存通信）
- ✅ API 更简单
- ✅ 实时通信
- ✅ 不需要持久化存储

#### 缺点
- ⚠️ 需要现代浏览器支持（已提供降级方案）

#### 实现示例

```javascript
// 标签页 A - 发送数据
import { useBroadcastChannel, BROADCAST_CHANNELS } from '@/utils/broadcastChannel';

const channel = useBroadcastChannel(BROADCAST_CHANNELS.DATA_SYNC);

function sendDataToOtherTabs(data) {
  channel.postMessage({
    type: 'data-update',
    data: data,
    timestamp: Date.now(),
  });
}

// 标签页 B - 接收数据
const channel = useBroadcastChannel(BROADCAST_CHANNELS.DATA_SYNC);

channel.onMessage((message) => {
  if (message.type === 'data-update') {
    console.log('收到其他标签页的数据:', message.data);
    // 处理数据...
  }
});
```

#### 在 Vue 组件中使用

```vue
<template>
  <div>
    <button @click="updateData">更新数据</button>
    <div>共享数据: {{ sharedData }}</div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';
import { useBroadcastChannel, BROADCAST_CHANNELS } from '@/utils/broadcastChannel';

const sharedData = ref(null);
let channel = null;
let unsubscribe = null;

onMounted(() => {
  // 创建频道
  channel = useBroadcastChannel(BROADCAST_CHANNELS.DATA_SYNC);

  // 监听其他标签页的数据更新
  unsubscribe = channel.onMessage((message) => {
    if (message.type === 'data-update') {
      sharedData.value = message.data;
      console.log('收到其他标签页的更新:', message.data);
    }
  });
});

onUnmounted(() => {
  // 清理资源
  if (unsubscribe) {
    unsubscribe();
  }
  if (channel) {
    channel.close();
  }
});

// 更新数据（会通知其他标签页）
const updateData = () => {
  const newData = { count: Math.floor(Math.random() * 100) };
  channel.postMessage({
    type: 'data-update',
    data: newData,
    timestamp: Date.now(),
  });
  sharedData.value = newData;
};
</script>
```

### 方案三：使用 SharedWorker（复杂场景）

适用于需要更复杂逻辑的跨标签页通信，比如：
- 需要后台处理
- 需要维护共享状态
- 需要复杂的消息路由

#### 实现示例

```javascript
// shared-worker.js
let sharedState = {};

self.addEventListener('connect', (e) => {
  const port = e.ports[0];

  port.onmessage = (event) => {
    if (event.data.type === 'get') {
      port.postMessage({ type: 'data', data: sharedState });
    } else if (event.data.type === 'set') {
      sharedState = { ...sharedState, ...event.data.data };
      // 通知所有连接的标签页
      // ...
    }
  };

  port.start();
});

// 在主线程中使用
const worker = new SharedWorker('shared-worker.js');
worker.port.onmessage = (e) => {
  console.log('收到共享数据:', e.data);
};
```

---

## sessionStorage 的使用场景

### ✅ 适合使用 sessionStorage 的场景

1. **临时表单数据**
   - 用户填写表单时，刷新页面后数据仍然保留
   - 关闭标签页后自动清除，保护隐私

```javascript
// 保存表单数据
function saveFormData(formData) {
  sessionStorage.setItem('form-data', JSON.stringify(formData));
}

// 恢复表单数据
function restoreFormData() {
  const saved = sessionStorage.getItem('form-data');
  if (saved) {
    return JSON.parse(saved);
  }
  return null;
}
```

2. **单页应用的临时状态**
   - 不需要跨标签页共享的临时状态
   - 标签页关闭后自动清理

```javascript
// 保存临时状态
sessionStorage.setItem('current-page', 'home');
sessionStorage.setItem('scroll-position', '100');
```

3. **不需要跨标签页共享的数据**
   - 每个标签页独立的数据
   - 避免数据冲突

```javascript
// 每个标签页独立的用户操作记录
const operations = JSON.parse(sessionStorage.getItem('operations') || '[]');
operations.push({ action: 'click', timestamp: Date.now() });
sessionStorage.setItem('operations', JSON.stringify(operations));
```

4. **标签页关闭后应自动清理的数据**
   - 临时缓存
   - 会话相关的数据

```javascript
// 临时缓存
sessionStorage.setItem('cache-key', JSON.stringify(cachedData));

// 标签页关闭后自动清除，无需手动清理
```

### ❌ 不适合使用 sessionStorage 的场景

1. **需要跨标签页共享的数据**
   - 使用 `localStorage` 或 `Broadcast Channel API`

2. **需要持久化的数据**
   - 使用 `localStorage`

3. **需要监听其他标签页变化的数据**
   - 使用 `localStorage` + `storage` 事件
   - 或使用 `Broadcast Channel API`

---

## 总结

### 核心要点

1. **sessionStorage 不跨标签页共享是设计如此**
   - 这是浏览器的标准行为，不是 bug
   - 目的是提供数据隔离和安全性

2. **sessionStorage vs localStorage**
   - `sessionStorage`: 单个标签页，标签页关闭时清除
   - `localStorage`: 所有同源标签页，需要手动清除

3. **跨标签页共享数据的方案**
   - **localStorage** + `storage` 事件（兼容性好，推荐）
   - **Broadcast Channel API**（性能更好，API 更简单）
   - **SharedWorker**（复杂场景）

4. **选择建议**
   - 临时数据、表单数据 → `sessionStorage`
   - 需要跨标签页共享 → `localStorage` 或 `Broadcast Channel API`
   - 需要持久化 → `localStorage`
   - 标签页关闭后自动清理 → `sessionStorage`

### 快速参考

```javascript
// sessionStorage - 单个标签页
sessionStorage.setItem('key', 'value');
const value = sessionStorage.getItem('key');
sessionStorage.removeItem('key');
sessionStorage.clear();

// localStorage - 所有标签页共享
localStorage.setItem('key', 'value');
const value = localStorage.getItem('key');
localStorage.removeItem('key');
localStorage.clear();

// 监听 localStorage 变化（其他标签页）
window.addEventListener('storage', (e) => {
  console.log('其他标签页修改了:', e.key, e.newValue);
});

// Broadcast Channel API - 跨标签页通信
import { useBroadcastChannel } from '@/utils/broadcastChannel';
const channel = useBroadcastChannel('my-channel');
channel.postMessage({ data: 'value' });
channel.onMessage((data) => {
  console.log('收到消息:', data);
});
```

---

## 参考资料

- [MDN - sessionStorage](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/sessionStorage)
- [MDN - localStorage](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/localStorage)
- [MDN - Broadcast Channel API](https://developer.mozilla.org/en-US/docs/Web/API/Broadcast_Channel_API)
- [MDN - Storage Event](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/storage_event)

---

**创建时间**: 2024年  
**版本**: 1.0.0
