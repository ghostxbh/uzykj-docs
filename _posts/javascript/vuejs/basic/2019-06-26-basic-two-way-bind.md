---
title: Vue.js双向数据绑定
date: 2019-06-26
sidebar: 'auto'
categories:
  - Javascript
tags:
  - Vuejs
author: ghostxbh
location: blog
summary: 双向数据绑定基于**MVVM**框架，vue属于MVVM框架。
---
# 双向数据绑定

双向数据绑定基于**MVVM**框架，vue属于MVVM框架

MVVM：M等于model，V等于view，即model改变影响view，view改变影响model
### 1.双向数据绑定
```vue
<!-- 双向数据绑定 -->
#必须在使用在表单里面
#使用v-model绑定数据，实现动态数据变化
<h3>{{msg}}</h3>
<input type="text" v-model="msg">

#js代码
export default {
    name: 'app',
    data() {
        return {
            msg: 'vue demo',
        }
    }
}
```

- 获取动态数据
```vue
<!-- 获取动态数据 -->
<button v-on:click="getMsg()">获取表单数据</button>

#js代码
methods: {
    getMsg() {
        alert(this.msg)
    }
}
```

- 设置表单数据
```vue
<!-- 设置动态数据 -->
<button v-on:click="setMsg()">设置表单数据</button>

#js代码
methods: {
    setMsg(){
        this.msg = '设置后的数据';
    },
}
```

### 2.使用ref绑定数据（使用ref进行dom操作）
```vue
#html代码
<input type="text" ref="textInfo"/>
<div ref="box">这里是一个box</div>
<!-- 获取动态数据 -->
<button v-on:click="getInfo()">获取表单数据</button>

#js代码
methods: {
    getInfo() {
        alert(this.$refs.textInfo.value);
        this.$refs.box.style.background = 'red';
    },
}
```

---
收录时间: 2019-06-26

<Vssue :title="$title" />
