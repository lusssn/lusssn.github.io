---
title: 微信小程序已知陷阱集锦
header-img: images/comm-header/experience.jpg
date: 2017-12-06 14:25:13
tags: 
  - 微信小程序
---
开发微信小程序主要依赖官方的开发文档，但是有很多陷阱得靠经验积累。到目前已经做了三个小程序了，本文记录开发中遇到的一些问题，持续更新。
更新时间：2017-12-22
<!-- more -->
___
### 一、组件
1.1. **scroll-view组件**高度设置为100%，不能触发上拉刷新和下拉加载事件。
1.2. 页面的第一个组件设置margin-top 和 最后一个组件的margin-bottom无效。
1.3. **textarea组件**
    * 文字会浮到顶层；
    * 不能设置tap；
    * IOS中placeholder不能超过两行；
    * IOS中文字会下一截
    * **textarea组件**上方不可以有高度会变化的区域，否则会导致textarea不能跟随页面一起滚动。
1.4. 获取 **input组件** 中的内容时，请同时绑定input事件和blur事件。
    * 若只绑定input事件，某些机型的中文输入法选中中文字的时候不会触发input事件，导致数据采集错误。
    * 若只绑定blur事件，则会因为***3.5***的问题导致数据采集错误。

### 二、API
2.1. 请小心使用**showNavigationBarLoading**，因为**hideNavigationBarLoading**有可能会无效（猜测是因为接口响应太快，hide的时候还没有show完毕）。
2.2. **onShareAppMessage**在IOS上正常，但在Android上有很多问题
    * 成功回调总会有shareTickets；
    * 打开聊天界面中的分享卡片，无论是给个人还是群，场景值定为1044；
    * app.js中onLoad、onShow的options里不一定携带 shareTicket。
2.3. A是一个tabbar页面，A的title通过**setNavigationBarTitle**动态设置。首次进入页面时title正常，跳转到别的页面再返回到A页，IOS中A页面的title就变成了A.json里的值。
2.4. 在一个tabbar页面中调用**switchTab**跳到当前页，不会触发onLoad、onShow。

### 三、事件
3.1. 事件的触发顺序
    * **单击（tap）** touchstart-->touchend-->tap
    * **长按（longtap）** touchstart-->longtap-->touchend-->tap  
        官方推荐使用**longpress**（1.5.0）代替
3.2. 页面第一次打开时，请注意**onLoad**中有异步事件，**onShow**中有期望同步处理的情况，因为执行**onLoad**后就会马上执行**onShow**。
3.3. 页面第一次打开时，开发工具中**onShow**只调用1次，真机上会调用2次。
3.4. A是tabBar的一页，B不是tabBar的一页，在A->B->A的过程中，B->A过程使用wx.switchTab(Object) 不会调用**onShow**，使用wx.navigateBack()会调用**onShow**。
3.5. 开发者工具中先触发**input组件**的blur再触发按钮的click，真机中则是先触发按钮的click再触发**input组件**的blur

### 四、文档没有的事
4.1. 为了体验效果更好，会在http请求数据前showToast，结束后hideToast，若接口响应很快会因为hide时候还没有show完毕，导致toast不能正常关闭，反而影响体验。所以是否需要**数据加载中的toast提示**，要根据接口响应速度酌情考虑。
4.2. 经过测试发现，**reLaunch**接口支持路由传参数，慎重使用。
4.3. 做页面刷新时，若是刷新普通页面使用**redirectTo**，若刷新tabbar页面需要使用**reLaunch**。
4.4. 模板传参方式不局限于文档中的方式，可以有；
```html
<!-- 若item是obj的一个属性，total是一个基本数据类型 -->
<template is="tplName" data="{{...obj, info: obj.item, total}}">
<template name="tplName">
	<!-- 1. ...obj像ES6的解构一样，可以直接使用其内部的属性 -->
	<view>{{item.name}}</view>
	<!-- 2. info: obj.item 指定一个参数名 -->
	<view>{{info.name}}</view>
	<!-- 3. 直接使用传入的参数做为参数名 -->
	<view>{{total}}</view>
</template>
```