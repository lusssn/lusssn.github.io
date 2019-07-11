---
title: 高德地图和D3js的结合（下）
headerimg: images/comm-header/experience.jpg
date: 2019-07-11 17:04:37
tags: JS插件
---
内容太多所以分成高德地图篇（上）、D3js篇（下）两个部分，本篇是对D3js使用的总结。
<!-- more -->
<!-- toc -->最近负责开发一个地图项目，用到了高德地图的SDK和D3js，记录一下探索过程和心得体会吧。

D3js是真的强，[API目录传送门](https://github.com/d3/d3/blob/master/API.md)。
在项目中只用到了：
- Selections
- Shapes
- Polygons
- Brushes

实战之后的体会分优缺点说说。

### 1. 优点

- 高效
最大的优点，我觉得是高效。
d3是基于svg做可视化的，这就是说需要直接操作svg节点，我们都知道直接操作页面节点很低效，但是d3却表现优秀。这也是我觉得d3神奇的地方。

- 交互开发友好
因为是基于svg节点的，所以另一大优点就是做起交互来很方便，想给哪个节点加事件就给哪个加。

- 灵活、高可控
还有一个优点是，d3提供的api都是最为基础的功能，并没有像Echarts、Highcharts这类图表库把交互、动画、主题等等打包好，只提供一部分内容可供配置修改，一切都可以DIY。

### 2. 缺点

- 高频功能也要DIY

- 跟React打不好配合

- 对比echarts做图表不太友好

【未完待续...】
