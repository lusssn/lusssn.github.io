---
layout: technology
title: Decorators 和 Annotations 的不同
tags:
  - Angular2
headerimg: images/comm-header/technology.jpg
abbrlink: b6c4c4a6
date: 2016-09-12 14:29:59
---
Java的Annotation用起来很是顺手，AtScript和TypeScript都是JavaScript的扩展语言，AtScript支持Annotation，而TypeScript支持Decorator。
<!-- more -->
2015年3月的ng-conf大会Angular团队宣布抛弃AtScript项目，转用TypeScript开发，而TypeScript团队也表示TypeScript正在吸收AtScript的优点，为即将到来的TypeScript 1.5版本引入注解的功能。

在[TypeScript 1.5的变更记录] [typescript-1.5]中我们找到了Decorator。

那么Annotation和Decorator有什么关系，又有什么区别呢？？

### 定义

Decorator:	作用是在编译时根据元数据内容对一个函数进行一些修改，其实质是一个函数。

Annotation:	是将元数据附加到代码中，如何附加是由transpiler来决定的。

### 详解AtScript之Annotations

首先看一段ng2代码：

```typescript
import { ComponentMetadata as Component } from '@angular/core';

@Component({
  selector: 'tabs',
  template: `
    <ul>
      <li>Tab 1</li>
      <li>Tab 2</li>
    </ul>
  `
})
export class Tabs { }
```

如果class Tabs没有@Component的注解，它就只是一个普通的类（类的实质也是一个方法）。
从import关键字，很容易想到，其实Component的内容是由框架实现的，Component的实质也是一个class。

那么我们如何做到用一个普通的class去改变另一个class的行为？
为什么可以只用在类前带上前缀@就可以当注释使用？

目前为止，还没有浏览器支持我们干些事情，以后也许可以，现在使用就需要对它们做一个转换。

转换的方式有很多，Babel，Traceur，TypeScript等等，每种方式的转译结果都不同。Traceur会将上面的代码转译成：

```javascript
var Tabs = (function () {
	function Tabs() {}

	Tabs.annotations = [
			new ComponentMetadata({...}),
	];

	return Tabs;
})
```

最后，Annotation内容变成了class的annotations属性里的实例调用。当然，这并不是所有的Annotation都会以这样的方式被转译。如果Annotation作为构造器的参数，那么就会被Traceur转译为class的parameters属性，就像是这样：

```typescript
class MyClass {

  constructor(@Annotation() foo) {
    ...
  }
}
```

转换之后：

```javascript
var MyClass = (function () {
  function MyClass() {}

  MyClass.parameters = [[new Annotation()]]; // 参数可能是多个Annotation

  return MyClass;
})
```

转译工作完成后，具体的一个类怎么成为一个Component的细节就是框架自己的事情了。
对于框架来说，它只关心从哪里可以获取到annotations属性和parameters属性，如果没有Traceur的转译，框架就不知道去哪里取数据了。

在AtScript中，annotations会转译成什么样子又是一种不同的转译实现方式。

### 详解TypeScript之Decorators

> Decorators make it possible to annotate and modify classes and properties at design time.

站在一个消费者的角度，如果可以自定义metadata用在代码中的什么地方是不是很爽？
事实上，在使用Decorator和Annotation时，它们的语法是一样的。

```typescript
import { ComponentMetadata as Component } from '@angular/core';

@Component({
  selector: 'tabs',
  template: `
    <ul>
      <li>Tab 1</li>
      <li>Tab 2</li>
    </ul>
  `
})
export class Tabs { }
```

用Decorator的方式转换会是下面这样：

```javascript
var __decorate = function (decorators, target, key, desc) {
  // ...
};
var core_1 = require('@angular/core');

__decorate( [
              core_1.Component({
                selector: 'tabs',
                template: "..."
              }), 
              __metadata('design:paramtypes', [])
            ], Tabs);
```

Decorator就是一个函数，让我们可以使用需要进行装饰的目标。

> 之前我们关心的是Annotation被转译到什么地方（比如放到某个属性里）。
> 现在，我们要做的是定义一个Decorator函数要做些什么事情。

所以，这也允许我们实现一个Decorator，让它能跟AtScript中的Annotation转译方式一样。

### TypeScript支持Annotation和Decorator？

2015年的ng-conf大会上，Turner宣布[TypeScript将会整合AtScript中引入的标注（annotation）特性] [angular-2-built-on-typescript]，该特性将在TypeScript 1.5+版本中发布。
实际上，最后TypeScript支持Decorator，却不知道Angular 2内部的特殊Annotation实现。在使用装饰器之前我们需要引入metadata注释的实现（不是引入一个装饰器），用来组成一个装饰器。

### 总结

从消费者的角度来看，Annotation和Decorator的作用和语法是一样的，它们的区别在于：

**Annotation** 消费者不可以控制metadata如何被添加到代码中。
**Decorator** 相当于是一个接口，用来构建某些最终作为注释的东西。

长远来看，我们只需要关注Decorator，因为以后它将会成为一个标准。

[>> 参考阅读 <<] [reference-read]

[typescript-1.5]: http://tslang.cn/docs/release-notes/typescript-1.5.html
[angular-2-built-on-typescript]: https://blogs.msdn.microsoft.com/typescript/2015/03/05/angular-2-built-on-typescript/
[reference-read]: http://blog.thoughtram.io/angular/2015/05/03/the-difference-between-annotations-and-decorators.html
