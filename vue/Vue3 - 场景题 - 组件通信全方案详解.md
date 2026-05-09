# Vue3 组件通信全方案（父传子 / 子传父 / 祖孙 / 跨层）

## 文档说明
 Vue3 `<script setup>` 语法下**最常用、面试必问、企业实战**的 4 大类组件通信方式，包含**完整可直接运行代码**，无冗余内容，复制即可使用。

---

# 一、父传子：defineProps（最基础、最常用）

## 适用场景

父组件 → 子组件 直接传值（静态值、动态值、对象、数组都可）

### 父组件 Parent\.vue

```vue
<template>
  <div class="parent">
    <h2>父组件</h2>
    <!-- 静态传值 + 动态绑定传值 -->
    <Child msg="来自父组件的消息" :count="number" />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import Child from './Child.vue'

// 定义响应式数据
const number = ref(100)
</script>
```

### 子组件 Child\.vue

```vue
<template>
  <div class="child">
    <h3>子组件</h3>
    <p>接收消息：{{ msg }}</p>
    <p>接收数字：{{ count }}</p>
  </div>
</template>

<script setup>
// 接收并校验父组件传递的数据
const props = defineProps({
  msg: {
    type: String,
    default: ''
  },
  count: {
    type: Number,
    default: 0
  }
})
</script>
```

---

# 二、子传父：defineEmits（自定义事件）

## 适用场景

子组件 → 父组件 传值 / 通知父组件执行逻辑

### 子组件 Child\.vue

```vue
<template>
  <div class="child">
    <h3>子组件</h3>
    <button @click="sendToParent">点击向父组件传值</button>
  </div>
</template>

<script setup>
// 1. 定义要触发的事件名
const emit = defineEmits(['send-data'])

const sendToParent = () => {
  // 2. 触发事件，传递参数
  emit('send-data', '子组件的数据', 666)
}
</script>
```

### 父组件 Parent\.vue

```vue
<template>
  <div class="parent">
    <h2>父组件</h2>
    <Child @send-data="handleReceive" />
  </div>
</template>

<script setup>
import Child from './Child.vue'

// 接收子组件传递的值
const handleReceive = (str, num) => {
  console.log('父组件接收：', str, num)
  alert(`收到子组件消息：${str}，数字：${num}`)
}
</script>
```

---

# 三、祖孙 / 跨多层通信：provide /inject

## 适用场景

祖父 → 孙 → 重孙……（跨多层传递，无需逐层写 props）

### 祖父组件 Grandfather\.vue

```vue
<template>
  <div>
    <h2>祖父组件</h2>
    <Father />
  </div>
</template>

<script setup>
import { provide, ref } from 'vue'
import Father from './Father.vue'

// 定义要共享的数据
const familyInfo = ref('祖父：全家共享的消息')

// 向后代提供数据
provide('family', familyInfo)
</script>
```

### 中间父组件 Father\.vue（无需处理，直接透传）

```vue
<template>
  <div>
    <h3>父组件（中间层）</h3>
    <Son />
  </div>
</template>

<script setup>
import Son from './Son.vue'
</script>
```

### 孙子组件 Son\.vue（接收数据）

```vue
<template>
  <div>
    <h4>孙子组件</h4>
    <p>接收跨层数据：{{ family }}</p>
  </div>
</template>

<script setup>
import { inject } from 'vue'

// 注入接收数据
const family = inject('family')
</script>
```

---

# 四、全局跨层 / 任意组件通信：mitt 事件总线

## 适用场景

兄弟组件、无嵌套组件、跨页面、任意组件之间通信

### 1\. 安装 mitt

```bash
npm i mitt
```

### 2\. 创建总线文件 utils/bus\.js

```js
import mitt from 'mitt'
export const bus = mitt()
```

### 3\. 发送方组件 A\.vue

```vue
<template>
  <button @click="sendMsg">向全局发送消息</button>
</template>

<script setup>
import { bus } from '@/utils/bus'

const sendMsg = () => {
  // 发送事件 + 数据
  bus.emit('global-message', '来自任意组件的全局消息')
}
</script>
```

### 4\. 接收方组件 B\.vue

```vue
<template>
  <p>接收全局消息：{{ msg }}</p>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import { bus } from '@/utils/bus'

const msg = ref('')

// 监听事件
const listenMsg = (data) => {
  msg.value = data
}

onMounted(() => {
  bus.on('global-message', listenMsg)
})

// 销毁时解绑，防止内存泄漏
onUnmounted(() => {
  bus.off('global-message', listenMsg)
})
</script>
```

---

# 五、通信方式选型总结

|通信场景|推荐方案|优点|
|---|---|---|
|父 → 子|defineProps|官方标准、简单直观|
|子 → 父|defineEmits|单向数据流、安全清晰|
|祖孙 / 多层|provide / inject|跨层方便、无需逐层传递|
|任意组件 / 全局|mitt 事件总线|灵活通用、无组件关系限制|

---
