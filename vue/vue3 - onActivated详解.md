# onActivated 事件详解：什么时候会触发？

## 📚 目录

1. [核心概念](#核心概念)
2. [触发时机](#触发时机)
3. [完整的生命周期流程](#完整的生命周期流程)
4. [对比表格](#对比表格)
5. [实际示例](#实际示例)
6. [使用场景建议](#使用场景建议)
7. [项目中的最佳实践](#项目中的最佳实践)
8. [注意事项](#注意事项)
9. [总结](#总结)

---

## 核心概念

`onActivated` 是 **Vue 3 的生命周期钩子**，仅在组件被 `<keep-alive>` 包裹时才会触发。

### 关键点

- ✅ 必须配合 `keep-alive` 使用
- ✅ 每次组件激活时都会触发
- ✅ 包括首次激活和从缓存恢复

---

## 触发时机

`onActivated` 会在以下情况触发：

### 1. 组件首次被激活
首次进入被 `keep-alive` 包裹的组件时触发。

### 2. 从其他路由切换回来
组件从缓存中恢复并显示时触发。

### 3. 从隐藏状态变为显示状态
在 `keep-alive` 内部切换时触发。

---

## 完整的生命周期流程

### 场景 1: 首次进入页面

```
1. 组件创建
2. onBeforeMount
3. onMounted
4. onActivated  ← 首次激活时触发
```

### 场景 2: 切换到其他页面再回来

```
1. 离开当前页面
   → onDeactivated  ← 组件被缓存时触发
   
2. 切换到其他页面（组件被缓存，不会销毁）

3. 回到当前页面
   → onActivated  ← 从缓存恢复时触发（不会触发 onMounted）
```

### 场景 3: 不使用 keep-alive

```
1. 离开页面
   → onUnmounted  ← 组件被销毁
   
2. 再次进入
   → onBeforeMount
   → onMounted  ← 重新创建组件（不会触发 onActivated）
```

---

## 对比表格

| 生命周期钩子 | 不使用 keep-alive | 使用 keep-alive |
|------------|-----------------|----------------|
| **onBeforeMount** | ✅ 每次进入都触发 | ✅ 仅首次进入触发 |
| **onMounted** | ✅ 每次进入都触发 | ✅ 仅首次进入触发 |
| **onActivated** | ❌ 不触发 | ✅ 每次激活都触发 |
| **onDeactivated** | ❌ 不触发 | ✅ 每次失活都触发 |
| **onUnmounted** | ✅ 离开时触发 | ❌ 不会触发（组件被缓存） |

---

## 实际示例

### 基础示例

```vue
<template>
  <div>
    <h1>我的页面</h1>
    <p>计数: {{ count }}</p>
    <button @click="count++">增加</button>
  </div>
</template>

<script setup>
import { ref, onMounted, onActivated, onDeactivated, onUnmounted } from 'vue';

const count = ref(0);

// 首次进入时触发（仅一次）
onMounted(() => {
  console.log('组件挂载 - 仅首次进入时触发');
  // 适合：初始化数据、创建定时器等一次性操作
  fetchInitialData();
});

// 每次激活时触发（包括首次）
onActivated(() => {
  console.log('组件激活 - 每次进入页面都触发');
  // 适合：刷新数据、恢复事件监听、重置状态等
  refreshData();
  restoreScrollPosition();
});

// 每次失活时触发
onDeactivated(() => {
  console.log('组件失活 - 离开页面时触发');
  // 适合：清理事件监听、暂停定时器等
  saveScrollPosition();
  pauseTimers();
});

// 不使用 keep-alive 时才会触发
onUnmounted(() => {
  console.log('组件卸载 - 不使用 keep-alive 时才会触发');
  // 适合：最终清理工作
  cleanup();
});
</script>
```

### 完整示例：带数据刷新和事件监听

```vue
<template>
  <div class="user-list">
    <div v-for="user in users" :key="user.id">
      {{ user.name }}
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onActivated, onDeactivated, onUnmounted } from 'vue';
import { getUserList } from '@/api/user';

const users = ref([]);
let scrollPosition = 0;
let refreshTimer = null;

// 获取用户列表
const fetchUsers = async () => {
  try {
    const data = await getUserList();
    users.value = data;
  } catch (error) {
    console.error('获取用户列表失败:', error);
  }
};

// 保存滚动位置
const saveScrollPosition = () => {
  const container = document.querySelector('.user-list');
  if (container) {
    scrollPosition = container.scrollTop;
  }
};

// 恢复滚动位置
const restoreScrollPosition = () => {
  const container = document.querySelector('.user-list');
  if (container) {
    container.scrollTop = scrollPosition;
  }
};

// 开始自动刷新
const startAutoRefresh = () => {
  refreshTimer = setInterval(() => {
    fetchUsers();
  }, 30000); // 每30秒刷新一次
};

// 停止自动刷新
const stopAutoRefresh = () => {
  if (refreshTimer) {
    clearInterval(refreshTimer);
    refreshTimer = null;
  }
};

// 首次挂载时执行
onMounted(() => {
  console.log('组件首次挂载');
  fetchUsers();
  startAutoRefresh();
});

// 每次激活时执行
onActivated(() => {
  console.log('组件激活 - 从缓存恢复');
  // 刷新数据
  fetchUsers();
  // 恢复滚动位置
  restoreScrollPosition();
  // 恢复自动刷新
  startAutoRefresh();
});

// 每次失活时执行
onDeactivated(() => {
  console.log('组件失活 - 被缓存');
  // 保存滚动位置
  saveScrollPosition();
  // 停止自动刷新
  stopAutoRefresh();
});

// 组件卸载时执行（不使用 keep-alive 时）
onUnmounted(() => {
  console.log('组件卸载');
  stopAutoRefresh();
});
</script>
```

---

## 使用场景建议

### ✅ 使用 `onActivated` 的场景

1. **每次进入页面都需要刷新数据**
   ```javascript
   onActivated(() => {
     fetchLatestData();
   });
   ```

2. **恢复事件监听器**
   ```javascript
   onActivated(() => {
     window.addEventListener('resize', handleResize);
   });
   
   onDeactivated(() => {
     window.removeEventListener('resize', handleResize);
   });
   ```

3. **重置组件状态**
   ```javascript
   onActivated(() => {
     formData.value = {};
     errors.value = {};
   });
   ```

4. **发送埋点统计**
   ```javascript
   onActivated(() => {
     ta.track('page_view', { page: 'home' });
   });
   ```

5. **恢复滚动位置**
   ```javascript
   onActivated(() => {
     const savedPosition = sessionStorage.getItem('scroll-position');
     if (savedPosition) {
       window.scrollTo(0, parseInt(savedPosition));
     }
   });
   ```

### ✅ 使用 `onMounted` 的场景

1. **只需要执行一次的操作**
   ```javascript
   onMounted(() => {
     // 初始化第三方库
     initThirdPartyLibrary();
   });
   ```

2. **初始化全局事件监听（需要在 onUnmounted 中清理）**
   ```javascript
   onMounted(() => {
     document.addEventListener('keydown', handleKeyDown);
   });
   
   onUnmounted(() => {
     document.removeEventListener('keydown', handleKeyDown);
   });
   ```

---

## 项目中的最佳实践

### 示例：Packing.vue 中的使用

```javascript
import { onActivated, onDeactivated, onUnmounted } from 'vue';

onActivated(() => {
  // 1. 埋点统计
  ta.track("enter_bag");
  
  // 2. 恢复事件监听
  if (typeof document !== "undefined") {
    document.addEventListener("click", handleClickOutside);
  }
  
  // 3. 重置状态
  packStatus.value = false;
  emit("onClear");
  
  // 4. 显示提示
  if (shouldShowFloatTips()) {
    showFloatTips.value = true;
    startAutoHideTimer();
  }
});

onDeactivated(() => {
  // 清理事件监听（避免内存泄漏）
  if (typeof document !== "undefined") {
    document.removeEventListener("click", handleClickOutside);
  }
  
  // 清理定时器
  if (autoHideTimer) {
    clearTimeout(autoHideTimer);
    autoHideTimer = null;
  }
});

// 组件卸载时也清理（双重保险）
onUnmounted(() => {
  if (autoHideTimer) {
    clearTimeout(autoHideTimer);
    autoHideTimer = null;
  }
  if (typeof document !== "undefined") {
    document.removeEventListener("click", handleClickOutside);
  }
});
```

### 最佳实践模式

```javascript
// 模式 1: 数据刷新
onActivated(() => {
  // 每次进入都刷新数据
  refreshData();
});

// 模式 2: 事件监听管理
let eventHandler = null;

onActivated(() => {
  eventHandler = () => {
    // 处理事件
  };
  window.addEventListener('scroll', eventHandler);
});

onDeactivated(() => {
  if (eventHandler) {
    window.removeEventListener('scroll', eventHandler);
    eventHandler = null;
  }
});

// 模式 3: 定时器管理
let timer = null;

onActivated(() => {
  timer = setInterval(() => {
    // 定期执行
  }, 1000);
});

onDeactivated(() => {
  if (timer) {
    clearInterval(timer);
    timer = null;
  }
});

// 模式 4: 状态重置
onActivated(() => {
  // 重置表单
  form.value = {};
  // 重置错误
  errors.value = {};
  // 重置分页
  currentPage.value = 1;
});
```

---

## 注意事项

### 1. 必须配合 keep-alive 使用

```vue
<!-- ✅ 正确：使用 keep-alive -->
<template>
  <keep-alive>
    <component :is="currentComponent" />
  </keep-alive>
</template>

<!-- ❌ 错误：不使用 keep-alive，onActivated 不会触发 -->
<template>
  <component :is="currentComponent" />
</template>
```

### 2. 执行顺序

首次进入时的执行顺序：
```
onBeforeMount → onMounted → onActivated
```

从缓存恢复时的执行顺序：
```
onActivated（不触发 onMounted）
```

### 3. 内存管理

**重要**：在 `onDeactivated` 中清理资源，避免内存泄漏：

```javascript
onActivated(() => {
  // 创建资源
  timer = setInterval(() => {}, 1000);
  window.addEventListener('resize', handleResize);
});

onDeactivated(() => {
  // 必须清理资源
  if (timer) {
    clearInterval(timer);
    timer = null;
  }
  window.removeEventListener('resize', handleResize);
});
```

### 4. 与 onMounted 的区别

| 特性 | onMounted | onActivated |
|------|-----------|-------------|
| **触发次数** | 仅首次进入 | 每次激活都触发 |
| **是否需要 keep-alive** | 不需要 | 必须需要 |
| **适用场景** | 一次性初始化 | 每次激活时的操作 |

### 5. 常见错误

#### 错误 1: 在 onMounted 中刷新数据

```javascript
// ❌ 错误：从缓存恢复时不会触发
onMounted(() => {
  fetchData(); // 只有首次进入会执行
});

// ✅ 正确：每次激活都会触发
onActivated(() => {
  fetchData(); // 每次进入都会执行
});
```

#### 错误 2: 忘记清理资源

```javascript
// ❌ 错误：没有清理，导致内存泄漏
onActivated(() => {
  timer = setInterval(() => {}, 1000);
});

// ✅ 正确：在 onDeactivated 中清理
onActivated(() => {
  timer = setInterval(() => {}, 1000);
});

onDeactivated(() => {
  if (timer) {
    clearInterval(timer);
    timer = null;
  }
});
```

---

## 总结

### 核心要点

1. **onActivated 必须配合 keep-alive 使用**
   - 不使用 `keep-alive` 时，`onActivated` 不会触发

2. **触发时机**
   - 首次激活：`onMounted` → `onActivated`
   - 再次激活：只触发 `onActivated`（不触发 `onMounted`）

3. **生命周期对比**
   - 使用 `keep-alive`：`onActivated` / `onDeactivated`
   - 不使用 `keep-alive`：`onMounted` / `onUnmounted`

4. **使用建议**
   - 每次激活都需要执行的操作 → `onActivated`
   - 只需要执行一次的操作 → `onMounted`
   - 记得在 `onDeactivated` 中清理资源

### 快速参考

```javascript
// 导入
import { onMounted, onActivated, onDeactivated, onUnmounted } from 'vue';

// 首次挂载（仅一次）
onMounted(() => {
  // 初始化操作
});

// 每次激活（包括首次）
onActivated(() => {
  // 刷新数据、恢复状态等
});

// 每次失活
onDeactivated(() => {
  // 清理资源
});

// 组件卸载（不使用 keep-alive 时）
onUnmounted(() => {
  // 最终清理
});
```

### 流程图

```
首次进入页面：
┌─────────────┐
│ 组件创建    │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ onBeforeMount│
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ onMounted   │ ← 仅首次触发
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ onActivated │ ← 首次激活
└─────────────┘

切换到其他页面再回来：
┌─────────────┐
│ onDeactivated│ ← 离开时触发
└─────────────┘
       │
       ▼
   组件被缓存
       │
       ▼
┌─────────────┐
│ onActivated │ ← 恢复时触发（不触发 onMounted）
└─────────────┘
```

---

## 参考资料

- [Vue 3 官方文档 - keep-alive](https://cn.vuejs.org/guide/built-ins/keep-alive.html)
- [Vue 3 官方文档 - 生命周期钩子](https://cn.vuejs.org/api/composition-api-lifecycle.html)
- [Vue 3 API 参考 - onActivated](https://cn.vuejs.org/api/composition-api-lifecycle.html#onactivated)

---

**创建时间**: 2024年  
**版本**: 1.0.0
