# Vue 3 Pinia 状态管理

## 简介

Pinia 是 Vue 官方推荐的状态管理库（Vuex 的继任者），支持 Vue 3 组合式 API，类型推导优秀，心智负担更低。

## 安装与注册

### 安装依赖

```bash
# npm
npm install pinia

# yarn
yarn add pinia
```

### 在应用入口注册

```js
// main.ts
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'

const app = createApp(App)
app.use(createPinia())
app.mount('#app')
```

## 创建 Store

### 1. 组合式写法（推荐）

类型推导更好，更符合 Vue 3 组合式 API 风格：

```js
// stores/useCounter.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useCounter = defineStore('counter', () => {
  const count = ref(0)
  const double = computed(() => count.value * 2)

  function increment() {
    count.value++
  }

  async function incrementLater() {
    await new Promise(r => setTimeout(r, 500))
    increment()
  }

  return { count, double, increment, incrementLater }
})
```

### 2. 选项式写法

更接近 Vuex 风格：

```js
// stores/useUser.ts
import { defineStore } from 'pinia'

export const useUser = defineStore('user', {
  state: () => ({ 
    name: 'Tom', 
    age: 20 
  }),
  
  getters: {
    isAdult: (state) => state.age >= 18,
  },
  
  actions: {
    setName(name) {
      this.name = name
    },
  },
})
```

## 在组件中使用

```vue
<!-- AnyComponent.vue -->
<script setup lang="ts">
import { storeToRefs } from 'pinia'
import { useCounter } from '@/stores/useCounter'

const counter = useCounter()
const { count, double } = storeToRefs(counter) // 解构保持响应式
const { increment, incrementLater } = counter
</script>

<template>
  <div>
    <p>{{ count }} / {{ double }}</p>
    <button @click="increment">+1</button>
    <button @click="incrementLater">延迟+1</button>
  </div>
</template>
```

## 持久化（本地存储）

### 安装插件

```bash
npm install pinia-plugin-persistedstate
```

### 配置插件

```js
// main.ts
import { createPinia } from 'pinia'
import piniaPluginPersistedstate from 'pinia-plugin-persistedstate'

const pinia = createPinia()
pinia.use(piniaPluginPersistedstate)

app.use(pinia)
```

### 在 Store 中使用

```js
// stores/useUser.ts
export const useUser = defineStore('user', {
  state: () => ({ 
    token: '',
    userInfo: null 
  }),
  
  persist: {
    // 持久化所有状态
    enabled: true,
    // 或只持久化特定字段
    // paths: ['token']
  },
})
```

## 高级功能

### 重置、订阅与热更新

```js
const store = useCounter()

// 重置为初始 state
store.$reset()

// 订阅 state 变化
const unsubscribe = store.$subscribe((mutation, state) => {
  console.log('状态变化:', mutation.type)
  // mutation.type: 'direct' | 'patch object' | 'patch function'
})

// HMR（开发时热更新）
if (import.meta.hot) {
  import.meta.hot.accept(acceptHMRUpdate(useCounter, import.meta.hot))
}
```

### 批量更新状态

```js
// 使用 $patch 批量更新
store.$patch({
  count: 100,
  name: 'New Name'
})

// 或使用函数形式
store.$patch((state) => {
  state.count = 100
  state.name = 'New Name'
})
```

## 与 Vuex 的主要差异

- **更轻量**：无需复杂的 mutation，直接 actions 改 state
- **类型友好**：TS 推导一流，组合式写法贴近 Vue 3
- **API 简洁**：学习成本低，生态完善（持久化、中间件等）
- **开发体验**：更好的 DevTools 支持

## 常见问题与最佳实践

### 1. 响应式解构

使用 `storeToRefs` 解构 state/getters，保持响应式：

```js
import { storeToRefs } from 'pinia'

const store = useCounter()
const { count, double } = storeToRefs(store) // ✅ 保持响应式
const { increment } = store // ✅ actions 可以直接解构
```

### 2. 在 setup() 外使用 store

```js
// 错误：在 setup 外直接使用
const store = useCounter()

// 正确：传入 pinia 实例
import { createPinia } from 'pinia'
const pinia = createPinia()
const store = useCounter(pinia)
```

### 3. SSR/多实例场景

每次请求创建新的 pinia 实例，避免跨请求泄漏：

```js
// server.js
app.use((req, res, next) => {
  const pinia = createPinia()
  req.pinia = pinia
  next()
})
```

### 4. 性能优化

```js
// 避免在模板中直接调用 store 方法
// 错误
<template>
  <div>{{ useCounter().count }}</div>
</template>

// 正确
<script setup>
const counter = useCounter()
</script>
<template>
  <div>{{ counter.count }}</div>
</template>
```
