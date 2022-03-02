---
title: 装饰器和反射
date: 2022-03-02
sidebar: 'auto'
categories:
  - Javascript
tags:
  - TypeScript
  - TS
  - 装饰器
  - 反射
author: tasaid
location: blog
summary: 顾名思义，"装饰器" (也叫 "注解")就是对一个类/方法/属性/参数的装饰。它是对这一系列代码的增强，并且通过自身描述了被装饰的代码可能存在的行为改变。
---
# 装饰器和反射
## 前言
在了解装饰器之前，我们先看一段代码：
```typescript
class User {
    name: string
    id: number
    
    constructor(name:string, id: number) {
        this.name = name
        this.id = id
    }
    
    changeName (newName: string) {
        this.name = newName
    }
}
```

这段代码声明了一个 Class 为 `User`，`User` 提供了一个实例方法 `changeName()` 用来修改字段 `name` 的值。

现在我们要在修改 `name` 之前，先对 `newName` 做校验，判断如果 `newName` 的值为空字符串，就抛出异常。

按照我们过去的做法，我们会修改 `changeName()` 函数，或者提供一个 `validaName()` 方法：
```typescript
class User {
    name: string
    id: number
    
    constructor(name:string, id: number) {
        this.name = name
        this.id = id
    }
    // 验证 Name
    validateName (newName: string) {
        if (!newName){
            throw Error('name is invalid')
        }
    }
    changeName (newName: string) {
        // 如果 newName 为空字符串，则会抛出异常
        this.validateName(newName)
        this.name = newName
    }
}
```

可以看到，我们新编写的 `validateName()`，侵入到了 `changeName()` 的逻辑中。如此带来一个弊端：

我们不知道 `changeName()` 里面可能还包含了什么样的隐性逻辑
`changeName()` 被扩展后逻辑不清晰
然后我们把调用时机从 `changeName()` 中抽出来，先调用 `validateName()`，再调用 `changeName()`：
```typescript
let user = new User('linkFly', 1)
    if (user.validateName('tasaid')) {
    user.changeName('tasaid')
}
```

但是上面的问题 1 仍然没有被解决，调用方代码变的十分啰嗦。那么有没有更好的方式来表现这层逻辑呢？

装饰器就用来解决这个问题："无侵入式" 的增强。

## 装饰器
顾名思义，"装饰器" (也叫 "注解")就是对一个 **类/方法/属性/参数** 的装饰。它是对这一系列代码的增强，并且通过自身描述了被装饰的代码可能存在的行为改变。

简单来说，装饰器就是对代码的描述。

由于装饰器是实验性特性，所以要在 `tsconfig.json` 里启用这个实验性特性：
```json
{
    "compilerOptions": {
        // 支持装饰器
        "experimentalDecorators": true,
    }
}
```

钢铁侠托尼·史塔克只是一个有血有肉的人，而他的盔甲让他成为了钢铁侠，盔甲就是对托尼·史塔克的装饰(增强)。

我们使用装饰器修改一下上面的例子：
```typescript
// 声明一个装饰器，第三个参数是 "成员的属性描述符"，如果代码输出目标版本(target)小于 ES5 返回值会被忽略。
const validate = function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    // 保存原来的方法
    let method = descriptor.value
    // 重写原来的方法
    descriptor.value = function (newValue: string) {
        // 检查是否是空字符串
        if (!newValue) {
            throw Error('name is invalid')
        } else {
            // 否则调用原来的方法
            method.call(this, newValue)
        }
    }
}

class User {
    name: string
    id: number
    constructor(name:string, id: number) {
        this.name = name
        this.id = id
    }

    // 调用装饰器
    @validate
    changeName (newName: string) {
        this.name = newName
    }
}
```

这里我们可以看到，`changeName` 的逻辑没有任何改变，但其实它的行为已经通过装饰器 `@validate` 增强。

这就是装饰器的作用。装饰器可以用很直观的方式来描述代码：
```typescript
class User {
    name: string
    
    @validateString
    set name (@required name: string) {
        this.name = name
    }
}
```

## 装饰器工厂
装饰器的执行时机如下：
```typescript
// 这是一个装饰器工厂，在外面使用 @god() 的时候就会调用这个工厂
function god(name: string) {
    console.log(`god(): evaluated ${name}`)
    // 这是装饰器，在 User 生成之后会执行
    return function (target, propertyKey: string, descriptor: PropertyDescriptor) {
        console.log('god(): called')
    }
}

class User {
    @god('test')
    test () { }
}
```

以上代码输出结果
```
god(): evaluated test
god(): called
```

我们也可以直接声明一个装饰器来使用（要注意和装饰器工厂的区别）：
```typescript
function god(target, propertyKey: string, descriptor: PropertyDescriptor) {
    console.log("god(): called")
}


class User {
    // 注意这里不是 @god()，没有 ()
    @god
    test () { }
}
```

## 装饰器全家族
装饰器家族有 4 种装饰形式，**注意，装饰器能装饰在类、方法、属性和参数上，但不能只装饰在函数上！**

### 类装饰器
类装饰器表达式会在运行时当作函数被调用，类的构造函数作为其唯一的参数。
```typescript
function sealed(constructor: Function) {
    Object.seal(constructor)
    Object.seal(constructor.prototype)
}

@sealed
class User { }
```

### 方法装饰器
方法装饰器表达式会在运行时当作函数被调用，传入下列 **3个参数**：

- 1、对于静态成员来说是类的构造函数，对于实例成员是类的原型对象
- 2、成员的名字
- 3、成员的属性描述符 `{value: any, writable: boolean, enumerable: boolean, configurable: boolean}`
```typescript
function god(name: string) {
    return function (target, propertyKey: string, descriptor: PropertyDescriptor) {
        // target: 对于静态成员来说是类的构造函数，对于实例成员是类的原型对象
        // propertyKey: 成员的名字
        // descriptor: 成员的属性描述符 {value: any, writable: boolean, enumerable: boolean, configurable: boolean}
    }
}

class User {
    @god('tasaid.com')
    sayHello () { }
}
```

### 访问器装饰器
和函数装饰器一样，只不过是装饰于访问器上的。
```typescript
function god(name: string) {
    return function (target, propertyKey: string, descriptor: PropertyDescriptor) {
        // target: 对于静态成员来说是类的构造函数，对于实例成员是类的原型对象
        // propertyKey: 成员的名字
        // descriptor: 成员的属性描述符 {value: any, writable: boolean, enumerable: boolean, configurable: boolean}
    }
}

class User {
    private _name: string
    // 装饰在访问器上
    @god('tasaid.com')
    get name () {
        return this._name
    }
}
```

### 属性装饰器
属性装饰器表达式会在运行时当作函数被调用，传入下列 **2个参数**：

- 1、对于静态成员来说是类的构造函数，对于实例成员是类的原型对象
- 2、成员的名字
```typescript
function god(target, propertyKey: string) {
    // target: 对于静态成员来说是类的构造函数，对于实例成员是类的原型对象
    // propertyKey: 成员的名字
}

class User {
    @god
    name: string
}
```

### 参数装饰器
参数装饰器表达式会在运行时当作函数被调用，传入下列 **3个参数**：

- 1、对于静态成员来说是类的构造函数，对于实例成员是类的原型对象
- 2、成员的名字
- 3、参数在函数参数列表中的索引
```typescript
const required = function (target, propertyKey: string, parameterIndex: number) {
    // target: 对于静态成员来说是类的构造函数，对于实例成员是类的原型对象
    // propertyKey: 成员的名字
    // parameterIndex: 参数在函数参数列表中的索引
}

class User {
    private _name : string;
    set name(@required name : string) {
        this._name = name;
    }
}
```

例如上面 `validate` 的例子可以用在参数装饰器上
```typescript
// 定义一个私有 key
const requiredMetadataKey = Symbol.for('router:required')

// 定义参数装饰器，大概思路就是把要校验的参数索引保存到成员中
const required = function (target, propertyKey: string, parameterIndex: number) {
    // 属性附加
    const rules = Reflect.getMetadata(requiredMetadataKey, target, propertyKey) || []
    rules.push(parameterIndex)
    Reflect.defineMetadata(requiredMetadataKey, rules, target, propertyKey)
}

// 定义一个方法装饰器，从成员中获取要校验的参数进行校验
const validateEmptyStr = function (target, propertyKey: string, descriptor: PropertyDescriptor) {
    // 保存原来的方法
    let method = descriptor.value
    // 重写原来的方法
    descriptor.value = function () {
        let args = arguments
        // 看看成员里面有没有存的私有的对象
        const rules = Reflect.getMetadata(requiredMetadataKey, target, propertyKey) as Array<number>
        if (rules && rules.length) {
            // 检查私有对象的 key
            rules.forEach(parameterIndex => {
                // 对应索引的参数进行校验
                if (!args[parameterIndex]) throw Error(`arguments${parameterIndex} is invalid`)
            })
        }
        return method.apply(this, arguments)
    }
}

class User {
    name: string
    id: number
    constructor(name:string, id: number) {
        this.name = name
        this.id = id
    }

    // 方法装饰器做校验
    @validateEmptyStr
    changeName (@required newName: string) { // 参数装饰器做描述
        this.name = newName
    }
}
```

![运行代码](https://static.tasaid.com/blogs/f5cc17abfcb4ecdc20ab0b9622704a28.jpeg)

## 元数据反射
反射，就是在运行时动态获取一个对象的一切信息：方法/属性等等，特点在于动态类型反推导。在 TypeScript 中，反射的原理是通过设计阶段对对象注入元数据信息，在运行阶段读取注入的元数据，从而得到对象信息。

反射可以获取对象的：

- 对象的类型
- 成员/静态属性的信息(类型)
- 方法的参数类型、返回类型
```typescript
class User {
    name: string = 'linkFly'
    
    say (myName: string): string {
        return `hello, ${myName}`
    }
}
```

例如上面的例子，在 TypeScript 中可以获取到这些信息：

- Class Name 为 `User`
- `User` 有一个属性名为 `name`，有一个方法 `say()`
- 属性 `name` 是 `string` 类型的，且值为 `linkFly`
- 方法 `say()` 接受一个 `string` 类型的参数，在 TypeScript 中，参数名是获取不到的
- 方法 `say()` 返回类型为 `string`

TypeScript 结合自身静态类型语言的特点，为使用了装饰器的代码声明注入了 3 组元数据：

- `design:type`: 成员类型
- `design:paramtypes`: 成员所有参数类型
- `design:returntype`: 成员返回类型

由于元数据反射也是实验性 API，所以要在 `tsconfig.json` 里启用这个实验性特性：
```json
{
    "compilerOptions": {
        "target": "ES5",
        // 支持装饰器
        "experimentalDecorators": true,
        // 装饰器元数据
        "emitDecoratorMetadata": true
    }
}
```

然后安装 `reflect-metadata`
```shell
npm i reflect-metadata --save
```

这样在装饰器中，就可以访问到由 TypeScript 注入的基本信息元数据：
```typescript
import 'reflect-metadata'

let meta = function (target: any, propertyKey: string) {
    
    // 获取成员类型
    let type = Reflect.getMetadata('design:type', target, propertyKey)
    // 获取成员参数类型
    let paramtypes = Reflect.getMetadata('design:paramtypes', target, propertyKey)
    // 获取成员返回类型
    let returntype = Reflect.getMetadata('design:returntype', target, propertyKey)
    // 获取所有元数据 key (由 TypeScript 注入)
    let keys = Reflect.getMetadataKeys(target, propertyKey)
    
    
    console.log(keys) // [ 'design:returntype', 'design:paramtypes', 'design:type' ]
    // 成员类型
    console.log(type) // Function
    // 参数类型
    console.log(paramtypes) // [String]
    // 成员返回类型
    console.log(returntype) // String
}
    
    
class User {
    // 使用这个装饰器就可以反射出成员详细信息
    @meta
    say (myName: string): string {
        return `hello, ${myName}`
    }
}
```

## 结语
Java 和 C# 由于是强类型编译型语言，所以反射就成了它们动态反推导数据类型的一个重要特性。

目前来说，JavaScript 因为其动态性，所以本身就包含了一些反射的特点：

- 遍历对象内所有属性
- 判断数据类型

TypeScript 补充了基础的类型元数据，只不过还是有些地方不够完善：在 TypeScript 中，参数名通过反射是获取不到的。

为什么获取不到呢？因为 JavaScript 本质上还是解释型语言，还迎合 Web 有一大特色：编译和压缩...

- 编译完了之后 Class Name 可能叫做 `User_1`
- 压缩完了之后参数 `myName` 可能叫 `m`
- 运行时可能传了 2 个，3 个，或者 N 个参数

`angular 1.x`中使用的依赖注入，采用传字符串那么蹩脚的方式，也是对 JavaScript 反射机制的不完善做出的一种妥协。

---
收录时间: 2022-03-02

<Vssue :title="$title" />