---
title: 对原型链的理解
tags: JavaScript
headerimg: images/comm-header/technology.jpg
date: 2016-04-11 00:00:00
---
又整理了一遍『原型』和『原型链』，例子是网上的，图是按照自己的理解画的，看完这篇文章整个人都通透了。
<!-- more -->
<!-- toc -->

### ChangeLog

- 2018-03-25 增加“拓展”、“继承”

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
- `cat`调用`price`

  ① 查看本身，没有`price`，根据 **\_\_proto\_\_** 找到 **animal prototype对象**。
  
  ② 在**animal prototype对象**中找到了`price`，所以 **`cat.price 结果为20`**。
  
  ③ 如果`无 animal.prototype=20 `，则会根据**animal prototype对象**的 **\_\_proto\_\_** 找到**Object prototype**，**Object prototype**中的 **\_\_proto\_\_** 指向null，若仍没有找到`price`，**`结果则为undefined`**。

- `tidy`调用`price`
  
  ① 查看本身，没有`price`，根据 **\_\_proto\_\_** 找到**dog prototype**。
  
  ② `dog prototype = animal`，这里可以这样理解： **prototype**本质也是一个对象，因此可以重新赋一个对象，无论是函数对象还是普通对象。事实上，每个 **prototype** 会有一个预定义的constructor属性用来引用它的函数对象。
  
  ③ 在**animal**中找到了`price`， 所以 **`tidy.price 结果为1000`**。
  
  ④ 如果``无 animal.price = 1000 ``，则会根据**animal**的 **\_\_proto\_\_** 找到下一个对象，最终都没有找到 `price`，**`结果就为undefined`**。

### 拓展
1. 创建一个对象的方式

  * {}、new Object()
  * 构造函数
  * Object.create()  
    此方法将新对象的__proto__更改并指向create的入参对象。

2. instance of  VS  constructor
**instance of 原理：**检查左边对象与右边对象是否在同一条原型链上。
**constructor原理：**取对象的__proto__属性指向的prototype对象上的constructor字段。
```javascript
cat instanceof animal === true
cat.__proto__ === animal.prototype

animal.prototype instanceof Object === true
animal.prototype.__proto__ === Object.prototype

cat instanceof Object === true
// but，
cat.constructor === animal // true
cat.constructor === Object // false
```

3. new运算符的原理

  * 创建一个空对象，它的__proto__等于构造函数的原型对象（可以用Object.create()完成）
  * 构造函数以第1步创建的对象做为上下文，是否会返回一个对象
  * 若第2步返回了对象，则使用该对象作为新实例，否则用第1步创建的对象作为新实例
  ```javascript
  var myNew = function (func) {
    var o = Object.create(func.prototype)
    var i = func.call(o)
    return typeof i === 'object' ? i : o
  }
  ```

### 继承
**类的声明：**
* function
* class

**生成实例：**new

**继承的几种方式：**
* 借助构造函数，父类的作用域指向子类
  * Parent.call(this)   // this是Child类的上下文
  * 缺点：不能继承原型链属性
* 借助原型链
  * Child.prototype = new Parent()
  * 缺点：子类所有实例共享原型对象；子类实例的constructor为Parent
* 组合方式
  * Parent.call(this)   // this是Child类的上下文
  * Child.prototype = Object.create(Parent.prototype)
  * Child.prototype.constructor = Child

