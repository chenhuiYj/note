

## 1.Redux 和 MobX 的相同点

1.都解决了状态管理中状态混乱，传递不便捷，无法有效同步状态的问题

2.通常都是与 React 之类的库一起使用。但不仅仅是 React

## 2.Redux 和 MobX 的不同点

### 2-1.数据流

#### redux 数据流
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/43cad858f1f74b6a932c7c4eb273e489~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)
图片来自：https://juejin.cn/post/7080795642217365541

初始阶段：调用一次 reducer，传入的 state 作为初始 state，并创建好 store；

更新阶段：

1.触发页面事件，调用 dispatch 把一个 action 发送到 store 中。

2.store 用之前的 state 和 action 中的数据再一次调用 reducer，并返回新的 state 作为新的 state。

3.store 再去通知所有订阅（就是使用这个 store 中的数据）过的 UI，通知他们 store 发生了变化。

4.每个订阅过 store 数据的 UI 组件都会检查它们需要的 state 部分是否被更新

发现数据被更新的每个 UI 组件强制使用新数据重新渲染，紧接着更新网页

#### Mobx 数据流

![](https://cn.mobx.js.org/flow.png)
图片来自：https://cn.mobx.js.org/

MobX 就是触发 actions，更改 state 的值，然后 Mobx 更改 state 的副作用就是更新 UI，看似比 Redux 的要简单很多，其实是 MobX 在背后默默的帮我们实现了。

### 2-2.存储的数据结构

1.Redux 默认是存储的一个原生的 JavaScript 对象，而 MobX 则是存贮了一个可观察的对象

2.Redux 需要手动追踪所有状态对象的变更

3.MobX 中可以监听可观察对象，当其变更时讲自动触发监听（感觉炸一看跟 Vue 的双向绑定挺像，但是细看其实 MobX 还是需要组件触发事件，随后再去更改数据）

### 2-3.不可变（Immutable）和可变（Mutable）

Redux 对状态的处理使用不可变值，通常是不可变的。只能通过 dispatch 和 action 进行修改，在原来状态的基础上返回一个新的状态对象。

而 MobX 这边可以直接更改状态，继而更改视图数据

### 2-4.函数式编程和面向对象编程

Redux 更拥抱函数式编程，这也更贴合 React 的思想（毕竟现在 React 的 Hooks，你们懂的），因为 Redux 拥抱函数式编程，因此使用纯函数，函数获取输入，并输出，没有其他副作用。

而 MobX 拥抱的面向对象编程，一个 store 就是一个对象。

### 2-5.一个 Store 和多个 Store

使用的时候好像都是多个 Store。在 Redux 中，其实是把状态保存在一个全局的 store 中或者全局的 state 中，一个数据来源保证了数据的稳定，并支持多个 reducer 可以去更改他。即使项目上是多个 store，但最后也是用 combineReducers 合成一个的，当在组件中使用的使用的时候，我们只不过是拿到了单一数据源中的下面一层的数据，而不是直接拿到某个 store 的中的所有值


在 MobX 中可以存在多个 Store，每个对象就是一个 Store，多个 Store 互相不受影响，需要更改某个 Store 中的对象，必须使用某个 Store 中的@action

> 其实在 Redux 中不是严格的单个 Store，我们也可以创建多个 Store，但是这样有违 Redux 的设计初衷，而且会使代码状态又混乱起来