## Vue 常见面试题（20 题）

1) Vue2 与 Vue3 的主要差异？
- Proxy 替代 defineProperty，支持深层/数组追踪；Composition API；更好 TS 支持；更小更快；Fragment/Teleport/Suspense。

2) 响应式原理（Vue2 vs Vue3）
- Vue2：getter/setter 劫持对象属性 + 依赖收集（Dep/Watcher），数组需重写方法。
- Vue3：Proxy 代理对象，Map 依赖图，精确追踪读取与写入，弱化对 API 限制。

3) 计算属性与侦听器的区别？
- 计算属性：基于依赖缓存，适合派生状态。
- 侦听器：观察副作用或异步处理，响应变化执行回调。

4) Composition API 与 Options API 的取舍？
- Composition 更利于逻辑复用与组织；Options 更易上手与规约化。团队与项目规模决定选择或混用。

5) Vue 中的虚拟 DOM 与模板编译？
- 模板编译为渲染函数（vdom），运行时 diff 更新真实 DOM；可用 JSX 或 `h` 手写渲染函数。

6) v-if 与 v-show 区别？
- v-if 条件渲染（创建/销毁），首屏少但切换慢。
- v-show 通过 CSS 隐藏，切换快但初始渲染成本高。

7) v-for 与 key 的使用规范？
- 提供稳定唯一 key，避免索引；v-for 先于 v-if 执行，避免同时使用在同一节点。

8) 父子组件通信方式？
- props/emit；v-model；`provide/inject`；事件总线（2.x）/全局状态（Pinia/Vuex）。

9) 何时使用 Vuex/Pinia？
- 跨页面共享复杂状态、可预测性与调试需求；简单场景用 `provide/inject` 或组合式函数即可。

10) 组件重渲染的触发与优化？
- 依赖变更触发渲染；避免在模板中创建新对象；`computed` 缓存；分割组件；`defineProps/defineEmits` 明确边界。

11) 异步组件与路由懒加载？
- `defineAsyncComponent` 或路由级 `component: () => import('...')`；可设置 loading/error/fallback，减少首屏体积。

12) 生命周期钩子（Vue2 vs Vue3）
- Vue2：created/mounted/updated/destroyed 等。
- Vue3：`onMounted`、`onUpdated`、`onUnmounted`、`onBeforeRouteLeave`（路由）等组合式钩子。

13) 自定义指令的应用场景？
- 直接操作 DOM 的通用行为：权限显示、懒加载、粘性、点击外部、自动聚焦。

14) 何为 Teleport 与 Suspense？
- Teleport：将子树渲染到 DOM 的其他位置（如模态对 body）。
- Suspense：协调异步组件加载，显示占位内容。

15) 模板编译优化有哪些？
- 静态提升、缓存内联事件、`v-once`、`memo` 指令（Vue3 内部）；合理拆分组件、避免大列表同步渲染。

16) keep-alive 的作用与原理？
- 缓存动态组件，保留状态与实例，减少重复创建；通过 include/exclude 控制范围，LRU 策略实现可控。

17) 路由导航守卫的执行顺序？
- 全局 beforeEach → 路由 beforeEnter → 组件内 beforeRouteEnter → 解析异步组件 → 全局 beforeResolve → 全局 afterEach。

18) SSR 与同构在 Vue 里的实现要点？
- `@vue/server-renderer`/Nuxt；数据预取、注水/脱水；避免浏览器专属 API；缓存与错误边界处理。

19) 如何与 TypeScript 高效结合？
- 使用 defineComponent、defineProps/Emits 宏、`as const` 限定、泛型组合式函数；Pinia 原生 TS 友好。

20) 常见性能陷阱？
- 在模板内创建大对象/函数；滥用深度 watch；不当使用 v-for+index；大型表格不做虚拟滚动；未释放监听与计时器。