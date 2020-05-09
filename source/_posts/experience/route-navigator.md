---
title: 路由导航组件的实现
date: 2018-07-19 17:19:40
tags: 实用Demo
---
PC端的WEB项目中常常会需要一个路由导航的组件，可以显示当前路由的父级，点击可以跳转至相应页面。
<!-- more -->
最终的实现方案看起来其实并不简单和优雅。
写在这里也希望同行能给点优化意见，今后做类似需求也可以有参考。

### 需求分析

1. 打开一个页面，显示当前页面的父级；
2. 点击非当前页的层级，跳转到相应页面；
3. 处理某些页面带有参数的情况；

  举个🌰 层级关系A -> B -> C
  - 打开A页面，显示`A`；
  - 打开B页面，显示`A / B`，其中A可点击；
  - 打开C页面，显示`A / B / C`，其中A、B可以点击。

### 关键点

1. 根据路由获得其层级关系；
2. 页面参数传递。

通过一个**配置文件**，记录路由的层级关系。配置内容包括：
显示名称（name）、跳转路由（path）、页面参数（query）、下一级（children）。

```json
[
  {
    name: '物料活动',
    path: '/activity/list',
    children: [
      {
        name: '活动详情',
        path: '/activity/detail',
        query: ['activity'],
        children: [
          {
            name: '门店活动详情',
            path: '/activity/shopDetail',
          },
        ],
      },
    ],
  },
  {
    name: '进行中的物料申请',
    path: '/apply/active/list',
    children: [
      {
        name: '详情',
        path: '/apply/active/detail',
      },
      {
        name: '填写',
        path: '/apply/active/edit',
      },
    ],
  },
  {
    name: '历史物料申请',
    path: '/apply/history',
  },
]
```

再按照深度优先原则遍历配置，进行路由匹配，最后得到一个记录层级关系的数组。

### 关键代码

```js
_getGuideFromConfig = (pathName, config, guides = []) => {
  for (let item of config) {
    let path = item.path
    // 命中压入配置，退出遍历
    if (
      pathName === path ||
      (Array.isArray(path) && path.includes(pathName))
    ) {
      guides.unshift({
        name: item.name,
        path: this._getPathWithQuery(item),
      })
      return guides
    }
    // 有下一层则继续遍历
    let children = item.children
    if (children) {
      this._getGuideFromConfig(pathName, children, guides)
    }
    // 如果guides长度不为0，认为已经找到相应配置，则从头部压入配置，退出遍历
    if (guides.length) {
      guides.unshift({
        name: item.name,
        path: this._getPathWithQuery(item),
      })
      return guides
    }
  }
  return guides
};

// 根据配置内容拼接pathname和query
_getPathWithQuery = config => {
  let { query = [], path } = config
  return query.reduce(
    (resPath, key, index) =>
      `${resPath}${index ? '&' : '?'}${key}=${this.query[key]}`,
    path
  )
};
```

### 缺陷

最大的一个缺陷就是**下一层路由必须携带父层级的所有参数**，
导致子页面的参数会越来越多，单独看哪些与当前页面无关的变量有点莫名其妙。
所以建议父子层级关系最好不要超过 3 层。

另一个缺陷是**一个路由只能有一种父层级关系**，即不能同时有`A / B / C`和`A / BB / C`。
原因很简单，因为是用路由匹配层级关系的，当C路由满足`A / B / C`层级后，遍历也就结束了。

### 结束

👶🏻 希望小伙伴们有更优雅的实现方式的话，可以交流一下
