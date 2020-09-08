---
title: 语法解析的思路验证数学公式
tags: 实用Demo
date: 2018-11-05 10:36:59
---
一个包含加减乘除和圆括号的基本数学公式，如何用JS去实现其语法正确性的判断呢？
<!-- more -->
一般情况，如果是个只有数字和符号的字符串，要验证它是不是一个正确的数学公式，可以有三种解法。

**1⃣️** 借助 `eval()` 函数，直接执行公式。
**2⃣️** 结合正则表达式，层层匹配来验证语法的合法性。
**3⃣️** 根据编译原理，进行语法解析。

解法 1⃣️ 中，如果字符串中语法符合 Javascript 的语法，却不符合数学公式的语法，不能被检测出来。
另外，**如果字符串不是一个真实的计算表达式，而是一个抽象的公式**，比如 `ax + b`，上面的解法 1⃣️ 和解法 2⃣️ 就不好做了，此时运用编译原理的思路就显得很自然。

根据编译原理，进行语法解析步骤也不复杂。
首先，我们要规定几个语法规则：
用 `A` 表示因子，`&` 表示数学运算符（加减乘除等），左右括号仍然分别用 `(`、`)` 表示。
1. A&A => A
1. (A) => A

若给定的式子，通过上述规则进行规约，最后得到一个因子 `A`，就认为这个式子是一个正确的数学计算公式。

有了上面的规定，就可以开始写代码啦！
悄悄说一句，代码只根据思考逻辑实现，并不是最为简洁高效的。

{% spoiler 入口函数实现 %}
```js
function mergeFormula(originArray = []) {
  let data = [...originArray]
  // 循环结束的条件：数组长度不大于1
  while (data.length > 1) {
    // 从末尾往前进行规约
    const item = data[data.length - 1]
    if (getItemType(item) === 'factor') {
      // 如果是因子，A&A => A
      data = mergeNormal(data)
    } else if (getItemType(item) === 'right') {
      // 如果是右括号，(A) => A
      data = mergeParenthesis(data)
    } else {
      return []
    }
  }

  if (data.length === 1 && getItemType(data[0]) === 'factor') {
    return data
  }
  return []
}
```
{% endspoiler %}

{% spoiler 语法 A&A => A 的规约 %}
```js
function mergeNormal(originArray = []) {
  let data = [...originArray]
  // 第一个项必须是因子
  const firstItem = data.pop()
  if (getItemType(firstItem) !== 'factor') {
    return []
  }
  // 第二个项必须是运算符
  const secondItem = data.pop()
  if (getItemType(secondItem) !== 'operator') {
    return []
  }
  // 循环结束的条件：数组长度为0
  while (data.length) {
    const item = data[data.length - 1]
    const type = getItemType(item)
    // 遇到有括号，进入语法2规约
    if (type === 'right') {
      data = mergeParenthesis(data)
    }
    // 遇到因子，返回结果，结束循环
    else if (type === 'factor') {
      return data
    }
    // 否则，规约失败，结束循环
    else {
      return []
    }
  }

  return []
}
```
{% endspoiler %}

{% spoiler 语法 (A) => A 的规约 %}
```js
function mergeParenthesis(originArray = []) {
  let data = [...originArray]
  let isParenthesis = false
  const temp = []
  // 第一个项必须是右括号
  const firstItem = data.pop()
  if (getItemType(firstItem) !== 'right') {
    return []
  }
  // 循环
  while (!isParenthesis && data.length) {
    const item = data.pop()
    const type = getItemType(item)
    // 遇到左括号，结束循环
    if (type === 'left') {
      isParenthesis = true
      break
    }
    // 遇到右括号，进入语法2规约
    if (type === 'right') {
      data.push(item)
      data = mergeParenthesis(data)
    }
    // 将项存入临时区
    else {
      temp.push(item)
    }
  }

  if (!isParenthesis) {
    return []
  }
  // 规约临时区
  const tempMerge = mergeFormula(temp)
  if (tempMerge.length) {
    return data.concat(tempMerge)
  }
  return []
}
```
{% endspoiler %}

⚠️ **代码说明**

1. 以上方法均**默认传入参数为数组类型**，应用前需要保证入参类型为数组。
{% spoiler 应用举例 %}
```js
const isFormula = (originArray = []) => {
  if (Array.isArray(originArray)) {
    const result = mergeFormula(originArray)
    return Boolean(result.length)
  }
  return false
}
```
{% endspoiler %}
2. 判断子项类型的工具方法：`getItemType` 根据具体情况具体实现，约定为：
  - 因子 -> factor
  - 运算符 -> operator
  - 左括号 -> left
  - 右括号 -> right
