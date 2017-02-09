---
title: 对原型链的理解
date: 2016-04-11
tags: 
  - JavaScript
header-img: images/comm-header/technology.jpg
---
之前花了几个小时彻彻底底的研究明白『原型』和『原型链』。
搭了个人blog找个东西放又整理了一遍，例子是网上的，图是按照自己的理解画的，看完这篇文章整个人都通透了哈哈哈。
<!-- more -->
代码实例：
```javascript
    var animal = function() {};
    var dog = function() {};

    animal.prototype.price = 20;
    animal.price = 1000;

    dog.prototype = animal;

    var cat = new animal();
    var tidy = new dog();

    console.log(cat.price);
    console.log(tidy.price);
```

* ![](case-1.png) 表示函数对象，函数对象拥有prototype（原型对象，本质是普通对象，Function prototype除外）。
  ![](case-2.png) 表示普通对象，没有prototype属性。new操作得到的对象为普通对象。

关系图：
![](explain.png)

### 1、 *cat* 调用 *price属性* 时，同样沿着原型链寻找该属性。

① 查看本身，没有 *price属性*，根据 *\_\_proto\_\_* 找到 *animal prototype对象*。

② 在 *animal prototype对象* 中找到了 *price属性*，所以 **cat.price 结果为20**。如果没有设置`` animal.prototype=20 ``，则会根据 *animal prototype对象* 的 *\_\_proto\_\_* 找到 *Object prototype*，*Object prototype* 中的 *\_\_proto\_\_* 指向null，若仍没有找到 *price属性*，结果则为undefined。

### 2、 *tidy* 调用 *price属性* 时，沿着原型链寻找该属性。

① 查看本身，没有 *price属性*，根据 *\_\_proto\_\_* 找到 *dog prototype*。

② dog prototype = animal ，这里可以这样理解： *prototype*本质也是一个对象，因此可以重新赋一个对象，无论是函数对象还是普通对象。事实上，每个 *prototype* 会有一个预定义的constructor属性用来引用它的函数对象。

③ 在 *animal* 中找到了 *price属性*， 所以 **tidy.price 结果为1000**。如果没有`` animal.price = 1000 ``，则会根据 *animal* 的 *\_\_proto\_\_* 找到下一个对象，最终都没有找到 *price属性*，结果就会为undefined。