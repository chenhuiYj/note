# React 兄弟组件互通（筛选栏 \+ 表格列表）\- 面试完整版

# 一、场景说明（面试必述）

\#\#\# 业务场景

两个同级兄弟组件（筛选栏FilterBar \+ 表格列表DataTable），需求：筛选栏修改筛选条件（关键词、状态），表格组件立刻根据最新筛选参数重新请求接口、刷新列表，实现数据联动。

\#\#\# 核心方案

采用 React 「状态提升」思想，将共享状态（筛选条件）统一放到两个兄弟组件的共同父组件中管理，通过 props 实现数据向下传递、行为向上回调，完成兄弟组件数据互通。

# 二、完整代码（拆分为3个独立组件，企业规范\+面试可直接写）

## 1\. 父组件：ListPage\.jsx（状态管理容器）

```jsx
import { useState } from 'react';
import FilterBar from './FilterBar';
import DataTable from './DataTable';

// 父组件：两个兄弟组件的共同容器，统一管理共享状态
const ListPage = () => {
  // 共享状态：筛选条件（兄弟组件均需使用/响应）
  const [filterParams, setFilterParams] = useState({
    keyword: '', // 关键词筛选
    status: '',  // 状态筛选（空=全部，1=正常，0=禁用）
  });

  // 供子组件1（筛选栏）调用，修改共享状态的回调方法
  const handleFilterChange = (newParams) => {
    setFilterParams(newParams); // 更新父组件的共享状态
  };

  return (
    <div style={ margin: '30px auto' }}>
      兄弟组件数据互通 - 筛选栏 + 表格联动
      {/* 子组件1：筛选栏（接收筛选条件，触发修改回调） */}
      <FilterBar
        filterParams={filterParams}
        onFilterChange={handleFilterChange}
      />
      {/* 子组件2：表格列表（接收筛选条件，监听变化重新请求） */}
<DataTable filterParams={filterParams} />
    
  );
};

export default ListPage;
```

## 2\. 子组件1：FilterBar\.jsx（筛选栏组件）

```jsx
// 子组件1：筛选栏（兄弟A）- 负责接收筛选条件、修改筛选参数
const FilterBar = ({ filterParams, onFilterChange }) => {
  // 关键词输入变化：更新筛选条件
  const handleKeywordChange = (e) => {
    // 保留原有筛选条件，只更新关键词（避免覆盖其他参数）
    onFilterChange({
      ...filterParams,
      keyword: e.target.value,
    });
  };

  // 状态下拉框变化：更新筛选条件
  const handleStatusChange = (e) => {
    onFilterChange({
      ...filterParams,
      status: e.target.value,
    });
  };

  return (
    <div style={Bottom: 16, padding: 12, border: '1px solid #eee' }}>
      <input
        placeholder="输入关键词搜索"
        value={handleKeywordChange}
        style={{ marginRight: 12, padding: 4, width: 200 }}
      />
      <select
        value={
        onChange={handleStatusChange}
        style={{ padding: 4, width: 120 }}
      >
        
  );
};

export default FilterBar;
```

## 3\. 子组件2：DataTable\.jsx（表格列表组件）

```jsx
import { useState, useEffect } from 'react';

// 子组件2：表格列表（兄弟B）- 接收筛选条件，监听变化并请求数据（已添加AbortController优化）
const DataTable = ({ filterParams }) => {
  const [list, setList] = useState([]); // 表格数据
  const [loading, setLoading] = useState(false); // 加载态

  // 核心：监听筛选条件变化，一旦变化就重新请求接口（添加取消请求逻辑）
  useEffect(() => {
    // 创建AbortController实例，用于取消请求
    const controller = new AbortController();
    // 获取取消信号
    const signal = controller.signal;

    fetchTableData(signal);

    // 组件卸载时，取消未完成的请求（解决卸载警告）
    return () => {
      controller.abort(); // 中断请求
    };
  }, [filterParams]); // 依赖筛选条件，参数一变就触发

  // 接口请求方法（添加取消请求支持，解决请求错乱）
  const fetchTableData = async (signal) => {
    setLoading(true);
    try {
      const res = await mockApi(filterParams, signal); // 携带筛选参数和取消信号请求
      setList(res); // 更新表格数据
    } catch (err) {
      // 忽略取消请求导致的异常（正常中断，无需报错）
      if (err.name !== 'AbortError') {
        console.error('请求表格数据失败：', err);
      }
    } finally {
      setLoading(false); // 无论成功失败，关闭加载态
    }
  };

  return (
    <div style={', borderRadius: 4 }}>
      {loading ? (
        // 加载中状态
        <div style={加载中...
      ) : (
        // 表格数据展示
        <ul style={0, padding: 12, listStyle: 'none' }}>
          {list.length === 0 ? (
            <li style={9' }}>暂无匹配数据
          ) : (
            list.map((item) => (<li 
                key={ '26px', borderBottom: '1px solid #f5f5f5', padding: 4  }}
              >
                名称：{item.name} —— 状态：{item.status === 1 ? '正常' : '禁用'}
              
            ))
          )}
        
      )}
    
  );
};

// 模拟后端接口（添加取消请求支持，适配AbortController）
function mockApi(params, signal) {
  return new Promise((resolve, reject) => {
    // 模拟接口请求延迟（500ms）
    const timer = setTimeout(() => {
      // 模拟原始数据（后端返回的全量数据）
      const originData = [
        { id: 1, name: 'React 项目', status: 1 },
        { id: 2, name: 'Vue 项目', status: 0 },
        { id: 3, name: '测试工程', status: 1 },
        { id: 4, name: 'React Hooks 实战', status: 1 },
        { id: 5, name: '废弃项目', status: 0 },
      ];

      // 前端模拟筛选逻辑（真实场景由后端实现，前端仅传参）
      const filteredData = originData.filter((item) => {
        // 关键词筛选（模糊匹配）
        const matchKeyword = item.name.includes(params.keyword);
        // 状态筛选（空值匹配所有，非空匹配对应状态）
        const matchStatus = params.status ? item.status == params.status : true;
        return matchKeyword && matchStatus;
      });

      resolve(filteredData); // 返回筛选后的数据
    }, 500);

    // 监听取消信号，若请求被取消，清除定时器并抛出异常
    signal.addEventListener('abort', () => {
      clearTimeout(timer); // 清除请求定时器
      reject(new DOMException('请求已被取消', 'AbortError')); // 抛出取消异常
    });
  });
}

export default DataTable;
```

# 三、核心逻辑拆解（面试口述重点）

1. 状态提升：将「筛选条件filterParams」这一共享数据，提升到两个兄弟组件的共同父组件（ListPage）中统一管理，保证单一数据源，避免数据割裂。

2. 数据向下传递：父组件通过props，将filterParams分别传给FilterBar和DataTable，两个子组件获取到最新的筛选条件。

3. 行为向上回调：FilterBar中修改筛选条件后，通过父组件传递的onFilterChange回调，将新的筛选参数传给父组件，触发父组件状态更新。

4. 联动刷新：父组件状态更新后，会重新渲染两个子组件；DataTable通过useEffect监听filterParams变化，一旦变化就重新请求接口，实现表格自动刷新。

# 四、面试官高频追问（3道必背，贴合本场景）

## 追问1：为什么兄弟组件不能各自维护state，非要做状态提升？

标准答案：

1. 数据一致性问题：如果FilterBar和DataTable各自存一份筛选条件，会导致数据割裂，筛选栏修改后，表格无法感知，数据不同步。

2. 联动失效：筛选栏修改参数后，表格不会自动刷新，需要额外写通信逻辑，代码冗余且易出bug。

3. 符合React设计思想：遵循「单向数据流」（数据自上而下传递，行为自下而上回调），单一数据源便于维护、排查问题。

## 追问2：如果组件嵌套层级很深，状态提升层层传props会有什么问题？怎么解决？

标准答案：

1. 存在问题：会出现「props drilling（props层层透传）」，中间无关组件需要接收并传递多余的props，代码冗余、维护成本高，且不利于组件复用。

2. 解决方案（按需选型）：
        

    - 简单跨层场景：使用React\.createContext \+ useContext实现跨层状态透传，无需层层传递props。

    - 复杂全局场景：使用Redux、Zustand、Jotai等全局状态管理库，统一管理全局共享状态。

    - 简单业务场景：可通过「组合组件」减少嵌套层级，避免过度透传。

## 追问3：本场景中，表格用useEffect监听筛选参数请求接口，有什么隐患？怎么优化？

标准答案：

1. 核心隐患：
        

    - 请求错乱：连续快速切换筛选条件，会发起多次异步请求，若旧请求晚于新请求返回，会覆盖新请求的数据，导致表格展示错误。

    - 卸载警告：组件卸载后，接口请求仍在进行，回调中setState会触发React警告（Can\&\#39;t perform a React state update on an unmounted component）。

2. 优化方案：
        

    - 请求防抖：给筛选操作添加防抖（如300ms），短时间内多次修改筛选条件，只执行最后一次请求，减少无效请求。

    - 取消请求：使用AbortController中断上一次未完成的请求，避免旧请求覆盖新数据。

    - 卸载拦截：组件卸载时标记状态，中断接口回调中的setState操作，避免警告。

    - 依赖规范：严格控制useEffect的依赖项，只依赖filterParams，避免不必要的重复请求。

# 五、面试加分短句（口述收尾）

简单兄弟组件联动，「状态提升」是最优方案，成本最低、代码最简洁；组件层级较深时，用Context跨层透传；全局复杂状态（如多页面共享），再引入状态管理库，按需选型，不做过度设计。

> （注：文档部分内容可能由 AI 生成）
