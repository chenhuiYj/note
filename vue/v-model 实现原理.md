首先，v-model 是一个语法糖，封装了 v-bind:value 和 @input 
### v-model
给出一个代码

```
<input v-model="something">
```
实际上是以下例子的语法糖

```
<input
  v-bind:value="something"
  v-on:input="something = $event.target.value">
```
### 补充1：model 属性
如果在组件内 value 以及 @input 事件可能会被占用，vue 提供了 model 属性，定义 v-model 绑定的属性以及事件，如下例子就是把 v-model 取值用 checked ，触发事件为 @change
```
props:{
  checked:{
     type:String,
     default:''
  }
},
model:{
  prop: 'checked',
  event: 'change'
}
```

### 补充2：在子组件使用 v-model 

有些时候为了方便绑定，会在子组件内绑定 v-model 这个实现很简单

父组件
```
<child v-model="val"></child>
```
子组件
```
<template>
    <div @click="handleClick"></div>
</template>

<script>
export default {
  props:{
    value:String
  },
  methods:{
    handleClick(){
        this.$emit('input',this.value+1)
    }
  }
}
</script>
```

### 补充3： sync 修饰符

目前 sync 修饰符一样可以实现双向绑定，在原理上和v-model相差无几

```
<component :foo.sync="bar"></component>
```

实际上是以下例子的语法糖

```
<component :foo="bar" @update:foo="val => bar = val"></component>
```