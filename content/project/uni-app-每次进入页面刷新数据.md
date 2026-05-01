在 uni-app 中实现每次进入页面刷新数据，有多种方案：
## 方案一：使用 onShow 生命周期（最常用）

每次页面显示时都会触发，适合大多数场景：

```vue
<script>
export default {
  data() {
    return {
      addressList: [],
      goodsDetail: {}
    }
  },
  onShow() {
    // 每次进入页面都会执行
    this.refreshData()
  },
  methods: {
    refreshData() {
      // 刷新数据的方法
      this.getAddressList()
      this.getGoodsDetail()
      this.getUserInfo()
    },
    async getAddressList() {
      // 获取地址列表
    },
    async getGoodsDetail() {
      // 获取商品详情
    }
  }
}
</script>
```

## 方案二：使用 onLoad + onShow 组合

避免首次加载重复请求：

```vue
<script>
export default {
  data() {
    return {
      isFirstLoad: true
    }
  },
  onLoad(options) {
    // 首次加载获取数据
    this.initData(options)
  },
  onShow() {
    // 非首次加载时刷新
    if (!this.isFirstLoad) {
      this.refreshData()
    }
    this.isFirstLoad = false
  },
  methods: {
    initData(options) {
      // 初始化数据
      this.getAddressList()
      this.getGoodsDetail()
    },
    refreshData() {
      // 刷新数据（不包含初始化逻辑）
      this.getAddressList()
      this.getGoodsDetail()
    }
  }
}
</script>
```

## 方案三：使用页面栈监听（适合需要监听从其他页面返回）

```vue
<script>
export default {
  data() {
    return {
      needRefresh: false
    }
  },
  onLoad(options) {
    // 监听页面返回事件
    uni.onUnload(() => {
      // 页面卸载时的处理
    })
    this.initData()
  },
  onShow() {
    // 检查是否需要刷新
    if (this.needRefresh) {
      this.refreshData()
      this.needRefresh = false
    }
  },
  methods: {
    // 在其他页面修改数据后，设置刷新标志
    setNeedRefresh() {
      this.needRefresh = true
    },
    refreshData() {
      // 刷新数据
      this.getAddressList()
      this.getGoodsDetail()
    }
  }
}
</script>
```

## 方案四：使用事件总线（适合跨页面通信）

```javascript
// utils/eventBus.js
import Vue from 'vue'
export const eventBus = new Vue()

// 使用示例
// 在需要刷新的页面监听事件
import { eventBus } from '@/utils/eventBus.js'

export default {
  onLoad() {
    // 监听刷新事件
    eventBus.$on('refreshOrderDetail', () => {
      this.refreshData()
    })
  },
  onUnload() {
    // 页面卸载时移除监听
    eventBus.$off('refreshOrderDetail')
  },
  methods: {
    refreshData() {
      this.getAddressList()
      this.getGoodsDetail()
    }
  }
}

// 在其他页面触发刷新
eventBus.$emit('refreshOrderDetail')
```

## 方案五：使用 Vuex/Store 状态管理

```javascript
// store/index.js
export const store = new Vuex.Store({
  state: {
    needRefresh: false,
    refreshKey: 0
  },
  mutations: {
    setNeedRefresh(state, payload) {
      state.needRefresh = payload
    },
    triggerRefresh(state) {
      state.refreshKey++
    }
  }
})

// 在页面中使用
import { mapState, mapMutations } from 'vuex'

export default {
  computed: {
    ...mapState(['refreshKey'])
  },
  watch: {
    // 监听 refreshKey 变化，自动刷新
    refreshKey() {
      this.refreshData()
    }
  },
  onShow() {
    // 也可以在这里判断
    if (this.needRefresh) {
      this.refreshData()
      this.setNeedRefresh(false)
    }
  },
  methods: {
    ...mapMutations(['setNeedRefresh']),
    refreshData() {
      this.getAddressList()
      this.getGoodsDetail()
    }
  }
}

// 在其他页面触发
this.setNeedRefresh(true)
// 或
this.$store.commit('triggerRefresh')
```

## 方案六：使用 uni-app 的页面通讯 API

```javascript
// 发送页面间消息
uni.$emit('refreshData', { type: 'order' })

// 在页面中监听
export default {
  onLoad() {
    // 监听事件
    uni.$on('refreshData', this.handleRefresh)
  },
  onUnload() {
    // 移除监听
    uni.$off('refreshData', this.handleRefresh)
  },
  methods: {
    handleRefresh(data) {
      if (data.type === 'order') {
        this.refreshData()
      }
    },
    refreshData() {
      this.getAddressList()
      this.getGoodsDetail()
    }
  }
}
```

## 方案七：针对你的订单页面的完整示例

```vue
<template>
  <view class="order-detail">
    <!-- 页面内容 -->
  </view>
</template>

<script>
import { getSecondHandGoodAddressList, getSecondHandGoodDetail, getSecondHandGoodOrder, postSecondHandPayConfirm } from '@/api/modules/secondarymarket'

export default {
  data() {
    return {
      // ... 其他数据
      isFirstLoad: true,
      refreshTimer: null
    }
  },
  onLoad(options) {
    // 首次加载
    this.handleOptions(options)
    this.initData()
  },
  onShow() {
    // 每次显示页面时刷新
    if (!this.isFirstLoad) {
      this.refreshData()
    }
    this.isFirstLoad = false
  },
  onHide() {
    // 页面隐藏时的处理
  },
  onUnload() {
    // 页面卸载时清除定时器
    if (this.refreshTimer) {
      clearTimeout(this.refreshTimer)
    }
  },
  methods: {
    handleOptions(options) {
      // 处理页面参数
      if (options.flag === 'chooseResponse') {
        this.secondHandGoodsId = options.secondHandGoodsId
        this.secondHandDemandId = options.secondHandDemandId
        // ... 其他参数处理
      }
    },
    
    async initData() {
      // 初始化所有数据
      uni.showLoading({ title: '加载中...' })
      try {
        await Promise.all([
          this.getAddressList(),
          this.getGoodsDetail(),
          this.getUserInfo(),
          this.handlePostSecondHandPayConfirm()
        ])
      } catch (error) {
        console.error('初始化失败:', error)
      } finally {
        uni.hideLoading()
      }
    },
    
    async refreshData() {
      // 刷新数据（静默刷新或带提示）
      try {
        await Promise.all([
          this.getAddressList(true), // 静默刷新
          this.getGoodsDetail(true),
          this.getUserInfo(true),
          this.handlePostSecondHandPayConfirm(true)
        ])
        // 可选：显示刷新成功提示
        // uni.showToast({ title: '数据已更新', icon: 'success' })
      } catch (error) {
        console.error('刷新失败:', error)
      }
    },
    
    async getAddressList(silent = false) {
      if (!silent) uni.showLoading({ title: '加载地址...' })
      try {
        const res = await getSecondHandGoodAddressList({
          addressBo: {
            page: 0,
            pageSize: 0,
            defaultStatus: true
          }
        })
        if (res.code === 0 || res.code === 200 || res.success) {
          this.addressList = res.data.records || []
          if (this.addressList.length > 0 && !this.selectedAddress) {
            this.selectedAddress = this.addressList[0]
          }
        }
      } catch (error) {
        console.error('获取地址列表失败:', error)
      } finally {
        if (!silent) uni.hideLoading()
      }
    },
    
    async getGoodsDetail(silent = false) {
      if (!silent) uni.showLoading({ title: '加载商品...' })
      try {
        const res = await getSecondHandGoodDetail({
          secondHandGoodsId: this.secondHandGoodsId
        })
        if (res.code === 0 || res.code === 200 || res.success) {
          this.goodsDetail = res.data
          // 处理配送方式
          if (this.goodsDetail.deliveryType?.includes('1') && this.goodsDetail.deliveryType?.includes('2')) {
            this.showTabFlag = '自提和快递'
          } else if (this.goodsDetail.deliveryType?.includes('1')) {
            this.showTabFlag = '自提'
          } else if (this.goodsDetail.deliveryType?.includes('2')) {
            this.showTabFlag = '快递'
          }
        }
      } catch (error) {
        console.error('获取商品详情失败:', error)
      } finally {
        if (!silent) uni.hideLoading()
      }
    }
  }
}
</script>
```

## 方案八：下拉刷新（用户手动刷新）

```vue
<template>
  <view class="order-detail" @touchstart="touchStart" @touchmove="touchMove" @touchend="touchEnd">
    <!-- 或者使用 scroll-view -->
    <scroll-view scroll-y="true" @refresherrefresh="onRefresh" :refresher-enabled="true" :refresher-triggered="triggered">
      <!-- 页面内容 -->
    </scroll-view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      triggered: false
    }
  },
  onPullDownRefresh() {
    // 页面配置 enablePullDownRefresh: true
    this.refreshData()
  },
  methods: {
    async refreshData() {
      try {
        await Promise.all([
          this.getAddressList(),
          this.getGoodsDetail()
        ])
      } finally {
        // 停止下拉刷新
        uni.stopPullDownRefresh()
        this.triggered = false
      }
    }
  }
}
</script>

<style>
/* 在 pages.json 中配置 */
{
  "path": "pages-ecology/secondarymarket/orderDetail",
  "style": {
    "enablePullDownRefresh": true,
    "backgroundColor": "#f5f5f5"
  }
}
</style>
```

## 推荐方案总结

| 场景 | 推荐方案 |
|------|---------|
| 简单页面，每次都需要刷新 | `onShow` + 刷新方法 |
| 需要区分首次加载和后续刷新 | `onLoad` + `onShow` + 标志位 |
| 跨页面通信刷新 | `uni.$emit` / `uni.$on` |
| 用户主动刷新 | 下拉刷新 `onPullDownRefresh` |
| 复杂数据状态管理 | Vuex + 监听器 |

**对于你的订单页面，推荐使用方案一（onShow）**，因为订单页每次进入都需要最新的数据。