# 数据请求

## 技术点：
* [vue-resource](https://github.com/pagekit/vue-resource)
* [axios](https://github.com/axios/axios)
* [fetch-jsonp](https://github.com/camsong/fetch-jsonp)

## 使用步骤：
### 1.vue-resource
+ 需要安装vue-resource模块，注意添加--save
```shell
npm install vue-resource --save
cnpm install vue-resource --save
```

+ main.js引入vue-resource
```vuejs
import VueResource from 'vue-resource'
```

+ vue使用vue-resource
```vuejs
Vue.use(VueResource)
```

+ 在组件中使用vue-resource
```vuejs
this.$http.get(api).then(res => {
  console.log(res);
}).catch(e => {
  console.log(e);
})
```

### 2.axios
+ 安装axios
```shell
npm install axios --save
cnpm install axios --save
```

+ 加载axios
```vuejs
import Axios from 'axios';
```

+ 使用axios
```vuejs
Axios.get(api).then(res => {
  console.log(res);
}).catch(e => {
  console.log(e);
})
```

### 3.fetch-jsonp
+ 安装fetch-jsonp
```shell
npm install fetch-jsonp --save
cnpm install fetch-jsonp --save
```

+ 加载fetch-jsonp
```vuejs
import FetchJsonp from 'fetch-jsonp';
```

+ 使用fetch-jsonp
```vuejs
FetchJsonp(api)
  .then(resp => resp.json())
  .then(data => {
    console.log(data);
  })
  .catch(e => console.log(e));
```

## 参考资料
[数据请求](https://www.jianshu.com/p/6b82722e2025)

## 源码示例
[vue数据请求](https://github.com/ghostxbh/VUE-Study/tree/master/vuedemo/demo09)
