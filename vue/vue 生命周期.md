可分为8个阶段创建前/后，渲染前/后，更新前/后，销毁前/后

![image](https://user-images.githubusercontent.com/27403818/83328943-40523400-a2b9-11ea-9bbc-7ff85ec28eac.png)


#### 创建前/后：

在 beforeCreated 阶段，vue 实例的挂载元素和数据对象都为，还未初始化。在 created 阶段，实例的 data 对象有了，el还没有。

#### 渲染前/后

在 beforeMount 阶段，vue 实例的 $e l和 data 都初始化了，但还是渲染之前为虚拟的 dom 节点，data.message 还未替换。在 mounted 阶段，vue 实例挂载完成，data.message 成功渲染。

#### 更新前/后
当 data 变化时，会触发 beforeUpdate 和 updated 方法。在 beforeUpdate 阶段，数据更新了，但是视图的 dom 结构还没有更新，在 updated 就更新了dom 结构

#### 销毁前/后
在执行destroy方法后，对 data 的改变不会再触发周期函数，说明此时 vue 实例已经解除了事件监听以及和 dom 的绑定，但是dom结构依然存在


#### 注意点

1.不能使用箭头函数作为生命周期钩子函数，因为找到 vue 实例的 this

比如 created: () => console.log(this.a) 或 vm.$watch('a', newValue => this.myMethod())。因为箭头函数并没有 this，this 会作为变量一直向上级词法作用域查找，直至找到为止，经常导致 Uncaught TypeError: Cannot read property of undefined 或 Uncaught TypeError: this.myMethod is not a function 之类的错误。

2.需要用到 nextTick 的场景

1.在Vue生命周期的 created() 钩子函数进行的 DOM 操作一定要放在 Vue.nextTick() 的回调函数中。原因是什么呢，原因是在 created() 钩子函数执行的时候 DOM 其实并未进行任何渲染，而此时进行DOM操作无异于徒劳，所以此处一定要将 DOM 操作的 js 代码放进 Vue.nextTick() 的回调函数中。与之对应的就是 mounted 钩子函数，因为该钩子函数执行时所有的 DOM 挂载和渲染都已完成，此时在该钩子函数中进行任何 DOM 操作都不会有问题 。

2.在数据变化后要执行的某个操作，当你设置 vm.someData = 'new value'，DOM 并不会马上更新，而是在异步队列被清除，也就是下一个事件循环开始时执行更新时才会进行必要的 DOM 更新。如果此时你想要根据更新的 DOM 状态去做某些事情，就会出现问题。。为了在数据变化之后等待 Vue 完成更新 DOM ，可以在数据变化之后立即使用 Vue.nextTick(callback) 。这样回调函数在 DOM 更新完成后就会调用。

3.mounted 不会承诺所有的子组件也都一起被挂载。如果你希望等到整个视图都渲染完毕，可以在 mounted 里面使用 vm.$nextTick 