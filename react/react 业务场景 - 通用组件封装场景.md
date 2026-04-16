

**3个通用组件** 的详细讲解、完整可运行代码及使用示例，覆盖核心考察点，可直接复制到项目或面试默写使用。

## 一、场景核心考点

通用组件封装是 React 必会场景，核心考察以下能力，回答时需主动体现：

1. **组件拆分思想**：抽离通用逻辑，实现业务与UI分离，保证组件复用性；

2. **Props 设计规范**：合理定义必填项、默认值，支持 props 透传，预留扩展性；

3. **插槽（children）用法**：支持自定义内容，满足不同业务场景的个性化需求；

4. **受控/非受控兼容**：组件状态可由外部控制，也可内部维护，提升灵活性；

5. **事件回调设计**：提供完整的回调函数（如 onClose、onConfirm），让外部感知组件状态变化；

6. **性能优化**：使用 React.memo 缓存组件，避免无效重渲染；

7. **边界场景处理**：覆盖 loading、空状态、禁用、异常等场景，保证组件健壮性；

8. **样式扩展性**：支持 style、className 透传，方便业务层自定义样式。

## 二、最常考 3 个通用组件（完整代码+使用示例）

以下组件均为面试高频考点，代码可直接复制运行，注释清晰，适配大多数项目场景。

### 1. 通用弹窗组件 Modal（最高频，必掌握）

#### 需求说明

- 可通过 visible 控制弹窗显示/隐藏；

- 支持自定义标题、内容（插槽）、底部按钮文本；

- 支持点击遮罩关闭、ESC 键关闭；

- 提供 onClose、onConfirm、onCancel 回调事件；

- 支持样式透传、自定义 className。

#### 完整组件代码

```jsx
import { memo, useEffect } from 'react';

/**
 * 通用弹窗组件
 * @param {boolean} visible - 弹窗显示状态（必填）
 * @param {string} title - 弹窗标题，默认"提示"
 * @param {React.ReactNode} children - 弹窗内容（插槽）
 * @param {() => void} onClose - 弹窗关闭回调
 * @param {() => void} onConfirm - 确认按钮回调
 * @param {() => void} onCancel - 取消按钮回调
 * @param {boolean} maskClosable - 点击遮罩是否关闭，默认true
 * @param {string} confirmText - 确认按钮文本，默认"确定"
 * @param {string} cancelText - 取消按钮文本，默认"取消"
 * @param {string} className - 自定义类名
 * @param {React.CSSProperties} style - 自定义样式
 * @returns {React.ReactNode} 弹窗组件
 */
const Modal = memo(({
  visible = false,
  title = '提示',
  children,
  onClose,
  onConfirm,
  onCancel,
  maskClosable = true,
  confirmText = '确定',
  cancelText = '取消',
  className = '',
  style = {}
}) => {
  // 关闭弹窗（统一触发onClose）
  const handleClose = () => {
    onClose?.(); // 可选链操作，避免报错
  };

  // 点击遮罩关闭（判断点击目标是遮罩本身）
  const handleMaskClick = (e) => {
    if (e.target === e.currentTarget && maskClosable) {
      handleClose();
    }
  };

  // 监听ESC键关闭弹窗
  useEffect(() => {
    const handleKeyDown = (e) => {
      if (e.key === 'Escape' && visible) {
        handleClose();
      }
    };
    // 绑定全局键盘事件
    window.addEventListener('keydown', handleKeyDown);
    // 组件卸载时解绑，避免内存泄漏
    return () => window.removeEventListener('keydown', handleKeyDown);
  }, [visible, handleClose]);

  // 弹窗隐藏时，不渲染任何内容
  if (!visible) return null;

  return (
    <div
      className={-mask ${className}`}
      onClick={handleMaskClick}
      style={{
        position: 'fixed',
        top: 0,
        left: 0,
        right: 0,
        bottom: 0,
        background: 'rgba(0, 0, 0, 0.5)',
        display: 'flex',
        alignItems: 'center',
        justifyContent: 'center',
        zIndex: 999, // 保证弹窗在最上层
        ...style // 样式透传，覆盖默认样式
      }}
    >
      <div
        className="modal-content"
        style={ '500px',
          borderRadius: '8px',
          padding: '20px',
          boxShadow: '0 2px 10px rgba(0, 0, 0, 0.1)',
          boxSizing: 'border-box'
        }}
      >
        {/* 弹窗标题 */}
        <div style={ '18px', fontWeight: 600, marginBottom: '16px' }}>
          {title}
        

        {/* 弹窗内容（插槽） */}
        <div style={
          {children}
        

        {/* 底部按钮区域 */}
        <div style={flex', justifyContent: 'flex-end', gap: '10px' }}>
          <button
            onClick={ {
              onCancel?.(); // 触发取消回调
              handleClose(); // 关闭弹窗
            }}
            style={{
              padding: '6px 16px',
              border: '1px solid #dcdfe6',
              borderRadius: '4px',
              background: '#fff',
              cursor: 'pointer',
              transition: 'all 0.2s'
            }}
            onMouseOver={(e) => e.target.style.background = '#f5f7fa'}
            onMouseOut={(e) => e.target.style.background = '#fff'}
          >
            {cancelText}
          <button
            onClick={ {
              onConfirm?.(); // 触发确认回调
              handleClose(); // 关闭弹窗
            }}
            style={{
              padding: '6px 16px',
              background: '#1677ff',
              color: '#fff',
              border: 'none',
              borderRadius: '4px',
              cursor: 'pointer',
              transition: 'all 0.2s'
            }}
            onMouseOver={(e) => e.target.style.background = '#0f62fe'}
            onMouseOut={(e) => e.target.style.background = '#1677ff'}
          >
            {confirmText}
          
  );
});

export default Modal;
```

#### 使用示例

```jsx
import { useState } from 'react';
import Modal from './Modal'; // 引入封装的弹窗组件

export default function TestModal() {
  // 控制弹窗显示/隐藏的状态
  const [visible, setVisible] = useState(false);

  // 确认按钮回调
  const handleConfirm = () => {
    console.log('点击了确认按钮');
    // 这里可写业务逻辑（如提交表单、删除数据等）
  };

  // 取消按钮回调
  const handleCancel = () => {
    console.log('点击了取消按钮');
  };

  return (
    <div style={0px' }}>
      {/* 打开弹窗的按钮 */}
      <button
        onClick={ setVisible(true)}
        style={{ padding: '8px 16px', background: '#1677ff', color: '#fff', border: 'none', borderRadius: '4px', cursor: 'pointer' }}
      >
        打开弹窗
      

      {/* 通用弹窗组件使用 */}<Modal
        visible={visible}
        title="用户操作确认"
        onClose={() => setVisible(false)} // 关闭弹窗时重置状态
        onConfirm={handleConfirm}
        onCancel={handleCancel}
        maskClosable={true} // 点击遮罩关闭
        confirmText="确认删除"
        cancelText="取消操作"
        // 自定义样式（可选）
        style={{ background: 'rgba(0, 0, 0, 0.6)' }}
      >
        {/* 插槽内容（自定义弹窗内容） */}
        <div style={14px', color: '#333' }}>
          确定要删除这条数据吗？删除后无法恢复，请谨慎操作！
        </Modal>

  );
}

```

### 2. 通用表格组件 Table（中后台必考）

#### 需求说明

- 接收 columns（列配置）、dataSource（表格数据）两个核心 props；

- 支持 loading 加载状态、空数据状态提示；

- 支持行点击事件 onRowClick；

- 支持自定义单元格渲染（render 函数）；

- 支持自定义空状态文本、行唯一标识。

#### 完整组件代码

```jsx
import { memo } from 'react';

/**
 * 通用表格组件
 * @param {Array} columns - 列配置（必填），每一项包含title、dataIndex、key、render（可选）
 * @param {Array} dataSource - 表格数据（必填）
 * @param {boolean} loading - 加载状态，默认false
 * @param {(row: any) => void} onRowClick - 行点击回调
 * @param {string} rowKey - 行唯一标识，默认"id"
 * @param {string} emptyText - 空数据提示文本，默认"暂无数据"
 * @returns {React.ReactNode} 表格组件
 */
const Table = memo(({
  columns = [],
  dataSource = [],
  loading = false,
  onRowClick,
  rowKey = 'id',
  emptyText = '暂无数据'
}) => {
  // 校验列配置和数据的合法性
  if (!columns.length) {
    console.warn('Table组件：columns配置不能为空');
    return null;
  }

  return (
    <div style={e5e7eb', borderRadius: '8px', overflow: 'hidden' }}>
      {/* 加载状态 */}
      {loading && (
        <div style={ 'center', color: '#666', background: '#f9fafb' }}>
          加载中...
        
      )}

      {/* 空数据状态 */}
      {!loading && dataSource.length === 0 && (
        <div style={: '#999', background: '#fff' }}>
          {emptyText}
        
      )}

      {/* 表格内容（有数据且非加载状态） */}
      {!loading && dataSource.length > 0 && (
        <table style={ '100%', borderCollapse: 'collapse', fontSize: '14px' }}>
          {/* 表头 */}
          <tr style={ '#f9fafb' }}>
              {columns.map((col) => (
                <th
                  key={ padding: '12px 16px',
                    textAlign: col.align || 'left',
                    borderBottom: '1px solid #e5e7eb',
                    color: '#333',
                    fontWeight: 600
                  }}
                >
                  {col.title}
                
              ))}
           

          {/* 表体 */}
          
            {dataSource.map((row, index) => (
              <tr
                key={row[rowKey] || index} // 行唯一标识，优先用rowKey，无则用索引
                onClick={() => onRowClick?.(row)} // 行点击回调
                style={{
                  cursor: onRowClick ? 'pointer' : 'default',
                  background: index % 2 === 0 ? '#fff' : '#f9fafb',
                  transition: 'background 0.2s'
                }}
                onMouseOver={(e) => e.currentTarget.style.background = '#f5f7fa'}
                onMouseOut={(e) => e.currentTarget.style.background = index % 2 === 0 ? '#fff' : '#f9fafb'}
              >
                {columns.map((col) => (
                  <td
                    key={
                      padding: '12px 16px',
                      borderBottom: '1px solid #f5f5f5',
                      color: '#666'
                    }}
                  >
                    {/* 自定义单元格渲染：有render函数则执行，无则显示dataIndex对应的值 */}
                    {col.render ? col.render(row[col.dataIndex], row, index) : row[col.dataIndex]}
                  
                ))}
              
            ))}
          
      )}
    
  );
});

export default Table;

```

#### 使用示例

```jsx
import Table from './Table'; // 引入封装的表格组件

export default function TestTable() {
  // 列配置（核心）
  const columns = [
    {
      title: '姓名',
      dataIndex: 'name', // 对应dataSource中每一项的key
      key: 'name', // 列唯一标识，必须唯一
      align: 'left'
    },
    {
      title: '年龄',
      dataIndex: 'age',
      key: 'age'
    },
    {
      title: '性别',
      dataIndex: 'gender',
      key: 'gender',
      // 自定义单元格渲染
      render: (gender) => {
        return gender === 1 ? '男' : '女';
      }
    },
    {
      title: '操作',
      key: 'action',
      // 自定义操作按钮
      render: (_, row) => (
        <div style={<button
            onClick={ handleEdit(row)}
            style={{ padding: '4px 8px', background: '#4096ff', color: '#fff', border: 'none', borderRadius: '4px', cursor: 'pointer' }}
          >
            编辑
         <button
            onClick={ handleDelete(row.id)}
            style={{ padding: '4px 8px', background: '#f56c6c', color: '#fff', border: 'none', borderRadius: '4px', cursor: 'pointer' }}
          >
            删除
          
      )
    }
  ];

  // 表格数据（核心）
  const dataSource = [
    { id: 1, name: '张三', age: 20, gender: 1 },
    { id: 2, name: '李四', age: 25, gender: 1 },
    { id: 3, name: '王五', age: 22, gender: 0 },
    { id: 4, name: '赵六', age: 18, gender: 0 }
  ];

  // 行点击回调
  const handleRowClick = (row) => {
    console.log('点击了行：', row);
  };

  // 编辑回调
  const handleEdit = (row) => {
    console.log('编辑：', row);
  };

  // 删除回调
  const handleDelete = (id) => {
    console.log('删除id：', id);
  };

  return (
    <div style={<Table
        columns={columns}
        dataSource={dataSource}
        onRowClick={handleRowClick}
        rowKey="id" // 用id作为行唯一标识
        emptyText="暂无用户数据"
        // loading={true} // 加载状态，按需开启
      />
    
  );
}

```

### 3. 通用输入框组件 Input（受控+防抖+校验）

#### 需求说明

- 受控组件，支持外部控制 value 和 onChange；

- 支持防抖功能（可配置延迟时间）；

- 支持最大长度限制、禁用状态；

- 支持清空按钮、前后缀图标；

- 支持样式透传，适配不同UI风格。

#### 完整组件代码

```jsx
import { memo, useState, useEffect, useCallback } from 'react';

/**
 * 通用输入框组件（受控+防抖+校验）
 * @param {string} value - 输入框值（受控，必填）
 * @param {(val: string) => void} onChange - 输入变化回调（必填）
 * @param {string} placeholder - 提示文本，默认"请输入"
 * @param {number} debounce - 防抖延迟时间（ms），0表示不防抖，默认0
 * @param {number} maxLength - 最大输入长度，可选
 * @param {boolean} disabled - 是否禁用，默认false
 * @param {boolean} allowClear - 是否显示清空按钮，默认true
 * @param {React.ReactNode} prefix - 前缀图标/文本，可选
 * @param {React.ReactNode} suffix - 后缀图标/文本，可选
 * @param {React.CSSProperties} style - 自定义样式
 * @returns {React.ReactNode} 输入框组件
 */
const Input = memo(({
  value,
  onChange,
  placeholder = '请输入',
  debounce = 0,
  maxLength,
  disabled = false,
  allowClear = true,
  prefix,
  suffix,
  style = {}
}) => {
  // 内部维护输入值，避免外部value频繁变化导致的抖动
  const [innerValue, setInnerValue] = useState(value);

  // 同步外部value：当外部传入的value变化时，更新内部值
  useEffect(() => {
    setInnerValue(value);
  }, [value]);

  // 防抖处理：延迟触发onChange
  useEffect(() => {
    if (debounce > 0) {
      const timer = setTimeout(() => {
        onChange?.(innerValue);
      }, debounce);
      // 组件卸载或innerValue变化时，清除定时器
      return () => clearTimeout(timer);
    }
    // 不防抖时，实时触发onChange
  }, [innerValue, debounce, onChange]);

  // 输入变化处理（缓存回调，避免频繁创建）
  const handleChange = useCallback((e) => {
    const val = e.target.value;
    // 限制最大长度
    if (maxLength && val.length > maxLength) {
      return;
    }
    setInnerValue(val);
    // 不防抖时，实时触发外部onChange
    if (debounce === 0) {
      onChange?.(val);
    }
  }, [onChange, debounce, maxLength]);

  // 清空输入框
  const handleClear = () => {
    setInnerValue('');
    onChange?.('');
  };

  return (
    <div style={ display: 'inline-block', width: '100%' }}>
      {/* 前缀内容 */}
      {prefix && (
        <span style={',
          left: '12px',
          top: '50%',
          transform: 'translateY(-50%)',
          color: '#999',
          pointerEvents: 'none' // 避免遮挡输入框点击
        }}>
          {prefix}
       
      )}

      {/* 输入框主体 */}
      <input
        value={innerValue}
        onChange={handleChange}
        placeholder={placeholder}
        maxLength={maxLength}
        disabled={disabled}
        style={{
          width: '100%',
          padding: `8px ${suffix || allowClear ? '30px' : '12px'} ${8}px ${prefix ? '30px' : '12px'}`,
          border: '1px solid #dcdfe6',
          borderRadius: '4px',
          outline: 'none',
          fontSize: '14px',
          boxSizing: 'border-box',
          backgroundColor: disabled ? '#f5f7fa' : '#fff',
          color: disabled ? '#999' : '#333',
          ...style
        }}
        onFocus={(e) => e.target.style.borderColor = '#4096ff'}
        onBlur={(e) => e.target.style.borderColor = '#dcdfe6'}
      />

      {/* 清空按钮 */}
      {allowClear && innerValue && !disabled && (
        <span
          onClick={
          style={{
            position: 'absolute',
            right: '12px',
            top: '50%',
            transform: 'translateY(-50%)',
            color: '#999',
            cursor: 'pointer',
            fontSize: '16px'
          }}
          onMouseOver={(e) => e.target.style.color = '#666'}
        >
          ✕
        
      )}

      {/* 后缀内容（优先级低于清空按钮） */}
      {suffix && !allowClear && (
        <span style={ '12px',
          top: '50%',
          transform: 'translateY(-50%)',
          color: '#999',
          pointerEvents: 'none'
        }}>
          {suffix}
        
      )}
    
  );
});

export default Input;

```

#### 使用示例

```jsx
import { useState } from 'react';
import Input from './Input'; // 引入封装的输入框组件

export default function TestInput() {
  // 受控状态：外部控制输入框值
  const [username, setUsername] = useState('');
  const [phone, setPhone] = useState('');

  return (
    <div style={<div style={<label style={', marginBottom: '8px', fontSize: '14px' }}>用户名<Input
          value={username}
          onChange={setUsername}
          placeholder="请输入用户名"
          maxLength={10}
          allowClear
          prefix="👤" // 前缀图标
          style={{ height: '40px' }}
        />
<div style={<label style={4px' }}>手机号（防抖）<Input
          value={phone}
          onChange={setPhone}
          placeholder="请输入手机号"
          debounce={500} // 防抖500ms，输入停顿后触发onChange
          maxLength={11}
          allowClear
          prefix="📱"
          style={{ height: '40px' }}
        />
      <div style={16px' }}>
        <label style={ fontSize: '14px' }}>禁用输入框<Input
          value="不可编辑"
          onChange={() => {}}
          disabled={true}
          style={{ height: '40px' }}
        />
      

      {/* 实时显示输入值 */}
      <div style={>
        用户名：{username}手机号：{phone}
  );
}

```

## 三、面试评分关键点（必记）

封装通用组件时，面试官主要看以下几点，回答时可主动提及：

1. Props 设计：必填项明确、提供合理默认值，支持透传，方便扩展；

2. 性能优化：使用 React.memo 缓存组件，避免无效重渲染；

3. 灵活性：支持插槽、自定义渲染，满足不同业务场景；

4. 健壮性：处理边界场景（loading、空状态、禁用、异常）；

5. 易用性：提供完整回调、清晰注释，降低使用成本；

6. 可维护性：逻辑拆分清晰，代码规范，便于后续修改。

## 四、面试应答总结话术（直接默写）

> 
>       封装通用组件时，我会遵循「复用性、扩展性、易用性」三个核心原则：
> 
>       1. 合理设计 Props，明确必填项与默认值，支持 style、className 透传，预留扩展空间；
> 
>       2. 使用 React.memo 缓存组件，避免无效重渲染，提升性能；
> 
>       3. 支持插槽和自定义渲染，满足不同业务场景的个性化需求；
> 
>       4. 提供完整的事件回调，让外部组件可控制当前组件状态，实现受控/非受控兼容；
> 
>       5. 全面处理边界场景，比如 loading、空状态、禁用等，保证组件健壮性；
>  6. 编写清晰注释，规范代码格式，提升组件可维护性和易用性。
>     
> 
> 