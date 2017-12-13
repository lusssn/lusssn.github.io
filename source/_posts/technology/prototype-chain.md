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
### 看一个实例
```javascript
var animal = function() {};
var dog = function() {};

animal.prototype.price = 20;
animal.price = 1000;

dog.prototype = animal;

var cat = new animal();
var tidy = new dog();
// 下面两行分别输出什么？
console.log(cat.price);
console.log(tidy.price);
```
### 需要知道
要理解原型和原型链首先要知道几个概念：
1. 在js里，继承机制是原型继承。继承的起点是 **对象的原型（Object prototype）**。
2. 一切皆为对象，只要是对象，就会有 **\_\_proto\_\_** 属性，该属性存储了指向其构造的指针。
    - **Object prototype**也是对象，其 **\_\_proto\_\_** 指向null。
3. 对象分为两种：**函数对象**和**普通对象**，只有函数对象拥有『原型』对象（prototype）。
    - **prototype**的本质是普通对象。
    - **Function prototype**比较特殊，是没有prototype的函数对象。
    - new操作得到的对象是普通对象。
4. 当调取一个对象的属性时，会先在本身查找，若无，就根据 **\_\_proto\_\_** 找到构造原型，若无，继续往上找。最后会到达顶层**Object prototype**，它的 **\_\_proto\_\_** 指向null，均无结果则返回undefined，结束。
5. 由 **\_\_proto\_\_** 串起的路径就是『原型链』。 

### 看图
将开篇的一段代码对象关系做了个关系图如下：
* ![](case-1.png) 表示函数对象
* ![](case-2.png) 表示普通对象

![](explain.png)

### 说话
#### 1、 `cat`调用`price`时，沿着原型链寻找该属性。

① 查看本身，没有`price`，根据 **\_\_proto\_\_** 找到 **animal prototype对象**。

② 在**animal prototype对象**中找到了`price`，所以 **`cat.price 结果为20`**。

③ 如果`无 animal.prototype=20 `，则会根据**animal prototype对象**的 **\_\_proto\_\_** 找到**Object prototype**，**Object prototype**中的 **\_\_proto\_\_** 指向null，若仍没有找到`price`，**`结果则为undefined`**。

#### 2、 `tidy`调用`price`时，沿着原型链寻找该属性。

① 查看本身，没有`price`，根据 **\_\_proto\_\_** 找到**dog prototype**。

② `dog prototype = animal`，这里可以这样理解： **prototype**本质也是一个对象，因此可以重新赋一个对象，无论是函数对象还是普通对象。事实上，每个 **prototype** 会有一个预定义的constructor属性用来引用它的函数对象。

③ 在**animal**中找到了`price`， 所以 **`tidy.price 结果为1000`**。

④ 如果``无 animal.price = 1000 ``，则会根据**animal**的 **\_\_proto\_\_** 找到下一个对象，最终都没有找到 `price`，**`结果就为undefined`**。