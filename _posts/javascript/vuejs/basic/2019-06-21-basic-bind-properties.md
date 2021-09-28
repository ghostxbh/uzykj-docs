---
title: Vue.js属性绑定
date: 2019-06-21
tags:
    - Vuejs
author: ghostxbh
location: blog
summary: 操作元素的`class`列表和内联样式是数据绑定的一个常见需求。
---
# 属性绑定

## Class 与 Style 属性绑定

### 介绍

操作元素的`` class ``列表和内联样式是数据绑定的一个常见需求。
因为它们都是属性，所以我们可以用`` v-bind ``处理它们：
只需要通过表达式计算出字符串结果即可。不过，字符串拼接麻烦且易错。
因此，在将`` v-bind ``用于`` class ``和`` style ``时，``Vue.js`` 做了专门的增强。
表达式结果的类型除了字符串之外，还可以是对象或数组。

### 1.属性绑定
```vue
<!-- 绑定属性 -->
<div v-bind:title="title">鼠标hover</div>
<div :title="title">鼠标悬浮</div>

<!-- 绑定地址 -->
<img v-bind:src="url" height="400" width="600"/>

<!-- html页面渲染 -->
<!-- 直接数据绑定 -->
<div>{{html}}</div>
<!-- 绑定html属性解析 -->
<div v-html="html"></div>

#js代码
<script>
    export default {
        name: 'app',
        data() {
            return {
                msg: 'vue demo',
                title: '这里是一个title',
                url: 'https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1561705454&di=33c13bd9a15f514bae71aaf8250ff305&imgtype=jpg&er=1&src=http%3A%2F%2Fp3.ssl.qhimg.com%2Ft017c9d8f1ba39d313f.jpg',
                html:'<h3>我是HTML文本</h3>'
            }
        }
    }
</script>
```

### 2.数据绑定的另一种
```vue
<!-- text数据绑定 -->
<div v-text="msg"></div>
```

### 3.class绑定
```vue
<!-- html -->
<div>
    <h3>div颜色列表</h3>
    <ul>
        <!-- 简单绑定class -->
        <li v-bind:class="'red'">
            item ---- key
        </li>
        <br/>
        <!-- 列表绑定 -->
        <li v-for="(item,key) in list" :class="{'red':key ==0,'blue':key==1,'green':key==2,'pink':key==3}">
            {{item}} ---- {{key}}
        </li>
    </ul>
</div>

<!-- js代码 -->
<script>
    export default {
        name: 'app',
        data() {
            return {
                list: ['红色', '蓝色', '绿色', '粉色'],
            }
        }
    }
</script>
```

### 4.style绑定
```vue
<!-- style绑定 -->
<div class="box" v-bind:style="{height:heightMax+'px',width:widthMax+'px'}">这里是一个div盒子</div>

<!-- js代码 -->
<script>
    export default {
        name: 'app',
        data() {
            return {
                 widthMax: 400,
                 heightMax: 200,
            }
        }
    }
</script>
```

## 参考资料
[属性绑定](https://cn.vuejs.org/v2/guide/class-and-style.html)

---
收录时间: 2019-06-21

<Vssue :title="$title" />
