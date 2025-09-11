# Vue2 和 Vue3 的区别详解

## 1. 架构层面的区别

### 1.1 响应式系统
**Vue2**: 使用 `Object.defineProperty` 实现响应式
**Vue3**: 使用 `Proxy` 实现响应式

```js
// Vue2 响应式原理
const data = { count: 0 };
Object.defineProperty(data, 'count', {
  get() {
    console.log('获取 count');
    return this._count;
  },
  set(newVal) {
    console.log('设置 count');
    this._count = newVal;
  }
});

// Vue3 响应式原理
const data = { count: 0 };
const proxy = new Proxy(data, {
  get(target, key) {
    console.log('获取', key);
    return target[key];
  },
  set(target, key, value) {
    console.log('设置', key, value);
    target[key] = value;
    return true;
  }
});
```

### 1.2 性能优化
- **Vue3**: 更小的包体积、更快的渲染速度、更好的 Tree-shaking
- **Vue2**: 相对较大的包体积

## 2. 组合式 API vs 选项式 API

### 2.1 Vue2 选项式 API
```vue
<template>
  <div>
    <h1>{{ title }}</h1>
    <p>计数: {{ count }}</p>
    <button @click="increment">增加</button>
    <button @click="decrement">减少</button>
  </div>
</template>

<script>
export default {
  name: 'Counter',
  data() {
    return {
      title: '计数器',
      count: 0
    };
  },
  computed: {
    doubleCount() {
      return this.count * 2;
    }
  },
  methods: {
    increment() {
      this.count++;
    },
    decrement() {
      this.count--;
    }
  },
  mounted() {
    console.log('组件已挂载');
  }
};
</script>
```

### 2.2 Vue3 组合式 API
```vue
<template>
  <div>
    <h1>{{ title }}</h1>
    <p>计数: {{ count }}</p>
    <p>双倍计数: {{ doubleCount }}</p>
    <button @click="increment">增加</button>
    <button @click="decrement">减少</button>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';

// 响应式数据
const title = ref('计数器');
const count = ref(0);

// 计算属性
const doubleCount = computed(() => count.value * 2);

// 方法
const increment = () => {
  count.value++;
};

const decrement = () => {
  count.value--;
};

// 生命周期
onMounted(() => {
  console.log('组件已挂载');
});
</script>
```

## 3. 组件定义方式

### 3.1 Vue2 组件定义
```vue
<template>
  <div class="user-card">
    <h3>{{ user.name }}</h3>
    <p>{{ user.email }}</p>
    <button @click="updateUser">更新用户</button>
  </div>
</template>

<script>
export default {
  name: 'UserCard',
  props: {
    user: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      isEditing: false
    };
  },
  methods: {
    updateUser() {
      this.isEditing = true;
    }
  }
};
</script>
```

### 3.2 Vue3 组件定义
```vue
<template>
  <div class="user-card">
    <h3>{{ user.name }}</h3>
    <p>{{ user.email }}</p>
    <button @click="updateUser">更新用户</button>
  </div>
</template>

<script setup>
import { ref } from 'vue';

// 定义 props
const props = defineProps({
  user: {
    type: Object,
    required: true
  }
});

// 响应式数据
const isEditing = ref(false);

// 方法
const updateUser = () => {
  isEditing.value = true;
};
</script>
```

## 4. 生命周期钩子

### 4.1 Vue2 生命周期
```vue
<script>
export default {
  data() {
    return {
      data: null
    };
  },
  beforeCreate() {
    console.log('beforeCreate');
  },
  created() {
    console.log('created');
    this.fetchData();
  },
  beforeMount() {
    console.log('beforeMount');
  },
  mounted() {
    console.log('mounted');
  },
  beforeUpdate() {
    console.log('beforeUpdate');
  },
  updated() {
    console.log('updated');
  },
  beforeDestroy() {
    console.log('beforeDestroy');
  },
  destroyed() {
    console.log('destroyed');
  },
  methods: {
    fetchData() {
      // 获取数据
    }
  }
};
</script>
```

### 4.2 Vue3 生命周期
```vue
<script setup>
import { 
  ref, 
  onBeforeMount, 
  onMounted, 
  onBeforeUpdate, 
  onUpdated, 
  onBeforeUnmount, 
  onUnmounted 
} from 'vue';

const data = ref(null);

// 组合式 API 中的生命周期
onBeforeMount(() => {
  console.log('onBeforeMount');
});

onMounted(() => {
  console.log('onMounted');
  fetchData();
});

onBeforeUpdate(() => {
  console.log('onBeforeUpdate');
});

onUpdated(() => {
  console.log('onUpdated');
});

onBeforeUnmount(() => {
  console.log('onBeforeUnmount');
});

onUnmounted(() => {
  console.log('onUnmounted');
});

const fetchData = () => {
  // 获取数据
};
</script>
```

## 5. 事件处理

### 5.1 Vue2 事件处理
```vue
<template>
  <div>
    <button @click="handleClick">点击我</button>
    <button @click="handleClickWithParam('hello')">带参数点击</button>
    <button @click="handleClickWithEvent($event)">获取事件对象</button>
  </div>
</template>

<script>
export default {
  methods: {
    handleClick() {
      console.log('按钮被点击');
    },
    handleClickWithParam(message) {
      console.log(message);
    },
    handleClickWithEvent(event) {
      console.log(event.target);
    }
  }
};
</script>
```

### 5.2 Vue3 事件处理
```vue
<template>
  <div>
    <button @click="handleClick">点击我</button>
    <button @click="handleClickWithParam('hello')">带参数点击</button>
    <button @click="handleClickWithEvent($event)">获取事件对象</button>
  </div>
</template>

<script setup>
const handleClick = () => {
  console.log('按钮被点击');
};

const handleClickWithParam = (message) => {
  console.log(message);
};

const handleClickWithEvent = (event) => {
  console.log(event.target);
};
</script>
```

## 6. 组件通信

### 6.1 父子组件通信

#### Vue2 方式
```vue
<!-- 父组件 -->
<template>
  <div>
    <ChildComponent 
      :message="parentMessage" 
      @child-event="handleChildEvent"
    />
  </div>
</template>

<script>
import ChildComponent from './ChildComponent.vue';

export default {
  components: {
    ChildComponent
  },
  data() {
    return {
      parentMessage: '来自父组件的消息'
    };
  },
  methods: {
    handleChildEvent(data) {
      console.log('收到子组件事件:', data);
    }
  }
};
</script>
```

```vue
<!-- 子组件 -->
<template>
  <div>
    <p>{{ message }}</p>
    <button @click="sendToParent">发送给父组件</button>
  </div>
</template>

<script>
export default {
  props: ['message'],
  methods: {
    sendToParent() {
      this.$emit('child-event', '子组件数据');
    }
  }
};
</script>
```

#### Vue3 方式
```vue
<!-- 父组件 -->
<template>
  <div>
    <ChildComponent 
      :message="parentMessage" 
      @child-event="handleChildEvent"
    />
  </div>
</template>

<script setup>
import { ref } from 'vue';
import ChildComponent from './ChildComponent.vue';

const parentMessage = ref('来自父组件的消息');

const handleChildEvent = (data) => {
  console.log('收到子组件事件:', data);
};
</script>
```

```vue
<!-- 子组件 -->
<template>
  <div>
    <p>{{ message }}</p>
    <button @click="sendToParent">发送给父组件</button>
  </div>
</template>

<script setup>
import { defineProps, defineEmits } from 'vue';

const props = defineProps(['message']);
const emit = defineEmits(['child-event']);

const sendToParent = () => {
  emit('child-event', '子组件数据');
};
</script>
```

### 6.2 跨组件通信

#### Vue2 使用 EventBus
```js
// eventBus.js
import Vue from 'vue';
export default new Vue();

// 组件A
import eventBus from './eventBus';
export default {
  methods: {
    sendMessage() {
      eventBus.$emit('message', 'Hello from A');
    }
  }
};

// 组件B
import eventBus from './eventBus';
export default {
  mounted() {
    eventBus.$on('message', (data) => {
      console.log(data);
    });
  }
};
```

#### Vue3 使用 provide/inject
```vue
<!-- 祖先组件 -->
<template>
  <div>
    <ParentComponent />
  </div>
</template>

<script setup>
import { provide, ref } from 'vue';
import ParentComponent from './ParentComponent.vue';

const sharedData = ref('共享数据');
provide('sharedData', sharedData);
</script>
```

```vue
<!-- 后代组件 -->
<template>
  <div>
    <p>{{ sharedData }}</p>
  </div>
</template>

<script setup>
import { inject } from 'vue';

const sharedData = inject('sharedData');
</script>
```

## 7. 指令变化

### 7.1 v-model 变化
```vue
<!-- Vue2 -->
<template>
  <div>
    <input v-model="message" />
    <p>{{ message }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      message: ''
    };
  }
};
</script>
```

```vue
<!-- Vue3 -->
<template>
  <div>
    <input v-model="message" />
    <p>{{ message }}</p>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const message = ref('');
</script>
```

### 7.2 自定义指令
```vue
<!-- Vue2 -->
<script>
export default {
  directives: {
    focus: {
      inserted(el) {
        el.focus();
      }
    }
  }
};
</script>
```

```vue
<!-- Vue3 -->
<script setup>
import { directive } from 'vue';

const vFocus = directive({
  mounted(el) {
    el.focus();
  }
});
</script>
```

## 8. 路由变化

### 8.1 Vue2 路由
```js
// router/index.js
import Vue from 'vue';
import VueRouter from 'vue-router';
import Home from '../views/Home.vue';

Vue.use(VueRouter);

const routes = [
  {
    path: '/',
    name: 'Home',
    component: Home
  }
];

const router = new VueRouter({
  routes
});

export default router;
```

```vue
<!-- 组件中使用 -->
<template>
  <div>
    <router-link to="/">首页</router-link>
    <router-view />
  </div>
</template>

<script>
export default {
  methods: {
    goToHome() {
      this.$router.push('/');
    }
  }
};
</script>
```

### 8.2 Vue3 路由
```js
// router/index.js
import { createRouter, createWebHistory } from 'vue-router';
import Home from '../views/Home.vue';

const routes = [
  {
    path: '/',
    name: 'Home',
    component: Home
  }
];

const router = createRouter({
  history: createWebHistory(),
  routes
});

export default router;
```

```vue
<!-- 组件中使用 -->
<template>
  <div>
    <router-link to="/">首页</router-link>
    <router-view />
  </div>
</template>

<script setup>
import { useRouter } from 'vue-router';

const router = useRouter();

const goToHome = () => {
  router.push('/');
};
</script>
```

## 9. 状态管理

### 9.1 Vue2 Vuex
```js
// store/index.js
import Vue from 'vue';
import Vuex from 'vuex';

Vue.use(Vuex);

export default new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment(state) {
      state.count++;
    }
  },
  actions: {
    increment({ commit }) {
      commit('increment');
    }
  }
});
```

```vue
<!-- 组件中使用 -->
<template>
  <div>
    <p>{{ $store.state.count }}</p>
    <button @click="increment">增加</button>
  </div>
</template>

<script>
import { mapState, mapActions } from 'vuex';

export default {
  computed: {
    ...mapState(['count'])
  },
  methods: {
    ...mapActions(['increment'])
  }
};
</script>
```

### 9.2 Vue3 Pinia
```js
// stores/counter.js
import { defineStore } from 'pinia';

export const useCounterStore = defineStore('counter', {
  state: () => ({
    count: 0
  }),
  actions: {
    increment() {
      this.count++;
    }
  }
});
```

```vue
<!-- 组件中使用 -->
<template>
  <div>
    <p>{{ counter.count }}</p>
    <button @click="counter.increment">增加</button>
  </div>
</template>

<script setup>
import { useCounterStore } from '@/stores/counter';

const counter = useCounterStore();
</script>
```

## 10. 主要区别总结

| 特性 | Vue2 | Vue3 |
|------|------|------|
| **API 风格** | 选项式 API | 组合式 API + 选项式 API |
| **响应式系统** | Object.defineProperty | Proxy |
| **性能** | 相对较慢 | 更快，包体积更小 |
| **TypeScript** | 支持有限 | 原生支持 |
| **多根节点** | 不支持 | 支持 Fragment |
| **Teleport** | 不支持 | 支持 |
| **Suspense** | 不支持 | 支持 |
| **Composition API** | 不支持 | 支持 |
| **setup 语法糖** | 不支持 | 支持 `<script setup>` |
| **生命周期** | beforeDestroy/destroyed | onBeforeUnmount/onUnmounted |

## 11. 迁移建议

1. **新项目**: 直接使用 Vue3
2. **现有项目**: 可以渐进式迁移，Vue3 兼容 Vue2 语法
3. **学习路径**: 先掌握 Vue2 基础，再学习 Vue3 新特性
4. **组合式 API**: 适合复杂组件逻辑，选项式 API 适合简单组件

Vue3 在保持 Vue2 易用性的同时，提供了更好的性能、更强的类型支持和更灵活的代码组织方式。
