---
title: Vue 引入 TypeScript
date: 2022-03-10
sidebar: 'auto'
categories:
  - Javascript
tags:
  - TypeScript
  - TS
  - Vue
author: tasaid
location: blog
summary: Vue在官方文档中有一节简单的介绍了如何引入TypeScript，可惜文档太过简单，真正投入生产还有许多的细节没有介绍。
---
# Vue 引入 TypeScript
### 引入 TypeScript
Vue 在 [官方文档中有一节简单的介绍了如何引入 TypeScript](https://cn.vuejs.org/v2/guide/typescript.html#main) ，可惜文档太过简单，真正投入生产还有许多的细节没有介绍。

我们对此进行了一系列探索，最后我们的风格是这样的：
```typescript
import { Component, Prop, Vue, Watch } from 'vue-property-decorator'
import { State, Action, Mutation, namespace } from 'vuex-class'
import Toast from 'components/Toast.vue'

const userState = namespace('business/user', State)

@Component({
    components: { Toast },
})
export default class extends Vue {
    // data
    title = 'demo'
    
    @Prop({ default: '' })
    text: string
    
    // store
    @userState userId
    
    // computed
    get name (): boolean {
    return this.title + this.text
    }
    
    // watch
    @Watch('text')
    onChangeText () { }
    
    // hooks
    mounted() { }
}
```

大体来说，Vue 引入 TypeScript 可以用到这些生态库：

- [vue-class-component](https://github.com/vuejs/vue-class-component) ：强化 Vue 组件，使用 TypeScript/装饰器 增强 Vue 组件
- [vue-property-decorator](https://github.com/kaorun343/vue-property-decorator) ：在 `vue-class-component` 上增强更多的结合 Vue 特性的装饰器
- [vuex-class](https://github.com/ktsn/vuex-class) ：在 `vue-class-component` 提供 `Vuex` 的绑定
- [vuex-ts-decorators](https://github.com/snaptopixel/vuex-ts-decorators) ：让 `Vuex` 和 TypeScript 结合

下面我们一步步来介绍 Vue 如何引入 TypeScript

### webpack 和 tsconfig.json 配置
TypeScript 为 Webpack 提供了 `ts-loader`：
```shell
npm i ts-loader --save-dev
```

webpack 配置如下：
```js
{
	module: {
		rules: [{
				test: /\.vue$/,
				loader: 'vue-loader',
				options: { /* ... */ },
			},
			{
				test: /\.ts$/,
				loader: 'ts-loader',
				options: {
					appendTsSuffixTo: [/\.vue$/],
				}
			},
		],
	}
}
```
`ts-loader` 会检索当前目录下的 `tsconfig.json` 文件，如果找不到会一层层往上找。

这里有一份参考的 `tsconfig.json` 配置：
```
{
	// 编译选项
	"compilerOptions": {
		// 输出目录
		"outDir": "./output",
		// 是否包含可以用于 debug 的 sourceMap
		"sourceMap": true,
		// 以严格模式解析
		"strict": true,
		// 采用的模块系统
		"module": "esnext",
		// 如何处理模块
		"moduleResolution": "node",
		// 编译输出目标 ES 版本
		"target": "es5",
		// 允许从没有设置默认导出的模块中默认导入
		"allowSyntheticDefaultImports": true,
		// 将每个文件作为单独的模块
		"isolatedModules": false,
		// 启用装饰器
		"experimentalDecorators": true,
		// 启用设计类型元数据（用于反射）
		"emitDecoratorMetadata": true,
		// 在表达式和声明上有隐含的any类型时报错
		"noImplicitAny": false,
		// 不是函数的所有返回路径都有返回值时报错。
		"noImplicitReturns": true,
		// 从 tslib 导入外部帮助库: 比如__extends，__rest等
		"importHelpers": true,
		// 编译过程中打印文件名
		"listFiles": true,
		// 移除注释
		"removeComments": true,
		"suppressImplicitAnyIndexErrors": true,
		// 允许编译javascript文件
		"allowJs": true,
		// 解析非相对模块名的基准目录
		"baseUrl": "./",
		// 指定特殊模块的路径
		"paths": {
			"jquery": [
				"node_modules/jquery/dist/jquery"
			]
		},
		// 编译过程中需要引入的库文件的列表
		"lib": [
			"dom",
			"es2015",
			"es2015.promise"
		]
	}
}
```

由于 TypeScript 默认并不支持 `*.vue` 后缀的文件，所以在 vue 项目中引入的时候需要创建一个 `vue-shims.d.ts` 文件，放在项目项目对应使用目录下，例如 `src/vue-shims.d.ts`
```typescript
declare module "*.vue" {
    import Vue from "vue";
    export default Vue;
}
```

意思是告诉 TypeScript `*.vue` 后缀的文件可以交给 `vue` 模块来处理。

而在代码中导入 `*.vue` 文件的时候，需要写上 `.vue` 后缀。原因还是因为 TypeScript 默认只识别 `*.ts` 文件，不识别 `*.vue` 文件：
```typescript
import Component from 'components/component.vue'
```

### vue-class-component
`vue-class-component` 对 Vue 组件进行了一层封装，让 Vue 的组件语法在结合了 TypeScript 语法之后更加扁平化：
```vue
<template>
  <div>
    <input v-model="msg">
    <p>msg: {{ msg }}</p>
    <p>computed msg: {{ computedMsg }}</p>
    <button @click="greet">Greet</button>
  </div>
</template>

<script lang="ts">
  import Vue from 'vue'
  import Component from 'vue-class-component'

  @Component
  export default class App extends Vue {
    // 初始化数据
    msg = 123

    // 声明周期钩子
    mounted () {
      this.greet()
    }

    // 计算属性
    get computedMsg () {
      return 'computed ' + this.msg
    }

    // 方法
    greet () {
      alert('greeting: ' + this.msg)
    }
  }
</script>
```

上面的代码和下面没有引入 `vue-class-component` 的语法一样：
```vue
export default {
    data () {
        return {
            msg: 123
        }
    }

    // 声明周期钩子
    mounted () {
        this.greet()
    }
    
    // 计算属性
    computed: {
        computedMsg () {
            return 'computed ' + this.msg
        }
    }
    
    // 方法
    methods: {
        greet () {
            alert('greeting: ' + this.msg)
        }
    }
}
```

### vue-property-decorator
`vue-property-decorator` 是在 `vue-class-component` 上增强了更多的结合 Vue 特性的装饰器，新增了这 7 个装饰器：
- `@Emit`
- `@Inject`
- `@Model`
- `@Prop`
- `@Provide`
- `@Watch`
- `@Component` (从 `vue-class-component` 继承)

这里仅列举常用的 `@Prop/@Watch/@Component`：
```typescript
import { Component, Emit, Inject, Model, Prop, Provide, Vue, Watch } from 'vue-property-decorator'

@Component
export class MyComponent extends Vue {
    
    @Prop()
    propA: number = 1
    
    @Prop({ default: 'default value' })
    propB: string
    
    @Prop([String, Boolean])
    propC: string | boolean
    
    @Prop({ type: null })
    propD: any
    
    @Watch('child')
    onChildChanged(val: string, oldVal: string) { }
}
```

相当于：
```vue
export default {
    props: {
        checked: Boolean,
        propA: Number,
        propB: {
            type: String,
            default: 'default value'
        },
        propC: [String, Boolean],
        propD: { type: null }
    }
    methods: {
        onChildChanged(val, oldVal) { },
    },
    watch: {
        'child': {
            handler: 'onChildChanged',
            immediate: false,
            deep: false
        },
    }
}
```

### vuex-class
`vuex-class` 在 `vue-class-component` 上，提供了 Vuex 的绑定的装饰器语法。

我们编写一个简单的 `Vuex store`：
```vue
import Vue from 'vue'
import Vuex from 'vuex'

// vuex
const store = new Vuex.Store({
    state: {
      name: 'linkFly'
    }
    modules: {
        demo: {
            // 带命名空间
            namespaced: true,
            state: {
                count: 0
            },
            mutations: {
                increment (state, n?: number) {
                    if (n != null )
                    state.count = n
                    else
                    state.count++
                }
            },
            actions: {
                increment ({ commit }) {
                    commit.commit('increment')
                }
            }
        }
    }
})
    
const app = new Vue({
    el: '#app',
    store,
    template: `<div class="app"></div>`
})
```

使用 `vuex-class` 之后：
```ts
import { Component, Vue } from 'vue-property-decorator'
import {
    State,
    Getter,
    Action,
    Mutation,
    namespace
} from 'vuex-class'

const ModuleState = namespace('demo', State)
const ModuleAction = namespace('demo', Action)
const ModuleMutation = namespace('demo', Mutation)

@Component
export class MyComp extends Vue {
    @ModuleState('count') count
    @ModuleAction increment
    @ModuleMutation('increment') mutationIncrement
    @State name
    
    created () {
        this.name // -> store.state.name => linkFly
        this.count // -> store.state.demo.count => 0
        this.increment() // -> store.dispatch('demo/increment')
        this.mutationIncrement(2) // -> store.commit('demo/increment', 2)
    }
}
```

### vuex-ts-decorators
由于使用 `vue-class-component`，在 Vue 组件中我们已经感受到了装饰器的强大语法糖，于是我们还希望在 Vuex Store 中也能使用装饰器的语法：`vuex-ts-decorators` 就是干这个事情的。

`vuex-ts-decorators` 可以让你结合装饰器来编写 Vuex Store。

由于 `vuex-ts-decorators` 提供的包是未经编译的 `*.ts` 代码，如果你排除了 `node_modules` 的编译，则需要在 `ts-loader` 中单独加上对它的编译：
```
{
	test: /\.ts$/,
	loader: 'ts-loader',
	// 加上对 vuex-ts-decorators 这个包的编译
	exclude: [/node_modules/, /node_modules\/(?!vuex-ts-decorators)/],
	options: {
		appendTsSuffixTo: [/\.vue$/],
	}
},
```

然后就可以愉快使用它来编写 Vuex Store 了。
```ts
import {module, action, mutation} from 'mfe-vuex-ts-decorators'


type actions = {}

type mutations = {
// 定义对应 mutations 的参数类型
    incrementMutation: number
}


type TypedDispatch = <T extends keyof actions>(type: T, value?: actions[T]) => Promise<any[]>;
type TypedCommit = <T extends keyof mutations>(type: T, value?: mutations[T]) => void;


@module({
    store: false,
    namespaced: true
})
class demo {
    // 用于类型检查，使 commit/dispatch 的方法可以找到并且可以被类型检查
    dispatch: TypedDispatch
    commit: TypedCommit

    // state
    count = 0

    // getter
    get countGetter(): string {
        return this.count
    }

    // action
    @action
    increment() {
        this.commit('incrementMutation');
    }

    // mutation
    @mutation
    incrementMutation(payload?: mutations['incrementMutation']) {
        if (n != null)
            this.count = payload
        else
            this.count++
    }
}
```

`vuex-ts-decorators` 文档没有提及 `@module` 装饰器的参数：
```vue
@module({
    store?: boolean = false; // 是否自动挂载到 Vuex Store 下，如果为 false 则
    modules?: Object; // 子 modules
    namespaced?: boolean; // 命名空间
} | (any, { decorators: any } => any)) // 也可以使用函数
```

#### 注意事项
`vuex-ts-decorators` 这个项目目前最后一次更新时间是 2017 年 2 月 21 日，原作者说自己正在开发对应的新项目，这个项目已经不再维护，但是目前还没有看到新项目的进展。

但可以肯定的是，原作者已经不再维护 `vuex-ts-decorators` 这个项目。

我们的做法是在这个项目上重新 fork 一个新项目，并且修正了遇到的隐藏 bug。后续我们会自己维护这个项目。(fork 到了内部项目中，后续在这个项目基础上进行二次开发，到时候会公布出来，当然，其实它的功能很简单完全可以自己开发一个)

目前已知 bug：

当使用 `@module({ store: false })` 后，被包装的 `class` 会返回处理后的 Vuex Store，结构为：`{ state: any, getters: any, modules: any, actions: any, mutations: any }`。

如果最后 `new Vuex.Store({ options: object })` 的时候传递的不是 `@module` 包装后的 Vuex Store(例如对这个 Vuex Store 做了一层深拷贝)，则会导致 `mutations` 中 `this` 丢失。

bug 点在 这一行，处理办法：
```js
store.mutations[name] = (state: any, ...args: any[]) => {
method.apply(store.state, args);
// 替换为
method.apply(state, args);
}
```

原因是因为 `vuex-ts-decorators` 一直在自己内部引用着生成的 Vuex Store，如果最终 `new Vuex.Store()` 的时候没有传递它自己生成的 Vuex Store 就会导致引用不正确。

### 结语
围绕 `vue-class-component` 展开的生态圈，如同一把利剑，通过各种装饰器，能够结合 TypeScript 将 Vue 武装的更加强大。在这篇文章发布之际，很巧的是：Vue 作者尤雨溪也宣布在 Vue 2.5 以后将全面支持 TypeScript(译文看这里)，并且**声明Vue 将尽力做到和 `vue-class-component` 兼容**。

前端的场景已经不再像当年一样简单的切图、画简单的样式和写简单的脚本。应用越来越庞大，需求越来越复杂。TypeScript 有一个很好的切入点：从语言的角度解决了大型 Web 应用的静态类型约束痛点。

相信随着时间的推移，会有越来越多的人和框架加入到 TypeScript 大军。

---
收录时间: 2022-03-10

<Vssue :title="$title" />