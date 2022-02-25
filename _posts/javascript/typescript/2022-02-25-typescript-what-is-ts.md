---
title: 什么是 TypeScript
date: 2022-02-25
sidebar: 'auto'
categories:
  - Javascript
tags:
  - TypeScript
  - TS
  - 静态类型
author: tasaid
location: blog
summary: TypeScript 简称 TS。TypeScript 是 JavaScript 的超集，就是在 JavaScript 上做了一层封装，封装出 TypeScript 的特性，当然最终代码可以编译为 JavaScript。
---
  
# 什么是 TypeScript

## 为什么选择 TypeScript
几个月前，团队里觉得很多项目跑的越来越臃肿，团队协作遇到了一个很大的难题：
如何让一个别人提供的方法产出一个一目了然的文档？因为一个项目总会涉及到多人协作：
**同学 A** 编写了函数 `a()`，而 **同学 B** 在调用函数 `a()` 的时候得一直撸着 API 文档才能知道 `a()` 需要什么参数，会返回什么参数。

而 **同学 A** 后续又改动了函数 `a()`，但是却忘记了更新文档，这时候新接手项目的 **同学 C** 看着 API 文档和函数 a() 一脸懵逼，问题浮出水面：团队协作中，提供的接口如何描述自身？

这其中涉及到的问题有：

接口如何描述自己的参数和返回值？
接口参数和返回值在无数需求迭代中改变了多次，而这个 API 对应的文档该如何更新？
数据格式如何描述
我们意识到 JavaScript 在逐渐复杂的 Web 应用中缺少一个很重要的东西：**静态类型**。

因为需要有静态类型我们才可以知道这个函数需要的参数是什么，有什么类型，返回值是什么类型，哪些字段是可能空，哪些字段又需要什么样的格式。

我们首先找到了业界融合于 JavaScript 的方案：[Flow](https://zhenyong.github.io/flowtype/)

后来对比之后发现：

- 1、生态圈，TypeScript 的生态圈明显在 Flow 之上，各大主流类库都提供了 TypeScript 的类型声明文件，配合 visual studio code 编码体验上比 Flow 强太多。
- 2、Flow 是侵入 JavaScript 的，对 JavaScript 做了一层增强；而 TypeScript 则为 JavaScript 超集，在 JavaScript 之上进行的语言抽象，最终编译成 JavaScript。

其中第二点很重要，因为团队成员对添加了 Flow 的 JavaScript 语法想当排斥，而我个人同样觉得 Flow 侵入 JavaScript 太过强烈，不如 TypeScript 直接做语言超集好，虽然二者出发点上没有太大区别，但从设计思想上来说 TypeScript 更吸引人。

TypeScript 的定位是做静态类型语言，而 Flow 的定位是类型检查器。

毕竟写着 Flow 的时候心里想的是我在写 JavaScript，而写 TypeScript 心里想的是我在写 TypeScript 🤣。

## 什么是 TypeScript
TypeScript 简称 TS。TypeScript 是 JavaScript 的超集，就是在 JavaScript 上做了一层封装，封装出 TypeScript 的特性，当然最终代码可以编译为 JavaScript。

TypeScript 早期的目标是为了让习惯编写强类型语言的后端程序员，能够快速的编写出前端应用（微软大法好），因为 JavaScript 没有强数据类型，所以 TypeScript 提供了静态数据类型，这是 TypeScript 的核心。

随着项目工程越来越大，越来越多的前端意识到静态数据类型的重要性，随着 TypeScript 的逐渐完善，支持者越来越多，静态数据类型的需求越来越强。于此同时， angular 2.x 这个领头羊率先使用 AtScript 开辟了静态数据类型战场。

JavaScript 行至今日，灵活，动态让它活跃在编程语言界一线。而灵活，动态使得它又十分神秘，只有运行才能得到答案。类型的补充填充了 JavaScript 的缺点，从 TypeScript 编译到 JavaScript，多了静态类型检查，而又保留了 JavaScript 的灵活动态。

简单来说：动态代码一时爽，重构全家火葬场。

### 静态类型
TypeScript 凭借 Microsoft 深厚的语言设计功底，设计的十分优雅和简单易用，学习成本非常低。

上面我们所说了，TypeScript 的核心就是静态数据类型，我们来简单了解一下静态数据类型和简单的类型推导，TypeScript 是以 *.ts 作为文件后缀的，我们创建一个 demo.ts 文件，写下这段代码：
```typescript
let num: number
```
从上面的代码中，我们可以知道变量 num 是 number 类型的，如果我们给 num 赋其他类型的值，则会报错：

![](https://static.tasaid.com/blogs/8e5603f8b97de041f81698251c5cc94a.jpeg)

是不是很简单？是的，这就是 TypeScript 的核心。

我们再来看看一个函数该如何表达：
```typescript
const fetch = function (url: string): Promise { }
```
`fetch()` 函数接收一个 `string` 类型的参数 url，返回一个 `Promise`。

以下是一个 JavaScript 的函数，不看方法内的写法我们完全不知道这个 API 会有哪些坑。
```javascript
export const fetch = function (url, params, user) {
    // dosomething
    
    return http(options).then(data => {
        return data
    }).catch(err => {
        return err
    })
}
```

这是 TypeScript 的写法：
```typescript
export const fetch = function (url: string | object, params?: any, user?: User): Promise<object | Error> {
    // dosomething

    return http(options).then(data => {
        return data
    }).catch(err => {
        return err
    })
}
```

这个 TypeScript 包含了很多信息：

- 1、`url` 可能是 `string` 或 `object` 类型
- 2、`params` 是可以不传的，也可以传递任何类型
- 3、`user` 要求是 `User` 类型的，当然也是可以不传
- 4、返回了一个 `Promise`，`Promise` 的求值结果可能是 `object`，也有可能是 `Error`

看到上面的信息后，我们大概知道可以这么调用并处理 `fetch` 的返回结果：

```typescript
let result = await fetch('https://uzykj.com', { id: 1 })

// fetch 可能会返回 Error
if (result instanceof Error) {
// 错误处理
}
```

是不是很有意思？TypeScript 在说话，TypeScript 在让代码描述自身。

![](https://static.tasaid.com/blogs/0488223f3b2ffbc24eb68ae4c6197540.png)

这就是静态数据类型的意义。静态类型在越复杂的应用中，需求越强烈。

这是 `react` 对于数据类型的约束：
```javascript
import PropTypes from 'prop-types'

component.propTypes = {
    optionalArray: PropTypes.array,
    optionalBool: PropTypes.bool,
    optionalFunc: PropTypes.func,
    requiredFunc: PropTypes.func.isRequired,
}
```

这是 `vue` 对于数据类型的约束：
```vue
Vue.component('component', {
    props: {
        optionalArray: Array,
        optionalBool: Boolean,
        optionalFunc: Function,
        requiredFunc: {
            type: Function,
            required: true
        }
    }
})
```

而引入了 `TypeScript` 之后，就会感受到真正流畅的数据类型约束：
```typescript
class Component {
    optionalArray?: Array<string> // string 类型的 数组
    optionalBool?: boolean // 写上 ? 号，就表示着这个属性可能为空
    optionalFunc?: (foo: string, bar: number) => boolean // 函数的参数，返回值都一目了然
    requiredFunc: () => void
}
```

## 和 ECMAScript 相比
TypeScript 是 JavaScript 的超集，目标也是对齐 ECMAScript 的标准规范和提案对齐，最终 TypeScript 也是编译为 JavaScript。

同时，和 JavaScript 规范标准 [ECMAScript 提案](https://github.com/tc39/ecma262) 相比，TypeScript 也一直在跟进 ECMAScript 的许多新特性。

例如当前来说比较深受大家喜爱的新特性：

- `import()` 动态表达式
- `decorators` 装饰器
- `async/await` 异步

而这些都可以编译到 `ECMAScript 3`（少数细节存在兼容性问题）。

## Tips
- 使用 [visual studio code](https://code.visualstudio.com/) 编辑器会体验到 TypeScript 强大的类型推导，毕竟两个都是微软亲儿子
- 一些 JavaScript 编写的大型的第三方库，都提供了 TypeScript 的类型声明文件（`*.d.ts` 文件）, 一般都放在包目录的 `types` 文件夹中。或者在 `@types/*` 仓库名下可以找到
- `babel` 是将高级版本的 JavaScript 编译为目标版本的 JavaScript，TypeScript 是将 TypeScript 编译为目标版本的 JavaScript。它们的编译是重叠的，也就是说 TypeScript 可以不再依赖 `babel` 编译。


## 资源
- TypeScript 中文网：[https://tslang.cn/](https://tslang.cn/)
- TypeScript 视频教程：[《TypeScript 精通指南》](https://nodelover.me/course/ts-basic)

---
收录时间: 2022-02-25

<Vssue :title="$title" />