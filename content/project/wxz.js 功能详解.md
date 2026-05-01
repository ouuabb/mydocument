# wxz.js 功能详解

## 简介

`wxz.js` 是项目中的一个核心工具文件，主要提供了以下几类功能：

- **定位服务**：获取用户位置、权限管理、地理编码等
- **地图服务**：打开地图选择位置、周边搜索、地址搜索等
- **行政区划**：获取省市区数据、根据关键词获取行政规划
- **工具函数**：数字转中文、图片主题色提取、UTF-8 编码、Base64 编码等

## 核心功能模块

### 1. 定位服务

#### 1.1 权限检查

```javascript
export const checkLocation = () => {
    return new Promise((r, j) => {
        // #ifdef MP-WEIXIN
        uni.getSetting({...});
        // #endif
        // #ifdef H5 || APP
        return r(1);
        // #endif
    });
};
```

**功能**：检查小程序位置权限并引导授权

**特点**：
- 支持多端（小程序、H5、APP）
- 对小程序做了特殊处理，引导用户授权
- 返回 Promise，便于异步调用

#### 1.2 获取位置信息

```javascript
export const getLocation = async () => {
    return new Promise((resolve, reject) => {
        uni.getLocation({
            isHighAccuracy: true,
            success: (res) => {
                resolve(res);
            },
            fail: (err) => {
                reject(err);
            }
        });
    });
};
```

**功能**：获取用户当前位置的经纬度信息

**参数**：无
**返回**：Promise，包含经纬度等位置信息

#### 1.3 逆地理编码

```javascript
export const getLocationName = (longitude, latitude) => {
    return new Promise((resolve, reject) => {
        uni.request({
            url: `https://restapi.amap.com/v3/geocode/regeo?output=json&location=${longitude},${latitude}&key=${gdkey}`,
            method: 'GET',
            success: (data) => {
                resolve({
                    ...data.data,
                    deptId: '',
                    latitude: latitude,
                    longitude: longitude
                });
            },
            fail: (err) => {
                reject(err);
            }
        });
    });
};
```

**功能**：根据经纬度获取详细地址信息

**参数**：
- `longitude`：经度
- `latitude`：纬度

**返回**：Promise，包含详细地址信息

### 2. 地图服务

#### 2.1 打开地图选择位置

```javascript
export let openLocation = async (callback) => {
    uni.showLoading({ title: '地图打开中' });
    uni.getLocation({
        type: 'gcj02',
        geocode: true,
        success: (res) => {
            uni.chooseLocation({
                latitude: parseFloat(res.latitude),
                longitude: parseFloat(res.longitude),
                geocode: true,
                success: (_res) => {
                    // 获取地址信息并回调
                }
            });
        }
    });
};
```

**功能**：打开地图让用户选择位置，并获取详细地址信息

**参数**：
- `callback`：回调函数，接收选择的位置信息

#### 2.2 周边搜索

```javascript
export let amapAround = (location, callback, params) => {
    let url = `https://restapi.amap.com/v3/place/around?location=${location}&radius=500&output=json&key=${gdkey}&datatype=base&offset=20&page=1&types=120000`;
    if (params) {
        url += '&' + uni.$u.queryParams(params, false);
    }
    uni.request({
        url: url,
        method: 'GET',
        complete: ({ data }) => {
            if (data.status == 1) {
                callback(data);
            }
        }
    });
};
```

**功能**：根据位置搜索周边 POI（兴趣点）

**参数**：
- `location`：中心点经纬度，格式为 "经度,纬度"
- `callback`：回调函数，接收搜索结果
- `params`：可选参数，用于扩展搜索条件

#### 2.3 地址搜索

```javascript
export let amapSearch = (keywords, callback, params) => {
    let url = `https://restapi.amap.com/v3/assistant/inputtips?output=json&keywords=${keywords}&key=${gdkey}&datatype=poi&types=120000`;
    if (params) {
        url += '&' + uni.$u.queryParams(params, false);
    }
    uni.request({
        url: url,
        method: 'GET',
        complete: ({ data }) => {
            if (data.status == 1) {
                callback(data);
            }
        }
    });
};
```

**功能**：根据关键词搜索地址

**参数**：
- `keywords`：搜索关键词
- `callback`：回调函数，接收搜索结果
- `params`：可选参数，用于扩展搜索条件

### 3. 行政区划

#### 3.1 获取省市区数据

```javascript
export const getDistricts = () => {
    return new Promise((resolve, reject) => {
        uni.request({
            method: 'get',
            url: `https://restapi.amap.com/v3/config/district?subdistrict=2&key=${gdkey}`,
            success: (res) => {
                if (res.statusCode === 200) {
                    resolve(res.data.districts[0].districts);
                } else {
                    reject(res);
                }
            },
            fail: (res) => {
                reject(res);
            }
        });
    });
};
```

**功能**：获取全国省市区数据

**返回**：Promise，包含省市区数据

#### 3.2 根据关键词获取行政规划

```javascript
export const getDistrict = (keywords, adcode) => {
    return new Promise((resolve, reject) => {
        uni.request({
            url: `https://restapi.amap.com/v3/config/district?output=json&keywords=${keywords}&key=${gdkey}&filter=${adcode.slice(0, 2) + '0000'}`,
            method: 'GET',
            success: (data) => {
                if (data.data.infocode == '10000') {
                    resolve(data.data.districts);
                }
            },
            fail: (err) => {
                reject(err);
            }
        });
    });
};
```

**功能**：根据关键词获取行政规划信息

**参数**：
- `keywords`：搜索关键词
- `adcode`：行政编码

**返回**：Promise，包含行政规划数据

### 4. 工具函数

#### 4.1 数字转中文

```javascript
export const toChinesNum = (num) => {
    let changeNum = ['零', '一', '二', '三', '四', '五', '六', '七', '八', '九'];
    let unit = ['', '十', '百', '千', '万'];
    // 实现逻辑...
    return overWan ? getWan(overWan) + '万' + getWan(noWan) : getWan(num);
};
```

**功能**：将数字转换为中文表示

**参数**：
- `num`：需要转换的数字

**返回**：转换后的中文字符串

#### 4.2 获取图片主题颜色

```javascript
export const getImageThemeColor = (that, path, canvasId, success = () => {}, fail = () => {}) => {
    // 实现逻辑...
};

function complementaryColor(r, g, b) {
    const avg = 255;
    const complementaryR = avg - r;
    const complementaryG = avg - g;
    const complementaryB = avg - b;
    return [Math.round(Math.abs(complementaryR)), Math.round(Math.abs(complementaryG)), Math.round(Math.abs(complementaryB))];
}
```

**功能**：获取图片的主题颜色和互补色

**参数**：
- `that`：Vue 实例
- `path`：图片路径
- `canvasId`：画布 ID
- `success`：成功回调，返回主题色和互补色
- `fail`：失败回调

#### 4.3 UTF-8 编码

```javascript
export const utf8Encode = (argString) => {
    // 实现逻辑...
    return utftext;
};
```

**功能**：将字符串转换为 UTF-8 编码

**参数**：
- `argString`：需要编码的字符串

**返回**：UTF-8 编码后的字符串

#### 4.4 Base64 编码

```javascript
export const base64Encode = (data) => {
    var b64 = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=';
    // 实现逻辑...
    return enc;
};
```

**功能**：将字符串转换为 Base64 编码

**参数**：
- `data`：需要编码的字符串

**返回**：Base64 编码后的字符串

## 技术特点

1. **多端支持**：通过条件编译（`#ifdef`）实现了对小程序、H5、APP 的适配
2. **Promise 封装**：所有异步操作都使用 Promise 封装，便于使用 async/await
3. **高德地图集成**：使用高德地图 API 实现地理编码、逆地理编码、周边搜索等功能
4. **工具函数丰富**：提供了多种实用的工具函数，如数字转中文、图片主题色提取等
5. **参数灵活**：支持可选参数扩展，如 `amapAround` 和 `amapSearch` 函数

## 使用示例

### 1. 获取当前位置并转换为详细地址

```javascript
import { checkLocation, getLocation, getLocationName } from '@/api/wxz.js';

async function getCurrentLocation() {
    try {
        // 检查权限
        await checkLocation();
        
        // 获取经纬度
        const location = await getLocation();
        
        // 获取详细地址
        const address = await getLocationName(location.longitude, location.latitude);
        
        console.log('当前位置：', address);
        return address;
    } catch (error) {
        console.error('获取位置失败：', error);
    }
}
```

### 2. 打开地图选择位置

```javascript
import { openLocation } from '@/api/wxz.js';

function chooseLocation() {
    openLocation((locationInfo) => {
        console.log('选择的位置：', locationInfo);
        // 处理选择的位置信息
    });
}
```

### 3. 搜索周边位置

```javascript
import { amapAround } from '@/api/wxz.js';

function searchNearby() {
    const location = '108.940175,34.265182'; // 中心点经纬度
    
    amapAround(location, (result) => {
        console.log('周边搜索结果：', result);
        // 处理搜索结果
    });
}
```

### 4. 获取省市区数据

```javascript
import { getDistricts } from '@/api/wxz.js';

async function loadDistricts() {
    try {
        const districts = await getDistricts();
        console.log('省市区数据：', districts);
        return districts;
    } catch (error) {
        console.error('获取省市区数据失败：', error);
    }
}
```

## 注意事项

1. **高德地图 API Key**：文件中使用了固定的高德地图 API Key（`gdkey`），如果需要在生产环境使用，建议替换为自己的 API Key

2. **权限管理**：
   - 小程序端需要用户授权位置权限
   - H5 和 APP 端默认返回授权成功

3. **异步操作**：所有定位和地图相关操作都是异步的，需要使用 Promise 或 async/await 处理

4. **错误处理**：建议在使用时添加适当的错误处理，特别是网络请求失败的情况

5. **性能优化**：
   - 对于频繁的位置请求，建议添加缓存机制
   - 图片主题色提取可能会消耗较多性能，建议在适当的时机使用

6. **兼容性**：
   - 部分功能可能在不同端的表现略有差异
   - 建议在不同端进行测试，确保功能正常

## 总结

`wxz.js` 是一个功能丰富的工具文件，提供了完整的定位、地图和工具函数功能。通过集成高德地图 API，实现了从位置获取、地址解析到周边搜索的全流程功能，同时还提供了一些实用的工具函数。

该文件的设计考虑了多端兼容性，使用了 Promise 封装异步操作，API 设计简洁易用，是项目中处理地理位置相关功能的核心模块。