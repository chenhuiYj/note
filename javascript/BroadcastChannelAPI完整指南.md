# Broadcast Channel API 完整指南

## 📚 目录

1. [什么是 Broadcast Channel API](#什么是-broadcast-channel-api)
2. [核心特点](#核心特点)
3. [浏览器兼容性](#浏览器兼容性)
4. [基本用法](#基本用法)
5. [完整工具类代码](#完整工具类代码)
6. [使用示例](#使用示例)
7. [应用场景](#应用场景)
8. [注意事项](#注意事项)

---

## 什么是 Broadcast Channel API？

Broadcast Channel API 是浏览器提供的**跨上下文通信 API**，允许：

- ✅ **不同标签页之间**通信
- ✅ **不同窗口之间**通信
- ✅ **主页面与 iframe** 之间通信
- ✅ **主线程与 Web Worker** 之间通信

### 核心概念

Broadcast Channel API 允许**同源的不同浏览器上下文**之间进行实时通信，无需服务器中转。

---

## 核心特点

1. **简单易用**：无需手动管理消息格式
2. **原生支持**：现代浏览器原生支持
3. **类型安全**：可以传递任意可序列化的数据（对象、数组、字符串等）
4. **自动清理**：关闭通道时自动清理资源
5. **降级方案**：已提供 localStorage 降级方案

---

## 浏览器兼容性

- ✅ Chrome 54+
- ✅ Firefox 38+
- ✅ Safari 15.4+
- ✅ Edge 79+

---

## 基本用法

### 1. 创建或加入频道

```javascript
const channel = new BroadcastChannel('my-channel');
```

### 2. 发送消息

```javascript
channel.postMessage({
  type: 'hello',
  message: 'Hello from Tab 1!',
  timestamp: Date.now(),
});
```

### 3. 接收消息

```javascript
channel.onmessage = (event) => {
  console.log('收到消息:', event.data);
};
```

### 4. 监听错误

```javascript
channel.onmessageerror = (event) => {
  console.error('消息错误:', event);
};
```

### 5. 关闭通道

```javascript
channel.close();
```

---

## 完整工具类代码

### `src/utils/broadcastChannel.js`

```javascript
/**
 * Broadcast Channel API 工具类
 * 用于在不同标签页、窗口、iframe 或 worker 之间进行通信
 */

/**
 * 创建或加入一个 Broadcast Channel
 * @param {string} channelName - 频道名称
 * @returns {Object} 返回频道对象，包含发送和接收消息的方法
 */
export function useBroadcastChannel(channelName) {
  if (!channelName || typeof channelName !== 'string') {
    throw new Error('频道名称必须是字符串');
  }

  // 检查浏览器是否支持 Broadcast Channel API
  if (typeof BroadcastChannel === 'undefined') {
    console.warn('浏览器不支持 Broadcast Channel API，将使用 localStorage 作为降级方案');
    return createLocalStorageFallback(channelName);
  }

  const channel = new BroadcastChannel(channelName);
  const listeners = new Set();

  /**
   * 发送消息到所有监听该频道的上下文
   * @param {*} message - 要发送的消息（可以是任意可序列化的数据）
   */
  const postMessage = (message) => {
    try {
      channel.postMessage({
        timestamp: Date.now(),
        data: message,
      });
    } catch (error) {
      console.error('发送消息失败:', error);
    }
  };

  /**
   * 监听频道消息
   * @param {Function} callback - 消息回调函数
   * @returns {Function} 返回取消监听的函数
   */
  const onMessage = (callback) => {
    if (typeof callback !== 'function') {
      throw new Error('回调函数必须是函数类型');
    }

    const handler = (event) => {
      try {
        callback(event.data.data, event);
      } catch (error) {
        console.error('处理消息时出错:', error);
      }
    };

    channel.addEventListener('message', handler);
    listeners.add(handler);

    // 返回取消监听的函数
    return () => {
      channel.removeEventListener('message', handler);
      listeners.delete(handler);
    };
  };

  /**
   * 监听消息错误
   * @param {Function} callback - 错误回调函数
   */
  const onError = (callback) => {
    if (typeof callback !== 'function') {
      throw new Error('回调函数必须是函数类型');
    }

    channel.addEventListener('messageerror', callback);
  };

  /**
   * 关闭频道
   */
  const close = () => {
    // 移除所有监听器
    listeners.forEach((handler) => {
      channel.removeEventListener('message', handler);
    });
    listeners.clear();

    // 关闭频道
    channel.close();
  };

  return {
    postMessage,
    onMessage,
    onError,
    close,
    // 暴露原始 channel 对象，用于高级用法
    channel,
  };
}

/**
 * 使用 localStorage 作为降级方案（用于不支持 Broadcast Channel 的浏览器）
 */
function createLocalStorageFallback(channelName) {
  const storageKey = `__broadcast_${channelName}__`;
  const listeners = new Set();

  const postMessage = (message) => {
    try {
      const data = {
        timestamp: Date.now(),
        data: message,
        channel: channelName,
      };
      localStorage.setItem(storageKey, JSON.stringify(data));
      // 立即触发 storage 事件（同源页面会收到）
      window.dispatchEvent(new StorageEvent('storage', {
        key: storageKey,
        newValue: JSON.stringify(data),
      }));
    } catch (error) {
      console.error('发送消息失败:', error);
    }
  };

  const onMessage = (callback) => {
    if (typeof callback !== 'function') {
      throw new Error('回调函数必须是函数类型');
    }

    const handler = (event) => {
      // 只处理当前频道的消息
      if (event.key === storageKey && event.newValue) {
        try {
          const messageData = JSON.parse(event.newValue);
          callback(messageData.data, event);
        } catch (error) {
          console.error('解析消息失败:', error);
        }
      }
    };

    window.addEventListener('storage', handler);
    listeners.add(handler);

    return () => {
      window.removeEventListener('storage', handler);
      listeners.delete(handler);
    };
  };

  const onError = (callback) => {
    console.warn('localStorage 降级方案不支持错误监听');
  };

  const close = () => {
    listeners.forEach((handler) => {
      window.removeEventListener('storage', handler);
    });
    listeners.clear();
    localStorage.removeItem(storageKey);
  };

  return {
    postMessage,
    onMessage,
    onError,
    close,
    channel: null, // localStorage 方案没有原始 channel
  };
}

/**
 * 预定义的频道名称常量
 */
export const BROADCAST_CHANNELS = {
  // 抽奖相关
  DRAW_UPDATE: 'draw-update',
  DRAW_RESULT: 'draw-result',
  
  // 用户相关
  USER_UPDATE: 'user-update',
  USER_LOGIN: 'user-login',
  USER_LOGOUT: 'user-logout',
  
  // 订单相关
  ORDER_UPDATE: 'order-update',
  ORDER_CREATE: 'order-create',
  
  // 通用
  DATA_SYNC: 'data-sync',
  NOTIFICATION: 'notification',
};
```

---

## 使用示例

### 示例 1: 基础用法

```javascript
import { useBroadcastChannel } from '@/utils/broadcastChannel';

// 创建频道
const channel = useBroadcastChannel('my-channel');

// 发送消息
channel.postMessage({
  type: 'hello',
  message: 'Hello from Tab 1!',
  timestamp: Date.now(),
});

// 监听消息
const unsubscribe = channel.onMessage((data, event) => {
  console.log('收到消息:', data);
  console.log('完整事件:', event);
  
  if (data.type === 'hello') {
    console.log('收到问候:', data.message);
  }
});

// 取消监听
// unsubscribe();

// 关闭频道
// channel.close();
```

### 示例 2: 在 Vue 组件中使用

```vue
<template>
  <div>
    <button @click="handleDrawComplete">完成抽奖</button>
  </div>
</template>

<script setup>
import { onMounted, onUnmounted } from 'vue';
import { useBroadcastChannel, BROADCAST_CHANNELS } from '@/utils/broadcastChannel';

let drawChannel = null;
let unsubscribe = null;

onMounted(() => {
  // 创建抽奖更新频道
  drawChannel = useBroadcastChannel(BROADCAST_CHANNELS.DRAW_UPDATE);

  // 监听其他标签页的抽奖更新
  unsubscribe = drawChannel.onMessage((data) => {
    if (data.type === 'draw-complete') {
      console.log('其他标签页完成了抽奖:', data);
      // 刷新当前页面的数据
      refreshDrawData();
    }
  });
});

onUnmounted(() => {
  // 清理资源
  if (unsubscribe) {
    unsubscribe();
  }
  if (drawChannel) {
    drawChannel.close();
  }
});

// 抽奖完成后，通知其他标签页
const handleDrawComplete = (drawResult) => {
  drawChannel.postMessage({
    type: 'draw-complete',
    result: drawResult,
    timestamp: Date.now(),
  });
};
</script>
```

### 示例 3: 跨标签页数据同步

```javascript
// 标签页 A（发送方）
import { useBroadcastChannel, BROADCAST_CHANNELS } from '@/utils/broadcastChannel';

const syncChannel = useBroadcastChannel(BROADCAST_CHANNELS.DATA_SYNC);

// 用户数据更新时，通知其他标签页
const updateUserData = (userData) => {
  syncChannel.postMessage({
    type: 'user-data-update',
    data: userData,
  });
};

// 标签页 B（接收方）
const receiveChannel = useBroadcastChannel(BROADCAST_CHANNELS.DATA_SYNC);

receiveChannel.onMessage((data) => {
  if (data.type === 'user-data-update') {
    // 更新本地用户数据
    updateLocalUserData(data.data);
  }
});
```

### 示例 4: 实时通知系统

```javascript
import { useBroadcastChannel, BROADCAST_CHANNELS } from '@/utils/broadcastChannel';

// 通知频道
const notificationChannel = useBroadcastChannel(BROADCAST_CHANNELS.NOTIFICATION);

// 发送通知
const sendNotification = (title, message, type = 'info') => {
  notificationChannel.postMessage({
    type: 'notification',
    title,
    message,
    notificationType: type,
    timestamp: Date.now(),
  });
};

// 接收通知（可以在任何标签页）
notificationChannel.onMessage((data) => {
  if (data.type === 'notification') {
    // 显示通知
    showNotification(data.title, data.message, data.notificationType);
  }
});
```

### 示例 5: 在 DevData.vue 中的应用

```javascript
// 在抽奖页面（DrawDetails.vue）发送消息
import { useBroadcastChannel, BROADCAST_CHANNELS } from '@/utils/broadcastChannel';

const drawUpdateChannel = useBroadcastChannel(BROADCAST_CHANNELS.DRAW_UPDATE);

// 抽奖完成后发送消息
const handleDrawComplete = (result) => {
  drawUpdateChannel.postMessage({
    type: 'draw-complete',
    prizePoolId: result.prizePoolId,
    boxId: result.boxId,
    timestamp: Date.now(),
  });
};

// 在 DevData.vue 中接收消息
import { onMounted, onUnmounted } from 'vue';
import { useBroadcastChannel, BROADCAST_CHANNELS } from '@/utils/broadcastChannel';

let drawUpdateChannel = null;
let unsubscribe = null;

onMounted(() => {
  drawUpdateChannel = useBroadcastChannel(BROADCAST_CHANNELS.DRAW_UPDATE);
  
  unsubscribe = drawUpdateChannel.onMessage((data) => {
    if (data.type === 'draw-complete') {
      // 刷新数据
      selectPrizePool(data.prizePoolId, { boxId: data.boxId });
      getUserPrizePoolData();
    }
  });
});

onUnmounted(() => {
  if (unsubscribe) {
    unsubscribe();
  }
  if (drawUpdateChannel) {
    drawUpdateChannel.close();
  }
});
```

### 示例 6: 错误处理

```javascript
const errorChannel = useBroadcastChannel('error-channel');

errorChannel.onError((event) => {
  console.error('消息错误:', event);
});
```

### 示例 7: 多频道监听

```javascript
// 同时监听多个频道
import { useBroadcastChannel, BROADCAST_CHANNELS } from '@/utils/broadcastChannel';

const channels = {
  draw: useBroadcastChannel(BROADCAST_CHANNELS.DRAW_UPDATE),
  user: useBroadcastChannel(BROADCAST_CHANNELS.USER_UPDATE),
  order: useBroadcastChannel(BROADCAST_CHANNELS.ORDER_UPDATE),
};

// 监听所有频道
Object.keys(channels).forEach((key) => {
  channels[key].onMessage((data) => {
    console.log(`收到 ${key} 频道的消息:`, data);
  });
});

// 清理所有频道
const cleanup = () => {
  Object.values(channels).forEach((channel) => {
    channel.close();
  });
};
```

---

## 与 localStorage + storage 事件的对比

| 特性 | Broadcast Channel | localStorage + storage |
|------|-------------------|----------------------|
| **性能** | ✅ 更快，直接内存通信 | ⚠️ 较慢，需要序列化/反序列化 |
| **数据大小限制** | ✅ 无限制（受内存限制） | ⚠️ 通常 5-10MB |
| **同步性** | ✅ 实时 | ⚠️ 可能有延迟 |
| **代码复杂度** | ✅ 简单 | ⚠️ 需要手动管理 |
| **浏览器支持** | ⚠️ 现代浏览器 | ✅ 所有浏览器 |

---

## 应用场景

### 1. 跨标签页数据同步
- 一个标签页更新数据，其他标签页自动刷新
- 适用于：DevData.vue 实时更新、购物车同步等

### 2. 用户状态同步
- 一个标签页登录/登出，其他标签页同步更新
- 适用于：多标签页应用的用户认证状态

### 3. 实时通知
- 在任意标签页显示通知
- 适用于：系统通知、消息提醒等

### 4. 协作功能
- 多用户实时协作
- 适用于：在线编辑、实时聊天等

---

## 注意事项

### 1. 同源限制
⚠️ **只能在同一域名、协议、端口之间通信**

```javascript
// ✅ 可以通信
https://example.com/page1  ↔  https://example.com/page2

// ❌ 不能通信
https://example.com  ↔  http://example.com
https://example.com  ↔  https://other.com
```

### 2. 数据序列化
⚠️ **只能传递可序列化的数据**

```javascript
// ✅ 可以传递
channel.postMessage({ name: 'John', age: 30 });
channel.postMessage([1, 2, 3]);
channel.postMessage('string');

// ❌ 不能传递
channel.postMessage(() => {}); // 函数
channel.postMessage(document.body); // DOM 元素
channel.postMessage(new Map()); // Map/Set 等
```

### 3. 内存管理
⚠️ **记得在组件卸载时关闭频道**

```javascript
onUnmounted(() => {
  if (unsubscribe) {
    unsubscribe();
  }
  if (channel) {
    channel.close();
  }
});
```

### 4. 浏览器兼容性
⚠️ **已提供 localStorage 降级方案**

工具类会自动检测浏览器支持，不支持时使用 localStorage 降级方案。

---

## 快速开始

### 1. 安装工具类

将 `broadcastChannel.js` 文件放到 `src/utils/` 目录下。

### 2. 在组件中使用

```javascript
import { useBroadcastChannel, BROADCAST_CHANNELS } from '@/utils/broadcastChannel';

// 创建频道
const channel = useBroadcastChannel(BROADCAST_CHANNELS.DRAW_UPDATE);

// 发送消息
channel.postMessage({ type: 'update', data: { count: 10 } });

// 监听消息
channel.onMessage((data) => {
  console.log('收到消息:', data);
});

// 清理
channel.close();
```

---

## 总结

Broadcast Channel API 是一个强大的跨上下文通信工具，特别适合：

- ✅ 跨标签页实时数据同步
- ✅ 用户状态同步
- ✅ 实时通知系统
- ✅ 多标签页协作功能

相比 localStorage + storage 事件，Broadcast Channel API 提供了：
- 更好的性能
- 更简单的 API
- 更实时的通信

---

## 参考资料

- [MDN - Broadcast Channel API](https://developer.mozilla.org/en-US/docs/Web/API/Broadcast_Channel_API)
- [Can I Use - Broadcast Channel](https://caniuse.com/broadcastchannel)

---

**创建时间**: 2024年
**版本**: 1.0.0
