watch 和 watchEffect 是两个用于响应式数据监听的 API，都能响应式的执行有副作用（除了返回预期结果外还会简介影响其它数据）的回调。

watch 和 watchEffect 两者的区别，主要有以下几个区别

### 依赖收集
watch 需要显式指定要监听的数据源（可以是响应式引用、计算属性、getter 函数等），也只追踪明确监听的数据源，不会追踪任何在回调中访问到的东西。仅在数据源确实改变时才会触发回调。watch 会避免在发生副作用时追踪依赖，因此能更加精确地控制回调函数的触发时机。

```html
<template>
  <div>
    <p>Count: {{ count }}</p>
    <p>{{ countStr }}</p>
    <button @click="increment">Increment</button>
  </div>
</template>
 
<script setup>
    import { ref, watch } from 'vue';
    const count = ref(0);
    const countStr = ref('');
  
    // 使用 watch 监听 count 的变化，count 有变化就会进入
    watch(count, () => {
      countStr.value=`watchEffect: count is now ${count.value}`
    });

    const increment = () => {
      count.value++;
    };

</script>
```

watchEffect 不需要指定依赖项，在同步执行过程中自动追踪所有能访问到的响应式属性。相当于运行一个函数，并“记住”这个函数访问了哪些响应式属性。当这些属性中的任何一个变化时，watchEffect 都会重新运行该函数这更方便，而且代码往往更简洁，但有时其响应性依赖关系会不那么明确。

```html
<template>
  <div>
    <p>Count: {{ count }}</p>
    <p>{{ countStr }}</p>
    <button @click="increment">Increment</button>
  </div>
</template>
 
<script setup>
    import { ref, watchEffect } from 'vue';
    const count = ref(0);
    const countStr = ref('');
  
    // 使用 watchEffect 监听 count 的变化，只要 count 变化就会进入。虽然在 watchEffect 里面也有写到 countStr，但实际上监听的依赖的只有 count
    watchEffect(() => {
      countStr.value=`watchEffect: count is now ${count.value}`
    });
    const increment = () => {
      count.value++;
    };

</script>
```

### 监听原理
watch 只监听传入的数据源，默认是浅监听，如果需要深度监听需要传入 deep:true 。

watchEffect 会监听函数里面用到的响应式属性

```html
<template>
  <div>
    <p>Count: {{ count }}</p>
    <p>{{ countStr }}</p>
    <button @click="increment">Increment</button>
  </div>
</template>
 
<script setup>
    import { ref, watchEffect } from 'vue';
    const count = ref({
      a:{
        a1:1,
        a2:11
      },
      b:2
    });
    const countStr = ref('');
  
    // 使用 watchEffect 监听 count 的变化
    watchEffect(() => {
      debugger
      countStr.value=`watchEffect: count is now ${count.value.a}`
    });
    const increment = () => {
      //不会触发监听，因为 watchEffect 回调函数收集到的依赖是 count.value.a。如果改了 count.value.a.a1，count.value.b 等情况都不会重新执行 watchEffect
      count.value.a.a1++

      //会触发监听，
      //count.value.a={
        //a1:13,
        //a2:14
      //};
    };

</script>
```
### 自动运行

watch 的回调函数不会在监听开始时立即执行，可以通过传递 immediate: true 选项，使得 watch 马上执行。

watchEffect 会立即执行一次传入的回调函数，并在依赖项发生变化时重新运行。

### 适用场景

watch 适用于需要细粒度控制的场景，比如需要在监听开始时立即执行操作，或者在监听停止之前做一些清理工作。比如需要监听表单字段的变化并执行实时验证逻辑；筛选条件变化后更新列表的逻辑；需要监听表单数据，根据表单的属性进行页面的渲染。例如实名验证时候，监听实名验证方式的字段，如果是身份证验证，就出现填写身份证的输入框。如果选择电话号验证，就出现填写手机号码和验证码的输入框。

watchEffect 适用于简单的副作用监听，不需要手动管理依赖项。比如修改 string 或者 array 数据后，马上显示对应长度。对单个字段进行校验等。

> 说到 watchEffect，不少人会把 watchEffect 和 computed 混淆，其实两者的区别挺大的。
>
> watchEffect 适用于只执行副作用,不需要返回值的响应式数据变化的场景。比如更新 DOM、调用外部 API 等。它会在依赖的响应式数据变化时自动重新执行。
>
> computed 适用于基于其他响应式数据计算新的值。它通常用于模板中，返回一个响应式的引用，该引用持有计算后的值，可以直接访问计算属性的值。例如统计购物车里面的商品总价。

