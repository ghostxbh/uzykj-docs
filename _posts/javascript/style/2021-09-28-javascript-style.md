---
title: JavaScript编码规范
date: 2021-09-28
tags:
    - JavaScript
    - Standard
author: ghostxbh
location: blog
summary: JavaScript编码规范。
---
**JavaScript编码规范**

## 一、命名
通常, 使用 `functionNamesLikeThis`, `variableNamesLikeThis`, `ClassNamesLikeThis`, `EnumNamesLikeThis`, `methodNamesLikeThis`, 和 `SYMBOLIC_CONSTANTS_LIKE_THIS`.

### 1.1 属性和方法
- 文件或类中的 私有 属性, 变量和方法名应该以下划线 `_` 开头.
- `_` 保护 `_` 属性, 变量和方法名不需要下划线开头, 和公共变量名一样.

### 1.2 方法和函数参数

可选参数以 `opt_` 开头.

函数的参数个数不固定时, 应该添加最后一个参数 `var_args` 为参数的个数. 你也可以不设置 `var_args` 而取代使用 `arguments`.

可选和可变参数应该在 `@param` 标记中说明清楚. 虽然这两个规定对编译器没有任何影响, 但还是请尽量遵守

### 1.3 Getters 和 Setters

`Getters` 和 `Setters` 并不是必要的. 但只要使用它们了, 就请将 `getters` 命名成 `getFoo()` 形式, 将 `setters` 命名成 `setFoo(value)` 形式. (对于布尔类型的 `getters`, 使用 `isFoo()` 也可.)

### 1.4 命名空间

JavaScript不支持包和命名空间.

不容易发现和调试全局命名的冲突, 多个系统集成时还可能因为命名冲突导致很严重的问题. 为了提高 JavaScript 代码复用率, 我们遵循下面的约定以避免冲突.

#### 1.4.1 为全局代码使用命名空间

在全局作用域上, 使用一个唯一的, 与工程/库相关的名字作为前缀标识. 比如, 你的工程是 "Project Sloth", 那么命名空间前缀可取为 `sloth.*`.

```js
var sloth = {};

sloth.sleep = function() {
  // 业务块
};
```

许多 JavaScript 库, 为你提供了声明你自己的命名空间的函数. 比如:

```js
goog.provide('sloth');

sloth.sleep = function() {
  // 业务块
};
```

#### 1.4.2 明确命名空间所有权

当选择了一个子命名空间, 请确保父命名空间的负责人知道你在用哪个子命名空间, 比如说, 你为工程 `sloths` 创建一个 `hats` 子命名空间, 那确保 Sloth 团队人员知道你在使用 `sloth.hats`.

#### 1.4.3 外部代码和内部代码使用不同的命名空间

"外部代码" 是指来自于你代码体系的外部, 可以独立编译. 内外部命名应该严格保持独立. 如果你使用了外部库, 他的所有对象都在 `foo.hats.*` 下, 那么你自己的代码不能在 `foo.hats.*` 下命名, 因为很有可能其他团队也在其中命名.

```js
foo.require('foo.hats');

/**
 * WRONG -- Do NOT do this.
 * @constructor
 * @extend {foo.hats.RoundHat}
 */
foo.hats.BowlerHat = function() {
};
```

如果你需要在外部命名空间中定义新的 API, 那么你应该直接导出一份外部库, 然后在这份代码中修改. 在你的内部代码中, 应该通过他们的内部名字来调用内部 API , 这样保持一致性可让编译器更好的优化你的代码.

```js
foo.provide('googleyhats.BowlerHat');

foo.require('foo.hats');

/**
 * @constructor
 * @extend {foo.hats.RoundHat}
 */
googleyhats.BowlerHat = function() {
  // 业务块
};

goog.exportSymbol('foo.hats.BowlerHat', googleyhats.BowlerHat);
```

#### 1.4.4 重命名那些名字很长的变量, 提高可读性

主要是为了提高可读性. 局部空间中的变量别名只需要取原名字的最后部分.

```js
/**
 * @constructor
 */
some.long.namespace.MyClass = function() {
};

/**
 * @param {some.long.namespace.MyClass} a
 */
some.long.namespace.MyClass.staticHelper = function(a) {
  // 业务块
};

myapp.main = function() {
  var MyClass = some.long.namespace.MyClass;
  var staticHelper = some.long.namespace.MyClass.staticHelper;
  staticHelper(new MyClass());
};
```

不要对命名空间创建别名.

```js
myapp.main = function() {
  var namespace = some.long.namespace;
  namespace.MyClass.staticHelper(new namespace.MyClass());
};
```

除非是枚举类型, 不然不要访问别名变量的属性.

```js
/** @enum {string} */
some.long.namespace.Fruit = {
  APPLE: 'a',
  BANANA: 'b'
};

myapp.main = function() {
  var Fruit = some.long.namespace.Fruit;
  switch (fruit) {
    case Fruit.APPLE:
      // 业务块
    case Fruit.BANANA:
      // 业务块
  }
};
myapp.main = function() {
  var MyClass = some.long.namespace.MyClass;
  MyClass.staticHelper(null);
};
```

不要在全局范围内创建别名, 而仅在函数块作用域中使用.

#### 1.4.5 文件名

文件名应该使用小写字符, 以避免在有些系统平台上不识别大小写的命名方式. 文件名以 `.js` 结尾, 不要包含除 `-` 和 `_` 外的标点符号(使用 `-` 优于 `_` ).

## 二、符号

### 2.1 字符串
单引号 (') 优于双引号 ("). 当你创建一个包含 HTML 代码的字符串时就知道它的好处了.
```js
var msg = 'This is some HTML';
```

### 2.2 括号
只在需要的时候使用
不要滥用括号, 只在必要的时候使用它.

对于一元操作符(如 `delete`, `typeof` 和 `void` ), 或是在某些关键词(如 `return`, `throw`, `case`, `new` )之后, 不要使用括号.

### 2.3 空格与缩进
不要混用空格和Tab，代码索引全部使用2个空格进行缩进。

开始一个项目，在写代码之前，选择软缩进（空格）或者 Tab（作为缩进方式），并将其作为最高准则。
为了可读, 设计2个字母宽度的缩进 `—` 这等同于两个空格或者两个空格替代一个 Tab。

如果你的编辑器支持，请总是打开 “显示不可见字符” 这个设置。好处是：
- 保证一致性
- 去掉行末的空格
- 去掉空行的空格
- 提交和对比更具可读性

### 2.4 分号
总是使用分号.
如果仅依靠语句间的隐式分隔, 有时会很麻烦. 你自己更能清楚哪里是语句的起止.

而且有些情况下, 漏掉分号会很危险:

```js
// 1.
MyClass.prototype.myMethod = function() {
  return 42;
}  // No semicolon here.

(function() {
  // Some initialization code wrapped in a function to create a scope for locals.
})();

var x = {
  'i': 1,
  'j': 2
}  // No semicolon here.

// 2.  Trying to do one thing on Internet Explorer and another on Firefox.
// I know you'd never write code like this, but throw me a bone.
[normalVersion, ffVersion][isIE]();

var THINGS_TO_EAT = [apples, oysters, sprayOnCheese]  // No semicolon here.

// 3. conditional execution a la bash
-1 == resultOfOperation() || die();
```

这段代码会发生些什么诡异事呢?

报 JavaScript 错误: 例子1上的语句会解释成, 一个函数带一匿名函数作为参数而被调用, 返回42后, 又一次被"调用", 这就导致了错误.
例子2中, 你很可能会在运行时遇到 'no such property in undefined' 错误, 原因是代码试图这样 `x[ffVersion][isIE]()` 执行.
当 `resultOfOperation()` 返回非 `NaN` 时, 就会调用die, 其结果也会赋给 `THINGS_TO_EAT` .

为什么?        
JavaScript 的语句以分号作为结束符, 除非可以非常准确推断某结束位置才会省略分号. 上面的几个例子产出错误, 均是在语句中声明了函数/对象/数组直接量, 但 闭括号('}'或']')并不足以表示该语句的结束. 在 JavaScript 中, 只有当语句后的下一个符号是后缀或括号运算符时, 才会认为该语句的结束.
遗漏分号有时会出现很奇怪的结果, 所以确保语句以分号结束.

## 三、其他规范

### 3.1 自定义异常
有时发生异常了, 但返回的错误信息比较奇怪, 也不易读. 虽然可以将含错误信息的引用对象或者可能产生错误的完整对象传递过来, 但这样做都不是很好, 最好还是自定义异常类, 其实这些基本上都是最原始的异常处理技巧. 所以在适当的时候使用自定义异常.

### 3.2 块内函数声明
不要在块内声明一个函数
不要写成:
```js
if (x) {
  function foo() {}
}
```

虽然很多 JS 引擎都支持块内声明函数, 但它不属于 ECMAScript 规范 (见 ECMA-262, 第13和14条). 各个浏览器糟糕的实现相互不兼容, 有些也与未来 ECMAScript 草案相违背. ECMAScript 只允许在脚本的根语句或函数中声明函数. 如果确实需要在块中定义函数, 建议使用函数表达式来初始化变量:
```js
if (x) {
  var foo = function() {}
}
```

### 3.3 明确作用域
任何时候都要明确作用域 提高可移植性和清晰度. 例如, 不要依赖于作用域链中的 `window` 对象. 可能在其他应用中, 你函数中的 `window` 不是指之前的那个窗口对象.

### 3.4 Tips and Tricks
JavaScript 小技巧

#### 3.4.1 True 和 False 布尔表达式
下面的布尔表达式都返回 false:

- null
- undefined
- '' 空字符串
- 0 数字0
  但小心下面的, 可都返回 true:

- '0' 字符串0
- [] 空数组
- {} 空对象
  下面段比较糟糕的代码:

`while (x != null) {`

你可以直接写成下面的形式(只要你希望 `x` 不是 `0` 和空字符串, 和 `false`):

`while (x) {`

如果你想检查字符串是否为 `null` 或空:

`if (y != null && y != '') {`

但这样会更好:

`if (y) {`

注意: 还有很多需要注意的地方, 如:
```
Boolean('0') == true
'0' != true
0 != null
0 == []
0 == false
Boolean(null) == false
null != true
null != false
Boolean(undefined) == false
undefined != true
undefined != false
Boolean([]) == true
[] != true
[] == false
Boolean({}) == true
{} != true
{} != false
```

#### 3.4.2 条件(三元)操作符 (?:)
三元操作符用于替代下面的代码:
```js
if (val != 0) {
  return foo();
} else {
  return bar();
}

// 你可以写成

return val ? foo() : bar();
```

在生成 HTML 代码时也是很有用的:
```js
var html = '<input type="checkbox"' +
    (isChecked ? ' checked' : '') +
    (isEnabled ? '' : ' disabled') +
    ' name="foo">';
```

#### 3.4.3 && 和 ||
二元布尔操作符是可短路的, 只有在必要时才会计算到最后一项.

"||" 被称作为 'default' 操作符, 因为可以这样:
```js
/** @param {*=} opt_win */
function foo(opt_win) {
  var win;
  if (opt_win) {
    win = opt_win;
  } else {
    win = window;
  }
  // ...
}
```

你可以使用它来简化上面的代码:
```js
/** @param {*=} opt_win */
function foo(opt_win) {
  var win = opt_win || window;
  // ...
}
```

"&&" 也可简短代码.比如:
```js
if (node) {
  if (node.kids) {
    if (node.kids[index]) {
      foo(node.kids[index]);
    }
  }
}
```

你可以像这样来使用:

```js
if (node && node.kids && node.kids[index]) {
  foo(node.kids[index]);
}

// 或者

var kid = node && node.kids && node.kids[index];
if (kid) {
  foo(kid);
}
```

不过这样就有点儿过头了:

```
node && node.kids && node.kids[index] && foo(node.kids[index]);
```

#### 3.4.4 使用 join() 来创建字符串
通常是这样使用的:

```js
function listHtml(items) {
  var html = '<div class="foo">';
  for (var i = 0; i < items.length; ++i) {
    if (i > 0) {
      html += ', ';
    }
    html += itemHtml(items[i]);
  }
  html += '</div>';
  return html;
}
```

但这样在 IE 下非常慢, 可以用下面的方式:
```js
function listHtml(items) {
  var html = [];
  for (var i = 0; i < items.length; ++i) {
    html[i] = itemHtml(items[i]);
  }
  return '<div class="foo">' + html.join(', ') + '</div>';
}
```

你也可以是用数组作为字符串构造器, 然后通过 `myArray.join('')` 转换成字符串. 不过由于赋值操作快于数组的 `push()`, 所以尽量使用赋值操作.

#### 3.4.5 遍历 Node List

Node lists 是通过给节点迭代器加一个过滤器来实现的. 这表示获取他的属性, 如 length 的时间复杂度为 O(n), 通过 length 来遍历整个列表需要 O(n^2).
```js
var paragraphs = document.getElementsByTagName('p');
for (var i = 0; i < paragraphs.length; i++) {
  doSomething(paragraphs[i]);
}
```

这样做会更好:

```js
var paragraphs = document.getElementsByTagName('p');
for (var i = 0, paragraph; paragraph = paragraphs[i]; i++) {
  doSomething(paragraph);
}
```

这种方法对所有的 collections 和数组(只要数组不包含 falsy 值) 都适用.

在上面的例子中, 也可以通过 firstChild 和 nextSibling 来遍历孩子节点.
```js
var parentNode = document.getElementById('foo');
for (var child = parentNode.firstChild; child; child = child.nextSibling) {
  doSomething(child);
}
```

### 3.5 注释
使用 JSDoc 中的注释风格. 行内注释使用 `// 变量` 的形式. 另外, 我们也遵循 `C++` 代码注释风格 . 这也就是说你需要:

- 版权和著作权的信息.
- 文件注释中应该写明该文件的基本信息(如, 这段代码的功能摘要, 如何使用, 与哪些东西相关), 来告诉那些不熟悉代码的读者.
  类, 函数, 变量和必要的注释.
- 期望在哪些浏览器中执行.
- 正确的大小写, 标点和拼写.
- 为了避免出现句子片段, 请以合适的大/小写单词开头, 并以合适的标点符号结束这个句子.

目前很多编译器可从 `JSDoc` 中提取类型信息, 来对代码进行验证, 删除和压缩. 因此, 你很有必要去熟悉正确完整的 [JSDoc](https://github.com/jsdoc/jsdoc) .

#### 3.5.1 顶层/文件注释

顶层注释用于告诉不熟悉这段代码的读者这个文件中包含哪些东西.
一般应用于对外主路径index文件，或者主入口文件.
应该提供文件的大体内容, 它的作者, 依赖关系和兼容性信息. 如下:

```js
// Copyright 2009 Google Inc. All Rights Reserved.
/**
 * @fileoverview Description of file, its uses and information
 * about its dependencies.
 * @author user@google.com (Firstname Lastname)
 */

// 或者简化这样

/*!
 * Socket.IO v4.1.2
 * (c) 2014-2021 Guillermo Rauch
 * Released under the MIT License.
 */
```

#### 3.5.2 类注释
每个类的定义都要附带一份注释, 描述类的功能和用法. 也需要说明构造器参数. 如果该类继承自其它类, 应该使用 `@extends` 标记. 如果该类是对接口的实现, 应该使用 `@implements` 标记.

```js
/**
 * Class making something fun and easy.
 * @param {string} arg1 An argument that makes this more interesting.
 * @param {Array.<number>} arg2 List of numbers to be processed.
 * @constructor
 * @extends {goog.Disposable}
 */
project.MyClass = function(arg1, arg2) {
  // ...
};
goog.inherits(project.MyClass, goog.Disposable);
```

#### 3.5.3 方法与函数的注释

提供参数的说明. 使用完整的句子, 并用第三人称来书写方法说明.
```js
/**
 * Converts text to some completely different text.
 * @param {string} arg1 An argument that makes this more interesting.
 * @return {string} Some return value.
 */
project.MyClass.prototype.someMethod = function(arg1) {
  // ...
};

/**
 * Operates on an instance of MyClass and returns something.
 * @param {project.MyClass} obj Instance of MyClass which leads to a long
 *     comment that needs to be wrapped to two lines.
 * @return {boolean} Whether something occured.
 */
function PR_someMethod(obj) {
  // ...
}
```

对于一些简单的, 不带参数的 `Getters`, 说明可以忽略.

```js
/**
 * @return {Element} The element for the component.
 */
goog.ui.Component.prototype.getElement = function() {
  return this.element_;
};
```

#### 3.5.4 枚举
```js
/**
 * Enum for tri-state values.
 * @enum {number}
 */
project.TriState = {
  TRUE: 1,
  FALSE: -1,
  MAYBE: 0
};
```

注意一下, 枚举也具有有效类型, 所以可以当成参数类型来用.

```js
/**
 * Sets project state.
 * @param {project.TriState} state New project state.
 */
project.setState = function(state) {
  // ...
};
```

#### 3.5.5 Typedefs

有时类型会很复杂. 比如下面的函数, 接收 Element 参数:

```js
/**
 * @param {string} tagName
 * @param {(string|Element|Text|Array.<Element>|Array.<Text>)} contents
 * @return {Element}
 */
goog.createElement = function(tagName, contents) {
  // 业务块
};
```

你可以使用 @typedef 标记来定义个常用的类型表达式.

```js
/** @typedef {(string|Element|Text|Array.<Element>|Array.<Text>)} */
goog.ElementContent;

/**
* @param {string} tagName
* @param {goog.ElementContent} contents
* @return {Element}
*/
goog.createElement = function(tagName, contents) {
  // 业务块
};

```

## 四、JavaScript 类型
强烈建议去使用编译器.
如果使用 JSDoc, 那么尽量具体地, 准确地根据它的规则来书写类型说明. 目前支持两种 JS2 和 JS1.x 类型规范.

### 4.1 JavaScript 类型语言

JS2 提议中包含了一种描述 JavaScript 类型的规范语法, 这里我们在 JSDoc 中采用其来描述函数参数和返回值的类型.

JSDoc 的类型语言, 按照 JS2 规范, 也进行了适当改变, 但编译器仍然支持旧语法.

| 名称 | 语法 | 描述  |  弃用语法  |
| --- | --- | ---  |  --------  |
|普通类型|	`{boolean}, {Window}, {goog.ui.Menu}` |	普通类型的描述方法.| |	
|复杂类型|	`{Array.<string>}` 字符串数组. `{Object.<string, number>}` 键为字符串, 值为整数的对象类型. | 参数化类型, 即指定了该类型中包含的一系列"类型参数". 类似于 Java 中的泛型.	||
|联合类型|	`{(number / boolean)}` 一个整数或者布尔值.	| 表示其值可能是 A 类型, 也可能是 B 类型	| `{(number,boolean)}, {number / boolean}, {(number / boolean)}` |
|记录类型|	`\{\{myNum: number, myObject\}\}` 由现有类型组成的类型.	| 表示包含指定成员及类型的值. 这个例子中, myNum 为 number 类型, myObject 为任意类型. 注意大括号为类型语法的一部分. 比如, `Array.<{length}>` , 表示一具有 length 属性的 Array 对象.||
|可为空类型|	`{?number}` 一个整型数或者为 NULL|	表示一个值可能是 A 类型或者 null. 默认, 每个对象都是可为空的. 注意: 函数类型不可为空.|	`{number?}` |
|非空类型|	`{!Object}` 一个对象, 但绝不会是 null 值.|	说明一个值是类型 A 且肯定不是 null. 默认情况下, 所有值类型 `(boolean, number, string, 和 undefined)` 不可为空.	| `{Object!}` |
|函数类型|	`{function(string, boolean)}` 具有两个参数 `( string 和 boolean)` 的函数类型, 返回值未知.| 说明一个函数.||	
|函数返回类型|	`{function(): number}` 函数返回一个整数.| 说明函数的返回类型.	||
|函数的 this 类型|	`{function(this:goog.ui.Menu, string)}` 函数只带一个参数 (string), 并且在上下文 `goog.ui.Menu` 中执行.|	说明函数类型的上下文类型.	||
|可变参数|	`{function(string, ...[number]): number}` 带一个参数 (字符类型) 的函数类型, 并且函数的参数个数可变, 但参数类型必须为 number.|	说明函数的可变长参数.	||
|可变长的参数 (使用 `@param` 标记)	`@param {...number} var_args` 函数参数个数可变.	|使用标记, 说明函数具有不定长参数.	||
|函数的 缺省参数|	`{function(?string=, number=)}` 函数带一个可空且可选的字符串型参数, 一个可选整型参数. = 语法只针对 function 类型有效.|	说明函数的可选参数.	||
|函数 可选参数 (使用 `@param` 标记)|	`@param {number=} opt_argument` number类型的可选参数.	|使用标记, 说明函数具有可选参数.	||
|所有类型|	`{*}`	|表示变量可以是任何类型.	||

### 4.2 JavaScript中的类型

| 类型示例 | 值示例 |	描述 |
| ------- | ----- | ---- |
|number|	1   1.0     -5   1e5     Math.PI||
|Number|    `new Number(true)` | Number 对象 |
|string|	'Hello'          "World"            `String(42)` | 字符串值 |
|String|	`new String('Hello')  `           `new String(42)` | 字符串对象 |
|boolean|	true          false         `Boolean(0)` | 布尔值 |
|Boolean|	`new Boolean(true)` | 布尔对象 |
|RegExp|	`new RegExp('hello')`           `/world/g` ||
|Date|	new Date             new Date()||
|null|	null ||
|undefined|     undefined ||
|void	| `function f() { return; }` | 没有返回值 |
|Array|	`['foo', 0.3, null]`            [] | 类型不明确的数组 |
|`Array.<number>`	 | [11, 22, 33] | 整型数组 |
|`Array.<Array.<string>>`	 | `[['one', 'two', 'three'], ['foo', 'bar']]` | 字符串数组的数组 |
|Object	 | {}            `{foo: 'abc', bar: 123, baz: null}` ||
|`Object.<string>`	| `{'foo': 'bar'}` | 值为字符串的对象. |
|`Object.<number, string>`	| `var obj = {};`                   `obj[1] = 'bar';` | 键为整数, 值为字符串的对象.   注意, JavaScript 中, 键总是被转换成字符串, 所以 `obj['1'] == obj[1]` . 也所以, 键在 `for...in` 循环中是字符串类型. 但在编译器中会明确根据键的类型来查找对象. |
|Function	| `function(x, y) { return x * y; }` | 函数对象 |
|`function(number, number): number`	 | `function(x, y) { return x * y; }` | 函数值 |
|SomeClass |	`/** @constructor */              function SomeClass() {}     new SomeClass();` ||
|SomeInterface| 	`/** @interface */               function SomeInterface() {}          SomeInterface.prototype.draw = function() {};` ||
|project.MyClass|	`/** @constructor */              project.MyClass = function () {}        new project.MyClass()` ||
|project.MyEnum|	 `/** @enum {string} */               project.MyEnum = { BLUE: '#0000dd', RED: '#dd0000' };` | 枚举 |
|Element|	 `document.createElement('div')` | DOM 中的元素 |
|Node|	`document.body.firstChild` | DOM 中的节点. |
|HTMLInputElement|	`htmlDocument.getElementsByTagName('input')[0]` | DOM 中, 特定类型的元素. |



---
收录时间: 2021-09-28

<Vssue :title="$title" />
