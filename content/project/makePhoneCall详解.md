# uni.makePhoneCall 拨打电话API详解

## 概述

`uni.makePhoneCall` 是 `uni-app` 框架提供的原生API，用于实现一键拨打电话功能。调用此API会自动打开手机拨号界面，并将指定手机号填充到拨号盘，用户只需点击拨号键即可直接拨打电话。

## 语法

```javascript
uni.makePhoneCall({
    phoneNumber: '要拨打的电话号码',
    success: (res) => {
        console.log('调用成功');
    },
    fail: (err) => {
        console.log('调用失败', err);
    }
});
```

## 参数说明

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| phoneNumber | String | 是 | 需要拨打的电话号码 |

## 回调说明

| 回调名称 | 类型 | 说明 |
|----------|------|------|
| success | Function | 接口调用成功的回调 |
| fail | Function | 接口调用失败的回调（如号码格式错误、用户拒绝等） |

## 使用示例

### 基础用法

```javascript
function makePhoneCall(phone) {
    uni.makePhoneCall({
        phoneNumber: phone
    });
}

// 调用
makePhoneCall('13800138000');
```

### 带确认提示的拨打

```javascript
function handlePhone(phone) {
    uni.showModal({
        title: '提示',
        content: '确认拨打 ' + phone + ' 吗？',
        success: (res) => {
            if (res.confirm) {
                uni.makePhoneCall({
                    phoneNumber: phone
                });
            }
        }
    });
}
```

### 带错误处理的拨打

```javascript
function safeMakePhoneCall(phone) {
    if (!phone) {
        uni.showToast({
            title: '暂无联系电话',
            icon: 'none'
        });
        return;
    }

    uni.makePhoneCall({
        phoneNumber: phone,
        success: () => {
            console.log('拨打电话成功');
        },
        fail: (err) => {
            console.error('拨打电话失败', err);
            uni.showToast({
                title: '拨打失败，请手动拨打',
                icon: 'none'
            });
        }
    });
}
```

## 注意事项

1. **手机号格式**：建议传入标准的11位手机号，API会自动校验格式
2. **用户授权**：部分平台（如小程序）需要用户授权才能调用
3. **平台差异**：
   - App端直接调起系统拨号界面
   - H5端可能因浏览器差异行为不同
   - 微信小程序需要用户触发才能调用
4. **隐私合规**：拨打电话前建议先询问用户，确认其意愿

## 项目中的实际应用

### 1. 联系商家（订单详情页）

```javascript
// pages-shop/purchasingMall/order/orderDetail.vue
function handlePhone(phone) {
    uni.showModal({
        title: '提示',
        content: '确认拨打' + phone + '吗？',
        success: (res) => {
            if (res.confirm) {
                uni.makePhoneCall({
                    phoneNumber: phone
                });
            }
        }
    });
}
```

### 2. 联系客服

```javascript
// pagesD/serviceCenter/serviceCenter.vue
uni.makePhoneCall({
    phoneNumber: '400-xxx-xxxx'
});
```

### 3. 一键拨打（提交成功页）

```javascript
// pagesD/success/success.vue
function makePhoneCall(phone) {
    uni.makePhoneCall({
        phoneNumber: phone
    });
}
```

```html
<!-- 模板中使用 -->
<text class="color_main u-m-l-10" @click="makePhoneCall(phone)">一键拨打</text>
```

## 常见问题

### Q: makePhoneCall 和 tel: 链接有什么区别？

| 特性 | makePhoneCall | tel: 链接 |
|------|---------------|----------|
| 调用方式 | API调用 | HTML链接 |
| 错误处理 | 支持 fail 回调 | 无法捕获错误 |
| 兼容性 | uni-app跨平台 | 仅H5端 |
| 用户体验 | 统一的调用体验 | 因浏览器而异 |

### Q: 为什么拨打电话没有反应？

可能的原因：
1. 手机号格式不正确（非标准11位数字）
2. 在不支持的平台调用（如某些H5场景）
3. 用户拒绝授权
4. 没有在点击事件中调用（小程序要求）

### Q: 可以打开电话本让用户选择联系人吗？

`uni.makePhoneCall` 只负责拨打电话，不能打开电话本选择联系人。如果需要此功能，可以结合以下方案：
- 使用 `uni.addPhoneContact` 添加联系人
- 使用手机通讯录选择器第三方插件
- H5端可使用 `<a href="tel:号码">` 方式

## 最佳实践建议

```javascript
/**
 * 通用的拨打电话方法
 * @param {string} phone - 电话号码
 * @param {boolean} showConfirm - 是否显示确认弹窗
 */
function callPhone(phone, showConfirm = true) {
    if (!phone) {
        uni.showToast({ title: '暂无联系电话', icon: 'none' });
        return;
    }

    const doCall = () => {
        uni.makePhoneCall({
            phoneNumber: phone,
            fail: () => {
                uni.showToast({
                    title: '拨打失败，可手动拨打',
                    icon: 'none',
                    duration: 2000
                });
            }
        });
    };

    if (showConfirm) {
        uni.showModal({
            title: '拨打电话',
            content: `确定拨打 ${phone} 吗？`,
            success: (res) => {
                if (res.confirm) {
                    doCall();
                }
            }
        });
    } else {
        doCall();
    }
}
```

## 总结

`uni.makePhoneCall` 是移动端应用中非常实用的API，能够帮助用户快速联系商家或客服。开发时应注意：
1. 始终提供电话号码的显示和确认机制
2. 添加适当的错误处理
3. 考虑用户体验，避免未经确认就直接拨打电话
4. 注意号码格式的校验