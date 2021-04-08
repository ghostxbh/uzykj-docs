---
title: Vue.js父子组件传值
date: 2019-07-11
tags:
    - Vue.js
author: ghostxbh
location: blog
summary: 父组件调用子组件时，绑定动态属性。
---
# 父子组件传值

## 技术点：
* [props](https://cn.vuejs.org/v2/guide/components-props.html)

## 使用步骤：
### 1.父组件调用子组件时，绑定动态属性
```vue
<Header :msg="msg" :title="title" :run="run" :home="this"></Header>
```

### 2.子组件里面通过props接收父组件传过来的数据
```vue
props: ['msg', 'title', 'run', 'home']

<p>{{msg}}</p>
<p>{{title}}</p>
<button @click="run('子组件参数')">执行run方法</button>
```

---
收录时间: 2019-07-11

<Vssue :title="$title" />
