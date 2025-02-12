## 1.基本使用

setup 是 Vue 3 的 组合式 API 系列里推出的一个全新组件选项，在创建组件之前执行，一旦 props 被解析。并作为组合式 API 的入口点。

整个组件相关的业务代码 (data，computed，function，生命周期钩子等等) ，都可以放在 setup 里执行。

setup 执行早，比 vue2的 beforeCreated 还要早，所以在 setup 中 this 会找不到组件实例。因此在 setup 中应该避免使用 this。如果一定要用 this，可以 getCurrentInstance()访问。

> 在 setup 中，根据打包方式，this 指向 undefined，或者 window 。

在模板中访问从 setup 返回的 ref 时，会自动浅层解包，因此无须在模板中为写 .value。当通过 this 访问时也会同样如此解包。

setup 内可以灵活的编写生命周期钩子，可以多次编写，最后会按注册顺序依次执行：

```javascript
setup() {
  onMounted(() => {})
  onMounted(() => {})
}
```

## 2.参数

setup 接收的两个参数(props, context)。

props：值为对象，包含：组件外部传递过来，且组件内部声明接收了的属性。如下面代码 child.vue 里面的 prop1 。

context：上下文对象

    attrs: 值为对象，包含：组件外部传递过来，但没有在 props 配置中声明的属性（如下面代码 child.vue 里面的 prop2 。）, 相当于 this.$attrs。
    slots: 收到的插槽内容, 相当于 this.$slots。
    emit: 分发自定义事件的函数, 相当于 this.$emit。
    expose：暴露公共属性（函数）

App.vue
```html
<script>
import { ref,onMounted } from 'vue'
import child from './child.vue'

export default{
  components:{child},
  setup(){
    const message = ref('Hello World!')
		const messageExpose = ref('Hello World!')
    return {
      message
    }
  }
}
</script>

<template>
  <child prop1="prop1 text" prop2="prop2 text">text of App.vue</child>
  <h1>{{ message }}</h1>
</template>
```

child.vue

```html
<script>
import { ref } from 'vue'
export default {
  props:['prop1'],
  setup(props,context){
    let msg = ref('this is msg')
    let prop1=props.prop1
    let prop2=context.attrs.prop2
    return {
      msg,
      prop1,
	  prop2
    }
  }
}
</script>

<template>
  <div>
    <p>
      {{prop1}}
    </p>
    <p>
      {{prop2}}
    </p>
    <p>
      {{msg}}
    </p>
    <slot/>
  </div>
</template>

```

渲染结果

![a](https://i.ibb.co/W2vFVn8/2025-01-17-164350.png)

### 暴露公共属性 expose

可以看到上面的截图，'childMsgExopse' 没有渲染出来，这个要分情况讨论。

下面把 App.vue 代码改一下
```html
<script>
import { ref,onMounted } from 'vue'
import child from './child.vue'

export default{
  components:{child},
  setup(){
    const childRef = ref(null)
    // 定义一个变量来存储从子组件获取的计数
    const childMsgExopse = ref('');
    // 在组件挂载后，从子组件获取计数并更新本地变量
    onMounted(() => {
      if (childRef.value) {
        childMsgExopse.value = childRef.value.msgExopse;
      }
    });
    return {
      childMsgExopse,
      childRef
    }
  }
}
</script>

<template>
  <child ref="childRef"/>
  <h2>{{childMsgExopse}}</h2>
</template>
```
然后再分情况讨论

> 下面的改动只改 child.vue ，不再修改 App.vue


1.选项式 API，如果不设置 expose 属性，那么默认所有属性，方法都可以被父元素访问。

下面把 child.vue 改成选项式 API 写法。

```html
<script>
export default {
  data() {
    return {
		msgExopse: 'this is msgExopse',
    }
  }
}
</script>

<template>
  <div></div>
</template>
```

渲染出来后，可以看到 'childMsgExopse' 被渲染出来了。

![](https://i.ibb.co/rHJc6QJ/2025-01-17-180007.png)

2.选项式 API，如果设置 expose 属性，那么只有 expose 的属性，方法可以被父元素访问。其他不可以被访问

```html
<script>
export default {
  //设置 expose 属性，只有 expose 属性里面的 data 或者方法才能被父元素访问，下面代码只有 msgExopse 能被父元素访问，其他父元素不可以访问
  expose:['msgExopse'],
  data() {
    return {
      msg: 'this is msg',
	  msgExopse: 'this is msgExopse',
    }
  }
}
</script>

<template>
  <div></div>
</template>
```

3.setup() 方式，父元素只能访问到 returu 或者 defineExpose 的属性。

```html
<script>
import { ref } from 'vue'
export default {
  setup(props,context){
		let msg = ref('this is msg')
    let msgExopse = ref('this is msgExopse')
    //使用 context.expose 把 msgExopse 设置为公共属性，msgExopse 可以被父组件访问
    //context.expose({msgExopse})
    return {
      //return 里面的变量（ msgExopse） 可以被父组件访问
      //msgExopse
    }
  }
}
</script>

<template>
  <div></div>
</template>
```

4.`<script setup>` 方式，所有属性和方法都是私有的，需要 defineExpose 方式把属性设置为公共属性后，父组件可以进行访问。

```html
<script setup>
    import { ref } from 'vue'
    let msgExopse = ref('this is msgExopse')

    //使用 defineExpose 把 msgExopse 设置为公共属性，msgExopse 可以被父元素访问
    defineExpose({
        msgExopse
    })
</script>

<template>
  <div></div>
</template>
```
