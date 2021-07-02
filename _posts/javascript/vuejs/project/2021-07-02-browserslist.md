---
title: browserslist 
date: 2021-07-02
tags:
  - browserslist 
author: ghostxbh 
location: blog 
summary: 你会发现有 package.json 文件里的 browserslist 字段 (或一个单独的 .browserslistrc 文件)，指定了项目的目标浏览器的范围。这个值会被 @babel/preset-env 和 Autoprefixer 用来确定需要转译的 JavaScript 特性和需要添加的 CSS 浏览器前缀。
---

# browserslist

> 本博文是翻译：[browserslist](https://github.com/browserslist/browserslist) 的内容。

Vue-cli官方文档开发模块的浏览器兼容这一块，提到了browserslist，原文如下：      
你会发现有 `package.json` 文件里的 `browserslist` 字段 (或一个单独的 `.browserslistrc` 文件)，指定了项目的目标浏览器的范围。
这个值会被 [@babel/preset-env](https://babeljs.io/docs/en/babel-preset-env.html)
和 [Autoprefixer](https://github.com/postcss/autoprefixer) 用来确定需要转译的 JavaScript 特性和需要添加的 CSS 浏览器前缀。

现在查阅这里了解如何指定浏览器范围。

下面是来自：[https://github.com/ai/browserslist](https://github.com/ai/browserslist) 的官方文档翻译的内容：

### Browserslist

在不同的前端工具之间共享目标浏览器和Node.js版本的配置。它用于:

- [Autoprefixer](https://github.com/postcss/autoprefixer)
- [Babel](https://github.com/babel/babel/tree/master/packages/babel-preset-env)
- [postcss-preset-env](https://github.com/csstools/postcss-preset-env)
- [eslint-plugin-compat](https://github.com/amilajack/eslint-plugin-compat)
- [stylelint-no-unsupported-browser-features](https://github.com/ismay/stylelint-no-unsupported-browser-features)
- [postcss-normalize](https://github.com/csstools/postcss-normalize)
- [obsolete-webpack-plugin](https://github.com/ElemeFE/obsolete-webpack-plugin)

当您将以下内容添加到`package.json`文件中时，所有工具都会自动找到目标浏览器:

```json
{
  "browserslist": [
    "defaults",
    "not IE 11",
    "not IE_Mob 11",
    "maintained node versions"
  ]
}

```

或者在 `.browserslistrc` 配置文件中配置：

```
# Browsers that we support

defaults
not IE 11
not IE_Mob 11
maintained node versions
```

开发人员在设置版本列表后，比如设置了`last 2 version`(最后2个版本)，可以避免手动更新版本.      
Browserslist将从配置文件中去查找：browserslist配置文件，.browserslistrc配置文件，或者package.json文件中的browserslist字段，或者各种环境配置等。
[Browserslist Example](https://github.com/browserslist/browserslist-example) 这个仓库显示了各个工具如何使用Browserslist。

### 工具
- [browserl.ist](https://browserl.ist/) 是用于检查某些查询将选择哪些浏览器的在线工具.
- `browserslist-ga` 和 `browserslist-ga-export` 下载你的网站浏览器统计信息，以便在我的统计查询中使用 > 0.5%.
- `browserslist-useragent-regexp` 将Browserslist查询编译到RegExp以测试浏览器useragent。
- `browserslist-useragent-ruby` 是一个Ruby库，用于按用户代理字符串检查浏览器是否与Browserslist相匹配。
- `browserslist-browserstack` 为Browserslist配置中的所有浏览器运行BrowserStack测试。
- `caniuse-api` 返回支持某些特定功能的浏览器. 在项目目录中运行npx browserslist以查看项目的目标浏览器。 此CLI工具是内置的，可在具有Autoprefixer的任何项目中使用.

### 最佳实践
下面是一个默认查询，它为大多数用户提供了合理的配置：

```
"browserslist":{
    defaults
}
```

- 如果要更改我们推荐的默认浏览器集，请将 last 2 versions、not dead 与使用编号（如 >0.2%）合并。这是因为 last n versions 本身不会添加流行的旧版本，而只使用高于 0.2%
  的百分比，从长远来看，会使流行的浏览器更受欢迎。就像我们使用Internet Explorer 6一样，我们可能会陷入垄断和停滞状态。请谨慎使用此设置。

- 仅当您使用一个浏览器为信息资讯制作网络应用程序时，才直接选择浏览器（最近2个Chrome版本）。 市场上有很多浏览器。 如果您要制作通用的Web应用程序，则应尊重浏览器的多样性。

- 不要因为不了解浏览器就删除它们。Opera Mini在非洲拥有1亿用户，在全球市场上比微软Edge更受欢迎。中国QQ浏览器的市场份额比Firefox和Safari加起来还要大。

### 查询指令
Browserslist将使用来自以下来源之一的浏览器和Node.js版本查询:

- 当前目录或父目录中package.json文件中的browserslist字段配置。 我们建议采用这种方式。
- 在`.browserslistrc`或者browserslist配置文件中配置。
- BROWSERSLIST环境变量中配置。
- 如果以上方法未产生有效结果，则Browserslist将使用默认值: `>0.5%,last 2 versions,Firefox ESR,not dead.`

### 组合查询
- or组合器可以使用关键字or或者使用,,last 1 version or >1%与last 1 version , >1%的表示方式是一样的。
- and支持查询组合来执行前一个查询的交集:last 1 version and >1%

如下所示，有3种不同的方法来组合查询。首先，您从单个查询开始，然后我们将这些查询组合起来以得到最终的列表:
显然，您不能从not组合器开始，因为没有左侧查询可以将其与之合并。

|查询组合器类型    | 图表表示 |    示例 |
| ----------    | ------  | --- |
| or combiner (union) |    并集    | > .5% or last 2 versions / > .5%, last 2 versions |
| and combiner (intersection) |    交集    | > .5% and last 2 versions |
| not combiner (relative complement) |    not    | > .5% and not last 2 versions / .5% or not last 2 versions / .5%, not last 2 versions |

> 测试查询的一种快速方法是在终端中执行`npx browserslist '> 0.5%, not IE 11'`。

全部列表 可以通过查询指定浏览器和Node.js版本(不区分大小写):

- defaults: Browserslist’s default browsers (`> 0.5%, last 2 versions, Firefox ESR, not dead`).
- `> 5%: browsers versions selected by global usage statistics. >=, < and <= work too.`

### 调试
在终端中运行`npx browserslist`命令，查看您的查询选择了哪些浏览器：

```
and_chr 78
and_ff 68
and_qq 1.2
and_uc 12.12
android 76
baidu 7.12
chrome 79
chrome 78
chrome 77
chrome 49
edge 18
edge 17
firefox 71
firefox 70
firefox 68
ie 11
ie_mob 11
ios_saf 13.3
ios_saf 13.2
ios_saf 13.0-13.1
ios_saf 12.2-12.4
kaios 2.5
op_mini all
op_mob 46
opera 64
opera 63
safari 13
safari 12.1
safari 5.1
samsung 10.1
samsung 9.2
```

### 浏览器
名称不区分大小写：

- Android for Android WebView.
- Baidu for Baidu Browser.
- BlackBerry or bb for Blackberry browser.
- Chrome for Google Chrome.
- ChromeAndroid or and_chr for Chrome for Android Edge for Microsoft Edge. 
- Electron for Electron framework. 
- It will be converted to Chrome version. 
- Explorer or ie for Internet Explorer. 
- ExplorerMobile or ie_mob for Internet Explorer Mobile. 
- Firefox or ff for Mozilla Firefox. 
- FirefoxAndroid or and_ff for Firefox for Android. 
- iOS or ios_saf for iOS Safari. 
- Node for Node.js. 
- Opera for Opera. 
- OperaMini or op_mini for Opera Mini. 
- OperaMobile or op_mob for Opera Mobile.
- QQAndroid or and_qq for QQ Browser for Android. 
- Safari for desktop Safari. 
- Samsung for Samsung Internet. 
- UCAndroid or and_uc for UC Browser for Android. 
- kaios for KaiOS Browser.

### 配置文件
#### package.json
```json
{
  "private": true,
  "dependencies": {
    "autoprefixer": "^6.5.4"
  },
  "browserslist": [
    "last 1 version",
    "> 1%",
    "IE 10"
  ]
}
```

#### `.browserslistrc`
```
# Browsers that we support

last 1 version
> 1%
IE 10 # sorry
```

---
收录时间: 2021-07-02

<Vssue :title="$title" />
