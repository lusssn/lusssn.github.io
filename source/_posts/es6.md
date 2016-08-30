---
title: ES6 新特性
date: 2016-08-15 10:06:39
tags: [ECMAScript 6]
---
组内给我分配了分享AngularJS 2.0的任务，为了更好的理解Angular2，比较粗略地学习了ES6。
ES6语法能使代码变得更加简洁，很人性化。
<!-- more -->

## 目录
1. [数组遍历](#1-数组遍历)
2. [Generators](#2-Generators)
3. [Template](#3-Template)
4. [Parameters](#4-Parameters)
5. [Destructuring](#5-Destructuring)
6. [Arrow Functions](#6-Arrow-Functions)
7. [Symbol](#7-Symbol)
8. [集合](#8-集合)
9. [Proxy](#9-Proxy)
10. [Class](#10-Class)
11. [Subclassing](#11-Subclassing)
12. [let和const](#12-let和const)
13. [模块系统](#13-模块系统)
14. [需要Babel或Traceur协助](#14-需要Babel或Traceur协助)
15. [其他](#15-其他)
16. [ES7](#16-ES7)

* * *

### 1. 数组遍历

> 原始的for循环，麻烦。 
> ES5中foreach不可跳出循环，也不可返回上层！ 
> 可以使用for (var index in myArray){}，但是index是字符串型的数字，误操作会带来大麻烦。 
> for-in是为普通对象设计的，还会遍历自定义属性（原型链上的属性）。 
> 某些情况下还会对数组进行随机遍历！

```javascript
for (var value of myArray) {
    console.log(value);}
```

`for-of` 可以遍历数组和大多数数组对象，不能用于普通对象，支持break、continue和return语句。 
可以使用Object.keys()转换普通对象：
```javascript
// 向控制台输出对象的可枚举属性
for (var key of Object.keys(someObject)) {
    console.log(key + ": " + someObject[key]);
}
```

* * *

### 2. Generators

Generator类似函数，可以称为生成器函数。 
使用 `function*` 进行声明。 
内部使用 `yeild` 替代普通函数中的return，可以使用多个 `yeild`，Generator执行过程遇到 `yeild` 就会暂停，后续再次触发Generator可恢复执行状态。 
与调用者处于同一个线程，有确定的连续执行顺序，永不并发。 
Generator实质是迭代器，拥有内建.next() 和 \[Symbol.iterator\]()方法的实现，你只需实现循环的部分。

#### 2.1 可选的.return()

.return()方法是迭代器接口支持的可选方法，Generator作为一种迭代器也支持这个方法。
迭代器 `返回{done:true}之前退出` 会自动调用该方法，然后触发生成器执行 `任一finally代码块`之后退出。

#### 2.2 可选的.next()的参数

.next()方法接受一个可选参数，参数稍后会作为yield表达式的返回值出现在生成器中。

#### 2.3 .throw(error)

当生成器执行到一个yield表达式并暂停后可以实现以下功能：

* 调用generator.next(value)，生成器从离开的地方恢复执行。
* 调用generator.return()，传递一个可选值，生成器只执行finally代码块并不再恢复执行。
* 调用generator.throw(error)，生成器表现得像是yield表达式调用一个函数并抛出错误。
* 什么也不做，生成器永远保持冻结状态。（是的，对于一个生成器来说，很可能执行到一个try代码块，永不执行finally代码块。这种状态下的生成器可以被垃圾收集器回收。）

** 请记住，生成器内部抛出的异常总是会传播到调用者。所以无论生成器是否捕获错误，generator.throw(error)都会抛出error并立即返回给你。*

#### 2.4 yield*

> 普通 `yield` 表达式只生成一个值，而 `yield*` 表达式可以通过迭代器进行迭代生成所有的值。
> 通过 `yield*` 可以实现生成器中调用生成器，从而实现**生成器的重构**。



[深入讲解Generator](http://jlongster.com/A-Study-on-Solving-Callbacks-with-JavaScript-Generators)

* * *

### 3. Template

#### 3.1 Template Strings

使用` ` `将模板字符串包起来，`${}` 类似jQuery中的模板占位符。
```javascript
function authorize(user, action) {
    if (!user.hasPrivilege(action)) {
        throw new Error(
            `用户 ${user.name} 未被授权执行 ${action} 操作。`);
    }
}
```

模板占位符中的代码可以是任意JavaScript表达式。   
不同于普通字符串，模板字符串可以多行书写。模板字符串中所有的空格、新行、缩进，都会原样输出在生成的字符串中。  
```javascript
$("#warning").html(`
    <h1>小心！>/h1>
    <p>未经授权打冰球可能受罚将近${maxPenalty}分钟。</p>`);
```

#### 3.2 Tagged Template

标签模板使用在模板字符串` ` `之前，作用是对字符串内容进行处理，简化了函数调用。 
任何ES6的成员表达式（MemberExpression）或调用表达式（CallExpression）都可作为标签使用。
```javascript
var message =
    SaferHTML`<p>${bonk.sender} 向你示好。</p>`;
    // SaferHTML是一个用于转换模板字符串中特殊字符的函数
    // 等效于：
    var message = SaferHTML(templateData, bonk.sender);
```

* * *

### 4. Parameters

#### 4.1 不定参数

```javascript
function containsAll(haystack, ...needles) {
    for (var needle of needles) {
        if (haystack.indexOf(needle) === -1) {
          return false;
        }
    }
    return true;
}
```

`needles` 就是ES6中存储不定参数的数组，相比 `arguments` 类数组，它可遍历。 
需要注意的是，只有最后一个参数可以标记 `...` 为不定参数。 
没有额外参数时，`needles` 就是一个空数组，永远不会是undefined。

#### 4.2 默认参数

```javascript
function animalSentenceFancy(animals2="tigers",
    animals3=(animals2 == "bears") ? "sealions" : "bears") {
    return `Lions and ${animals2} and ${animals3}! Oh my!`;
}
```

默认参数表达式自左向右求值。   
传递undefined等同于不传值。  

* * *

### 5. Destructuring

解构赋值允许你使用类似数组或对象字面量的语法将数组和对象的属性赋给各种变量。 
被解构的值需要被强制转换为对象，解构不能转换成对象的值，会抛出异常。 
解构能转换为对象的值时，被解构的值必须要包含一个迭代器，否则将得到结果是undefined。 
进行解构时，可以指定一个默认值。 
```javascript
// 示例：对任意深度的嵌套数组进行解构
var [foo, [[bar], baz]] = [1, [[2], 3]];
console.log(foo); // 1
console.log(bar); // 2
console.log(baz); // 3

// 示例：在对应位留空来跳过被解构数组中的某些元素
var [,,third] = ["foo", "bar", "baz"];
console.log(third); // "baz"

// 示例：解构对象时，属性名与变量名一致可简写
var { foo, bar } = { foo: "lorem", bar: "ipsum" };
console.log(foo); // "lorem"

// 示例：解构对象并赋值给未声明变量时
{ blowUp } = { blowUp: 10 }; // Syntax error 语法错误
({ safe } = {}); // No errors 没有语法错误

// 示例：指定一个默认值
var [missing = true] = [];
console.log(missing); // true

// 示例：嵌套解构对象
var complicatedObj = {
    arrayProp: [
        "Zapp",
        { second: "Brannigan" }
    ]
};
var { arrayProp: [first, { second }] } = complicatedObj;
console.log(first);　// "Zapp"
console.log(second);　// "Brannigan"
```

> 解构的实际应用： 
> * 函数参数定义 
> * 配置对象参数 
> * 与ES6迭代器协议协同使用 
> * 多重返回值 
> * 使用解构导入部分CommonJS模块 

* * *

### 6. Arrow Functions

|        符号  |            作用 |
| :---------- | :-------------- |
| <\!\-\-  | 单行注释        |
| -->         | "趋向于"操作符   |
| <=          | 小于等于        |
| =>          | Arrow Functions |

```javascript
// ES5
var selected = allJobs.filter(function (job) {
    return job.isSelected();
});
// ES6
var selected = allJobs.filter(job => job.isSelected());

// ES5
var total = values.reduce(function (a, b) {
    return a + b;
}, 0);
// ES6
var total = values.reduce((a, b) => a + b, 0);

// ES5
$("#confetti-btn").click(function (event) {
    playTrumpet();
    fireConfettiCannon();
});
// ES6
$("#confetti-btn").click(event => {
    playTrumpet();
    fireConfettiCannon();
});

// 为与你玩耍的每一个小狗创建一个新的空对象
var chewToys = puppies.map(puppy => {});   // 这样写会报Bug！
var chewToys = puppies.map(puppy => ({})); //
```

> { 是唯一一个有歧义的字符，所以用小括号包裹对象字面量是唯一一个你需要牢记的小窍门。 

**箭头函数没有它自己的this值**，箭头函数内的this值继承自外围作用域。 
通过object.method()语法调用的方法使用非箭头函数定义，这些函数需要从调用者的作用域中获取一个有意义的this值。 
其它情况全都使用箭头函数。 
箭头函数不会获取它们自己的arguments对象。 
```javascript
{
    // ...
    addAll: function addAll(pieces) {
        var self = this;
        _.each(pieces, function (piece) {
            self.add(piece);
        });
    },
    // ...
}

// ES6
{
    // ...
    addAll: function addAll(pieces) {
        _.each(pieces, piece => this.add(piece));
    },
    // ...
}
// ES6的方法语法
    {
        // ...
        addAll(pieces) {
            _.each(pieces, piece => this.add(piece));
        },
        // ...
    }
```

* * *

### 7. Symbol

> JavaScript的第七种原始类型。

获取Symbol的方法：

  * **Symbol()。** 每次调用都会返回一个新的唯一symbol。
  * **Symbol.for(string)。** 这种方式会访问Symbol注册表，注册表可以供多个页面共享使用。
  * **使用标准定义的symbol。** 例如：Symbol.iterator，标准根据一些特殊用途定义了少许的几个symbol。

* * *

### 8. 集合
#### 8.1 Set

> 与其他语言的Set集合相似，一个Set集合中不会包含相同的元素。

ES6中Set集合支持的操作有：

|        操作  |            说明 |
| :---------- | :-------------- |
| new Set  | 创建一个新的、空的Set |
| new Set(iterable) | 从任何可遍历数据中提取元素，构造出一个新的集合 |
| set.size | 获取集合元素个数 |
| set.has(value) | 包含性检测 |
| set.add(value) | 添加一个新元素，会返回集合自身 |
| set.delete(value) | 删除一个元素，会返回集合自身 |
| set\[Symbol.iterator\]() | 返回一个新的遍历整个集合的迭代器 |
| set.forEach(f) | 类似于数组的.forEach()方法 |
| set.clear() | 清空集合 |
| set.keys() |  |
| set.values() |  |
| set.entries() |  ||

#### 8.2 Map

ES6中Set集合支持的操作有：

|        操作  |            说明 |
| :---------- | :-------------- |
| new Map  | 创建一个新的、空的Map |
| new Map(iterable) | 从任何可遍历数据中提取元素，构造出一个新的集合 |
| map.size | 获取集合元素个数 |
| map.has(value) | 包含性检测 |
| map.get(key) | 添加一个新元素，会返回集合自身 |
| map.set(key, value) |  |
| map.delete(key) | 按键名删除一项 |
| map\[Symbol.iterator\]() | 返回一个新的遍历整个集合的迭代器 |
| map.forEach(f) | 类似于数组的.forEach()方法 |
| map.clear() | 清空集合 |
| map.keys() |  |
| map.values() |  |
| map.entries() |  ||

遍历Map或Set的顺序就是其中元素的插入顺序。

WeakMap和WeakSet被设计来完成与Map、Set几乎一样的行为，除了以下一些限制：
* WeakMap只支持new、has、get、set 和delete。
* WeakSet只支持new、has、add和delete。
* WeakSet的值和WeakMap的键必须是对象。

两种弱集合都不可迭代，除非专门查询或给出你感兴趣的键，否则不能获得一个弱集合中的项。

* * *

### 9. Proxy

`Proxy`可以接受两个参数：`target`、`handler`。
`Proxy.revocable(target, handler)函数`可以像new Proxy(target, handler)一样创建代理，但是创建好的代理后续可被解除。

#### 9.1 target

代理的行为很简单：将代理的所有内部方法转发至目标。
简单来说，如果调用proxy.\[\[Enumerate\]\]()，就会返回target.\[\[Enumerate\]\]()。
然而proxy并不是target的拷贝，`proxy !== target`。

#### 9.2 handler

handler对象可以设置内部方法拦截方法，所有的[handler方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy/handler)均是可选的。
```
> var tree = Tree();
> tree
	{ }
> tree.branch1.branch2.twig = "green";
> tree
	{ branch1: { branch2: { twig: "green" } } }
> tree.branch1.branch3.twig = "yellow";
	{ branch1: { branch2: { twig: "green" },
			   branch3: { twig: "yellow" } } }
```

使用Proxy实现上面的效果：
```javascript
function Tree() {
	return new Proxy({}, handler);
}
var handler = {
	get: function (target, key, receiver) {
		if (!(key in target)) {
			target[key] = Tree();  // 自动创建一个子树
		}
		return Reflect.get(target, key, receiver);
	}
};
```

** ES6定义了一个新的 [Reflect对象](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect)，用于实现只执行委托给目标的默认行为这一需求*

** 如果你想在Node.js或io.js环境中使用代理，首先你需要添加名为harmony-reflect的polyfill，然后在执行时启用一个非默认的选项（--harmony_proxies），这样就可以暂时使用V8中实现的老版本代理规范。*

* * *

### 10. Class

JavaScript也在不断地朝面向对象编程靠拢。像其他语言一样，JS中终于有一个`句法结构`可以用来处理面向对象设计的问题了，其被命名为`Class`。

要实现一个Circle类，传统方法如下：
```javascript
function Circle(radius) {
	this.radius = radius;
		Circle.circlesMade++;
	}
	Circle.draw = function draw(circle, canvas) { /* Canvas绘制代码 */ }
		Object.defineProperty(Circle, "circlesMade", {
	get: function() {
		return !this._count ? 0 : this._count;
	},
	set: function(val) {
		this._count = val;
	}
	});
	Circle.prototype = {
		area() {
			return Math.pow(this.radius, 2) * Math.PI;
		},
	get radius() {
		return this._radius;
	},
	set radius(radius) {
		if (!Number.isInteger(radius))
			throw new Error("圆的半径必须为整数。");
		this._radius = radius;
	}
};
```

然而实际我们更希望像这样来定义一个类：
```javascript
var obj = {
	// 现在不再使用function关键字给对象添加方法
	// 而是直接使用属性名作为函数名称。
	method(args) { ... },
	// 只需在标准函数的基础上添加一个“*”，就可以声明一个生成器函数。
	*genMethod(args) { ... },
	// 借助|get|和|set|可以在行内定义访问器。
	// 只是定义内联函数，即使没有生成器。
	// 注意通过这种方式装载的getter不能接受参数
	get propName() { ... },
	// 注意通过这种方式装载的setter至少接受一个参数
	set propName(arg) { ... },
	// []语法可以用于任意支持预计算属性名的地方，来满足上面的第4中情况。
	// 这意味着你可以使用symbol，调用函数，联结字符串
	// 或其它可以给property.id求值的表达式。
	// 这个语法对访问器或生成器同样有效，我在这里只是举个例子。
	[functionThatReturnsPropertyName()] (args) { ... }
};
```

Class定义语法实现了这样的方式：
```javascript
class Circle {
	// 方法构造函数（method constructor）指定出唯一的构造函数
	constructor(radius) {
		this.radius = radius;
		Circle.circlesMade++;
	};
	// 可选的static和const关键字
	static draw(circle, canvas) {
		// Canvas绘制代码
	};
	static get circlesMade() {
		return !this._count ? 0 : this._count;
	};
	static set circlesMade(val) {
		this._count = val;
	};
	area() {
		return Math.pow(this.radius, 2) * Math.PI;
	};
	get radius() {
		return this._radius;
	};
	set radius(radius) {
		if (!Number.isInteger(radius))
			throw new Error("圆的半径必须为整数。");
		this._radius = radius;
	};
}
```

*注意：重定义Class会抛出重定义错误。*

* * *

### 11. Subclassing

> 同其他语言一样，JavaScript拥有了`Class`这样的句法结构，但是还需要一个更简单方便的继承机制。
> 关键词 `extends` 用来声明子类关系。

某些情况下，如果在子类中重新定义了一个方法，有时又想绕开这个方法去使用父类中的同名方法，应该怎样做呢？
> 关键词 `super` 可以绕开我们在子类中定义的属性，直接从子类的原型开始查找属性，从而绕过子类中覆盖父类上的同名方法。

执行基类的构造函数前，this对象没有被分配，从而我们无法得到一个确定的this值。
因此，在子类的构造函数中，调用super构造函数之前访问this会触发一个引用错误（ReferenceError）。

使用new创建的实例，可以在其基类的构造函数中使用 `new.target` 来确定子类的类型。
new.target在任何函数中都是合法的，如果函数不是通过new调用，new.target将被赋值为undefined。

* * *

### 12. let和const

> JavaScript中变量（var）的作用域是函数体的全部。这实际上是JavaScript设计初的bug。

#### 12.1 let

* let声明的变量拥有**块级作用域**，并保留了良好的提升（hoisting）特性。
* let声明的**全局变量不是全局对象的属性**。
* 形如for (let x...)的循环在**每次迭代时都为x创建新的绑定**。
  如果一个for (let...)循环执行多次并且循环保持了一个闭包，那么每个闭包将捕捉一个循环变量的不同值作为副本，而不是所有闭包都捕捉循环变量的同一个值。
  这种情况适用于现有的三种循环方式：`for-of`、`for-in`、以及传统的用分号分隔的`类C循环`。
* let声明的变量**在到达其被定义的代码行之前使用，会触发错误（ReferenceError）**。因为直到控制流到达该变量被定义的代码行时才会被装载。
* **用let重定义变量会抛出一个语法错误（SyntaxError）**。

#### 12.2 const

* const声明的变量**只可以在声明时赋值**，不可随意修改，否则会导致语法错误（SyntaxError）。
* **用const声明变量后必须要赋值**，否则也抛出语法错误。

* * *

### 13. 模块系统

> 一个js文件就是一个模块。

**export list**：公开的内容列表。
```javascript
export {detectCats, Kittydar};
function detectCats(canvas, options) { ... }
class Kittydar { ... }

// 重命名名称
function v1() { ... }
function v2() { ... }
export {
	v1 as streamV1,
	v2 as streamV2,
	v2 as streamLatestVersion
};
```

**import**：导入模块内容。
```javascript
// 导入多个名称
import {detectCats, Kittydar} from "kittydar.js";
// 重命名名称
import {flip as flipOmelet} from "eggs.js";
import {flip as flipHouse} from "real-estate.js";
```

**Default exports**：
```javascript
import $ from "jquery";
// 等价于：
import {default as $} from "jquery";

var colors = require("colors/safe"); 
// ES6等效代码
import colors from "colors/safe";

// 默认导出方式
let myObject = {
	field1: value1,
	field2: value2
};
export {myObject as default};
// 等效于：
export default {
	field1: value1,
	field2: value2
};

// 导入一个模块命名空间对象，将它的所有属性都导出
import * as cows from "cows";
```

**聚合模块**
```javascript
// world-foods.js - 来自世界各地的好东西

// 导入"sri-lanka"并将它导出的内容的一部分重新导出
export {Tea, Cinnamon} from "sri-lanka";

// 导入"equatorial-guinea"并将它导出的内容的一部分重新导出
export {Coffee, Cocoa} from "equatorial-guinea";

// 导入"singapore"并将它导出的内容全部导出
export * from "singapore";
```

这些`export-from`语句每一个都好比是在一条`import-from语句后伴随着一个export`。
与真正的导入内容的方法不同的是，这些导入内容再重新导出的方法不会在作用域中绑定你导入的内容。
如果打算用world-foods.js中的Tea来写一些代码，别用这种方法导入模块，你会发现当前模块作用域中根本找不到Tea。

**ES6的模块加载过程**：

`语法解析`，检查语法错误 --> 递归`加载`被导入的模块 --> `连接` --> `运行时`在每一个新加载的模块体内执行所有语句
> `连接阶段`每遇到一个新加载的模块，为其`创建作用域`并将模块内声明的所有绑定填充到该作用域中，其中包括由其它模块导入的内容。

**ES6中模块系统需要注意**：
* 只可以在模块的最外层作用域使用import和export，不可在条件语句中使用，也不能在函数作用域中使用import。
* 所有导出的标识符一定要在源代码中明确地导出它们的名称，不能通过编写代码遍历一个数组然后用数据驱动的方式导出一堆名称。
* 模块对象被冻结了，所以你无法hack模块对象并为其添加polyfill风格的新特性。
* 一个模块的所有依赖必须在模块代码运行前完全加载、解析并且及早连接，不存在一种通过import来按需懒加载的语法。
* import模块产生的错误没有错误恢复机制。一个app可能囊括了上百个模块，一旦有一个模块无法加载或连接，所有的模块都不会运行，而且你不能在try/catch代码块中捕捉import的错误信息。
* 不支持在模块加载依赖前运行其它代码的钩子，这也意味着无法控制模块的依赖加载过程。

* * *

### 14. 需要Babel或Traceur协助

> 通过 [Babel](https://babeljs.io/) 或Google的 [Traceur](https://github.com/google/traceur-compiler) 可以将ES6代码转译为Web友好的ES5代码。

[使用Broccoli + Babel 构建的小demo](http://www.infoq.com/cn/articles/es6-in-depth-babel-and-broccoli)

** 注意第二个示例中，Brocfile.js 中 babelPath 的处理，原文中有bug。*

* * *

### 15. 其他

> 以上并不是ES6中所有的新特性，还有更多内容是一些没有标准化，但早已被各大厂商广泛实现并使用的；还有一些特性采自过去其它的标准。

* **定型数组（Typed arrays）、数组缓冲（ArrayBuffer）和数据视图（DataView）**
  \- 来自WebGL标准，被用于实现Canvas、Web Audio还有WebRTC等。
  \- 可以协助处理大量原始的二进制数据或数值型数据。
* **Promise**
  \- 简而言之，它只不过是一段异步JS编程的构建代码块，它可以为你提供一些非立即可用的值。
  \- [深入剖析Promise](http://www.html5rocks.com/zh/tutorials/es6/promises/)
* **块级作用域内部函数**
  \- ES6将此用法标准化，函数声明会被提升到封闭代码块的顶部。
* **函数名称**
  \- 所有主流的JS引擎都会为命名函数添加一个`.name属性`。
  \- 之前通过函数表达式声明的函数被认为是未命名函数，新标准通过命名感知的方式为其也添加了`.name属性`。
* **展开运算符（...）**
* **Object.assign(target, ...sources)**
* **完全尾调用**
  \- 在ES6标准中所有的实现也都必须是“尾递归（Tail-Recursive）。
* **升级Unicode版本**
  \- ES6要求至少支持Unicode 5.1.0中的所有字符。
* **长Unicode转义序列**
  \- 你现在可以直接写`\u{13021}`啦。
* **更好地支持基本多文种平面（BMP）以外的字符**
* **Unicode正则表达式**
  \- ES6中的正则表达式支持一个新的修饰符：`u`。
  \- 在加入修饰符u的正则表达式中，BMP以外的字符都被视为单个字符，而不是两个相互分离的码元。
  \- 如果不加u，`/./`只能匹配字符`&#x1f62d;`的前半部分，但是`/./u`就可以匹配整个字符。
* **粘性正则表达式**
  \- 该特性为正则表达式添加了一个粘性修饰符：`y`。
  \- 它只会查找始于特定偏移量的匹配，偏移量给自它的.lastIndex属性；如果没有相应匹配，它会立即返回null。
* **官方版国际化标准**
  \- 标准规定，任何提供国际化相关特性的ES6实现必须支持ECMAScript 2015国际化API规范——[ECMA-402](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl)
* **二进制和八进制数字字面量**
  \- 新标准中使用`0o表示八进制`，使用`0b表示二进制`。
* **新的Number函数和常量**
  \- [专门为Number打造的特性](http://www.ecma-international.org/ecma-262/6.0/index.html#sec-properties-of-the-number-constructor)
* **新的Math函数**
  \- Math.sinh()、Math.cosh()、Math.tanh()、Math.asinh()、Math.acosh()、Math.atanh()：双曲三角函数和它们的逆函数
  \- Math.cbrt(x)：用于计算立方根
  \- Math.hypot(x, y)：用于计算直角三角形斜边
  \- Math.log2(x)和Math.log10(x)：用于计算同底对数
  \- Math.clz32(x)：帮助计算整数对数的函数
  \- Math.sign(x)：可以用来获取数字的符号
  \- Math.fround(x)：用于支持32位浮点数操作
  \- Math.imul(x, y)：用以计算x的y次幂
  \- 其它的一些函数。

* * *

### 16. ES7

> ES7标准的提案已经在进行中，再说一些ES7中有趣的特性。

* **乘方操作符**
  \- 2 ** 8将返回256。
* **Array.prototype.includes(value)**
  \- 如果数组中包含给定值将返回true。
* **单指令多数据（SIMD）**
  \- 暴露现代CPU提供的128位SIMD指令， 这些指令可以对2、4或8组相邻的数组元素同时进行算数运算。
* **Async异步函数**
  \- Async函数与生成器类似，但是专门用于异步编程操作。
  \- 当你调用生成器时，它会返回一个迭代器；当你调用async函数时，它会返回一个promise。
  \- 生成器使用yield关键词来暂停并产生值；async函数使用await关键词来暂停并等待promise。
* **定型对象**
  \- 定型数组中包含着定型元素，定型对象与定型数组的概念类似，简单来说定型对象中的属性都拥有确定的类型。
* **类和属性修饰符**
  \- 下面的`@debug.logWhenCalled`就是修饰符。
```javascript
import debug from "jsdebug";

class Person {
	@debug.logWhenCalled
	hasRoundHead(assert) {
		return this.head instanceof Spheroid;
	}
	...
}
```

[ES6更多内容](http://www.infoq.com/cn/es6-in-depth/)