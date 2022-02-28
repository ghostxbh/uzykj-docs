---
title: 基础特性和类型推导
date: 2022-02-28
sidebar: 'auto'
categories:
  - Javascript
tags:
  - TypeScript
  - TS
  - 基础特性
author: tasaid
location: blog
summary: 从前几年热门的 MVC 一直到现在热门的 MVVM，我们发现无论是 MVC(Model-View-Controller) 还是 MVVM(Model-View-ViewModel)，我们始终抛不开一个关键的地方 —— 数据层：Model。
---
  
# 基础特性和类型推导
## 基础类型
我们先简单的声明一些变量：
```typescript
let a: number
let b = true  // 有默认值的情况，甚至不需要声明类型，ts 会自动推导
let c: [string, number] // 元组
enum Color {Red, Green, Blue} // 枚举
let d: { name: string } = { name: 'linkFly' }
```

当我们给这些变量赋错误的类型值的时候，会抛出类型错误异常。

![错误提示](https://static.tasaid.com/blogs/47d85e4a5d954994c4cac8d15682d127.png)

是不是很简单，TypeScript 优秀的设计使得即使你没有接触过它，但是仍然能够读懂它。

## 复杂类型
```typescript
// array
let list_a: number[] = [1, 2, 3]
let list_b: Array<number> = [1, 2, 3] // number 类型的数组
let list_c: [string, number] = ['linkFly', 0]

// any
let notSure: any = 4
notSure = true // any 类型可以自由赋值

// 函数类型
let fn: (id: string) => number = (id) => 1
// 这里使用了 ECMAScript 6 的箭头函数，和下面的代码等价
let fn: (id: string) => number = function (id) {
return 1
}
```

![类型推导](https://static.tasaid.com/blogs/82b705a6a6154a2aedfb66bb6a3b8f84.png)

## 高级类型
```typescript
// 联合类型, foo 是 string 或 number
let foo: string | number = 1

// 类型断言，强制使用兼容类型中的某一类型
(foo as string)

// 类型保护(判断)
if (typeof foo === 'string') {
// dosomething
}
// 类型保护(判断)
if (foo instanceof String) {
// dosomething
}
```

![类型保护](https://static.tasaid.com/blogs/205cbccb401525d6016ec0cf0eaa31f1.jpeg)

## Model
从前几年热门的 `MVC` 一直到现在热门的 `MVVM`，我们发现无论是 `MVC(Model-View-Controller)` 还是 `MVVM(Model-View-ViewModel)`，我们始终抛不开一个关键的地方 —— 数据层：`Model`。

因为本质上整个页面的操作都是在进行数据流动，页面展现本质上都是数据，而我们通过 Model 来描述数据。

这是一个简单的 `Model` 演示：
```typescript
let user : { id: number, name: string } = { id: 1, name: 'linkFly' }
```

在 TypeScript （或者是所有强 OO 语言）中，推荐以 `Model` 来描述数据的方式也就是 `Class`。

这一小节只简单介绍 `Class` 和 泛型，实际项目中可能还会牵扯更多更强大的 OO 概念：接口、抽象类、继承类、继承属性。

这些知识不是一蹴而就的，而是需要在项目中不断探索不断组合的。

### Class 类
所有类型的根本都是类，TS 中声明一个类的语法非常简单，可读性很高。

注意，TS 中类型是核心，当你想把一个项目从 JavaScript 迁移到 TypeScript 的时候，需要为项目中补充大量的类型，而这些类型大部分都是基于 `Class` 构建的。

这是一个简单的类：
```typescript
class User {
    id: number
    name: string
}

let user: User = { id: 1, name: 'linkFly' }
```

当然随着需求的不同，也可以补充很多细节：
```typescript
class User {
// 只读属性
    readonly id: number
    
    // 存取器, get/set
    private _name: string
    get name(): string {
        // dosomething
        return this._name
    }
    set name (name: string) {
        console.log('this is set method')
        // dosomething
        this._name = name
    }
    
    // 构造函数
    constructor (id: number, theName: string) {
        // 只读属性只能在构造函数里初始化
        this.id = id
        this._name = theName
    }
    
    // 实例方法
    say () {
        console.log(`name: ${this.name}`)
    }
    
    // 静态方法（类方法）
    static print () {
        console.log('static method')
    }
}

let user = new User(1, 'linkFly')
user.name = 'tasaid' // 会输出 'this is set method'
user.say() // 实例方法
User.print() // 静态方法
```

![Model_User](https://static.tasaid.com/blogs/60f27dab245dc4d56c84b445aef2087f.png)


### 泛型
泛型是用来解决类型重用的问题。

例如下面一个函数，只能传递 `number` 的参数并返回：
```typescript
function identity(arg: number): number {
    // dosomething
    return arg
}
```

现在想传递一个 `string` 类型的参数，然后也返回它，这个时候就可以使用泛型，使用泛型可以接收任意类型并返回：
```typescript
// 这个 T 就是泛型，也可以叫其他名字
function identity<T>(arg: T): T {
    // dosomething
    return arg
}

identity<string>('linkfly')
identity('linkfly') // 自动推导
identity(0)
identity(true)
```

我们可以轻松的使用泛型来实现数据包装：
```typescript
function fetch<T>(url: string): Promise<T> {
// 远程请求数据并返回结果
    return http(url).then(data => {
        return data as T
    })
}

class User {
    name: string
}

// 泛型使用
let user = fetch<User>('https://tasaid.com/user')
```

## Tips
[TypeScript 中文网](https://tslang.cn/docs/handbook/decorators.html) 可以看到完整 TS 类型。
在项目初期使用 TS 中，会需要很大的时间和精力，去编写和架构基本业务类型(Models)，在此之后会越来越方便快捷。
一些没有行为只需要做类型检查的类型（没有方法的 Models），可以使用 [TypeScript 声明文件 （*.d.ts）](https://tslang.cn/docs/handbook/declaration-files/introduction.html) ，例如：
```typescript
declare namespace Models {
    interface GPS {
        lat: number
        lng: number
    }
}
```

这个系列的文章不会讲解 TypeScript 的声明文件，但是它是 TypeScript 中不可缺少的一部分。

尽量减少使用 `any` 类型，它意味着类型不可控
某些变量或者第三方库中属性无法感知，使用 `as` 强制进行类型推导即可。
```typescript
window.tempName = 'linkFly'
// Errors: [ts] Property 'tempName' does not exist on type 'Window'.

// 强制推导
(window as any).tempName = 'linkFly'
```

## Resources
- [TypeScript - 基础类型](https://www.tslang.cn/docs/handbook/basic-types.html)
- [TypeScript - 规范](https://www.tslang.cn/docs/handbook/declaration-files/do-s-and-don-ts.html)

---
收录时间: 2022-02-28

<Vssue :title="$title" />