---
title: JS 中 new Date 的坑
date: 2016-01-24
tags: 
  - JavaScript
header-img: images/comm-header/technology.jpg
---
实习的时候做个需求，要判断几个时间区间是否包含、重叠，做时间的计算踩了 ``new Date()`` 的坑。new完时间竟然不对了(o´·д·)」
<!-- more -->
``new Date`` 可以传入一个日期字符串来生成对象的，官方规定 **日期字符串需要符合 RFC2822 或者 ISO8601 的格式**。

- *RFC2822* 格式如果不带时区，``new Date`` 会当做本地时区处理。
- *ISO8601* 格式则会当做 UTC 时区处理。

```javascript
// Sat Jan 02 2016 00:00:00 GMT+0800 (中国标准时间)
new Date("Jan 02 2016") 
// Sat Jan 02 2016 00:00:00 GMT+0800 (中国标准时间)
new Date("01/02/2016") 
// Sat Jan 02 2016 00:00:00 GMT+0800 (中国标准时间)
new Date("2016/01/02") 

// Sat Jan 02 2016 00:00:00 GMT+0800 (中国标准时间)
new Date("01-02-2016") 
// Sat Jan 02 2016 08:00:00 GMT+0800 (中国标准时间)     ISO8601
new Date("2016-01-02") 

// Sat Jan 02 2016 00:00:00 GMT+0800 (中国标准时间)
new Date("2016/01-02") 
// Sat Jan 02 2016 00:00:00 GMT+0800 (中国标准时间)
new Date("2016-01/02") 
```

### 解决方案：

- 每次格式化日期都严格指定时区
- 或者使用工具 moment.js

这个问题在 ``Date.parse`` 中也同样存在。

不同浏览器对Date进行格式化的区别，如：

```javascript
// 都支持
new Date('Thu Mar 31 2016 16:56:00 GMT+0800');
new Date('2016/3/31 16:56');

// 仅chrome支持
new Date('2016.3.31 16:56');
new Date('2016-3-31 16:56');

// ie不支持
new Date('2016 3 31 16:56'); 
new Date('2016-03-31T09:13:00.000Z');
```

以上chrome都支持，然而firefox和ie却不全支持，以上只是举例还并不完全。
所以我们在编写代码的时候要是想都兼容的话还是将参数全部拆分分别传入

```javascript
// 注意月份需要-1
new Date('2016','2','31','17','13');
```