## Options API

试想下，如果页面(App.vue)有两个功能，一个是 todoList,一个是计数器。如果按照选项式 API 方式显示，代码大致是下面这样
```html
<template>
  <div id="app">
    <!--todoList-->
    <input type="text" v-model="val" @keyup.enter="addTodo">
    <ul>
      <li v-for="todo in todos" :key="todo.id">{{todo.title}}</li>
    </ul>
    <!--计数器-->
    <div>count:{{count}}</div>
    <div>double:{{double}}</div>
    <div><button @click="addCount">加 1</button></div>
  </div>
</template>

<script>
export default {
  data(){
    return{
            //todoList 的 data
      val:'',
      todos:[ 
        {id:0, title:'吃饭', done:false},
        {id:1, title:'睡觉', done:false},
        {id:2, title:'学习', done:false},
      ],
	  //计数器的 data
      count:1
    }
  },
 methods:{
	//todoList 的方法
    addTodo(){
      this.todos.push({
        id:this.todos.length,
        title:this.val,
        done:false
      })
      this.val = ''
},
//计数器的方法
    addCount(){
      this.count++
    }
  },
//计数器的计算属性
  computed: {
    double() {
      return this.count * 2
    }
  }
}
</script>
```
渲染后的页面如下

[![pEF0DxS.png](https://s21.ax1x.com/2025/01/16/pEF0DxS.png)](https://imgse.com/i/pEF0DxS)

在 input 输入内容，按下回车，就会加一个 todo。点击一次“加 1”按钮，count 就会加1。

想必大家也看出来了，选项式 API 的一个特点就是，每个功能都会有 data,computed,methods 等配置项。大致呈现下面的模式（一个颜色对应一个功能）

![图片来自：https://juejin.cn/post/6891640356543627278](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4146605abc9c4b638863e9a3f2f1b001~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

图片来自：https://juejin.cn/post/6891640356543627278


选项式 API 写法的优势，可以直观的看到 data,computed,methods 等选项的代码。写法也很简单。

选项式 API 写法的劣势也比较明显。从上面代码看到一个功能的代码会交叉放在每个功能都会有 data,computed,methods 等配置项上。代码看着没什么问题，但是实际开发中，代码的数量和复杂度远远超过上面的例子。当其中一个功能需要修改的时候，就要在 data,computed,methods 等配置项里面找到那个功能打代码进行修改。上面例子代码少，不复杂修改还是很简单的，如果文件代码多，逻辑复杂时候，修改的成本就要高很多了。

要解决这个问题，可以使用 mixin。

比如抽离一个 count.js
```javascript
export default {
  data() {
    return {
      count:1
    }
  },
  computed: {
    double() {
      return this.count * 2
    }
  },
  methods:{
    addCount(){
      this.count++
    }
  }
}
```
App.vue
```html
<template>
  <div id="app">
    <!--todoList-->
    <input type="text" v-model="val" @keyup.enter="addTodo">
    <ul>
      <li v-for="todo in todos" :key="todo.id">{{todo.title}}</li>
    </ul>
    <!--计数器-->
    <div>count:{{count}}</div>
    <div>double:{{double}}</div>
    <div><button @click="addCount">加 1</button></div>
  </div>
</template>

<script>
import Count from './count'
export default {
  //通过 mixins 引入 Count
  mixins:[Count],
  data(){
    return{
      val:'',
      todos:[ 
        {id:0, title:'吃饭', done:false},
        {id:1, title:'睡觉', done:false},
        {id:2, title:'学习', done:false},
      ]
    }
  },
  methods:{
    addTodo(){
      this.todos.push({
        id:this.todos.length,
        title:this.val,
        done:false
      })
      this.val = ''
    }
  }
}
</script>
```
这样写，代码确实拆分了，但是 mixin 问题也暴露出来了，App.vue 里面的 template 上，count，double，addCount 这些属性完全不知道从哪来的，不知道是 mixin，还是全局 install，还是 Vue.prototype.count 设置的，数据来源完全模糊，调试非常麻烦。

而且有的时候，mixin 带来的 count，double，addCount 这些属性，App.vue 如果也要使用，到时候又不知道 this 的 count，double，addCount 这些属性从哪里来的。

mixin 的问题还不止这个，还有可能造成命名冲突，比如 count.js 里面的 count 改成 val，就会和 App.vue 的 val 产生冲突了。mixin 的特性，可能压根不知道是哪两个变量冲突了，有会给调试带来麻烦。

如果想复用选项式 API 写法，就得先把上面的问题避免。

## Composition API

同样的页面，下面使用 Composition API 方式实现一次。
```html
<template>
  <div id="app">
    <input type="text" v-model="val" @keyup.enter="addTodo">
    <ul>
      <li v-for="todo in todos" :key="todo.id">{{todo.title}}</li>
    </ul>
    <div>count:{{count}}</div>
    <div>double:{{double}}</div>
    <div><button @click="add">加1</button></div>
  </div>
</template>

<script setup>
import {reactive, ref, computed} from 'vue'
//todoList
let val = ref('')
let todos = reactive([ 
    {id:0, title:'吃饭', done:false},
    {id:1, title:'睡觉', done:false},
    {id:2, title:'lsp', done:false},
])
function addTodo(){
  todos.push({
    id:todos.length,
    title:val.value,
    done:false
  })
  val.value = ''
}
//计数器
let count = ref(1)
function add(){
    count.value++
}
let double = computed(()=>count.value*2)
</script>
```
可以看到，todoList 的代码（data，methods）和计数器的代码（data，methods，computed）按照功能分开一块一块了。这样一来，如果需要修改其中一个功能，就不需要交叉的看其他功能的代码了，代码会比 options API 方式更清晰。

代码大概是下面这样

![https://juejin.cn/post/6891640356543627278](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d05799744a6341fd908ec03e5916d7b6~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

图片截自：https://juejin.cn/post/6891640356543627278

至于复用，上面的代码可能一下看不出来效果，下面抽离出 todo.js 和 count.js，复用效果就出来了。

App.vue
```html
<template>
  <div id="app">
    <input type="text" v-model="val" @keyup.enter="addTodo">
    <ul>
      <li v-for="todo in todos" :key="todo.id">{{todo.title}}</li>
    </ul>
    <div>count:{{count}}</div>
    <div>double:{{double}}</div>
    <div><button @click="add">加1</button></div>
  </div>
</template>

<script setup>

import useToDo from './todo'
import useCounter from './count'

let {val, todos, addTodo} = useToDo()


let {val:count,double,add} = useCounter() 
</script>
```
todo.js
```javascript
import {reactive, ref} from 'vue'
export default function useToDo(){
  let val = ref('')
  let todos = reactive([ 
      {id:0, title:'吃饭', done:false},
      {id:1, title:'睡觉', done:false},
      {id:2, title:'lsp', done:false},
  ])
  function addTodo(){
    todos.push({
      id:todos.length,
      title:val.value,
      done:false
    })
    val.value = ''
  }
  return {val, todos, addTodo}
}
```
count.js
```javascript
import {ref, computed} from 'vue'
export default function useCounter(){
    let val = ref(1)
    function add(){
        val.value++
    }
    let double = computed(()=>val.value*2)
    return {val, double, add}
}
```

这样一改写，复用就能看出来了，复用的改写也很简单。

想必大家也看到了，实际上 Composition API 就是把代码分成了，很多个 function，再引入

![https://juejin.cn/post/6891640356543627278](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4146605abc9c4b638863e9a3f2f1b001~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

图片截自：https://juejin.cn/post/6891640356543627278

Composition API 的劣势就是学习难度会稍微高一些，但不算太难。以及 Composition API 是 vue3之后才支持，网上的资源，生态肯定不如 options api，但随着时间发展，这些问题不大。

|-------|组合式 API|选项式 API|
|-------|----------|---------|
|‌逻辑复用|更好，通过可组合函数实现|较差，通常通过 mixin 实现，但存在作用域不清晰、命名冲突等问题|
|‌代码组织|更清晰，按逻辑关注点组织代码|直观，但复杂组件中代码可能分散在多个选项中|
|‌响应式系统|更灵活，通过 ref 和 reactive 控制|相对固定，主要通过 data 选项|
|‌学习难度|较难，需要理解新的概念和模式|简单，符合传统面向对象编程的思维方式|
|‌IDE 支持|较好，但依赖于特定插件或配置|良好，多数 IDE 都有良好支持|
|‌文档和资源|逐渐增加，但相对于选项式 API 较少|丰富，有大量教程和社区支持|

## Composition API 和 Options API 如何公用？
> 首先说明，不建议这种写法。在一个组件里面，如果用了 Options API，就继续使用 Options API，如果用了 Composition API 就继续用 Composition API。

> 如果一个基于 Options API 开发的组件需要和一个基于 Composition API 开发的组件进行整个，就可以在一个选项式 API 的组件中通过 setup() 选项来使用组合式 API。
```html
<script>
import { ref } from 'vue'

export default {
  data(){
    return {
      msgOption:'守候-options'
    }
  },
  setup() {
    const msgSetup = ref('守候-setup')
    return {
      msgSetup
    }
  },
}
</script>

<template>
  <h1>{{ msgOption }}</h1>
  <input v-model="msgOption" />

  <h1>{{ msgSetup }}</h1>
  <input v-model="msgSetup" />
</template>
```

[![pEF042T.png](https://s21.ax1x.com/2025/01/16/pEF042T.png)](https://imgse.com/i/pEF042T)

但是需要注意 data 不要重名等。如果重名就会出现问题，如下实例
```html
<script>
import { ref } from 'vue'

export default {
  data(){
    return {
      msg:'守候-options'
    }
  },
  setup() {
    const msg = ref('守候-setup')
    const handleUpdateMsgForSetup=function(){
      msg.value='修改后-setup'
    }
    return {
      msg,
      handleUpdateMsgForSetup
    }
  },

  methods:{
    handleUpdateMsgForOptions(){
      this.msg='修改后-options'
    }
  },

  mounted() {
    console.log(this.msg) // 0
  }
}
</script>

<template>
  <h1>{{ msg }}</h1>
  <input v-model="msg" />

  <p @click="handleUpdateMsgForOptions">handleUpdateMsgForOptions</p>
  <p @click="handleUpdateMsgForSetup">handleUpdateMsgForSetup</p>
</template>
```

[![pEF0hGV.png](https://s21.ax1x.com/2025/01/16/pEF0hGV.png)](https://imgse.com/i/pEF0hGV)

上面的例子可以看到，在 data 和 setup 同时声明了 msg，但是拿的是 setupData。那是因为，vue3 的写法是，如果 setupData 里面有值，就先拿 setupData 的值。setupData 没有就再从 data 里面取值。

自然而然点击 handleUpdateMsgForSetup，msg 的值就会变成‘修改后-setup’，点击 handleUpdateMsgForOptions，msg 的值就会变成‘修改后-options’


参考链接

https://juejin.cn/post/6892017198450081800
https://cn.vuejs.org/guide/extras/composition-api-faq
https://juejin.cn/post/6894993303486332941