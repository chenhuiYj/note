### 简介

Pinia 是 Vue 官方推荐的状态管理库（Vuex 的继任者），支持 Vue 3 组合式 API，类型推导优秀，心智负担更低。


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

创建 Store（两种写法）
组合式（推荐，类型推导更好）

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

选项式（更接近 Vuex 风格）
```js
// stores/useUser.ts
import { defineStore } from 'pinia'

export const useUser = defineStore('user', {
  state: () => ({ name: 'Tom', age: 20 }),
  getters: {
    isAdult: state => state.age >= 18,
  },
  actions: {
    setName(name: string) {
      this.name = name
    },
  },
})
```
在组件中使用
```html
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

持久化（本地存储）
可用社区插件 pinia-plugin-persistedstate：

```js
import persist from 'pinia-plugin-persistedstate'
const pinia = createPinia()
pinia.use(persist)
app.use(pinia)
```

```js
// stores/useUser.ts
export const useUser = defineStore('user', {
  state: () => ({ token: '' }),
  persist: true, // 或 { paths: ['token'] }
})
```

重置、订阅与热更新

```js

const store = useCounter()

// 重置为初始 state（仅选项式写法自带 reset，可自实现）
store.$reset?.()

// 订阅 state 变化
const unsubscribe = store.$subscribe((mutation, state) => {
  // mutation.type: 'direct' | 'patch object' | 'patch function'
})

// HMR（开发时）
if (import.meta.hot) {
  import.meta.hot.accept(acceptHMRUpdate(useCounter, import.meta.hot))
}
```


### 与 Vuex 的主要差异（简要）

- 更轻：无需复杂的 mutation，直接 actions 改 state
- 类型更友好：TS 推导一流，组合式写法贴近 Vue 3
- API 简洁：学习成本低，生态完善（持久化、中间件等）


### 常见问题速记

- 使用 `storeToRefs` 解构 state/getters，保持响应式
- 在 `setup()` 外使用 store：需传入 pinia 实例或在组件树中使用后再引入
- SSR/多实例：每次请求创建新的 pinia 实例，避免跨请求泄漏