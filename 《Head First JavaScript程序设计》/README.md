# 《Head First JavaScript程序设计》

> 书中代码：[https://github.com/bethrobson/Head-First-JavaScript-Programming/](https://github.com/bethrobson/Head-First-JavaScript-Programming/)

## 7 类型、相等、转换等

### 相等运算符如何转换操作数

#### 1. 比较数字与字符串

> 将把字符串转换为数字，再对两个数字进行比较。

* 比较两个字符串时，根据字母排列顺序来确定谁大谁小。

#### 2. 比较布尔值和其它类型

将把布尔值转为数字，再进行比较。`true` 将被转换为 `1`，而 `false` 将被转换为 `0`。

```javascript
0 < true;  // true
1 == true; // true
```

#### 3. 比较 null 和 undefined

这两个值的比较结果为 `true` 。它们其实表示的都是 没有值 （没有值的变量和没有值的对象），因此认为它们相等。

## 8 综合应用：编写一个应用程序 

### 编码技巧

```js
// Enter 键触发点击事件
function handleKeyPress(e) {
  var fireButton = document.getElementById('fireButton');
  if (e.keyCode === 13) {
    fireButton.click();
    return false;
  }
}
```

## 9 异步编码：处理事件

* 浏览器维护着一个事件队列，不断的从这人队列中取出事件，并调用相应的事件处理程序来处理它们——如果有的话。

## 10 函数是一等公民：自由的函数

### 函数声明和函数表达式差别

* 使用函数声明时，函数将在`执行代码前`创建；而使用函数表达式时，函数将在`运行阶段`执行代码时创建。
* 使用函数声明时，将创建一个与函数同名的变量并指向函数。



