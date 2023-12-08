---
title: vue/vue
comments: true
aside: true
top_img: false
date: 2020-11-20 15:21:09
tags:
description:
mathjax:
katex:
categories:
cover: false
---
>Vue (读音 /vju:/，类似于 view) 是一套用于构建用户界面的渐进式框架。

### 快速开始
---

打开 `https://codesandbox.io/` 在线代码编辑器,创建一个static环境,新建一个 `.html` 文件，然后通过如下方式引入 Vue

```html
<!DOCTYPE html>
<html>
<head>
    <title>My first Vue app</title>
    <!-- 开发环境版本，包含了有帮助的命令行警告 -->
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
</head>
<body>
<div id="app">
    {{ message }}
</div>
<!-- vue的使用要从创建vue对象开始 -->
<script>
    var app = new Vue({
        el: '#app',  <!-- 设置vue可以操作的html内容范围 -->
        data: {   <!-- 保存vue.js中要显示到html页面的数据 -->
           message: 'Hello Vue!',
           methods:{},  <!-- 定义函数 -->
        　　watch:{},  <!-- 监听属性-->
        　　filters:{},<!-- 定义过滤器 -->
        }
    })
</script>
</body>
</html>
```

### 声明式渲染
---
Vue.js 的核心是一个允许采用简洁的模板语法来声明式地将数据渲染进 DOM 的系统,
数据和 DOM 已经被建立了关联，所有东西都是响应式的。我们要怎么确认呢？打开你的浏览器的 JavaScript 控制台 (就在这个页面打开)，
并修改 app.message 的值，你将看到上例相应地更新。
```html
<div id="app">
  {{ message }}
</div>
var app = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!'  <!--修改 app.message 的值，你将看到上例相应地更新。-->
  }
})
```

### 指令 `v-`
---
`v-bind attribute 被称为指令。指令带有前缀 v-，以表示它们是 Vue 提供的特殊 attribute。`

##### `v-bind`
```html
<div id="app-2">
  <span v-bind:title="message">
    鼠标悬停几秒钟查看此处动态绑定的提示信息！
  </span>
</div>
var app2 = new Vue({
  el: '#app-2',
  data: {
    message: '页面加载于 ' + new Date().toLocaleString()
  }
})
```
该指令的意思是：“将这个元素节点的 title attribute 和 Vue 实例的 message property 保持一致”。
例如输入 app2.message = '新消息'，就会再一次看到这个绑定了 title attribute 的 HTML 已经进行了更新。

##### `v-if` 条件
```html
<div id="app-3">
  <p v-if="seen">现在你看到我了</p>
</div>
var app3 = new Vue({
  el: '#app-3',
  data: {
    seen: true
  }
})
```
控制台输入 app3.seen = false，你会发现之前显示的消息消失了。
这个例子演示了我们不仅可以把数据绑定到 DOM 文本或 attribute，还可以绑定到 DOM 结构。
此外，Vue 也提供一个强大的过渡效果系统，可以在 Vue 插入/更新/移除元素时自动应用过渡效果。

##### `v-for` 循环
绑定数组的数据来渲染一个项目列表：
```html
<div id="app-4">
  <ol>
    <li v-for="todo in todos">
      {{ todo.text }}
    </li>
  </ol>
</div>
var app4 = new Vue({
  el: '#app-4',
  data: {
    todos: [
      { text: '学习 JavaScript' },
      { text: '学习 Vue' },
      { text: '整个牛项目' }
    ]
  }
})
```
控制台里，输入 app4.todos.push({ text: '新项目' })，你会发现列表最后添加了一个新项目。

##### `v-on` 事件监听器
```html
<div id="app-5">
  <p>{{ message }}</p>
  <button v-on:click="reverseMessage">反转消息</button>
</div>
var app5 = new Vue({
  el: '#app-5',
  data: {
    message: 'Hello Vue.js!'
  },
  methods: {
    reverseMessage: function () {
      this.message = this.message.split('').reverse().join('')
    }
  }
})
```
v-on 指令添加一个事件监听器，通过它调用在 Vue 实例中定义的方法

##### `v-model`
```html
<div id="app-6">
  <p>{{ message }}</p>
  <input v-model="message">
</div>
var app6 = new Vue({
  el: '#app-6',
  data: {
    message: 'Hello Vue!'
  }
})
```
v-model 指令，它能轻松实现表单输入和应用状态之间的双向绑定。

### 组件化应用构建
组件系统是 Vue 的另一个重要概念，因为它是一种抽象，允许我们使用小型、独立和通常可复用的组件构建大型应用。
在 Vue 里，一个组件本质上是一个拥有预定义选项的一个 Vue 实例。在 Vue 中注册组件很简单：

```html
// 定义名为 todo-item 的新组件
Vue.component('todo-item', {
  template: '<li>这是个待办项</li>'
})

var app = new Vue(...)
```
现在你可以用它构建另一个组件模板：
```html
<ol>
  <!-- 创建一个 todo-item 组件的实例 -->
  <todo-item></todo-item>
</ol>
```
`或`
```html
<div id="app-7">
  <ol>
    <!--
      现在我们为每个 todo-item 提供 todo 对象
      todo 对象是变量，即其内容可以是动态的。
      我们也需要为每个组件提供一个“key”，稍后再
      作详细解释。
    -->
    <todo-item
      v-for="item in groceryList"
      v-bind:todo="item"
      v-bind:key="item.id"
    ></todo-item>
  </ol>
</div>
Vue.component('todo-item', {
  // todo-item 组件现在接受一个
  // "prop"，类似于一个自定义 attribute。
  // 这个 prop 名为 todo。
  props: ['todo'],
  template: '<li>{{ todo.text }}</li>'
})

var app7 = new Vue({
  el: '#app-7',
  data: {
    groceryList: [
      { id: 0, text: '蔬菜' },
      { id: 1, text: '奶酪' },
      { id: 2, text: '随便其它什么人吃的东西' }
    ]
  }
})
```



