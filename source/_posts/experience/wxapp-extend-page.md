---
title: 微信小程序Page扩展
tags: 微信小程序
date: 2018-03-12 15:29:54
---
实际的项目开发中，页面的初始化几乎是统一的，在某些场景下还需要扩展小程序页面的生命周期钩子。
<!-- more -->

### 1. 场景和需求描述
- 页面加载时，首先调用`onLoad`，此时需要解码页面路由中的参数。
- 在`onLoad`中可能要请求页面的初始化数据，此时得同步获取access token后才能发起http请求。
- 从其他页面返回时，调用`onShow`，可能要刷新页面数据，此时得同步检查access token有效后才能发起http请求，
- 第一次进入页面时，onLoad和onShow均会被调用，要避免重复检查access token。
- 一些操作（例如埋点统计PV）需要在检查access token有效后，每次进入页面都被执行。经过上述改造后第一次进入页面时，onShow将不会被执行，因此新增`onAfterAuth`钩子。

### 2. 源码
```javascript
import wxUtil from '../utils/wxUtil'

const page = function (config) {
  config.global = config.global || {}
  Page({
    ...config,
    /**
     * 预处理onLoad
     * config中needReadyInOnLoad为'close'时，则关闭onLoad中的ready校验
     */
    onLoad (options) {
      // options 解码
      const urlParams = JSON.parse(JSON.stringify(options))
      for (const prop in urlParams) {
        if (urlParams.hasOwnProperty(prop)) {
          urlParams[prop] = decodeURIComponent(urlParams[prop])
        }
      }
      if (config.needReadyInOnLoad === 'close') {
        typeof config.onLoad === 'function' && config.onLoad.call(this, urlParams)
        return
      }
      // 在onLoad中调用ready的原因：
      // 1. ready方法会检验token是否有效，若失效则会发起请求获取新的token。页面若要发起请求加载数据，建议在页面加载时就校验token的有效性
      // 2. 因为ready的token校验是一个异步过程，所以要避免在onShow中出现二次调用ready的情况。否则会导致第一次的调用被服务器取消，出现错误。
      // 3. ready除了校验token外，还会从后台获取全局变量存在getApp().config中
      wxUtil.ready().then(() => {
        this.global._firstReadyCompleted = true
        // 使用后台返回的、存在getApp().config中全局变量
        // 发起网络请求
        typeof config.onAfterAuth === 'function' && config.onAfterAuth.call(this)
        typeof config.onLoad === 'function' && config.onLoad.call(this, urlParams)
      })
    },
    /**
     * 预处理onShow
     * config中needReadyInOnShow为'close'时，则关闭onShow中的ready校验
     */
    onShow () {
      if (config.needReadyInOnShow === 'close') {
        typeof config.onShow === 'function' && config.onShow.call(this)
        return
      }
      // 若有onShow中校验token的情况，要判断第一次ready是否完成
      if (!this.global._firstReadyCompleted && config.needReadyInOnLoad !== 'close') {
        this.global._firstReadyCompleted = true
        return
      }
      wxUtil.ready().then(() => {
        typeof config.onAfterAuth === 'function' && config.onAfterAuth.call(this)
        typeof config.onShow === 'function' && config.onShow.call(this)
      })
    }
  })
}

export default page

```

### 3. 其他说明
- wxUtil内容请看 [封装微信小程序API](/experience/ea8cef36.html)
