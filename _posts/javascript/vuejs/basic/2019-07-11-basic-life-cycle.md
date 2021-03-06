---
title: Vue.js生命周期
date: 2019-07-11
tags:
    - Vue.js
author: ghostxbh
location: BeiJing
summary: Vue.js的生命周期。
---
# 生命周期

如下图
![life](http://file.uzykj.com/lifecycle.png)

```vuejs
//生命周期相关函数
beforeCreate() {
    console.log('实例创建之前');
},
created() {
    console.log('实例创建完成');
},
beforeMount() {
    console.log('模版编译之前');
},
mounted() {                     /** 请求数据，操作DOM*/
    console.log('模版编译完成');
},
beforeUpdate(){
    console.log('数据更新之前');
},
updated(){
    console.log('数据更新完成');
},
beforeDestroy() {               /** 页面销毁前，保存页面数据*/
    console.log('模版销毁之前');
},
destroyed() {
    console.log('模版销毁完成');
},
```


## 参考资料
[生命周期](https://cn.vuejs.org/v2/guide/instance.html#%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E5%9B%BE%E7%A4%BA)

## 源码示例
[vue生命周期](https://github.com/ghostxbh/VUE-Study/tree/master/vuedemo/demo08)


---
收录时间: 2019-07-11

<Vssue :title="$title" />
