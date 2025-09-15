## React 常见面试题（20 题）

1) React 的核心思想是什么？
- **声明式 UI**：根据状态渲染视图。
- **组件化**：UI 拆分为可复用组件。
- **单向数据流**：数据自上而下流动，便于追踪状态变化。

2) 虚拟 DOM 是什么？为什么比直接操作 DOM 更快？
- 虚拟 DOM 是对真实 DOM 的轻量 JS 对象描述。
- 通过 diff 算法找出最小变更集，再批量更新真实 DOM，减少重排重绘与跨语言边界开销。

3) React 组件有哪几种？
- **函数组件**（推荐）：无实例、更轻量，可使用 Hooks。
- **类组件**：有生命周期与实例，逐步淡出。
- **受控/非受控组件**：表单输入是否受 state 控制。

4) 受控组件与非受控组件的区别？
- 受控：表单值由 state 驱动，onChange 同步更新，便于校验与回放。
- 非受控：使用 `ref` 读写 DOM 值，干预更少，整合第三方库方便。

5) 为什么 Hooks 不能在条件或循环中调用？
- Hooks 依赖调用顺序稳定以匹配内部索引；条件/循环会破坏顺序，导致状态错位。

6) 常见 Hooks 及使用场景？
- `useState`：本地状态。
- `useEffect`：副作用（网络请求、订阅、DOM）。
- `useMemo`/`useCallback`：计算/函数记忆，优化重渲染。
- `useRef`：持久化可变值/DOM 引用。
- `useReducer`：复杂状态逻辑集中管理。

7) useEffect 与 useLayoutEffect 的区别？
- `useEffect`：异步、在绘制后执行，不阻塞渲染。
- `useLayoutEffect`：同步、在布局与绘制前执行，适合测量/同步布局，可能造成卡顿。

8) React 如何进行性能优化？
- 组件拆分、减少重渲染；`memo`、`useMemo`、`useCallback`。
- 列表虚拟化（react-window/react-virtualized）。
- 避免在 render 创建新引用；按需渲染、代码分割、懒加载。
- Key 合理选择，避免索引作为 key 导致错位更新。

9) key 的作用是什么？
- 唯一标识列表项，帮助 diff 精确匹配，减少不必要的卸载/重建；稳定且可预测。

10) 何为受控渲染与条件渲染？
- 受控渲染：根据状态精确控制渲染输出。
- 条件渲染：通过条件表达式决定是否/渲染什么。

11) Context 的作用与注意点？
- 跨层级传递数据（主题、国际化、用户）。
- 注意：滥用会导致广泛重渲染；可拆分 context、使用选择器或 memo 化 value。

12) Redux 的核心概念？
- 单一 store、不可变 state、纯函数 reducer、单向数据流、middleware 处理副作用/扩展。

13) Redux 与 Context 的区别？
- Context 是传递通道，不含状态管理模式。
- Redux 提供完整约束（时间旅行、devtools、中间件、不可变）。

14) React 中如何处理副作用与数据请求？
- 在 `useEffect` 中发起；处理取消、错误与加载态；封装 hooks；结合 SWR/React Query 管理缓存与重试。

15) React 18 并发特性简述（如 Suspense/Transition）
- Suspense：等待异步数据时优雅显示 fallback。
- Transition：标记非紧急更新，提升交互流畅度。
- 自动批处理：跨事件源的 setState 合并。

16) 严格模式（StrictMode）做了什么？
- 仅开发启用，帮助发现副作用、废弃 API 使用，双调用某些生命周期/初始化以暴露问题。

17) 如何避免子组件不必要渲染？
- `React.memo`、稳定的 props 引用、拆分上下文、选择器、将频繁变化 state 下沉。

18) SSR/同构渲染的优势与要点？
- 优势：首屏更快、SEO 友好、可预取数据。
- 要点：避免仅浏览器可用 API；数据注水/脱水；缓存策略。

19) 表单库选择与对比（Formik/React Hook Form）
- RHF 更轻量性能好，依赖非受控与注册机制；Formik 直观、生态多，但重渲染更多。

20) 常见优化陷阱？
- 过度使用 memo 带来复杂度；错误依赖数组导致数据不同步；索引作为 key；在 render 中做重计算/创建对象。
