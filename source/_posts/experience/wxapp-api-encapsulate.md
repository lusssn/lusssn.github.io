---
title: 封装微信小程序API
tags: 微信小程序
date: 2018-01-10 18:30:13
---
微信小程序官方提供的API很多用起来都有点鸡肋，针对不同项目的业务需求做二次封装是有必要的，这篇文章记一些有通用性的封装，若读者觉得有不好的地方，欢迎指出。
<!-- more -->
<!-- toc -->
* * *
### ChangeLog

- 2019-03-19 更新异步aip转为Promise风格、消息提示、页面信息传递

### 1. 异步api转为Promise风格

> 小程序中的异步api，风格都是统一为成功接受success参数作为回调，失败接受fail作为参数回调。比起Promise风格来说，感觉后者对开发者来说更加友好。

{% spoiler 异步api转为Promise风格 %}
```javascript
/**
 * 将大多数微信接口转换成promise形式
 * 配置项中的success和fail回调会失效
 * complete回调用finally处理
 * @param fn
 * @returns {Function}
 */
export const promisifyWxApi = fn => {
  return args => {
    return new Promise((resolve, reject) => {
      fn({
        ...args,
        success: res => resolve(res),
        fail: res => reject(res),
      })
    })
  }
}
```
{% endspoiler %}

#### 1.1 使用举例

```javascript
promisifyWxApi(wx.chooseImage)({
  count: 1,
}).then(res => {
  console.log(res.tempFilePaths.pop())
}, () => {
  console.error('选择图片失败')
})
```

### 2. 路由跳转

> 官方提供了四种页面跳转的方式，接受参数类型是一个Json，使用起来复杂又麻烦。
> 建议合并四种方式，简化参数形式。

{% spoiler 路由跳转 %}
```javascript
// 需要手动配置tabbar页面列表
const TABBAR_PAGES = ['productList', 'me']
/**
 * 路由跳转重载
 * @param pageName String 必填，页面文件名称
 * @param urlParams Json 页面参数
 * @param close Boolean/String 跳转方式，true：关闭当前页再跳转；'all'：关闭所有页面再跳转
 */
const navigateTo = (pageName, urlParams, close) => {
  const objParams = {}
  let isTabbarPage = TABBAR_PAGES.indexOf(pageName) !== -1
  let strParams = ''
  for (const key in urlParams) {
    if (!urlParams.hasOwnProperty(key)) {
      continue
    }
    // 对参数做编码，所以如果参数值包含中文，取用时要解码
    strParams += `${key}=${encodeURIComponent(urlParams[key])}&`
  }
  strParams = strParams.replace(/&$/, '')

  objParams.url = `/pages/${pageName}/${pageName}${strParams ? `?${strParams}` : ''}`

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
{% endspoiler %}

#### 2.1 使用举例

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

#### 2.2 特殊说明

- 需要**手动配置**tabbar页面列表。
- 方法会对**参数做编码**，所以如果参数值包含中文，取用时要解码。
- **tabbar页面**只能通过reLaunch实现关闭当前页。


### 3. 刷新当前页

> 如果程序缓存的token失效，或者遇到某些异常时，需要重新刷新加载页面。
> 此方法是对方法1的扩展。

```javascript
function refresh () {
  const page = getCurrentPages().pop()
  const route = (page.route.split('/'))[2]
  navigateTo(route, page.options, true)
}
```


### 4. 消息提示

> 官方提供了 **showToast** 和 **showLoading** 两个消息提示的API，使用频率很高，Json的参数类型就显得很麻烦了。

{% spoiler Toast提示重载 %}
```javascript
/**
 * Toast提示重载
 * @param title String，必填
 * @param icon String，默认warning
 * @param others Object，其他配置
 */
const showToast = (title = '', icon = 'warning', others) => {
  const params = {
    title,
    icon,
    mask: true,
    duration: 3000,
    ...others,
  }
  if (icon === 'warning') {
    params.image = '../../images/warning-o.png'
  }
  return promisifyWxApi(wx.showToast)(params)
}
```
{% endspoiler %}

#### 4.1 使用举例
```javascript
// 提示等待
wxUtil.showToast('加载中...', 'loading', { ducation: 2000 }) 
// 提示成功
wxUtil.showToast('操作成功', 'success')
// 提示失败
wxUtil.showToast('手机号必填')
```


### 5. 发起网络请求
> 实际业务中需要对全部或绝大部分网络请求做统一的错误处理和数据预处理。

{% spoiler request重载 %}
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
{% endspoiler %}

#### 5.1 使用举例

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

#### 5.2 特殊说明

- 在实际项目中，方法应该根据接口的Header信息、出参格式稍作调整。

### 6. 图片预览

> 官方提供的图片预览API，接受的参数同样是Json类型，回调方法一般使用不到。可以二次封装做一些简化。

{% spoiler 预览图片重载 %}
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
{% endspoiler %}

#### 6.1 使用举例

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

### 7. 页面信息传递

> 很多业务场景都需要在页面间传递状态，传递方式要满足使用简单，状态值易于管理的要求。

{% spoiler 页面信息传递 %}
```javascript
const NOTICE = {
  edit: Symbol('edit')
}
APP({
  /**
   * 设置全局的消息变量
   * @param key
   * @param value 若为undefined，则删除对应键值
   */
  setNotice(key, value) {
    const noticeKey = NOTICE[key]
    if (!noticeKey) {
      return
    }
    if (value === undefined) {
      delete this.global[noticeKey]
      return
    }
    this.setConfig({ [noticeKey]: value })
  },
  /**
   * 根据键值获取对应消息值
   * @param key
   * @returns {*}
   */
  getNotice(key) {
    const noticeKey = NOTICE[key]
    if (!noticeKey) {
      return ''
    }
    return this.global[noticeKey]
  },
  /**
   * 检查键值对应消息值，符合条件则callback并删除对应键值
   * @param key
   * @param value
   * @param callback
   */
  checkNotice(key, value, callback) {
    if (this.getNotice(key) === value) {
      this.setNotice(key)
      typeof callback === 'function' && callback()
    }
  },
})
```
{% endspoiler %}

#### 7.1 使用举例

场景：希望在编辑页提交数据之后，回到详情页，通知详情页刷新页面数据。

```javascript
// in edit.js
const APP = getApp()
Page({
  submit () {
    APP.setNotice('edit', true)
  }
})
```
```javascript
// in detail.js
const APP = getApp()
Page({
  onShow () {
    APP.checkNotice('edit', true, () => {
      console.log('Page edit has been changed')
    })
  }
})
```

#### 7.2 特殊说明

- setNotice (key, value)
  + 设置全局的消息变量
  + 若value值缺省，则会**删除**对应键值
- getNotice (key)
  + 根据键值获取对应消息值
- checkNotice (key, value, callback)
  + 检查键值对应消息值，符合条件则callback并**删除**对应键值
- 3个方法为了全局内可调用，**写在app.js里**。
- **key应该放在变量NOTICE中**统一管理，借助Symbol做唯一性处理。
