# Vue组件

vue组件类似nodejs组件
[module](http://nodejs.cn/api/modules.html)，
可以把公共的页面，如页面头部、底部抽离出来，提供给其他页面重复使用

## 组件使用步骤
### 1.引入组件
```js
import Header from "./Header";
```

### 2.挂载组件
```vue
components: {
    'v-head': Header,
}
```

### 3.模版中使用组件
```vue
<template>
    <div>
        <v-head></v-head>
    </div>
</template>
```
## 参考示例

### 1.Header.vue
```vue
<template>
    <div>
        <h3>页面头部组件</h3>
    </div>
</template>

<script>
    export default {
        name: "Header"
    }
</script>
<style scoped>
    h3{
        background: bisque;
        text-align: center;
    }
</style>
```
**css**样式在其他页面调用时会出现dom重复，样式错乱，可以使用id标记或者`<style scoped>`作用域

### 2.Footer.vue
```vue
<template>
    <div>
       <h3>页面底部组件</h3>
    </div>
</template>

<script>
    export default {
        name: "Footer"
    }
</script>

<style scoped>
    h3{
        background: silver;
        text-align: center;
    }
</style>
```

### 3.Home.vue
```vue
<template>
    <div>
        <v-head></v-head>
        <h2>{{msg}}</h2>
        <v-foot></v-foot>
    </div>
</template>

<script>
    //必须在js中导入组件
    import Header from "./Header";
    import Footer from "./Footer";

    export default {
        name: "Home",
        data() {
            return {
                msg: '这里是首页模块'
            }
        },
        //定义组件vue标签名，不能跟html标签重复
        components: {
            'v-head': Header,
            'v-foot': Footer,
        }
    }
</script>

<style scoped>
    h2{
        background: pink;
        text-align: center;
    }
</style>
```

### 4.App.vue根组件引入
```vue
<template>
    <div id="app">
        <v-home></v-home>
    </div>
</template>

<script>
    import Home from "./components/Home";

    export default {
        name: 'app',
        data() {
            return {
                msg: '',
            }
        },
        methods: {},
        components: {
            'v-home': Home,
        }
    }
</script>
```
template模版中切记使用一个根节点包裹html代码

## 参考资料
[组件](https://cn.vuejs.org/v2/guide/components.html)

## 源码示例
[vue组件](https://github.com/ghostxbh/VUE-Study/tree/master/vuedemo/demo07)
