# 定义方法、对象事件
### 监听事件
可以用 `v-on` 指令监听 `DOM` 事件，并在触发时运行一些 `JavaScript` 代码。

代码：
```vue
<div id="example-1">
  <button v-on:click="counter += 1">Add 1</button>
  <p>The button above has been clicked {{ counter }} times.</p>
</div>
```

```js
var example1 = new Vue({
  el: '#example-1',
  data: {
    counter: 0
  }
})
```

输出：
```vue
<div id="example-1">
  <button @click="#">Add 1</button>
  <p>The button above has been clicked 0 times.</p>
</div>
```


## 代码示例
### 1.定义执行方法

```vue
<!-- 点击事件方法 -->
<h3>{{msg}}</h3>
<!-- 写法一 -->
<button v-on:click="clickOne()">点击事件一</button>
<br>
<br>

<!-- 写法二 -->
<button @click="clickTwo()">点击事件二</button>

<!-- js代码 -->
<script>
    methods: {
        clickOne() {
            alert('点击事件一');
        },
        clickTwo() {
            alert('点击事件二');
        },
    }
</script>
```

### 2.获取数据和设置数据
```vue
<!-- 获取data的值 -->
<button @click="getMsg()">获取值</button>
<br>
<br>

<!-- 设置data的值 -->
<button @click="setMsg()">设置值</button>

<!-- js代码 -->
<script>
export default {
    name: 'app',
    data() {
        return {
            msg: 'vue demo',
        }
    },
    methods: {
        getMsg() {
            alert(this.msg);
        },
        setMsg() {
            this.msg = '设置后的数据';
        },
    },
}    
</script>
```

### 3.批量数据执行事件
```vue
<!-- 请亲数据 -->
<button @click="reqData()">请亲数据</button>
<hr>
<ul>
    <li v-for="(item,key) in list">
        {{key}} ===== {{item}}
    </li>
</ul>

<!-- js代码 -->
<script>
    export default {
        name: 'app',
        data() {
            return {
                list: [],
            }
        },
        methods: {
            reqData() {
                for (let i = 0; i < 10; i++) {
                    this.list.push('这是第' + i + '条数据')
                }
            },
        },
    }   
</script> 
```

### 4.传值和绑定数据执行事件
```vue
<!-- 传值事件 -->
<button @click="deleteData('1')">传参事件(1)</button>
<br>
<br>

<!-- 输入传值事件 -->
<input type="text" ref="num">
<br>
<button @click="properData()">绑定传参事件</button>

<!-- js代码 -->
<script>
    methods: {
        deleteData(val) {
            alert(val);
        },
        properData() {
            let num = this.$refs.num.value;
            alert(num);
        },
    }
</script>
```

### 5.事件对象
```vue
<!-- 事件对象 -->
<button data-aid='890' @click="eventFn($event)">事件对象</button>

<!-- js代码 -->
<script>
    methods: {
        eventFn(e) {
            console.log(e);
            //dom节点 e.srcElement
            e.srcElement.style.background = 'red';
            //获取自定义数据
            console.log(e.srcElement.dataset.aid);
        }
    }
</script>
```
### 事件修饰符
在事件处理程序中调用 `event.preventDefault()` 或 `event.stopPropagation()` 是非常常见的需求。
尽管我们可以在方法中轻松实现这点，但更好的方式是：方法只有纯粹的数据逻辑，而不是去处理 DOM 事件细节。

为了解决这个问题，`Vue.js` 为 `v-on` 提供了事件修饰符。之前提过，修饰符是由点开头的指令后缀来表示的。

- stop
- prevent
- capture
- self
- once
- passive

```vue
<!-- 阻止单击事件继续传播 -->
<a v-on:click.stop="doThis"></a>

<!-- 提交事件不再重载页面 -->
<form v-on:submit.prevent="onSubmit"></form>

<!-- 修饰符可以串联 -->
<a v-on:click.stop.prevent="doThat"></a>

<!-- 只有修饰符 -->
<form v-on:submit.prevent></form>

<!-- 添加事件监听器时使用事件捕获模式 -->
<!-- 即元素自身触发的事件先在此处理，然后才交由内部元素进行处理 -->
<div v-on:click.capture="doThis">...</div>

<!-- 只当在 event.target 是当前元素自身时触发处理函数 -->
<!-- 即事件不是从内部元素触发的 -->
<div v-on:click.self="doThat">...</div>
```

使用修饰符时，顺序很重要；相应的代码会以同样的顺序产生。
因此，用`v-on:click.prevent.self` 会阻止**所有的点击**，
而 ``v-on:click.self.prevent``只会阻止对元素自身的点击。

- 新增

```vue
<!-- 点击事件将只会触发一次 -->
<a v-on:click.once="doThis"></a>
```

不像其它只能对原生的 DOM 事件起作用的修饰符，``.once ``修饰符还能被用到自定义的组件事件上。如果你还没有阅读关于组件的文档，现在大可不必担心。

Vue 还对应`addEventListener`中的 `passive` 选项提供了 `.passive` 修饰符。
```vue
<!-- 滚动事件的默认行为 (即滚动行为) 将会立即触发 -->
<!-- 而不会等待 `onScroll` 完成  -->
<!-- 这其中包含 `event.preventDefault()` 的情况 -->
<div v-on:scroll.passive="onScroll">...</div>
```

这个`.passive`修饰符尤其能够提升移动端的性能。

不要把`.passive`和`.prevent`一起使用，因为`.prevent`将会被忽略，同时浏览器可能会向你展示一个警告。
请记住，`.passive`会告诉浏览器你不想阻止事件的默认行为。

### 按键修饰符
在监听键盘事件时，我们经常需要检查详细的按键。
Vue 允许为`v-on`在监听键盘事件时添加按键修饰符：

```vue
<!-- 只有在 `key` 是 `Enter` 时调用 `vm.submit()` -->
<input v-on:keyup.enter="submit">
```

你可以直接将`KeyboardEvent.key`暴露的任意有效按键名转换为 `kebab-case` 来作为修饰符。

```vue
<input v-on:keyup.page-down="onPageDown">
```
在上述示例中，处理函数只会在`$event.key`等于`PageDown`时被调用。

#### 按键码

**keyCode** 的事件用法已经被废弃了并可能不会被最新的浏览器支持。

使用 keyCode 特性也是允许的：

```vue
<input v-on:keyup.13="submit">
```

为了在必要的情况下支持旧浏览器，Vue 提供了绝大多数常用的按键码的别名：

- enter
- tab
- delete (捕获“删除”和“退格”键)
- esc
- space
- up
- down
- left
- right

有一些按键 (.esc 以及所有的方向键) 在 IE9 中有不同的 key 值, 如果你想支持 IE9，这些内置的别名应该是首选。

你还可以通过全局 config.keyCodes 对象自定义按键修饰符别名：
```js
// 可以使用 `v-on:keyup.f1`
Vue.config.keyCodes.f1 = 112
```

### 系统修饰键

- 新增

可以用如下修饰符来实现仅在按下相应按键时才触发鼠标或键盘事件的监听器。

- ctrl
- alt
- shift
- meta
注意：在 Mac 系统键盘上，meta 对应 command 键 (⌘)。在 Windows 系统键盘 meta 对应 Windows 徽标键 (⊞)。在 Sun 操作系统键盘上，meta 对应实心宝石键 (◆)。在其他特定键盘上，尤其在 MIT 和 Lisp 机器的键盘、以及其后继产品，比如 Knight 键盘、space-cadet 键盘，meta 被标记为“META”。在 Symbolics 键盘上，meta 被标记为“META”或者“Meta”。

例如：
```vue
<!-- Alt + C -->
<input @keyup.alt.67="clear">

<!-- Ctrl + Click -->
<div @click.ctrl="doSomething">Do something</div>
```
请注意修饰键与常规按键不同，在和`keyup`事件一起用时，事件触发时修饰键必须处于按下状态。
换句话说，只有在按住 `ctrl` 的情况下释放其它按键，
才能触发`keyup.ctrl`。而单单释放`ctrl`也不会触发事件。
如果你想要这样的行为，请为`ctrl`换用`keyCode：keyup.17`。

#### `.exact`修饰符

- 新增

`.exact`修饰符允许你控制由精确的系统修饰符组合触发的事件。
```vue
<!-- 即使 Alt 或 Shift 被一同按下时也会触发 -->
<button @click.ctrl="onClick">A</button>

<!-- 有且只有 Ctrl 被按下的时候才触发 -->
<button @click.ctrl.exact="onCtrlClick">A</button>

<!-- 没有任何系统修饰符被按下的时候才触发 -->
<button @click.exact="onClick">A</button>
```

#### 鼠标按钮修饰符

- 新增

- left
- right
- middle

这些修饰符会限制处理函数仅响应特定的鼠标按钮。

## 参考资料
[事件](https://cn.vuejs.org/v2/guide/events.html)

## 源码示例
[vue事件](https://github.com/ghostxbh/VUE-Study/tree/master/vuedemo/demo05)
