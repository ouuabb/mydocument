## 概述

在 uni-app 中，`uni.$emit` 和 `uni.$on` 是全局事件总线（Event Bus）的实现方式，用于在不同页面、组件之间进行跨页面通讯和数据传递。它们基于 Vue 的观察者模式，允许在应用的任何地方触发和监听事件。

## 核心 API

### uni.$emit(eventName, data)

触发全局事件，可携带需要传递的数据。

**参数：**
- `eventName`：String 类型，事件名称
- `data`：任意类型，需要传递的数据

### uni.$on(eventName, callback)

监听全局事件，当事件被触发时执行回调函数。

**参数：**
- `eventName`：String 类型，事件名称
- `callback`：Function 类型，事件触发时的回调函数，接收 `data` 参数

### uni.$once(eventName, callback)

监听全局事件，但只触发一次，第一次触发后自动移除监听器。

### uni.$off(eventName, callback)

移除全局事件监听器。

**参数：**
- `eventName`：String 类型，事件名称（可选，不传则移除所有事件）
- `callback`：Function 类型，要移除的回调函数（可选）

## 使用场景

1. **非父子组件通信**：跨越多层级的组件通信
2. **跨页面传值**：从 A 页面跳转到 B 页面后，A 页面需要通知 B 页面执行某些操作
3. **兄弟组件通信**：同一页面内没有直接关联的组件之间通信
4. **全局状态同步**：多个页面需要响应同一数据变化

## 基本使用示例

### 示例 1：页面间传值

**发送数据页面（pages/send/send.vue）：**
```vue
<template>
  <view>
    <button @click="sendData">发送数据到接收页</button>
  </view>
</template>

<script>
export default {
  methods: {
    sendData() {
      // 触发全局事件，传递数据
      uni.$emit('updateUserInfo', {
        name: '张三',
        age: 25,
        city: '北京'
      })
      
      // 跳转到接收页面
      uni.navigateTo({
        url: '/pages/receive/receive'
      })
    }
  }
}
</script>
```

**接收数据页面（pages/receive/receive.vue）：**
```vue
<template>
  <view>
    <text>用户名：{{ userInfo.name }}</text>
    <text>年龄：{{ userInfo.age }}</text>
    <text>城市：{{ userInfo.city }}</text>
  </view>
</template>

<script>
export default {
  data() {
    return {
      userInfo: {
        name: '',
        age: '',
        city: ''
      }
    }
  },
  onLoad() {
    // 页面加载时注册监听器
    uni.$on('updateUserInfo', this.handleUpdateUserInfo)
  },
  onUnload() {
    // 页面卸载时移除监听器，防止内存泄漏
    uni.$off('updateUserInfo', this.handleUpdateUserInfo)
  },
  methods: {
    handleUpdateUserInfo(data) {
      this.userInfo = data
      console.log('接收到数据：', data)
    }
  }
}
</script>
```

### 示例 2：组件间通信

**子组件 A（components/ChildA.vue）：**
```vue
<template>
  <view>
    <button @click="sendToChildB">向子组件B发送消息</button>
  </view>
</template>

<script>
export default {
  methods: {
    sendToChildB() {
      uni.$emit('childEvent', {
        message: '来自子组件A的消息',
        timestamp: Date.now()
      })
    }
  }
}
</script>
```

**子组件 B（components/ChildB.vue）：**
```vue
<template>
  <view>
    <text>收到的消息：{{ receivedMsg }}</text>
  </view>
</template>

<script>
export default {
  data() {
    return {
      receivedMsg: ''
    }
  },
  mounted() {
    uni.$on('childEvent', this.handleChildEvent)
  },
  beforeDestroy() {
    uni.$off('childEvent', this.handleChildEvent)
  },
  methods: {
    handleChildEvent(data) {
      this.receivedMsg = data.message
      uni.showToast({
        title: '收到新消息',
        icon: 'success'
      })
    }
  }
}
</script>
```

### 示例 3：传递复杂数据

```vue
<script>
export default {
  methods: {
    sendComplexData() {
      const complexData = {
        id: 1001,
        name: '商品名称',
        price: 99.99,
        tags: ['热销', '新品'],
        specs: {
          color: '红色',
          size: 'L'
        },
        updateTime: new Date().toISOString()
      }
      
      uni.$emit('complexDataEvent', complexData)
    }
  }
}
</script>
```

## 完整生命周期使用模板

```vue
<script>
export default {
  data() {
    return {
      // 存储从事件接收的数据
      eventData: null
    }
  },
  
  // 页面/组件创建时注册监听器
  created() {
    this.registerEvents()
  },
  
  // 页面加载时（仅页面有效）
  onLoad(options) {
    // 可以在这里注册需要在页面加载时立即监听的事件
    uni.$on('pageSpecificEvent', this.pageEventHandler)
  },
  
  // 组件挂载完成（仅组件有效）
  mounted() {
    // 组件专用事件监听
  },
  
  // 页面显示时（每次进入页面都会触发）
  onShow() {
    // 刷新数据等操作
  },
  
  // 页面隐藏时
  onHide() {
    // 可以暂停一些操作
  },
  
  // 页面/组件销毁前移除监听器
  beforeDestroy() {
    this.unregisterEvents()
  },
  
  // 页面卸载时（仅页面有效）
  onUnload() {
    // 确保移除所有监听器
    uni.$off('pageSpecificEvent', this.pageEventHandler)
  },
  
  methods: {
    // 批量注册事件监听
    registerEvents() {
      uni.$on('event1', this.handler1)
      uni.$on('event2', this.handler2)
      uni.$on('event3', this.handler3)
    },
    
    // 批量移除事件监听
    unregisterEvents() {
      uni.$off('event1', this.handler1)
      uni.$off('event2', this.handler2)
      uni.$off('event3', this.handler3)
    },
    
    handler1(data) {
      console.log('事件1触发', data)
    },
    
    handler2(data) {
      console.log('事件2触发', data)
    },
    
    handler3(data) {
      console.log('事件3触发', data)
    },
    
    pageEventHandler(data) {
      this.eventData = data
    }
  }
}
</script>
```

## 注意事项与最佳实践

### ⚠️ 重要注意事项

1. **必须成对使用**
   - 在 `onLoad`、`mounted` 或 `created` 中注册的监听器，必须在 `onUnload`、`beforeDestroy` 中使用 `uni.$off` 移除

2. **避免重复注册**
   - 多次调用 `uni.$on` 注册同一事件会导致回调函数执行多次

3. **内存泄漏风险**
   - 未及时移除的监听器会造成内存泄漏，尤其是在页面频繁切换的场景中

4. **事件命名规范**
   - 使用有意义的命名，避免冲突，建议采用命名空间方式
   - 示例：`user:login`、`order:create`、`cart:update`

5. **页面跳转时机**
   - 如果先 `uni.$emit` 再跳转页面，目标页面可能还没注册监听器
   - 建议在目标页面 `onLoad` 中先注册监听器，再通过其他方式触发事件

### ✅ 最佳实践

1. **统一管理事件常量**
```javascript
// utils/eventConstants.js
export const EVENTS = {
  USER_LOGIN: 'user:login',
  USER_LOGOUT: 'user:logout',
  CART_UPDATE: 'cart:update',
  ORDER_PAY_SUCCESS: 'order:paySuccess'
}
```

2. **封装事件总线工具**
```javascript
// utils/eventBus.js
class EventBus {
  // 注册事件
  on(event, callback) {
    uni.$on(event, callback)
  }
  
  // 触发事件
  emit(event, data) {
    uni.$emit(event, data)
  }
  
  // 注册一次性事件
  once(event, callback) {
    uni.$once(event, callback)
  }
  
  // 移除事件
  off(event, callback) {
    if (callback) {
      uni.$off(event, callback)
    } else {
      uni.$off(event)
    }
  }
  
  // 移除所有事件
  clear() {
    uni.$off()
  }
}

export default new EventBus()
```

3. **使用示例**
```vue
<script>
import eventBus from '@/utils/eventBus'
import { EVENTS } from '@/utils/eventConstants'

export default {
  onLoad() {
    // 使用封装的事件总线
    eventBus.on(EVENTS.USER_LOGIN, this.handleUserLogin)
  },
  
  onUnload() {
    eventBus.off(EVENTS.USER_LOGIN, this.handleUserLogin)
  },
  
  methods: {
    handleUserLogin(userData) {
      console.log('用户登录', userData)
    },
    
    triggerLogin() {
      eventBus.emit(EVENTS.USER_LOGIN, {
        userId: '123',
        userName: '张三'
      })
    }
  }
}
</script>
```

## 与其他传值方式的对比

| 方式 | 优点 | 缺点 | 适用场景 |
|------|------|------|----------|
| **uni.$emit/$on** | 简单灵活，跨任意组件/页面 | 需手动管理监听器，调试困难 | 简单跨页面通信 |
| **Vuex/Pinia** | 状态集中管理，调试工具完善 | 学习成本高，代码量较大 | 大型应用，复杂状态管理 |
| **props/$emit** | 直观，数据流清晰 | 仅适用于父子组件 | 简单的父子组件通信 |
| **uni.setStorageSync** | 持久化存储 | 异步操作，性能较差 | 需要持久化的数据 |
| **url 传参** | 简单直接 | 只能传递简单数据，有长度限制 | 少量参数传递 |

## 常见问题与解决方案

### Q1: 接收不到事件数据

**原因：**
- 监听器注册晚于事件触发
- 事件名称不一致
- 监听器被提前移除

**解决方案：**
```javascript
// 确保在事件触发前注册监听器
onLoad() {
  uni.$on('myEvent', this.handler)
  // 然后再触发事件或跳转页面
}
```

### Q2: 事件被多次触发

**原因：**
- 重复注册监听器
- 页面未销毁，重复进入导致重复注册

**解决方案：**
```javascript
// 方案1：注册前先移除
onLoad() {
  uni.$off('myEvent', this.handler)
  uni.$on('myEvent', this.handler)
}

// 方案2：使用 once
onLoad() {
  uni.$once('myEvent', this.handler)
}
```

### Q3: 内存泄漏问题

**解决方案：**
```javascript
export default {
  onLoad() {
    // 保存回调函数引用，便于移除
    this.eventHandler = this.handleEvent.bind(this)
    uni.$on('myEvent', this.eventHandler)
  },
  
  onUnload() {
    // 使用保存的引用进行移除
    if (this.eventHandler) {
      uni.$off('myEvent', this.eventHandler)
    }
  }
}
```

## 总结

`uni.$emit` 和 `uni.$on` 是 uni-app 中简单实用的跨页面通信方案，适合中小型应用或简单的通信场景。使用时需要注意：

1. **及时清理监听器**，避免内存泄漏
2. **统一管理事件名称**，使用常量定义
3. **注意生命周期**，确保事件触发时监听器已注册
4. **避免过度使用**，复杂状态管理建议使用 Vuex 或 Pinia

合理运用事件总线可以大大简化跨组件通信的代码复杂度，提高开发效率。