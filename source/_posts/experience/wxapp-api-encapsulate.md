---
title: 二次封装微信小程序API
header-img: images/comm-header/experience.jpg
date: 2018-01-10 18:30:13
tags: 
  - 微信小程序
---
微信小程序官方提供的API很多用起来都有点鸡肋，针对不同项目的业务需求做二次封装是有必要的，这篇文章记一些有通用性的封装，若读者觉得有不好的地方，欢迎指出。
<!-- more -->
* * *
## 目录
1. [路由跳转](#1. 路由跳转)
1. [刷新当前页](#2. 刷新当前页)
1. [消息提示](#3. 消息提示)
1. [发起网络请求](#4. 发起网络请求)
1. [图片预览](#5. 图片预览)

### 1. 路由跳转
> 官方提供了四种页面跳转的方式，接受参数类型是一个Json，使用起来复杂又麻烦。
> 建议合并四种方式，简化参数形式。

```javascript
// 需要手动配置tabbar页面列表
const TABBAR_PAGES = ['productList', 'me']
/**
 * Toast提示重载
 * @param pageName String 必填，页面文件名称
 * @param urlParams Json 页面参数
 * @param close Boolean/String 跳转方式，true：关闭当前页再跳转；'all'：关闭所有页面再跳转
 */
function navigateTo (pageName, urlParams, close) {
  const objParams = {}
  let isTabbarPage = TABBAR_PAGES.indexOf(pageName) !== -1
  let strParams = ''
  for (const key in urlParams) {
    if (!urlParams.hasOwnProperty(key)) {
      continue
    }
    // 对参数做编码，所以如果参数值包含中文，取用时要解码
    strParams += key + '=' + encodeURIComponent(urlParams[key]) + '&'
  }
  strParams = strParams.replace(/&$/, '')

  objParams.url = '/pages/' + pageName + '/' + pageName + (strParams ? '?' + strParams : '')

  // tabbar页面只能通过reLaunch实现关闭当前页
  if (close === 'all' || (isTabbarPage && close)) {
    wx.reLaunch(objParams)
  } else if (isTabbarPage) {
    wx.switchTab(objParams)
  } else if (close) {
    wx.redirectTo(objParams)
  } else {
    wx.navigateTo(objParams)
  }
}
```

#### 1.1 使用举例

假设productList是一个tabbar页面，detail是一个普通页面。
```javascript
// wx.switchTab 到tabbar页
wxUtil.navigateTo('productList') 
// wx.reLaunch 关掉所有页面到tabbar页
wxUtil.navigateTo('productList', '', true)
// wx.reLaunch 关掉所有页面到tabbar页
wxUtil.navigateTo('productList', '', 'all') 
// wx.navigateTo 到普通页
wxUtil.navigateTo('detail', {
  productId: 'sq12345'
}) 
// wx.redirectTo 关闭当前页到普通页
wxUtil.navigateTo('detail', {
  productId: 'sq12345'
}, true)
```

#### 1.2 特殊说明
- 需要**手动配置**tabbar页面列表。
- 方法会对**参数做编码**，所以如果参数值包含中文，取用时要解码。
- **tabbar页面**只能通过reLaunch实现关闭当前页。


### 2. 刷新当前页
> 如果程序缓存的token失效，或者遇到某些异常时，需要重新刷新加载页面。
> 此方法是对方法1的扩展。

```javascript
function refresh () {
  const page = getCurrentPages().pop()
  const route = (page.route.split('/'))[2]
  navigateTo(route, page.options, true)
}
```


### 3. 消息提示
> 官方提供了 **showToast** 和 **showLoading** 两个消息提示的API，使用频率很高，Json的参数类型就显得很麻烦了。

```javascript
/**
 * Toast提示重载
 * @param title String，必填
 * @param icon String，默认loading
 * @param duration Number，默认1500，单位ms
 */
function showToast (title, icon, duration) {
  const param = {
    icon: 'loading'
  }
  if (!title) {
    return
  }
  param.title = title
  if (duration && typeof duration === 'number') {
    param.duration = duration
  }
  if (icon) {
    param.icon = icon
  }
  wx.showToast(param)
}
```

#### 3.1 使用举例
```javascript
// 提示等待
wxUtil.showToast('加载中...', '', 20000) 
// 提示成功
wxUtil.showToast('操作成功', 'success')
// 提示失败
wxUtil.showToast('手机号必填')
```


### 4. 发起网络请求
> 实际业务中需要对全部或绝大部分网络请求做统一的错误处理和数据预处理。

```javascript
// 自定义请求头内容
const headers = {
  'content-type': 'application/x-www-form-urlencoded'
}
/**
 * request重载
 * @param params Json
 * @param success function
 * @param error function
 */
function ajax (params, success, error) {
  const config = getApp().config
  const curTime = (new Date().getTime()) / 1000
  // 1. 当前config里有app_token且过期了，刷新页面
  if (config.APP_TOKEN && curTime >= config.TIME_EXPIRE) {
    showToast('登录失效，正在刷新页面')
    delete config.APP_TOKEN
    setTimeout(() => {
      refresh()
    }, 1000)
    return
  }
  wx.request({
    method: 'POST', // 默认post方式
    dataType: 'json',
    header: headers,
    success (res) {
      if (!res || res.statusCode !== 200 || !res.data) {
        error && error()
        return
      }
      // res.data为接口返回的内容，假定为 {errno: xxx, msg: xxx, data: xxx} 的结构
      if (!res.data.errno) {
        success && success(res.data.data)
        return
      }
      error && error(res.data.errno, res.data.msg)
    },
    fail () {
      error && error()
    },
    ...params
  })
}
```

#### 4.1 使用举例

假设接口返回的内容格式为：
```json
{
  "errno": "xxx",
  "msg": "xxx",
  "data": "xxx"
}
```

```javascript
wxUtil.ajax({
  url: 'xxx',
  data: {
    key: 'xxx'
  }
}, (res) => {
  // res是接口返回的内容
}, (errno, msg) => {
  // errno是接口返回的错误号
  // msg是接口返回的错误信息
})
```

#### 4.2 特殊说明
- 在实际项目中，方法应该根据接口的Header信息、出参格式稍作调整。

### 5. 图片预览
> 官方提供的图片预览API，接受的参数同样是Json类型，回调方法一般使用不到。可以二次封装做一些简化。
```javascript
/**
 * 预览图片重载
 * @param index Number 当前图片下标
 * @param urls Array 预览的图片url数组
 */
function previewImage (index, urls) {
  if (!Array.isArray(urls) || index < 0 || index > urls.length - 1) {
    return
  }
  wx.previewImage({
    current: urls[index],
    urls: urls
  })
}
```

#### 5.1 使用举例
```javascript
Page({
  data: {
    imgList: ['1.jpg', '2.jpg', '3.jpg']
  },
  bindPreviewImg (e) {
    const self = this
    const index = e.currentTarget.dataset.index
    wxUtil.previewImage(index, self.data.imgList)
  }
})
```