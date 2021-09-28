---
title: NodeJS编码规范
date: 2021-09-28
tags:
    - Nodejs
    - Standard
author: ghostxbh
location: blog
summary: NodeJS编码规范。
---
**NodeJS编码规范**

## 一、编码规范
- 所有项目默认格式统一为`UTF-8`
- 使用javascript/typescript语言编码
- 轻量级框架选用Express/Koa/Fast/Meteor，企业级框架选用Egg/Nest，实用通讯选用Socket.io

### 1.1 注释规范
- 核心业务、通用业务的接口或者类上面必须有注释，示例：
```js
/**
 * @date: 2021/09/28
 * @author: ghostxbh
 * @desc: 程序节点，此为所有扩展节点的继承父级
 */
class ProgramNode {
  constructor(api, model){
    this.api = api;
    this.model = model;
  }
  
  /**
   * 获取节点标题
   * @param {string} name 传入名称
   * @param {参数类型} 参数 参数描述
   *
   * @return {string} name 标题名称
   * @return {返回类型} 返回参数 返回描述
   */
  getTitle(name);
}
```

- 主要业务逻辑的代码必须有相关流程注释，单行开头需要空格，示例：
```js
// 获取接口名称
const apiName = this.api.getName();
```

- 提供给外部/对接使用的SDK、API、npm库需要有日期、版本注释、作者(ghostxbh)，示例：
```js
/**
 * @date 2021/09/28
 * @version v0.1.0
 * @author ghostxbh
 */
class SDK {
  // 提供对外业务...
}
```

### 1.2 代码目录结构
```
+ demo-server
    + logs
    + docs
    + src
        + config
        + exception
        + extend
        + middleware
        + router
        + service
        + utils
    + test
    - app.js
    - package.json
    - README.md
```

### 1.3 命名规范

#### 1.3.1 文件命名
- 统一使用英文小写命令。
- 多个描述英文之间使用`-`符合连接。
- 如需要标明文件用途，可命名为`xxx.service.js`。
- 不允许使用拼音和中文命名。
- 示例：`user.service.js`、`user-dept-relation.service.js`。

#### 1.3.2 函数、方法命名
- 方法与函数的命名需要见名知意。
- 使用首字母小写的驼峰命名法，示例：`getOrderById()`。也可以使用下划线命名法，示例：`get_order_by_id()`。选择其中一种统一风格。
- 通用函数/方法使用`_`命名开头，示例：`_getSession()`。

#### 1.3.3 常量、变量命名
- 常量使用全英文大写，多个语义之间用`_`连接，示例：`GLOBEL_CONFIG`。
- 变量必须要有关键词接收，建议使用`const`替代`var`，若是代码需要使用`let`。
- 变量使用首字母小写的驼峰命名法，示例：`userAge`。
- 变量需要见名知意，不允许使用拼音和中文命名。
- 非必要尽量写全变量，如len、max、min等。
- boolean类型变量使用is / has开头。

### 1.4 符号规范

#### 1.4.1 引号规范
- 字符串统一使用单引号''，需要拼接使用英文模式下的`~`字符，如：`${param}`。

#### 1.4.2 分号
- 完整的代码片段需要以分号结尾，示例：`const user = this.session.getUser();`。

#### 1.4.3 逗号
- 数组变量、对象变量存在子元素都需要以逗号结尾，示例：
```js
const user = {
  name: '小明',
  age: 18,
  no: '1009918',
};
```

### 1.5 严格模式
所有代码均在`'use strict'`严格模式下开发

## 二、RESTful API 规范

### 2.1 请求协议
| 协议头 | 描述 | 应用 |
| ----- | --- | --- |
| http | 网络请求 | 接口数据、页面请求 |
| https | ssl网络请求 | 接口数据、页面请求 |

### 2.2 请求动作
| 请求方法 | 功能 |
| ----- | --- |
|  GET  | 获取资源 |	
|  POST | 新增资源 |	
|  PUT  | 更新整个资源 |	
| PATCH | 更新个别资源 |	
| DELETE| 删除资源 |	

### 2.3 API版本号
通过版本号可以区分api的版本     
通过`/api/v1/**`代表v1版本      
通过`/api/v2/**`代表v2版本

注意：API分2种，内部服务/客户端使用，第三方调用的情况。     
内部使用按照 [请求动作](#22-%E8%AF%B7%E6%B1%82%E5%8A%A8%E4%BD%9C) 正常使用，第三方调用只允许提供GET/POST请求动作的API接口。

### 2.4 URL路径
RESTful API的所有操作都是针对特定资源进行的。资源就是URL所表示的，URL需要符合以下规范：
- 只能是名词不能是动词，必须是小写字符
- 不可使用下划线'_'，可以使用连字符'-'
- CRUD不可出现在URL中
- 参数列表要用encode
- 避免层级过深的URI，尽量使用查询参数代替路径中的实体导航，如GET `/user?sex=female&age=30`

具体形式如下：
- `/api/{资源名}/{描述名}`
- `/api/{资源名}/{对象id}/{描述名}`

示例：
- GET http://www.demo.com/api/v1/user/1 获取用户1的信息
- POST http://www.demo.com/api/v1/user/login 登录
- PUT http://www.demo.com/api/v1/user/1 更新用户1的全部信息
- DELETE http://www.demo.com/api/v1/user/1 删除用户1
- PATCH http://www.demo.com/api/v1/user/1 更新用户1部分信息
- GET http://www.demo.com/api/v1/user/1/role 获取用户1的权限信息

特殊情况：      
GET查询条件过多时，可选择使用POST请求提交数据

### 2.5 请求体和响应体
- 请求格式
```
{
  "data": {
    // 请求数据
  }
}
```

- 响应格式
```
{
  "success": true, // 响应状态
  "code": 200,  // 响应编码
  "message": "ok",  // 响应信息
  "data": {
    // 响应数据
  }
}
```

### 2.6 分页与排序
使用page的分页参数      
使用sort做排序参数字段
使用params做条件筛选对象

- 响应
```json
{
  "success": true,  // 响应状态
  "code": 200,      // 响应编码
  "message": "ok",  // 响应信息
  "data": {
                    // 响应数据
  },
  "condition": {    // 参数对象
    "page": {       // 分页对象
      "pageNo": 1,  
      "pageSize": 10,
      "count": 225,
      "total": 23
    }, 
    "sort": {       // 排序对象
      "age": 1,
      "createTime": -1
    },
    "params": {    // 条件对象
      "name": "小明"
    }
  }
}
```


### 2.7 状态码
所有返回状态码均使用英文提示 禁止出现中文提示

- 成功状态码

| 状态码	| 定义 |
| ----- | --- |
| 200   | 请求成功。客户端向服务器请求数据，服务器返回相关数据 |
| 201	| 资源创建成功。客户端向服务器提供数据，服务器创建资源 | 
| 202	| 请求被接收。但处理尚未完成 | 
| 204	| 客户端告知服务器删除一个资源，服务器移除它 | 

- 错误状态码

| 状态码	| 定义 |
| ----- | --- |
| 400	| 请求无效。数据不正确，请重试 |
| 401	| 请求没有权限。缺少API token，无效或者超时 |
| 403	| 请求未被授权。当前权限无法获取指定的资源 |
| 404	| 请求失败。请求资源不存在 |
| 406	| 请求失败。请求头部不一致，请重试 |
| 422	| 请求失败。请验证参数 |

- 服务器错误状态码

| 状态码	| 定义 |
| ----- | --- |
| 500	| 服务器发生错误，请检查服务器 |
| 502	| 网关错误 |
| 503	| 服务不可用，服务器暂时过载或维护 |
| 504	| 网关超时 |

- 自定义状态码

示业务而定

### 2.8 请求格式
- Content-Type:`application/json` 数据发送
- Content-Type:`multipart/form-data` 文件上传

---
收录时间: 2021-09-28

<Vssue :title="$title" />
