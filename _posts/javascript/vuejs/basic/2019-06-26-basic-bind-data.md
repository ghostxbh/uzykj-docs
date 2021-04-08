---
title: Vue.js数据绑定
date: 2019-06-26
tags:
    - Vue.js
author: ghostxbh
location: blog
summary: Vue.js 使用了基于 HTML 的模板语法，允许开发者声明式地将 DOM 绑定至底层 Vue 实例的数据。
---
# 数据绑定

## 介绍
Vue.js 使用了基于 HTML 的模板语法，允许开发者声明式地将 DOM 绑定至底层 Vue 实例的数据。
所有 Vue.js 的模板都是合法的 HTML ，所以能被遵循规范的浏览器和 HTML 解析器解析。

在底层的实现上，Vue 将模板编译成虚拟 DOM 渲染函数。结合响应系统，Vue 能够智能地计算出最少需要重新渲染多少组件，并把 DOM 操作次数减到最少。

如果你熟悉虚拟 DOM 并且偏爱 JavaScript 的原始力量，你也可以不用模板，直接写渲染 (render) 函数，使用可选的 JSX 语法。

### 文本
数据绑定最常见的形式就是使用“Mustache”语法 (双大括号) 的文本插值：

```vue
<span>Message: {{ msg }}</span>
```

Mustache 标签将会被替代为对应数据对象上 msg 属性的值。无论何时，绑定的数据对象上 msg 属性发生了改变，插值处的内容都会更新。
通过使用 v-once 指令，你也能执行一次性地插值，当数据改变时，插值处的内容不会更新。但请留心这会影响到该节点上的其它数据绑定：

```vue
<span v-once>这个将不会改变: {{ msg }}</span>
```

### 原始 HTML

双大括号会将数据解释为普通文本，而非 HTML 代码。为了输出真正的 HTML，你需要使用 v-html 指令：

```vue
<p>Using mustaches: {{ rawHtml }}</p>
<p>Using v-html directive: <span v-html="rawHtml"></span></p>
```

这个 span 的内容将会被替换成为属性值 rawHtml，直接作为 HTML——会忽略解析属性值中的数据绑定。注意，你不能使用 v-html 来复合局部模板，因为 Vue 不是基于字符串的模板引擎。反之，对于用户界面 (UI)，组件更适合作为可重用和可组合的基本单位。
你的站点上动态渲染的任意 HTML 可能会非常危险，因为它很容易导致 XSS 攻击。请只对可信内容使用 HTML 插值，绝不要对用户提供的内容使用插值。

### 特性

Mustache 语法不能作用在 HTML 特性上，遇到这种情况应该使用 v-bind 指令：

```vue
<div v-bind:id="dynamicId"></div>
```

对于布尔特性 (它们只要存在就意味着值为 true)，v-bind 工作起来略有不同，在这个例子中：

```vue
<button v-bind:disabled="isButtonDisabled">Button</button>
```

如果 isButtonDisabled 的值是 null、undefined 或 false，则 disabled 特性甚至不会被包含在渲染出来的 `<button>` 元素中。

### JavaScript 表达式

迄今为止，在我们的模板中，我们一直都只绑定简单的属性键值。但实际上，对于所有的数据绑定，Vue.js 都提供了完全的 JavaScript 表达式支持。

```vue
{{ number + 1 }}

{{ ok ? 'YES' : 'NO' }}

{{ message.split('').reverse().join('') }}

<div v-bind:id="'list-' + id"></div>
```

这些表达式会在所属 Vue 实例的数据作用域下作为 JavaScript 被解析。有个限制就是，每个绑定都只能包含单个表达式，所以下面的例子都不会生效。

```vue
<!-- 这是语句，不是表达式 -->
{{ var a = 1 }}

<!-- 流控制也不会生效，请使用三元表达式 -->
{{ if (ok) { return message } }}
```

模板表达式都被放在沙盒中，只能访问全局变量的一个白名单，如 Math 和 Date 。你不应该在模板表达式中试图访问用户定义的全局变量。

## 代码示例

### 1.vue结构 
HTML所有的代码需有一个根包裹起来 `<div id="app">`

```vue
<!-- html模版 -->
<template>
    <div id="app">
        <h3>{{msg}}</h3>
    </div>
</template>

<!-- js业务 -->
<script>
    export default {
        name: 'app',
        data() {
            return {
                msg: 'vue demo',
            }
        }
    }
</script>

<!-- css样式 -->
<style>
    #app {
        font-family: 'Avenir', Helvetica, Arial, sans-serif;
    }
</style>
```

### 2.数据绑定
```vue
<!-- js业务逻辑 -->
<script>
    export default {
        name: 'app',
        data() {
            return {
                msg: 'vue demo',//返回字符
                obj: {          //返回对象
                    name: '张麻子',
                    age: 35,
                    job: '麻匪'
                },
                //返回数组
                arr:['一筒','二筒','三筒','四筒'],
            }
        }
    }
</script>

<!-- html数据绑定 -->
<template>
    <div id="app">
        <h3>{{msg}}</h3>
        <br/>
        <h3>{{obj.name}}的个人简介</h3>
        <ul>
            <li>姓名：{{obj.name}}</li>
            <li>年龄：{{obj.age}}</li>
            <li>职业：{{obj.job}}</li>
        </ul>
        <hr/>
        <h3>{{obj.name}}的手下</h3>
        <ul>
            <li v-for="item in arr">{{item}}</li>
        </ul>
    </div>
</template>
```

### 3.复杂数组
```vue
<!-- js数据 -->
<script>
    export default {
        name: 'app',
        data() {
            return {
                list: [
                    {
                        name: '图书馆',
                        category: [
                            {
                                name: '文学',
                                num: 2800,
                            },
                            {
                                name: '理学',
                                num: 7908,
                            }
                        ]
                    },
                    {
                        name: '天文馆',
                        goods: [
                            {
                                good: '望远镜A2',
                                caliber: 10,
                            },
                            {
                                good: '望远镜L8',
                                caliber: 30,
                            },
                        ]
                    },
                ]
            }
        }
    }
</script>

<!-- html数据绑定 -->
<h3>建筑类</h3>
<ul>
    <span v-for="i in list">名称：{{i.name}}
        <span v-for="j in i.category">
            <li>名称：{{j.name}}</li>
            <li>数量：{{j.num}}</li>
        </span>
        <span v-for="j in i.goods">
            <li>名称：{{j.good}}</li>
            <li>数量：{{j.caliber}}cm</li>
        </span>
    </span>
</ul>
```


## 参考资料
[模版语法](https://cn.vuejs.org/v2/guide/syntax.html)

---
收录时间: 2019-06-26

<Vssue :title="$title" />
