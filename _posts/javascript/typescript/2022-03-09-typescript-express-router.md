---
title: Express 路由进化
date: 2022-03-09
sidebar: 'auto'
categories:
  - Javascript
tags:
  - TypeScript
  - TS
  - express
  - 路由
author: tasaid
location: blog
summary: 实现路由的配置通过装饰器实现，并且实现一层路由逻辑的封装。
---
# Express 路由进化
## express 路由
首先我们来看一个简单的 express 路由 (router):
```javascript
// 对网站首页的访问返回 "Hello World!"
app.get('/', function (req: Request, res: Response) {
    res.send('Hello World!')
});

app.post('/user', function (req: Request, res: Response) {
    res.send(`User Id ${req.query.id}`)
})
```

在上面的路由代码我们演示了一个普通流水线式的路由。

基于上一篇文章中我们学到的装饰器和反射的知识，我们将要实现**路由的配置通过装饰器实现**，并且实现一层路由逻辑的封装。

## 路由进化
基于装饰器和反射，我们要实现的路由最终效果是这样的：
```typescript
class Home {
    @path('/user')
    @httpGet
    user (id: string) {
        return `User Id ${id}`
    }
}
```
```
GET  HTTP/1.1
Host: /user?id=uzykj.com
```

这段代码相比传统的路由配置，优点如下：

- 将路由的配置抽离成为了装饰器，让整个 `router` 函数内部只需要处理业务逻辑即可，路由配置简单明了
- 隐藏 `req` 和 `res`，每个 `router` 直接返回结果即可，无需自己再输出结果

### 装饰器: HTTP Method
我们先编写 `HTTP Method` 的装饰器，我们将实现两个装饰器，分别叫做 `httpGet` 和 `httpPost`，对应 HTTP Method 的 `GET/POST`。

原理上，我们会将 `router` 配置的数据都挂到使用装饰器的方法上。
```typescript
import 'reflect-metadata'

export const symbolHttpMethodsKey = Symbol("router:httpMethod")

export const httpGet = function (target: any, propertyKey: string) {
    // 挂载到调用装饰器的方法上
    Reflect.defineMetadata(symbolHttpMethodsKey, 'get', target, propertyKey)
}

export const httpPost = function (target: any, propertyKey: string) {
    Reflect.defineMetadata(symbolHttpMethodsKey, 'post', target, propertyKey)
}
```

### 装饰器: path
有了上面 HTTP Method 装饰器的实现，我们再实现 `path` 装饰器将会很简单。

当然，我们还可以在 `path` 中实现对原方法的封装：隐藏 `req` 和 `res`，并对 `router` 的输出结果进行封装。

注意这里使用的是装饰器工厂：
```typescript
import 'reflect-metadata'

export const symbolPathKey = Symbol.for('router:path')

export let path = (path: string): Function => {
    return function (target: any, propertyKey: string, descriptor: TypedPropertyDescriptor<Function>) {
        Reflect.defineMetadata(symbolPathKey, path, target, propertyKey)
        if (!descriptor.value) return
        // 覆盖掉原来的 router method，在外层做封装
        let oldMethod = descriptor.value
        descriptor.value = function (req: Request, res: Response) {
            const params = Object.assign({}, req.body, req.query)
            let methodResult = oldMethod.call(this, params)
            // 输出返回结果
            res.send(methodResult)
        }
    }
}
```

### Router? Controller!
现在，我们需要将所有的 `Router` 按照自己的业务规则/或者自定义的其他规则进行归类 —— 然后提取出对应的 `Class`，例如下面的 `User Class` 就是把用户信息所有的 `router` 都归类在一起：
```typescript
class User {
    @httpPost
    @path('/user/login')
    login() { }
    
    @httpGet
    @path('/user/exit')
    exit() { }
}
```

然后在 `express` 配置的入口逻辑那里，把 `class` 对应的方法遍历一遍，然后使用 `reflect-metadata` 反射对应的 `router` 配置即可：
```typescript
import 'reflect-metadata'
// 装饰器挂载数据的 key
import { symbolHttpMethodsKey, symbolPathKey } from './decorators'

const createController = (app: Express) => {
    let user = new User()
    for (let methodName in user) {
        let method = user[methodName]
        if (typeof method !== 'function') break
        // 反射得到挂载的数据
        let httpMethod = Reflect.getMetadata(symbolHttpMethodsKey, user, methodName)
        let path = Reflect.getMetadata(symbolPathKey, user, methodName)
        
        // app.get('/', () => any)
        app[httpMethod](path, method)
    }
}
```

至此，我们的 `express` 路由进化完毕，效果如下：

![路由进化](https://static.tasaid.com/blogs/1e9c56855d825cd3c6e22af5553f4840.png)

完整的例子可以参考 [Github](https://github.com/linkFly6/ts-router-to-controller) 。

## 结语
装饰器目前在 ECMAScript 新提案中的 [建议征集的第二阶段(Stage 2)](https://github.com/tc39/proposal-decorators) ，由于装饰器在其他语言中早已实现，例如 Java 的注解(Annotation) 和 C# 的特性(Attribute)，所以纳入 ECMAScript 规范只是时间问题了。

装饰器来装饰路由，并且封装 `router` 操作的的思路缘起 `.NET MVC` 架构：

![.NET MVC](https://static.tasaid.com/blogs/3a2e5a58317f1e155f34684febcd921f.png)

`angular 2.x` 使用也引入了装饰器作为核心开发，随着规范的推进，相信装饰器进入大家视野，应用的场景也会越来越多。

---
收录时间: 2022-03-09

<Vssue :title="$title" />