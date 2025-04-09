### 1.$root 

当前组件树的根 Vue 实例。如果当前实例没有父实例，此实例将会是其自己

### 2.$parent

父实例，如果当前实例有的话，否则就是 undefined

### 3.$children

当前实例的直接子组件。需要注意 $children 并不保证顺序，也不是响应式的。如果你发现自己正在尝试使用 $children 来进行数据绑定，考虑使用一个数组配合 v-for 来生成子组件，并且使用 Array 作为真正的来源。

实例：[初识vue 2.0（10）：使用$parent、$children父子组件通信(https://www.cnblogs.com/phptree/p/10082684.html)

### 4.$attrs 
包含了父作用域中不作为 prop 被识别 (且获取) 的 attribute 绑定 (class 和 style 除外)。当一个组件没有声明任何 prop 时，这里会包含所有父作用域的绑定 (class 和 style 除外)，并且可以通过 v-bind="$attrs" 传入内部组件——在创建高级别的组件时非常有用。

```html
//gran.vue

<parent :val='守候'></parent>

// parent.vue

<child></child>

// 在 child 里面可以使用 $attrs 接收 val，不管 parent 里面有没有使用 props 接收
```

### 5.$options

用于当前 Vue 实例的初始化选项。需要在选项中包含自定义 property 时会有用处：

```javascript
new Vue({
  customOption: 'foo',
  created: function () {
    console.log(this.$options.customOption) // => 'foo'
  }
})
```