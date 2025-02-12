Provide：祖先组件提供一个值，可以被后代组件注入。

Inject：注入一个由祖先组件或整个应用 (通过 app.provide()) 提供的值。

App.vue
```html
<script setup>
import { ref,provide } from 'vue'
import child from './child.vue';
import grandson from './grandson.vue';

// 定义和提供一个数据
const msg = ref('Hello World!')
provide('rootMsg',msg)


// 定义和提供一个更新数据的回调函数
function updateMsg(str){
  msg.value=str
}
provide('updateMsg',updateMsg)

</script>

<template>
  <h1>{{ msg }}</h1>
  <input v-model="msg" />
  <child>
    <grandson/>
  </child>
</template>
```
child.vue
```html
<script setup>
</script>

<template>
  <div>
    <slot />
  </div>
</template>
```
grandson.vue
```html
<script setup>
import { inject } from 'vue';
// 注入共享数据和更新函数
let msg = inject('rootMsg')
let updateMsg = inject('updateMsg')

// 请求更新数据的函数
function handleUpdateMsg(){
  updateMsg('守候')
}
</script>

<template>
  <div @click="handleUpdateMsg">
    这是子组件拿到的数据：{{msg}}
  </div>
</template>
```