---
title: 高德地图和D3js的结合（上）
headerimg: images/comm-header/experience.jpg
date: 2019-05-24 16:42:35
tags:
---
最近负责开发一个地图项目，用到了高德地图的SDK和D3js，记录一下探索过程和心得体会吧。
<!-- more -->
<!-- toc -->

内容太多所以分成高德地图篇（上）、D3js篇（下）两个部分。本篇是对高德地图使用的总结。

高德地图算是一款国内比较成熟的地图SDK了吧。
完成这次项目用到了：
- [地图生命周期和状态](https://lbs.amap.com/api/javascript-api/reference/map)
- [基础类](https://lbs.amap.com/api/javascript-api/reference/core)：Pixel、LngLat、Bounds
- [覆盖物](https://lbs.amap.com/api/javascript-api/reference/overlay)：Marker、Polygon、Rectangle、OverlayGroup
- [信息窗体](https://lbs.amap.com/api/javascript-api/reference/infowindow)：InfoWindow
- [图层](https://lbs.amap.com/api/javascript-api/reference/layer)：MassMarks、DistrictLayer
- [自建图层](https://lbs.amap.com/api/javascript-api/reference/self-own-layers)：CustomLayer
- [搜索服务](https://lbs.amap.com/api/javascript-api/reference/search)：Autocomplete、DistrictSearch
- [地图控件](https://lbs.amap.com/api/javascript-api/reference/map-control)：Scale、ToolBar

通过看官方文档就能上手的部分就不啰嗦了。这里面比较坑的是图层、自建图层和搜索服务。
* * *
### 1. 麻点图层 MassMarks
> 需求说明
> ☝🏼 将3000多个数据，以图标的方式渲染到地图上
> ✌🏼 图标的颜色和形状取决于接口返回的字段
> 👌🏼 图标的尺寸根据地图当前的缩放级别动态调整

因为高德地图已经有高效渲染海量点数据的API，直接用就是了。
需要自己稍微处理一下的部分就是图标的形状、颜色和尺寸。

#### 1.1 图标尺寸
设置尺寸很容易，根据缩放级别设置相应的图标尺寸，修改配置项中的`style > size`即可（size支持数组或者`AMap.Size`类）。
```javascript
const MASS_STYLE_MAPPING = {
  9: [4, 4], // 20km
  10: [6, 6], // 10km
  11: [10, 10], // 5km
  12: [10, 10], // 2km
  13: [20, 20], // 1km
  14: [20, 20], // 500m
  15: [22, 22], // 200m
  16: [22, 22], // 200m
  17: [26, 26], // 100m
  18: [26, 26], // 50m
}
```

#### 1.2 图标形状
设置图标形状对应配置`style > url`。
文档中只说url是图标地址，string类型，看示例用的是网络图片地址。
经过测试摸索后发现，**是支持svg字符串的**，为了方便配置图标颜色，svg当然是比图片灵活许多。
最后的实现方案：
  1. 接口返回的数据包含该点的图标名称
  2. 前端静态维护名称-path的映射
  3. 渲染图标的时候，根据图标名称找到对应的path，动态生成svg字符串
  
贴一下前端动态生成svg字符串的代码：
```javascript
const SVG_MAPPING = {
  user: {
    viewBox: '0 0 20 20',
    path: 'M19 19h-18v-1c0-5 4-9 9-9s9 4 9 9v1zM3.1 17h13.9c-0.5-3.4-3.4-6-6.9-6s-6.5 2.6-7 6z M10 11c-2.8 0-5-2.2-5-5s2.2-5 5-5 5 2.2 5 5-2.2 5-5 5zM10 3c-1.7 0-3 1.3-3 3s1.3 3 3 3 3-1.3 3-3-1.3-3-3-3z'
  }
}
/**
* 动态生成svg字符串
* @param iconName 图标名称
* @param color 图标颜色
* @param bgType 如需添加背景色，传入rect或者circle
* @returns {string}
*/
const getSvg = (iconName, color, bgType) => {
  const svg = SVG_MAPPING[iconName]
  if (!svg) {
    return ''
  }
  let bg = ''
  if (bgType === 'rect') {
    const box = svg.viewBox.split(' ')
    bg = `%3Cpath fill='%23fff' d='M0 0h${box[2]}v${box[3]}H0z'/%3E`
  } else if (bgType === 'circle') {
    const r = svg.viewBox.split(' ')[2] / 2
    bg = `%3Ccircle fill='%23fff' cx='${r}' cy='${r}' r='${r}'/%3E`
  }
  return `data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='${svg.viewBox}'%3E${bg}%3Cpath fill='${color}' d='${svg.path}'/%3E%3C/svg%3E`
}
```

#### 1.3 图标颜色
图标是什么颜色是接口返回的，在动态生成svg字符串的时候，同时传入颜色字段就可以了。
* * *
### 2. 行政区图层 DistrictLayer
> 需求说明
> ☝🏼 页面可以进行城市切换
> ✌🏼 地图上显示当前城市地图，并绘制行政区域边界线

![](shanghai.png)

#### 2.1 覆盖物
最开始的方案，是先用DistrictSearch根据城市名称获得城市的adcode，再用城市adcode和level去查询下面的行政区边界，最后绘制成`AMap.Polygon`类。
**优点：**可以自定义边界线的粗细程度。
**缺点：**耗费时间，大概要600ms。

#### 2.2 图层
后来发现高德2018-09-12就发布了简易行政区图层插件。
这个插件用起来很简单，重点是特别快，大概200ms完成绘制。
但是它真的太简单，只支持到大部分市级的行政区边界，太仓、辛集都没有，重庆绘制出来不包括郊县。
**优点：**绘制高效，调用简单。
**缺点：**覆盖的地区不全，不能调整边界线的粗细程度。

#### 2.3 覆盖物结合图层
综合上面两种方式的优缺点，最后采用二合一方式，贴一个伪代码：
```javascript
const map = new AMap.Map('amap')
const districtLayer = new AMap.DistrictLayer.Province({
  styles: {
    fill: 'rgba(204,243,255,0.3)',
    'county-stroke': '#CC66CC', // 中国区县边界
  }
})
const drawDistrictBounds = cityName => {
  // 用DistrictSearch根据城市名称获得城市的adcode、level。实现见本文4.1
  const city = getAMapCity(cityName)
  if (city.level === 'district') {
    // 用DistrictSearch查询行政区边界，绘制Polygon。实现见本文4.2
    drawDistrictBoundsByCustom(city.adcode, city.level)
    return
  }
  // 处理重庆郊县
  if (city.adcode === '500100') {
    districtLayer.setDistricts([city.adcode, 500200])
  } else {
    districtLayer.setDistricts([city.adcode])
  }
  map.add(districtLayer)
}
```
* * *
### 3. 自建图层 CustomLayer
> 需求说明
> 自定义图层内容
> 图层状态可以同步地图的缩放、平移、尺寸变化状态

高德的自定义图层功能必须给点个赞，这个功能给地图产品带来更多的可能。除了CustomLayer，还有TileLayer、ImageLayer、CanvasLayer、VideoLayer。不是很明白细化这些图层的原因是什么，感觉有个万能的CustomLayer就够了。

使用起来也简单：
1. 加载CustomLayer插件
1. 创建CustomLayer实例，初始化render方法，此方法会在地图状态变化时被调用
1. 添加自定义图层到地图实例

需要注意的是，render方法会有被频繁调用可能，最好是做一个**防抖判断**。
* * *
### 4. 搜索服务 DistrictSearch

坑点有二
⚠️ 高德的行政级别划分跟我们的认知不一致。举个例子，重庆市我认为是市级别，但是高德是省级；太仓市我认为是市级别，实际又是区级。
⚠️ 个别省市的行政区划分很特殊，目前发现的是重庆和苏州，重庆由重庆郊县和主城区组成，苏州由苏州工业园区和主城区组成。

封装了两个搜索的方法，自我感觉并不是很满意，再慢慢优化吧。
下面的代码引入了ramda工具库，缩写为R。

#### 4.1 搜索城市信息
- searcher：DistrictSearch实例
- keyword：关键字（城市名称或adcode）
- level：搜索级别

```javascript
const getAMapCity = (searcher, keyword, level = 'city') => {
  return new Promise((resolve, reject) => {
    // base-不返回行政区边界坐标点
    searcher.setExtensions('base')
    // 0-不返回下级行政区
    searcher.setSubdistrict(0)
    searcher.search(keyword, (status, result) => {
      if (status !== 'complete') {
        reject(result)
        return
      }
      const district = R.find(R.propEq('level', level))(result.districtList) || result.districtList[0]
      if (!district) {
        reject(result)
        return
      }
      resolve(district)
    })
  })
}
```

#### 4.2 搜索区域边界线
- searcher：DistrictSearch实例
- others：更多配置
  + keyword：关键字（城市名称或adcode）
  + level：搜索级别
  + subDistrict：是否获取下一级的行政区边界 1-返回，0-不返回
  
```javascript
const districtSearch = (searcher, others = {}) => {
  return new Promise((resolve, reject) => {
    const { keyword, level, subDistrict = 1 } = others
    // all-返回行政区边界坐标点
    searcher.setExtensions('all')
    // 1-返回下一级行政区
    searcher.setSubdistrict(subDistrict)
    searcher.search(keyword, async (status, result) => {
      if (status !== 'complete') {
        reject(result)
        return
      }
      const district = R.find(R.propEq('level', level))(result.districtList)
      if (!district) {
        reject(result)
        return
      }
      const { boundaries, districtList = [] } = district
      // 如果是城市级别，进一步获取下一级行政区
      if (level === 'city' && !R.isEmpty(districtList)) {
        const next = districtList.map(item => {
          return districtSearch(searcher, { keyword: item.adcode, level: item.level })
        })
        const group = await Promise.all(next)
        // 如果是重庆市，处理重庆郊县
        if (district.adcode === '500100') {
          const { districtList: districtJiao } = R.find(R.propEq('adcode', '500200'))(result.districtList)
          const nextJiao = districtJiao.map(item => {
            return districtSearch(searcher, { keyword: item.adcode, level: item.level })
          })
          Promise.all(nextJiao).then(jiaoGroup => {
            resolve(group.concat([jiaoGroup]))
          })
          return
        }
        // 如果是苏州市，处理苏州工业园区
        if (district.adcode === '320500') {
          const districtPark = R.find(R.propEq('adcode', '320571'))(result.districtList)
          const industrialPark = districtPark.boundaries.map(item => {
            return new AMap.Polygon({ path: item })
          })
          resolve(group.concat([industrialPark]))
          return
        }
        resolve(group)
        return
      }
      // 生成行政区划polygon
      resolve(boundaries.map(item => (new AMap.Polygon({ path: item }))))
    })
  })
}
```
* * *
### 5. 杂七杂八的记录
有些琐碎的、我觉得挺坑的点，也顺便记录在这里。
- 🙄**OverlayGroup的作用很鸡肋**
我还以为`AMap.OverlayGroup`是批量绘制覆盖物的对象，实际测试下来并不是，我估计它的底层只是代替开发者做了一个循环，因为对性能没有一点儿帮助。

- 🙄**获取Bounds中心点存在问题**
创建一个`AMap.Rectangle`时需要传入矩形的bounds参数，文档是说用东北-西南角坐标。亲测下来，这样会在创建完矩形后，获取bounds中心点时，经度为负数。
解决办法就是使用西北-东南角坐标。

- 😡**moveend事件触发的时机和文档不符**
给地图绑定事件的时候，意外发现ToolBar控制、键盘控制地图缩放时，moveend事件没有被触发，但是鼠标滚轮控制缩放时就会触发moveend事件，真的是奇怪。
解决办法是全局添加一个标记值isZoomEnd，绑定zoomend事件，触发时置isZoomEnd为true，在moveend回调中阻止重复处理，并改isZoomEnd为false。

- 😤**clearMap接口效率奇低**
清除地图上所有覆盖物的API `clearMap`，能把人逼疯。我们的业务场景下地图上绘制了成千上万的覆盖物，切换城市时这些覆盖物需要一次性清除掉，绘制新的。8000个覆盖物要耗费十几秒！实在恼火，且无能为力。
* * *
高德只支持了点数据的海量绘制，需要海量绘制覆盖物的时候，性能就下滑得难以让人接受了，不得不另寻出路。
借助CustomLayer，就可以自定义覆盖物的绘制方式，canvas也好，svg也好。

下面一篇文就继续总结高德地图配合D3js的开发经验。