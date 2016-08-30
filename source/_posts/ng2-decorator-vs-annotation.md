---
title: Decorators 和 Annotations 的不同
date: 2016-08-30 14:29:59
tags: [Angular2]
---
Java的Annotation用起来很是顺手，AtScript和TypeScript都是JavaScript的扩展语言，AtScript支持Annotation，而TypeScript支持Decorator。
5月份的ng-conf大会Angular团队宣布抛弃AtScript项目，转用TypeScript开发，而TypeScript团队也表示TypeScript正在吸收AtScript的优点，为即将到来的TypeScript 1.5版本引入注解的功能。
在[TypeScript 1.5的变更记录](http://tslang.cn/docs/release-notes/typescript-1.5.html)中我们找到了Decorator。
那么Annotation和Decorator有什么关系，又有什么区别呢？？
<!-- more -->

Decorator:	作用是在编译时根据元数据内容对一个函数进行一些修改。
Annotation:	是将元数据附加到代码中，如何附加是由transpiler来决定的。
装饰器是一个函数，transpiler并不管函数内容是什么。

所以，我们可以通过装饰器来创建注释

Typescript支持Decorator，但是不知道angular2的内部注释实现。

上面的代码，core_1.Component( { // 这里是注释的东西 } )
