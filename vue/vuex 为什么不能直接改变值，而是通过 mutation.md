 vuex 不但是一种全局修改数据的工具，更重要的意义是在于把跨组件的交互，拆分为基于状态管理的处理。

使用如 vuex 本身就是希望基于这样一个数据结构的约定，使得项目代码更加直观和简单。每一个 state 
树对应整个项目的一个数据，每一个 mutation 执行完成后都可以更新到一个新的状态。这样 devtools 就可以打个 snapshot 存下来。可以尝试开着 devtool 调用一个异步的 action，可以清楚地看到它所调用的 mutation 是何时被记录下来的，并且可以立刻查看 mutation 对应的状态。

所以，通过commit 提交 mutation 的方式来修改 state 时，vue的调试工具能够记录每一次 state 的变化，这样方便调试。但是如果是直接修改state，则没有这个记录，那样做会使状态不受我们管控。如果是多个模块需要引用一个state的，然后每个人可能由不同的人开发，如果直接修改值，可能会造成数据的混乱，Mutation 也记录不到，到时候调试会有麻烦。但是通过 mutation 修改 state 这样设计，就有个统一的地方进行逻辑运算去修改。如果逻辑有变动，修改一个地方就可以了。

有点类似 react 小书的配图

![image](https://user-images.githubusercontent.com/27403818/84636483-3a9a5680-af27-11ea-99c7-a60eefcd0269.png)


> vuex 更改state 是 dispatch 给 action，action 再 commit 给 mutation，这样实现了单向数据改动 区别普通双向绑定



### 扩展知识

#### 1.如果判断 state 是通过 mutation 修改还是直接修改值？

Vuex 中修改state的唯一渠道就是执行 commit(‘xx’, payload) 方法，其底层通过执行 this._withCommit(fn) 设置 _committing 标志变量为 true ，然后才能修改 state，修改完毕还需要还原_committing 变量。外部修改虽然能够直接修改 state，但是并没有修改 _committing 标志位，所以只要 watch 一下 state ，state change时判断是否 _committing 值为 true，即可判断修改的合法性。