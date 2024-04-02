---
title: Vue3 核心笔记
date: 2024-03-23 12:39:46
updated: 2024-03-23 12:39:46
tags: [Vue, Vue3, Front-End]
categories: Vue
---

本文使用以下版本：

```
"pinia": "2.1.7",
"vue": "3.4.21",
"vue-router": "4.3.0"
```



<!-- more -->

# Vue3 核心

## 什么是Vue

Vue (发音为 /vjuː/，类似 **view**) 是一款用于构建用户界面的 JavaScript 框架。它基于标准 HTML、CSS 和 JavaScript 构建，并提供了一套声明式的、组件化的编程模型：

- **声明式渲染**：Vue 基于标准 HTML 拓展了一套模板语法，使得我们可以声明式地描述最终输出的 HTML 和 JavaScript 状态之间的关系。
- **响应性**：Vue 会自动跟踪 JavaScript 状态并在其发生变化时响应式地更新 DOM。

你可以用不同的方式使用 Vue：

- 无需构建步骤，渐进式增强静态的 HTML。
- 在任何页面中作为 Web Components 嵌入
- 单页应用 (SPA)
- 全栈 / 服务端渲染 (SSR)
- Jamstack / 静态站点生成 (SSG)
- 开发桌面端、移动端、WebGL，甚至是命令行终端中的界面

## [单文件组件](https://cn.vuejs.org/guide/introduction.html#single-file-components)

**单文件组件** (也被称为 `*.vue` 文件，英文 Single-File Components，缩写为 **SFC**)：

- 一种类似 HTML 格式的文件来书写 Vue 组件；
- Vue 的单文件组件会将一个组件的逻辑 (JavaScript)，模板 (HTML) 和样式 (CSS) 封装在同一个文件里。
- 需要构建工具的支持。

下面我们将用单文件组件的格式重写上面的计数器示例：

```html
<script setup>
import { ref } from 'vue'
const count = ref(0)
</script>

<template>
  <button @click="count++">Count is: {{ count }}</button>
</template>

<style scoped>
button {
  font-weight: bold;
}
</style>
```

使用单文件组件创建项目时，通常使用基于 [Vite](https://vitejs.dev/) 的构建设置：

```shell
npm create vue@latest
```

在项目被创建后，通过以下步骤安装依赖并启动开发服务器：

```shell
$ cd <your-project-name>
$ npm install
$ npm run dev
```

## 通过`<script>`标签引入Vue并直接使用

借助 script 标签直接通过 CDN 来使用 Vue 时，

- 不涉及“构建步骤”，无需构建工具支持。
- 但是，你将无法使用单文件组件 (SFC) 语法。

在这些场景中你可以将 Vue 看作一个更加声明式的 jQuery 替代品。

### 使用全局构建版本

```html
<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>

<div id="app">{{ message }}</div>

<script>
  const { createApp, ref } = Vue

  createApp({
    setup() {
      const message = ref('Hello vue!')
      return {
        message
      }
    }
  }).mount('#app')
</script>
```

### 使用 ES 模块构建版本

```html
<div id="app"></div>

<script type="module">
  import { createApp, ref } from 'https://unpkg.com/vue@3/dist/vue.esm-browser.js'

  createApp({
    template: `<div>message is {{ message }}</div>`,
    setup() {
      const message = ref('Hello Vue!')
      return {
        message
      }
    }
  }).mount('#app')
</script>
```

### 启用 Import maps

```html
<script type="importmap">
  {
    "imports": {
      "vue": "https://unpkg.com/vue@3/dist/vue.esm-browser.js"
    }
  }
</script>

<div id="app">{{ message }}</div>

<script type="module">
  import { createApp, ref } from 'vue'

  createApp({
    setup() {
      const message = ref('Hello Vue!')
      return {
        message
      }
    }
  }).mount('#app')
</script>
```



## API 风格

### 选项式API (Options API)

```html
<script>
export default {
  // data() 返回的属性将会成为响应式的状态
  // 并且暴露在 `this` 上
  data() {
    return {
      count: 0
    }
  },

  // methods 是一些用来更改状态与触发更新的函数
  // 它们可以在模板中作为事件处理器绑定
  methods: {
    increment() {
      this.count++
    }
  },

  // 生命周期钩子会在组件生命周期的各个不同阶段被调用
  // 例如这个函数就会在组件挂载完成后被调用
  mounted() {
    console.log(`The initial count is ${this.count}.`)
  }
}
</script>

<template>
  <button @click="increment">Count is: {{ count }}</button>
</template>
```



### 组合式 API (Composition API)

```html
<script setup>
import { ref, onMounted } from 'vue'

// 响应式状态
const count = ref(0)

// 用来修改状态、触发更新的函数
function increment() {
  count.value++
}

// 生命周期钩子
onMounted(() => {
  console.log(`The initial count is ${count.value}.`)
})
</script>

<template>
  <button @click="increment">Count is: {{ count }}</button>
</template>
```



## setup

### `setup()`

[可以在一个组件中同时使用选项式 API 和 组合式 API 吗？](https://cn.vuejs.org/guide/extras/composition-api-faq.html#can-i-use-both-apis-in-the-same-component)可以。你可以在一个选项式 API 的组件中通过 [`setup()`](https://cn.vuejs.org/api/composition-api-setup.html) 选项来使用组合式 API。

`setup()` 钩子是在组件中使用组合式 API 的入口，通常只在以下情况下使用：

1. 需要在非单文件组件中使用组合式 API 时。
2. 需要在基于选项式 API 的组件中集成基于组合式 API 的代码时。

使用[响应式 API](https://cn.vuejs.org/api/reactivity-core.html) 来声明响应式的状态，在 `setup()` 函数中返回的对象会暴露给模板和组件实例。其他的选项也可以通过组件实例来获取 `setup()` 暴露的属性。

```html
<script>
import { ref } from 'vue'

export default {
  setup() {
    const count = ref(0)
    
    // setup() 自身并不含对组件实例的访问权，即在 setup() 中访问 this 会是 undefined。
    // 你可以在选项式 API 中访问组合式 API 暴露的值，但反过来则不行。
    // console.log(this.name)

    // 返回值会暴露给模板和其他的选项式 API 钩子
    return {
      count
    }
  },
  data(){
      return {
          name: 'name'
      }
  }
  mounted() {
    // 其他的选项也可以通过组件实例来获取 `setup()` 暴露的属性。
    // 当通过 this 访问时也会自动浅层解包。
    console.log(this.count) // 0
  }
}
</script>

<template>
  <!-- 在模板中访问从 setup 返回的 ref 时，它会自动浅层解包，因此你无须再在模板中为它写 .value。 -->
  <button @click="count++">{{ count }}</button> 
</template>
```

### setup 访问 props

`setup` 函数的第1个参数是组件的 `props`。和标准的组件一致，一个 `setup` 函数的 `props` 是响应式的。

```ts
export default {
  props: {
    title: String
  },
  setup(props) {
    console.log(props.title)
  }
}
```

如果需要解构 `props` 对象，使用 [toRefs()](https://cn.vuejs.org/api/reactivity-utilities.html#torefs) 和 [toRef()](https://cn.vuejs.org/api/reactivity-utilities.html#toref) 这两个工具函数，否则结构出来的变量会失去响应性。

```ts
import { toRefs, toRef } from 'vue'

export default {
  setup(props) {
    // 将 `props` 转为一个其中全是 ref 的对象，然后解构
    const { title } = toRefs(props)
    // `title` 是一个追踪着 `props.title` 的 ref
    console.log(title.value)

    // 或者，将 `props` 的单个属性转为一个 ref
    const title = toRef(props, 'title')
  }
}
```

### Setup 上下文

传入 `setup` 函数的第2个参数是一个 **Setup 上下文**对象。上下文对象暴露了其他一些在 `setup` 中可能会用到的值：

```ts
export default {
  setup(props, context) {
    // 该上下文对象是非响应式的，可以安全地解构
    const { attrs, slots, emit, expose } = context
    
    // 透传 Attributes（非响应式的对象，等价于 $attrs）
    console.log(attrs)

    // 插槽（非响应式的对象，等价于 $slots）
    console.log(slots)

    // 触发事件（函数，等价于 $emit）
    console.log(emit)

    // 暴露公共属性（函数）
    console.log(expose)
  }
}
```

`attrs` 和 `slots` 都是**有状态（但不是响应式）**的对象：

- 它们总是会随着组件自身的更新而更新。
- 应当避免解构它们，并始终通过 `attrs.x` 或 `slots.x` 的形式使用其中的属性。
- 如果你想要基于 `attrs` 或 `slots` 的改变来执行副作用，那么你应该在 `onBeforeUpdate` 生命周期钩子中编写相关逻辑。

### 暴露公共属性

`expose` 函数用于显式地限制该组件暴露出的属性，当父组件通过[模板引用](https://cn.vuejs.org/guide/essentials/template-refs.html#ref-on-component) (组件的`ref`属性) 访问该组件的实例时，将仅能访问 `expose` 函数暴露出的内容：

```ts
export default {
  setup(props, { expose }) {
    // 让组件实例处于 “关闭状态”
    // 即不向父组件暴露任何东西
    expose()

    const publicCount = ref(0)
    const privateCount = ref(0)
    // 有选择地暴露局部状态
    expose({ count: publicCount })
  }
}
```

### setup 返回函数

`setup`除了返回对象，还可以返回一个函数。修改默认脚手架里的`AboutView.vue`：

![](image-20240327174312645.png)

```html
<template>
  <div class="about">

  </div>
</template>


<script lang="ts">
export default {
  setup() {
    return () => '<p><i>haha</i></p>'
  }
}
</script>
```

页面上会直接显示`setup`返回的内容：

![](image-20240327174204982.png)

通常，`setup`返回函数时，会与渲染函数一起使用。

### 与渲染函数一起使用

`setup` 也可以返回一个[渲染函数](https://cn.vuejs.org/guide/extras/render-function.html)，此时在渲染函数中可以直接使用在同一作用域下声明的响应式状态：

```ts
import { h, ref } from 'vue'

export default {
  setup() {
    const count = ref(0)
    return () => h('div', count.value)
  }
}
```

返回一个渲染函数将会阻止我们返回其他东西，所以，如果我们想通过模板引用将这个组件的方法暴露给父组件，需要通过调用 [`expose()`](https://cn.vuejs.org/api/composition-api-setup.html#exposing-public-properties) 解决这个问题：

```ts
import { h, ref } from 'vue'

export default {
  setup(props, { expose }) {
    const count = ref(0)
    const increment = () => ++count.value

    expose({
      increment // 父组件可以通过模板引用来访问这个 increment 方法
    })

    return () => h('div', count.value)
  }
}
```

### `<script setup>`

使用`setup()` 函数需要手动`return`要暴露的状态和方法，当需要暴露大量的状态和方法时非常繁琐。幸运的是，我们可以通过使用[单文件组件 (SFC)](https://cn.vuejs.org/guide/scaling-up/sfc.html) 搭配 `<script setup>` 来大幅度地简化代码：

```html
<script setup>
import { ref } from 'vue'

const count = ref(0)

function increment() {
  count.value++
}
</script>

<template>
  <button @click="increment">
    {{ count }}
  </button>
</template>
```

`<script setup>` 中的顶层的导入、声明的变量和函数可在同一组件的模板中直接使用。你可以理解为模板是在同一作用域内声明的一个 JavaScript 函数。



## 创建Vue应用

每个 Vue 应用都是通过 [`createApp`](https://cn.vuejs.org/api/application.html#createapp) 函数创建一个新的 **应用实例**：

```js
import { createApp } from 'vue'

const app = createApp({
  /* 根组件选项 */
})
```

### [根组件](https://cn.vuejs.org/guide/essentials/application.html#the-root-component)

我们传入 `createApp` 的对象实际上是一个组件，每个应用都需要一个“根组件”，其他组件将作为其子组件。

使用单文件组件作为根组件，直接从另一个文件中导入根组件：

```js
import { createApp } from 'vue'
// 从一个单文件组件中导入根组件
import App from './App.vue'

const app = createApp(App)
```

使用无构建步骤的方式创建组件对象作为根组件：

> 注意：使用`template`选项提供模板时，必须使用带有编译器的Vue打包版本。否则会报错：
>
> `[Vue warn]: Component provided template option but runtime compilation is not supported in this build of Vue. Configure your bundler to alias "vue" to "vue/dist/vue.esm-bundler.js".`

```js
import { createApp, ref } from 'https://unpkg.com/vue@3/dist/vue.esm-bundler.js'

const app = createApp({
  setup() {
    const count = ref(0)
    return { count }
  },
  template: `<div>count is {{ count }}</div>`
})
```

或者：

```js
import { createApp, ref } from 'https://unpkg.com/vue@3/dist/vue.esm-browser.js'
const App = {
  setup() {
    const count = ref(0)
    return { count }
  },
  template: `<div>count is {{ count }}</div>`
}

const app = createApp(App)
```

或者拆分模块：

```js
// my-component.js
import { createApp, ref } from 'https://unpkg.com/vue@3/dist/vue.esm-bundler.js'
export default {
  setup() {
    const count = ref(0)
    return { count }
  },
  template: `<div>count is {{ count }}</div>`
}
```

```js
import { createApp, ref } from 'https://unpkg.com/vue@3/dist/vue.esm-bundler.js'
import MyComponent from './my-component.js'

const app = createApp(MyComponent)
```

### 挂载应用

应用实例必须在调用了 `.mount()` 方法后才会渲染出来。该方法接收一个“容器”参数，可以是一个实际的 DOM 元素或是一个 CSS 选择器字符串：

```html
<div id="app"></div>
```

```js
app.mount('#app')
```

#### DOM 中的根组件模板

根组件的模板通常是组件本身的一部分，但也可以直接通过在挂载容器内编写模板来单独提供：

```html
<div id="app">
  <button @click="count++">{{ count }}</button>
</div>
```

```js
import { createApp } from 'vue'

const app = createApp({
  data() {
    return {
      count: 0
    }
  }
})

app.mount('#app')
```

当根组件没有设置 `template` 选项时，Vue 将自动使用容器的 `innerHTML` 作为模板。

### [多个应用实例](https://cn.vuejs.org/guide/essentials/application.html#multiple-application-instances)

应用实例并不只限于一个。`createApp` API 允许你在同一个页面中创建多个共存的 Vue 应用，而且每个应用都拥有自己的用于配置和全局资源的作用域。

```js
const app1 = createApp({
  /* ... */
})
app1.mount('#container-1')

const app2 = createApp({
  /* ... */
})
app2.mount('#container-2')
```

## 模板语法

### 文本插值

双大括号标签会被替换为[相应组件实例中](https://cn.vuejs.org/guide/essentials/reactivity-fundamentals.html#declaring-reactive-state) `msg` 属性的值。同时每次 `msg` 属性更改时它也会同步更新。

```html
<span>Message: {{ msg }}</span>
```

### 原始HTML

双大括号会将数据解释为纯文本，而不是 HTML。若想插入 HTML，你需要使用 [`v-html` 指令](https://cn.vuejs.org/api/built-in-directives.html#v-html)：

```html
<p>Using text interpolation: {{ rawHtml }}</p>
<p>Using v-html directive: <span v-html="rawHtml"></span></p>
```

![](image-20240401175510523.png)

### [Attribute 绑定](https://cn.vuejs.org/guide/essentials/template-syntax.html#attribute-bindings)

双大括号不能在 HTML attributes 中使用。想要响应式地绑定一个 attribute，应该使用 [`v-bind` 指令](https://cn.vuejs.org/api/built-in-directives.html#v-bind)。

`v-bind` 指令指示 Vue 将元素的 `id` attribute 与组件的 `dynamicId` 属性保持一致。如果绑定的值是 `null` 或者 `undefined`，那么该 attribute 将会从渲染的元素上移除。

```html
<div v-bind:id="dynamicId"></div>
```

简写：

```html
<div :id="dynamicId"></div>
```

Vue3.4及以后版本支持同名简写：

```html
<!-- 与 :id="id" 相同 -->
<div :id></div>

<!-- 这也同样有效 -->
<div v-bind:id></div>
```

### [布尔型 Attribute](https://cn.vuejs.org/guide/essentials/template-syntax.html#boolean-attributes)

[布尔型 attribute](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Attributes#布尔值属性) 依据 `true` / `false` 值来决定 attribute 是否应该存在于该元素上。

当 `isButtonDisabled` 为[真值](https://developer.mozilla.org/en-US/docs/Glossary/Truthy)或一个空字符串 (即 `<button disabled="">`) 时，元素会包含这个 `disabled` attribute。而当其为其他[假值](https://developer.mozilla.org/en-US/docs/Glossary/Falsy)时 attribute 将被忽略：

```html
<button :disabled="isButtonDisabled">Button</button>
```

### [动态绑定多个值](https://cn.vuejs.org/guide/essentials/template-syntax.html#dynamically-binding-multiple-attributes)

如果你有像这样的一个包含多个 attribute 的 JavaScript 对象：

```js
const objectOfAttrs = {
  id: 'container',
  class: 'wrapper'
}
```

通过不带参数的 `v-bind`，你可以将它们绑定到单个元素上：

```html
<div v-bind="objectOfAttrs"></div>
```

### [使用 JavaScript 表达式](https://cn.vuejs.org/guide/essentials/template-syntax.html#using-javascript-expressions)

 Vue 在所有的数据绑定中都支持完整的 JavaScript 表达式：

```html
{{ number + 1 }}

{{ ok ? 'YES' : 'NO' }}

{{ message.split('').reverse().join('') }}

<div :id="`list-${id}`"></div>
```

这些表达式都会被作为 JavaScript ，以当前组件实例为作用域解析执行。

在 Vue 模板内，JavaScript 表达式可以被使用在如下场景上：

- 在文本插值中 (双大括号)
- 在任何 Vue 指令 (以 `v-` 开头的特殊 attribute) attribute 的值中

#### [仅支持表达式](https://cn.vuejs.org/guide/essentials/template-syntax.html#expressions-only)

每个绑定仅支持**单一表达式**，也就是一段能够被求值的 JavaScript 代码。一个简单的判断方法是是否可以合法地写在 `return` 后面。因此，下面的例子都是**无效**的：

```html
<!-- 这是一个语句，而非表达式 -->
{{ var a = 1 }}

<!-- 条件控制也不支持，请使用三元表达式 -->
{{ if (ok) { return message } }}
```

#### [调用函数](https://cn.vuejs.org/guide/essentials/template-syntax.html#calling-functions)

可以在绑定的表达式中使用一个组件暴露的方法：

```html
<time :title="toTitleDate(date)" :datetime="date">
  {{ formatDate(date) }}
</time>
```

> 绑定在表达式中的方法在组件每次更新时都会被重新调用，因此**不**应该产生任何副作用，比如改变数据或触发异步操作。

### 指令 Directives

指令是带有 `v-` 前缀的特殊 attribute。指令 attribute 的期望值为一个 JavaScript 表达式 (除了少数几个例外： `v-for`、`v-on` 和 `v-slot`)。一个指令的任务是在其表达式的值变化时响应式地更新 DOM。

以下[`v-if`](https://cn.vuejs.org/api/built-in-directives.html#v-if) 指令会基于表达式 `seen` 的值的真假来移除/插入该 `<p>` 元素。

```html
<p v-if="seen">Now you see me</p>
```

#### [参数 Arguments](https://cn.vuejs.org/guide/essentials/template-syntax.html#arguments)

某些指令会需要一个“参数”，在指令名后通过一个冒号隔开做标识。例如用 `v-bind` 指令来响应式地更新一个 HTML attribute：

```html
<a v-bind:href="url"> ... </a>

<!-- 简写 -->
<a :href="url"> ... </a>
```

这里 `href` 就是一个参数，它告诉 `v-bind` 指令将表达式 `url` 的值绑定到元素的 `href` attribute 上。在简写中，参数前的一切 (例如 `v-bind:`) 都会被缩略为一个 `:` 字符。

另一个例子是 `v-on` 指令，它将监听 DOM 事件：

```html
<a v-on:click="doSomething"> ... </a>

<!-- 简写 -->
<a @click="doSomething"> ... </a>
```

这里的参数是要监听的事件名称：`click`。`v-on` 有一个相应的缩写，即 `@` 字符。我们之后也会讨论关于事件处理的更多细节。

#### [动态参数](https://cn.vuejs.org/guide/essentials/template-syntax.html#dynamic-arguments)

同样在指令参数上也可以使用一个 JavaScript 表达式，需要包含在一对方括号内：

```html
<template>
<!--
注意，参数表达式有一些约束，
参见下面“动态参数值的限制”与“动态参数语法的限制”章节的解释
-->
<a v-bind:[attributeName]="url"> ... </a>

<!-- 简写 -->
<a :[attributeName]="url"> ... </a>
</template>

<script setup>
const attributeName = 'href';
</script>
```

相似地，你还可以将一个函数绑定到动态的事件名称上：

```html
<template>
<a v-on:[eventName]="doSomething"> ... </a>

<!-- 简写 -->
<a @[eventName]="doSomething"> ... </a>
</template>

<script setup>
const eventName = 'click';
</script>
```

动态参数值的限制：

- 动态参数中表达式的值应当是一个字符串，或者是 `null`。特殊值 `null` 意为显式移除该绑定。其他非字符串的值会触发警告。

动态参数值的限制：

- 动态参数表达式因为某些字符的缘故有一些语法限制，比如空格和引号，在 HTML attribute 名称中都是不合法的。如`<a :['foo' + bar]="value"> ... </a>`这会触发一个编译器警告。
- 当使用 DOM 内嵌模板 (直接写在 HTML 文件里的模板) 时，我们需要避免在名称中使用大写字母，因为浏览器会强制将其转换为小写。如`<a :[someAttr]="value"> ... </a>`会被转换为`:[someattr]`。

#### [修饰符 Modifiers](https://cn.vuejs.org/guide/essentials/template-syntax.html#modifiers)

修饰符是以点开头的特殊后缀，表明指令需要以一些特殊的方式被绑定。例如 `.prevent` 修饰符会告知 `v-on` 指令对触发的事件调用 `event.preventDefault()`：

```html
<form @submit.prevent="onSubmit">...</form>
```

## 响应式基础

### `ref()`

在组合式 API 中，推荐使用 [`ref()`](https://cn.vuejs.org/api/reactivity-core.html#ref) 函数来声明响应式状态：

```js
import { ref } from 'vue'

const count = ref(0)
```

`ref()` 接收参数，并将其包裹在一个带有 `.value` 属性的 ref 对象中返回：

```js
const count = ref(0)

console.log(count) // { value: 0 }
console.log(count.value) // 0

count.value++
console.log(count.value) // 1
```

在模板中使用 ref 时，我们**不**需要附加 `.value`：

```html
<div>{{ count }}</div>
```

你也可以直接在事件监听器中改变一个 ref：

```html
<button @click="count++">
  {{ count }}
</button>
```

在同一作用域内声明更改 ref 的函数：

```js
const count = ref(0)

function increment() {
    // 在 JavaScript 中需要 .value
    count.value++
}
```

然后把方法暴露给事件监听器：

```html
<button @click="increment">
    {{ count }}
</button>
```

Ref 可以持有任何类型的值，包括深层嵌套的对象、数组或者 JavaScript 内置的数据结构，比如 `Map`。

Ref 会使它的值具有深层响应性。这意味着即使改变嵌套对象或数组时，变化也会被检测到：

```js
import { ref } from 'vue'

const obj = ref({
  nested: { count: 0 },
  arr: ['foo', 'bar']
})

function mutateDeeply() {
  // 以下都会按照期望工作
  obj.value.nested.count++
  obj.value.arr.push('baz')
}
```

非原始值将通过 [`reactive()`](https://cn.vuejs.org/guide/essentials/reactivity-fundamentals.html#reactive) 转换为响应式代理。

#### [DOM 更新时机](https://cn.vuejs.org/guide/essentials/reactivity-fundamentals.html#dom-update-timing)

当你修改了响应式状态时，DOM 会被自动更新。但 DOM 更新不是同步的。Vue 会在“next tick”更新周期中缓冲所有状态的修改，以确保不管你进行了多少次状态修改，每个组件都只会被更新一次。

要等待 DOM 更新完成后再执行额外的代码，可以使用 [`nextTick()`](https://cn.vuejs.org/api/general.html#nexttick) 全局 API：

```js
import { nextTick } from 'vue'

async function increment() {
  count.value++
  await nextTick()
  // 现在 DOM 已经更新了
}
```

### `reactive()`

与将内部值包装在特殊对象中的 `ref()` 不同，`reactive()` 将使对象本身具有响应性：

```js
import { reactive } from 'vue'

const state = reactive({ count: 0 })
```

在模板中使用：

```html
<button @click="state.count++">
  {{ state.count }}
</button>
```

响应式对象是 [JavaScript 代理](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)，Vue 能够拦截对响应式对象所有属性的访问和修改，以便进行依赖追踪和触发更新。

`reactive()` 将深层地转换对象：当访问嵌套对象时，它们也会被 `reactive()` 包装。当 ref 的值是一个对象时，`ref()` 也会在内部调用`reactive()`。

**`reactive()` API 有一些局限性：**

1. **有限的值类型**：它只能用于对象类型 (对象、数组和如 `Map`、`Set` 这样的[集合类型](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects#keyed_collections))。它不能持有如 `string`、`number` 或 `boolean` 这样的[原始类型](https://developer.mozilla.org/en-US/docs/Glossary/Primitive)。

2. **不能替换整个对象**：由于 Vue 的响应式跟踪是通过属性访问实现的，因此我们必须始终保持对响应式对象的相同引用。这意味着我们不能轻易地“替换”响应式对象，因为这样的话与第一个引用的响应性连接将丢失。而应该使用`Object.assign()`来将新对象的值复制给响应式对象：

```js
let state = reactive({ count: 0 })

// 上面的 ({ count: 0 }) 引用将不再被追踪。响应性连接已丢失！
state = reactive({ count: 1 })

// 使用Object.assign()不会丢失响应性连接
state = Object.assign(state, {count: 1})
```

3. **对解构操作不友好**：当我们将响应式对象的原始类型属性解构为本地变量时，或者将该属性传递给函数时，我们将丢失响应性连接。当需要对`reactive()`创建的响应性对象进行解构时，需要使用`toRefs()`或`toRef()`。

```js
const state = reactive({ count: 0 })

// 当解构时，count 已经与 state.count 断开连接
let { count } = state
// 不会影响原始的 state
count++

// 该函数接收到的是一个普通的数字
// 并且无法追踪 state.count 的变化
// 我们必须传入整个对象以保持响应性
callSomeFunction(state.count)
```

使用`toRefs` 消费者组件可以解构/展开返回的对象而不会失去响应性：

```js
const state = reactive({
  foo: 1,
  bar: 2
})

const stateAsRefs = toRefs(state)

// 这个 ref 和源属性已经“链接上了”
state.foo++
console.log(stateAsRefs.foo.value) // 2

stateAsRefs.foo.value++
console.log(state.foo) // 3

// 使用 toRefs 对 state 进行解构，不会丢失响应性。
const { bar } = toRefs(state)
```

由于这些限制，我们建议使用 `ref()` 作为声明响应式状态的主要 API。



## watch

- 作用：监视数据的变化（和Vue2中的`watch`作用一致）；

- 特点：Vue3中的`watch`只能监视以下4种数据：

  - `ref`定义的数据（基本类型、对象）；

  - `reactive`定义的对象；
  
  - getter/effect函数，即返回一个值的函数；
  
  - 或者以上类型的数组；

1. `watch`监视`ref`：

```html

```

   



## watchEffet

- 立即运行一个函数，同时响应式地追踪器其中所使用的响应式变量，并在其中的响应式变量更该时重新执行该函数。
- 对比`watch`：
  - `watch`和`watchEffect`都能监听响应式数据的变化；
  - `watch`需要手动指定监视的数据；
  - `watchEffet`不需要手动指定监视的数据，而是通过识别函数中所使用的响应式数据，自动判断需要监视的数据。



## 模板引用：标签中的ref属性

- `ref`属性在HTML原生标签中时，获取`ref`属性所引用的标签；

```html
<template>
	<div>
        <h1>Hello</h1>
        <h1 ref='world'>World</h1>
    </div>
</template>
<script setup lang="ts">
	import { ref } from 'vue'
    const world = ref(); // world将引用<h1 ref='world'>World</h1>HTML原生标签元素
</script>
```

- `ref`属性在组件标签中时，获取`ref`属性所引用的组件实例；

```html
<template>
	<div>
        <h1>Hello</h1>
        <Person ref='person' />
    </div>
</template>
<script setup lang="ts">
import { ref } from 'vue'
const person = ref(); // person将引用Person组件的实例
</script>
```

## props

`defineProps`是编译器宏，不需要导入就可使用，用来定义组件的属性：

```html
// Person.vue
<template>
	<div>
        <h1>{{ msg }}</h1>
    </div>
</template>
<script setup lang="ts">
let x = defineProps(['msg']); // 定义组件属性msg
console.log(x.msg); // haha
</script>
```

在父组件中使用组件`Person`：

```html
<template>
	<Person msg="haha" /> <!-- 父组件为Person组件msg属性设置值 -->
</template>
```

使用泛型指定props类型：

```html
// Person.vue
<template>
	<div>
        <h1>{{ msg }}</h1>
    </div>
</template>
<script setup lang="ts">
let x = defineProps<{ msg: string }>(); // 定义组件属性msg，指定类型为string
console.log(x.msg); // haha
</script>
```

可选props：

```html
// Person.vue
<template>
	<div>
        <h1>{{ msg }}</h1>
    </div>
</template>
<script setup lang="ts">
let x = defineProps<{ msg?:string }>(); // 定义组件可选属性msg
console.log(x.msg); // haha
</script>
```

使用`withDefaults`定义props默认值：

```html
// Person.vue
<template>
	<div>
        <h1>{{ msg }}</h1>
    </div>
</template>
<script setup lang="ts">
let x = withDefaults(defineProps<{ msg?:string }>(), {
    msg: 'yoyo'
})
console.log(x.msg); // 在父组件没有传递msg属性时，输出yoyo
</script>
```

当props是对象，如数组时，默认值需要由函数返回：

```html
// Person.vue
<template>
	<div>
        <h1>{{ msg }}</h1>
    </div>
</template>
<script setup lang="ts">
let x = withDefaults(defineProps<{ msg?:Array<string> }>(), {
    msg: ()=> ['haha', 'yoyo']
})
console.log(x.msg); // 在父组件没有传递msg属性时，输出['haha', 'yoyo']
</script>
```

## 生命周期

### Vue2生命周期

1. `beforeCreate`：初始化选项式API前；
2. `created`：初始化选项式API后；
3. `beforeMount`: 初始渲染前；
4. `mounted`：初始渲染、创建和插入DOM节点后；
5. `beforeUpdate`：重新渲染前；
6. `updated`：重新渲染后；
7. `beforeDestroy`：组件销毁前；
8. `destroyed`：组件销毁后；

```html
<script>
export default {
    data(){
        
    },
    methods{
    
	},
    beforeCreate(){
    
	},
    created(){
        
    },
    beforeMount(){
        
    },
    mounted(){
        
    },
    beforeUpdate(){
        
    },
    updated(){
        
    },
    beforeDestroy(){
        
    },
    destroyed(){
        
    }
}
</script>
```



### Vue3生命周期

1. `setup`：将 Vue2 的`beforeCreate`和`created`合并了；
2. `onBeforeMount`： 初始渲染前；
3. `onMounted`：初始渲染、创建和插入DOM节点后；
4. `onBeforeUpdate`：重新渲染前；
5. `onUpdated`：重新渲染后；
6. `onBeforeUnmount`：组件取消挂载前，相当于 Vue2 的`beforeDestroy`；
7. `onUnmounted`：组件取消挂载后，相当于 Vue2 的`destroyed`；

```html
<script>
import { ref } from 'vue'
export default {
    setup(){
        console.log('创建前');
    
        const val = ref(0);

        console.log('创建后');
        
        onBeforeMount(()=>{
            console.log('初始渲染前');
        });
        
        onMounted(()=>{
            console.log('初始渲染后');
        })
        
        onBeforeUpdate(()=>{
            console.log('更新前');
        })
        
        onUpdated(()=>{
            console.log('更新后');
        });
        
        onBeforeUnmount(()=>{
            console.log('卸载前');
        });
        
        onUnmounted(()=>{
            console.log('卸载后');
        });
    }
}
</script>
```

使用`setup`标签属性：

```html
<script setup>
import { ref } from 'vue'

console.log('创建前');

const val = ref(0);

console.log('创建后');

onBeforeMount(()=>{
	console.log('初始渲染前');
});

onMounted(()=>{
	console.log('初始渲染后');
})

onBeforeUpdate(()=>{
	console.log('更新前');
})

onUpdated(()=>{
	console.log('更新后');
});

onBeforeUnmount(()=>{
	console.log('卸载前');
});

onUnmounted(()=>{
	console.log('卸载后');
});
</script>
```

多组件时，子组件先于父组件被创建。

## Hooks

# 路由 vue-router

## 基本使用步骤

使用`vue-router`创建SPA基本步骤：

1. 创建路由View组件；
2. `createRouter`创建`router`并配置路由参数；
3. `app.use(router)`引入创建好的`router`；
4. 在路由视图View呈现页添加`<RouterLink>`导航和`<RouterView>`；

步骤1：创建路由视图组件。路由视图组件通常放在项目目录的`views/`目录或`pages/`目录中；一般组件放在`components/`目录中。

```html
// HomeView.vue
<script setup lang="ts">
import TheWelcome from '../components/TheWelcome.vue'
</script>

<template>
  <main>
    <TheWelcome />
  </main>
</template>
```

```html
// AboutView.vue

<template>
  <div class="about">
    <h1>This is an about page</h1>
  </div>
</template>

<style>
@media (min-width: 1024px) {
  .about {
    min-height: 100vh;
    display: flex;
    align-items: center;
  }
}
</style>
```

步骤2：`createRouter`创建`router`并配置路由参数：

```ts
import { createRouter, createWebHistory } from 'vue-router'
import HomeView from '../views/HomeView.vue'

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    {
      path: '/',
      name: 'home',
      component: HomeView
    },
    {
      path: '/about',
      name: 'about',
      // route level code-splitting
      // this generates a separate chunk (About.[hash].js) for this route
      // which is lazy-loaded when the route is visited.
      component: () => import('../views/AboutView.vue')
    }
  ]
})

export default router
```

步骤3：`app.use(router)`引入创建好的`router`；

```ts
const app = createApp(App)
app.use(router)
app.mount('#app')
```

步骤4：在路由视图View呈现页添加`<RouterLink>`导航和`<RouterView>`；

```html
<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
</script>

<template>
  <header>
    <div class="wrapper">
      <nav>
        <RouterLink to="/">Home</RouterLink>
        <RouterLink to="/about">About</RouterLink>
      </nav>
    </div>
  </header>

  <RouterView />
</template>
```

## 路由器模式

**history模式：**

- 优点：URL中不带`#`，更加美观，更接近传统网站的URL；
- 缺点：项目上线后，需要后端配合处理路径问题，否则刷新网页会出现404错误；

```ts
import { createRouter, createWebHistory } from 'vue-router'

const router = createRouter({
    history: createWebHistory(), // history模式
})
```

**hash模式：**

- 优点：兼容性更好，不需要后端处理路径问题；
- 缺点：URL中带有`#`，不太美观；对 SEO 不友好；

```ts
import { createRouter, createWebHashHistory } from 'vue-router'

const router = createRouter({
    history: createWebHashHistory(), // hash模式
})
```

## 命名路由

```ts
import { createRouter, createWebHistory } from 'vue-router'
import HomeView from '../views/HomeView.vue'

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    {
      path: '/',
      name: 'home', // 命名路由
      component: HomeView
    },
    {
      path: '/about',
      name: 'about', // 命名路由
      // route level code-splitting
      // this generates a separate chunk (About.[hash].js) for this route
      // which is lazy-loaded when the route is visited.
      component: () => import('../views/AboutView.vue')
    }
  ]
})

export default router
```



## `RouterLink`中`to`属性的2种写法
**字符串写法：**

```html
<RouterLink to="/">首页</RouterLink>
<RouterLink to="/about">关于</RouterLink> 
```

**对象写法：**

```html
<!-- 路径跳转 -->
<RouterLink :to="{path: '/'}">关于</RouterLink>
<RouterLink :to="{path: '/about'}">关于</RouterLink>

<!-- 命名路由跳转 -->
<RouterLink :to="{name: 'home'}">关于</RouterLink>
<RouterLink :to="{name: 'about'}">关于</RouterLink>
```



## 嵌套路由

```ts
const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    {
      path: '/',
      name: 'home',
      component: HomeView
    },
    {
      path: '/news',
      name: 'news',
      component: NewsView,
      children:[ // 子路由
          {
              path: 'sport',
      		 name: 'sport',
              component: SportView,
          },
          {
              path: 'economy',
              name: 'economy',
              component: EconomyView,
          }
      ]
    },
    {
      path: '/about',
      name: 'about', // 命名路由
      // route level code-splitting
      // this generates a separate chunk (About.[hash].js) for this route
      // which is lazy-loaded when the route is visited.
      component: () => import('../views/AboutView.vue')
    }
  ]
})

export default router
```

```html
<RouterLink to="/news/sport">体育</RouterLink>
<RouterLink to="/news/economy">财经</RouterLink>
```

## 路由的querystring参数

**在`RouterLink`的`to`属性中传参：**

- 写法1：

```html
<RouterLink to="/news/sport?id=1&title=sportnews&content=arsenalwin">体育</RouterLink>
<RouterLink to="/news/economy?id=2&title=economynews&content=btcfalldown">财经</RouterLink>
```

- 写法2：

```html
<!-- 路径跳转 -->
<RouterLink :to="{
    path:'/news/sport',
    query:{
		id: news:id,
         title: news.title,
         content: news.content
    	}
	}">体育</RouterLink>

<!-- 命名路由 -->
<RouterLink :to="{
    name:'economy',
    query:{
		id: news:id,
         title: news.title,
         content: news.content
    	}
	}">财经</RouterLink>
```

**在路由视图中使用`useRoute` hooks接收参数：**

```html
<template>
	<ul>
    	<li>{{ route.query.id }}</li>
        <li>{{ route.query.title }}</li>
        <li>{{ route.query.content }}</li>
	</ul>
</template>

<script setup lang="ts">
import { useRoute } from 'vue-router'

let route = useRoute();
</script>
```

## 路由的params参数

配置路由，在`path`属性中使用 params 参数占位符：

```ts
const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    {
      path: '/',
      name: 'home',
      component: HomeView
    },
    {
      path: '/news',
      name: 'news',
      component: NewsView,
      children:[ // 子路由
          {
              path: 'sport/:id/:title/:content', // params占位参数
      		 name: 'sport',
              component: SportView,
          },
          {
              path: 'economy/:id/:title/:content?', // params占位参数；使用问号?指定可选参数
              name: 'economy',
              component: EconomyView,
          }
      ]
    },
    {
      path: '/about',
      name: 'about', // 命名路由
      // route level code-splitting
      // this generates a separate chunk (About.[hash].js) for this route
      // which is lazy-loaded when the route is visited.
      component: () => import('../views/AboutView.vue')
    }
  ]
})

export default router
```

在`RouterLink`中进行传参：

```html
<template>
    <!-- 写法1 -->
	<RouterLink to="/news/sport/1/sportnews/arsenalwin">体育</RouterLink>
	<RouterLink :to="`/news/economy/${news.id}/${news.title}/${news.content}`">财经</RouterLink>
    
    <!-- 写法2: to属性将对象作为params参数传给路由视图组件时，只能使用命名视图路由（name属性），不能使用path属性 -->
    <!-- 注意写法2不允许将对象作为参数 -->
    <RouterLink :to="{
         name:'sport',
         params: {
             id: news.id,
             title: news.title,
             content: news.content
         }
    }">体育</RouterLink>
    <RouterLink :to="{
         name:'economy',
         params: {
             id: news.id,
             title: news.title,
             content: news.content
         }
    }">财经</RouterLink>
    
<RouterView />
</template>

<script>
const news = {
    id: 'id',
    title: 'title',
    content: 'content'
}
</script>
```

在路由视图中获取参数：

```html
<template>
	<ul>
    	<li>{{ route.params.id }}</li>
        <li>{{ route.params.title }}</li>
        <li>{{ route.params.content }}</li>
	</ul>
</template>

<script setup lang="ts">
import { useRoute } from 'vue-router'

const route  = useRoute();
</script>
```

## 路由的props配置

在路由配置中将路由的`props`设置为`true`：

```ts
const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    {
      path: '/',
      name: 'home',
      component: HomeView
    },
    {
      path: '/news',
      name: 'news',
      component: NewsView,
      children:[ // 子路由
          {
              name: 'sport',
              path: 'sport/:id/:title/:content', 
              component: SportView,
              // 写法1：将路由收到的所有params参数作为props传给路由组件
              props: true,
          },
          {
              name: 'economy',
              path: 'economy',
              component: EconomyView,
              // 写法2：props是一个函数，这种写法可以自己决定将什么作为props传给路由组件
              // 此函数接收一个RouteLink传递的路由信息作为参数，可以通过返回其中的特定属性来决定将哪种参数传给路由组件
              props: (route)=>{
                  // 返回路由的params参数
                  // return route.params
                  // 返回路由的querystring参数
                  return route.query
              }
              /*
              // 写法3(不常用)：props是一个对象，可以自己决定将什么作为props传给路由组件
              props: {
                  id: 'id',
                  title: 'title',
                  content: 'content'
              }
              */
          }
      ]
    },
    {
      path: '/about',
      name: 'about', // 命名路由
      // route level code-splitting
      // this generates a separate chunk (About.[hash].js) for this route
      // which is lazy-loaded when the route is visited.
      component: () => import('../views/AboutView.vue')
    }
  ]
})

export default router
```

在`RouterLink`中传递参数的方式与传params参数的方式相同：

```html
<template>
    <!-- 写法1：将路由收到的所有params参数作为props传给路由组件 -->
    <!-- 如前文所述，下面2种写法等价 -->
	<RouterLink to="/news/sport/1/sportnews/arsenalwin">体育</RouterLink>
    <RouterLink :to="{
         name:'sport',
         params: {
             id: news.id,
             title: news.title,
             content: news.content
         }
    }">体育</RouterLink>
    
    <!-- 写法2: props为函数时可以自由决定传递给路由组件的参数 -->
    <!-- 上文配置中返回的路由的querystring，即将querystring作为props传递给路由组件 -->
    <!-- 如前文所述，下面2种写法等价 -->
    <RouterLink to="/news/economy?id=2&title=economynews&content=btcfalldown">财经</RouterLink>
    <RouterLink :to="{
    name:'economy',
    query:{
		id: news:id,
         title: news.title,
         content: news.content
    	}
	}">财经</RouterLink>
    
<RouterView />
</template>

<script setup lang="ts">
const news = {
    id: 'id',
    title: 'title',
    content: 'content'
}
</script>
```

在路由视图中获取参数：

```html
<template>
	<ul>
    	<li>{{ id }}</li>
        <li>{{ title }}</li>
        <li>{{ content }}</li>
	</ul>
</template>

<script setup lang="ts">
defineProps(['id', 'title', 'content']) // 使用defineProps接收RouterLink传来的params参数
</script>
```

## `RouterLink`的`replace`属性

在`RouterLink`组件中使用`replace`属性，则在路由跳转时会使用`replace`模式，浏览器不会留下此URL的历史记录。

```html
<RouterLink replace to="/">首页</RouterLink>
<RouterLink replace to="/about">关于</RouterLink> 
```

与`replace`模式对应的是`push`模式，`push`模式会将访问的URL压入历史记录栈中，从而保存浏览器历史记录。

## 编程式导航

`<RouterLink>`在浏览器渲染后会变成 HTML 原生`<a>`标签，当需要实现点击`<button>`跳转URL，或基于编程来实现URL跳转时，即想要脱离`<RouterLink>`实现路由跳转，需使用编程式路由导航。

简单示例，实现当前组件挂载3秒后自动跳转到路由`/news`：

```html
<template>
</template>

<script setup lang="ts">
import { onMounted } from 'vue'
import { useRouter } from 'vue-router'
    
// 使用useRouter hook 获取 router 对象
const router = useRouter()

// 在当前组件挂载3秒后跳转到路由/news
onMounted(()=>{
    setTimeout(()=>{
        router.push('/news') // push方法即使用push路由模式跳转URL
    }, 3000)
})
</script>
```

示例2，使用编程式导航实现以下`<RouterLink>`路由导航功能：

```html
<template>
  <header>
    <div class="wrapper">
      <nav>
        <RouterLink to="/">Home</RouterLink>
        <RouterLink to="/about">About</RouterLink>
        <RouterLink to="/news/sport">NewsSport</RouterLink>
        <RouterLink to="/news/economy">NewsEconomy</RouterLink>
      </nav>
    </div>
  </header>

  <RouterView />
</template>

<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
</script>
```

实现：

```html
<template>
  <header>
    <div class="wrapper">
      <nav>
        <button @click="toHome">Home</button>
        <button @click="toAbout">About</button>
        <button @click="toNewsSport(news)">NewsSport</button>
        <button @click="toNewsEconomy(news)">NewsEconomy</button>
      </nav>
    </div>
  </header>

  <RouterView />
</template>

<script setup lang="ts">
import { reactive } from 'vue'
import { RouterLink, RouterView, useRouter } from 'vue-router'

const news = reactive({
    id: 'id',
    title: 'title',
    content: 'content'
});

const router = useRouter();

function toHome(){
    // 使用push模式跳转
    router.push('/')
}

function toAbout(){
	// 使用replace模式跳转
    router.replace('/')
}

function toNewsSport(news){
    // push方法接收的参数类型基本与<RouterLink>的to属性接收的参数相同
    // 可以接受字符串形式的路由路径，如上面2个方法中那样；
    // 也可以向路由视图组件传递querystring参数和params参数：
    // router.push('/news/sport?id=id&title=title&content=content')
    // router.push(`/news/sport?id=${news.id}&title=${news.title}&content=${news.content}`)
    
    // router.push('/news/sport/id/title/content')
    router.push(`/news/sport/${news.id}/${news.title}/${news.content}`)
}

function toNewsEconomy(news){
    // 也可以接受对象作为参数，并且也可以向路由视图组件传递querystring参数和params参数：
    router.push({
         path:'/news/economy',
         query: {
             id: news.id,
             title: news.title,
             content: news.content
         }
    })
    
    router.push({
         name:'economy',
         params: {
             id: news.id,
             title: news.title,
             content: news.content
         }
    })
}
</script>
```

## 重定向

在路由配置中进行重定向配置：

```ts
const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    {
      path: '/',
      name: 'home',
      component: HomeView,
      redirect: '/home' // 访问路由/时重定向至/home
    },
    {
      path: '/about',
      name: 'about', 
      component: () => import('../views/AboutView.vue')
    }
  ]
})

export default router
```

# Pinia

安装`pinia`：

```bash
npm i pinia
```

在项目入口`main.ts`文件中引入`pinia`：

```ts
import { createApp } from 'vue'
import App from './App.vue'
// 引入pinia
import { createPinia } from 'pinia'

const app = createApp(App);
// 创建pinia
const pinia = createPinia();
// 安装pinia
app.use(pinia);
app.mount('#app');
```

## 使用Pinia存储和读取数据

根据约定，`store`目录用来存放Pinia定义的`useStore`对象的源文件，文件名通常是与其相关功能相对应。

![](image-20240326214945605.png)

定义 `useCountStore`：

```ts
import { defineStore } from 'pinia'

const useCountStore = defineStore('count', {
    // state方法所返回的对象的属性是各个实际存储数据(state)的变量
    state(){
        return {
            sum: 0 // 实际存储数据的state
        }
    }
})
```

在组件中存取`sum`：

```html
<template>
    <p>
        {{ countStore.sum }}
        {{ countStore.$state.sum }}
    </p>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue'
import { useCountStore } from '@/store/count'

const countStore = useCountStore()

// 以下方式都可以取得sum
console.log(countStore.sum)
console.log(countStore.$state.sum)
</script>
```

## 修改数据的3种方式

**方法1，直接赋值：**

```ts
countStore.sum = 0
```

**方法2，使用`$patch`方法：**

```ts
countStore.$patch({
    sum:888
})

// 如果countStore中有多个存有多个state，也可以一次性修改多个state
countStore.$patch({
    sum:888,
    name: 'name',
    age: 18
})
```

**方法3，在定义store时，定义action方法：**

```ts
import { defineStore } from 'pinia'

const useCountStore = defineStore('count', {
    // actions里面放置的是一个一个的方法，用于响应组件中的“动作”
    actions:{
        increment(){
            this.sum++
            console.log(this.sum)
        },
        add(value){ // action方法也可以接收参数
            this.sum += value
            console.log(this.sum)
        }
    }
    // state方法所返回的对象的属性是各个实际存储数据(state)的变量
    state(){
        return {
            sum: 0 // 实际存储数据的state
        }
    }
})
```

在组件中调用action方法：

```html
<template>
    <p>
        {{ countStore.sum }}
        {{ countStore.$state.sum }}
    </p>
    <button @click="increment">
        increment
    </button>
    <button @click="add(3)">
        add
    </button>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue'
import { useCountStore } from '@/store/count'

const countStore = useCountStore()

function increment(){
    countStore.increment();
}

//
function add(value){
    countStore.add(value);
}
</script>
```

## `storeToRefs`

```html
<script setup lang="ts">
import { ref, reactive } from 'vue'
import { useCountStore, storeToRefs } from '@/store/count'

const countStore = useCountStore()
// 使用storeToRefs结构countStore不会丢失响应式
const { sum } = storeToRefs(countStore)
</script>
```

## `getters`

store的getters类似于Vue组件的计算属性：

```ts
import { defineStore } from 'pinia'

const useCountStore = defineStore('count', {
    state(){
        return {
            sum: 1
        }
    },
    actions:{
        increment(){
            this.sum++
            console.log(this.sum)
        },
        add(value){
            this.sum += value
            console.log(this.sum)
        }
    },
    // 类似于vue组件的计算属性
    getters: {
        bigSum(state){
            return state.sum *10
        }
    }
})
```

取得getters中的值：
```html
<script setup lang="ts">
import { ref, reactive } from 'vue'
import { useCountStore, storeToRefs } from '@/store/count'

const countStore = useCountStore()
const { sum, bigSum } = storeToRefs(countStore)

console.log(sum) // 1
console.log(bigSum) // 10 因为 bigSum = sum * 10 
</script>
```

## `$subscribe`方法

store的`$subscribe`方法可以监控store中state的变化，类似于Vue组件的watch：

```html
<script setup lang="ts">
import { ref, reactive } from 'vue'
import { useCountStore, storeToRefs } from '@/store/count'

const countStore = useCountStore()

countStore.$subscribe((mutation, state)=>{
    console.log(mutation, state)
})
</script>
```

## store的组合式写法

```html
<script setup lang="ts">
import { ref, computed } from 'vue'
import { defineStore } from 'pinia'

export const useCounterStore = defineStore('counter', () => {
  // 组合式写法的state
  const count = ref(0)
  
  // 组合式写法的getter
  const doubleCount = computed(() => count.value * 2)
  
  // 组合式写法的action
  function increment() {
    count.value++
  }

  return { count, doubleCount, increment }
})
</script>
```

# 组件间通信

## props

props 是使用频率最高的组件通信方式，常用于进行父子组件间相互传递数据：

- 父传子：父组件直接向子组件的属性值传递数据；
- 子传父：父组件向子组件的属性值传递一个函数，通过此函数间接获得子组件中的数据。

### 父传子

父组件直接通过子组件的props向子组件传递数据。

父组件：

```html
<template>
	<div class="father">
        <h3>父组件</h3>
        <h4>车：{{ car }}</h4>
        <Child :car="car"/>
    </div>
</template>
<script setup lang="ts">
import Child from './Child.vue'
import { ref } from 'vue'

const car = ref('car')
</script>
```

子组件：

```html
<template>
	<div class="child">
        <h3>子组件</h3>
        <h4>玩具：{{ toy }}</h4>
        <h4>符给的车：{{ car }}</h4>
    </div>
</template>
<script setup lang="ts">
import { ref } from 'vue'

const toy = ref('toy')

// 定义props用于接收来自父组件传递的数据
defineProps(['car'])
</script>
```

### 子传父

“子传父”通过向子的props传递一个函数，来间接拿到数据。

父组件：

```html
<template>
	<div class="father">
        <h3>父组件</h3>
        <h4>车：{{ car }}</h4>
        <!-- 将函数getToy传给子组件 -->
        <Child :car="car" :sendToy="getToy"/>
    </div>
</template>
<script setup lang="ts">
import Child from './Child.vue'
import { ref } from 'vue'

const car = ref('car')

function getToy(value: string){
    console.log('father got', value)
}
</script>
```

子组件：

```html
<template>
	<div class="child">
        <h3>子组件</h3>
        <h4>玩具：{{ toy }}</h4>
        <h4>符给的车：{{ car }}</h4>
        <!-- 子组件调用父组件传来的函数，通过向此函数传参向父组件传递数据 -->
        <button @click="sendToy(toy)">
            把玩具给父亲
        </button>
    </div>
</template>
<script setup lang="ts">
import { ref } from 'vue'

const toy = ref('toy')

// 定义props用于接收来自父组件传递的数据
defineProps(['car', 'sendToy'])

</script>
```

## 自定义事件

通常也可以使用自定义事件来实现 *子组件向父组件* 传递数据。

父组件：

```html
<template>
	<div class="father">
        <h3>父组件</h3>
        <h4>车：{{ car }}</h4>
        <!-- 为子组件的自定义事件sendToy绑定回调函数为getToy -->
        <Child @send-toy="getToy"/>
    </div>
</template>
<script setup lang="ts">
import Child from './Child.vue'
import { ref } from 'vue'
    
const car = ref('car')

function getToy(value: string){
    console.log('father got', value)
}
</script>
```

子组件：

```html
<template>
	<div class="child">
        <h3>子组件</h3>
        <h4>玩具：{{ toy }}</h4>
        <!-- 点击按钮，子组件触发sendToy事件 -->
        <button @click="emit('send-toy')">
            把玩具给父亲
        </button>
        <!-- 不仅可以触发sendToy事件，还可以传递数据 -->
        <button @click="emit('send-toy', toy)">
            把玩具给父亲
        </button>
    </div>
</template>
<script setup lang="ts">
import { ref } from 'vue'

const toy = ref('toy')
// 定义自定义事件sendToy
const emit = defineEmits(['send-toy'])

</script>
```

## mitt

[mitt](https://github.com/developit/mitt) 是一个只有200Bytes的事件发送/订阅库，也可以用来实现组件间通信，并且不仅可以实现父子组件之间的通信，还可以实现兄弟组件甚至任意组件间通信。

使用以下命令安装：

```
$ npm install --save mitt
```

通常将工具库放到项目`src`目录下的`utils`目录中：

![](image-20240327214623386.png)

```ts
// emitter.ts

// 引入mitt
import mitt from 'mitt'

const emitter = mitt()

// 绑定事件
/*
emitter.on('event1', ()=>{
    console.log('event1 emitted')
})

emitter.on('event2', ()=>{
    console.log('event2 emitted')
})
*/

// 暴露emitter
export default emitter
```

实例：

```html
<template>
	<div class="child1">
        <h3>子组件1</h3>
        <h4>玩具：{{ toy }}</h4>
        <!-- 为子组件的自定义事件sendToy绑定回调函数为getToy -->
        <button @click="emitter.emit('send-toy', toy)">
            把玩具给子组件2
        </button>
    </div>
</template>
<script setup lang="ts">
import { ref } from 'vue'
import emitter from '@/utils/emitter'

const toy = ref('toy')

</script>
```

```html
<template>
	<div class="child2">
        <h3>子组件2</h3>
        <h4>子组件1给的玩具：{{ toy }}</h4>
    </div>
</template>
<script setup lang="ts">
import { ref } from 'vue'
import emitter from '@/utils/emitter'

const toy = ref('')

// 绑定事件回调
emitter.on('send-toy', (value)=>{
    console.log('收到来自子组件1的', value);
    toy.value = value
})

// 在组件卸载后解绑事件
onUnmounted(()=>{
    emitter.off('send-toy')
})

</script>
```

## 组件`v-model`

通常，我们使用`v-model`实现双向绑定：

```html
<!-- v-model 用在html标签上 -->
<input type="text" v-model="username">
```

`v-model`其实是以下代码的语法糖，这两行代码其实是等价的：

```html
<input type="text" :value="username" @input="username = $event.target.value">
```

### 底层机制

**其实现双向绑定的本质是：**

- `:value="username"`将`username`的值赋给`<input>`的`value`属性，实现了将`username`的值显示在输入框中。
- `@input="username = $event.target.value"`又将输入框中的值传递给`username`。
- `$event.target.value`中，`$event`是事件对象（在这个例子中是`<input>`元素的input事件）；`target`是被触发事件的目标对象（在这个例子中是`<input>`元素）；`value`为`value`属性，即输入框中的值。

那么接下来以此为思路封装一个`<MyInput>`组件，实现父子组件之间的数据双向绑定：

```html
<template>
  <div class="about">
    <h1>This is an about page</h1>
        <!-- v-model 用在组件标签上 -->
        <MyInput :modelValue="username" @update:modelValue="username = $event"></MyInput>
    </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import MyInput from "@/components/MyInput.vue"

const username = ref('zhangsan')
</script>
```

`MyInput.vue`：

```html
<template>
	<input type="text" :value="modelValue" @input="emit('update:modelValue', $event.target.value)">
</template>

<script setup lang="ts">
defineProps(['modelValue'])
const emit = defineEmits(['update:modelValue'])
</script>
```

**`<MyInput>`实现的本质是：**

- `MyInput`组件定义了一个名为`modelValue`的props。父组件通过这个props将`username`的值传递给`modelValue`。 然后，`MyInput`组件又将`modelValue`收到的值赋给`value`，从而将`username`的内容显示在输入框中。
- `MyInput`组件定义了一个自定义事件`update:modelValue`。当`input`事件被触发时，触发自定义事件`update:modelValue`并将输入框中的值传递给父组件。父组件在自定义事件`update:modelValue`被触发时，将`MyInput`通过事件传递来的值`$event`赋给`username`。此处由于`update:modelValue`是自定义事件，所以`$event`就是通过自定义事件传递的值。

当然，在父组件里你也可以这样写：

```html
<template>
  <div class="about">
    <h1>This is an about page</h1>
    <MyInput :modelValue="username" @update:modelValue="getUsername"></MyInput>
    <p>{{ username }}</p>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import MyInput from "@/components/MyInput.vue"

const username = ref('zhangsan')

function getUsername(value) {
  username.value = value
}
</script>
```

更进一步，也可以在父组件里直接使用`v-model`语法糖：

```html
<template>
  <div class="about">
    <h1>This is an about page</h1>
    <MyInput v-model="username"></MyInput>
    <p>{{ username }}</p>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import MyInput from "@/components/MyInput.vue"

const username = ref('zhangsan')
</script>
```

### 单向数据流

那么为什么`MyInput`不能像下面这样直接使用`v-model`双向绑定props呢？

```html
<template>
    <input type="text" v-model="props.modelValue">
</template>

<script setup lang="ts">
const props = defineProps(['modelValue'])
</script>
```

可以看到eslint报错了：

![](image-20240330141835059.png)

**这其实由于Vue只支持 *单向数据流*，即：**

- 父级组件传递给props的响应式变量更新时会向下传递给子组件，子组件中所有的props都会更新为最新的值；
- 但是反过来则不行，不应该在一个子组件内修改props。

同时，如果`MyInput`像上面那样写，可以看到在Vue DevTools中修改父组件的`username`后，子组件的中输入框中的内容也跟着变化了：

![](image-20240330142605365.png)

但是在Vue DevTools中查看子组件`MyInput`，可以看到props是不能编辑的，同时`update:model-value`也是没有声明的。并且，当你直接修改输入框中的内容后，`props.modelValue`的值也没有变，父组件中`username`的值也没有变。预想中的双向绑定并没有实现。

![](image-20240330143125207.png)

### `v-model`的参数

组件上的 `v-model` 是可以接受一个参数的。如果不指定参数，则默认为`modelValue`，即`v-model`等价于`v-model:modelValue`。也就是说，组件上的`v-model`如果不添加参数，则默认是将值传递给组件的prop `modelValue`。

同样，相应地，组件中自定义事件的默认事件名为`update:modelValue`。

```html
<template>
  <div class="about">
    <h1>This is an about page</h1>
    <!-- 双向绑定到userName props -->
    <MyInput v-model:userName="username"></MyInput>
    <p>{{ username }}</p>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import MyInput from "@/components/MyInput.vue"

const username = ref('zhangsan')
</script>
```

`MyInput`组件：

```html
<template>
    <input type="text" :value="userName" @input="emit('update:userName', $event.target.value)">
</template>

<script setup lang="ts">
// 相应地，子组件中也要把props修改为userName
defineProps(['userName'])
// 相应地，自定义事件也要修改为update:userName
const emit = defineEmits(['update:userName'])
</script>
```

### 多个`v-model`绑定

父组件：

```html
<template>
  <div class="about">
    <h1>This is an about page</h1>
    <MyInput v-model="nickname" v-model:userName="username"></MyInput>
    <p>{{ nickname }}</p>
    <p>{{ username }}</p>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import MyInput from "@/components/MyInput.vue"

const nickname = ref('nickName');
const username = ref('zhangsan')
</script>
```

`MyInput`组件：

```html
<template>
    <input type="text" :value="modelValue" @input="emit('update:modelValue', $event.target.value)">
    <input type="text" :value="userName" @input="emit('update:userName', $event.target.value)">
</template>

<script setup lang="ts">
defineProps(['modelValue', 'userName'])
const emit = defineEmits(['update:modelValue', 'update:userName'])
</script>
```

渲染后效果：

![](image-20240330150656285.png)

## `defineModel`

Vue3.4版本新增的`defineModel`可以简化父子组件之间的双向绑定。是目前官方推荐的双向绑定实现方式。也就是说，从Vue3.4开始，不必再使用上节中的方式实现父子组件之间的双向绑定。

父组件：

```html
<template>
  <div class="about">
    <h1>This is an about page</h1>
    <MyInput v-model="value"></MyInput>
    <p>{{ value }}</p>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import MyInput from "@/components/MyInput.vue"

const value = ref()
</script>
```

`MyInput`组件：

```html
<template>
    <input type="text" v-model="model">
</template>

<script setup lang="ts">
const model = defineModel()
</script>
```

![](image-20240327235845488.png)

渲染后效果为：

![](image-20240327235425665.png)

在子组件的输入框中输入字符，可以看到`<p>`中出现输入的字符，看来父组件的`value`和子组件输入框中的值确实实现了双向绑定。

### `v-model`的参数

使用`defineModel`时，`v-model`也可以接收一个参数：

```html
<MyInput v-model:userName="username"></MyInput>
```

父组件：

```html
<template>
  <div class="about">
    <h1>This is an about page</h1>
    <MyInput v-model:userName="username"></MyInput>
    <p>{{ username }}</p>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import MyInput from "@/components/MyInput.vue"

const username = ref('zhangsan')
</script>
```

在`MyInput`组件中可以通过将字符串作为第一个参数传递给 `defineModel()` 来支持相应的参数：

```html
<template>
    <input type="text" v-model="userName">
</template>

<script setup lang="ts">
const userName = defineModel('userName')
</script>
```

如果需要额外的 prop 选项，应该在 model 名称之后传递：

```ts
const title = defineModel('title', { required: true })
```

### 多个`v-model`绑定

父组件：

```html
<template>
  <div class="about">
    <h1>This is an about page</h1>
    <MyInput v-model="nickname" v-model:userName="username"></MyInput>
    <p>{{ nickname }}</p>
    <p>{{ username }}</p>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import MyInput from "@/components/MyInput.vue"

const nickname = ref('nickName');
const username = ref('zhangsan')
</script>
```

`MyInput`组件：

```html
<template>
    <input type="text" v-model="NickName">
    <input type="text" v-model="UserName">
</template>

<script setup lang="ts">
const NickName = defineModel()
const UserName = defineModel('userName')
</script>
```

## `$attrs`

组件的`$attts`属性可用来实现祖先组件、父组件与孙组件相互通信。其中，孙组件与祖先组件通信需借助祖先组件传递给孙组件一个函数来间接实现，类似于子组件使用props与父组件通信。

### 祖传孙

![](image-20240330182802645.png)

`GrandFather.vue`：

```html
<template>
    <div id="GrandFather">
        <h1>GrandFather</h1>
        <p>a: {{ a }}</p>
        <p>b: {{ b }}</p>
        <p>c: {{ c }}</p>
        <p>d: {{ d }}</p>
        <!-- 以下两行是等价的，v-bind是可以传入对象的 -->
        <!-- <Father :a="a" :b="b" :c="c" :d="d" /> -->
        <Father v-bind="{ a, b, c, d }" />
    </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import Father from './Father.vue';

const a = ref('a')
const b = ref('b')
const c = ref('c')
const d = ref('d')
</script>
```

`Father.vue`：

```html
<template>
    <div id="Father">
        <h1>Father</h1>
        <p>a: {{ a }}</p>
        <p>未接收的其余props: {{ $attrs }}</p>
        <Child v-bind="$attrs" />
    </div>
</template>

<script setup lang="ts">
import Child from './Child.vue'
defineProps(['a'])

</script>
```

`Child.vue`：

```html
<template>
    <div id="Child">
        <h1>Child</h1>
        <p>未接收的其余props: {{ $attrs }}</p>
    </div>
</template>

<script setup lang="ts">

</script>
```

渲染后为：

![](image-20240330183054059.png)

### 孙传祖

通过将祖先组件的方法传递给孙组件，孙组件可以间接修改祖先组件中的响应式对象：

![](image-20240330184416815.png)

`GrandFather.vue`：

```html
<template>
    <div id="GrandFather">
        <h1>GrandFather</h1>
        <p>a: {{ a }}</p>
        <p>b: {{ b }}</p>
        <p>c: {{ c }}</p>
        <p>d: {{ d }}</p>
        <!-- 将方法changeA传递给Father -->
        <Father v-bind="{ a, b, c, d, changeA }" />
    </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import Father from './Father.vue';

const a = ref('a')
const b = ref('b')
const c = ref('c')
const d = ref('d')

function changeA(newValue: string) {
    a.value = newValue
}
</script>
```

`Father.vue`：

```html
<template>
    <div id="Father">
        <h1>Father</h1>
        <p>a: {{ a }}</p>
        <p>未接收的其余props: {{ $attrs }}</p>
        <Child v-bind="$attrs" />
    </div>
</template>

<script setup lang="ts">
import Child from './Child.vue'
defineProps(['a'])
</script>
```

`Child.vue`：

```html
<template>
    <div id="Child">
        <h1>Child</h1>
        <p>未接收的其余props: {{ $attrs }}</p>
        <!-- 点击按钮修改祖先组件中的ref a -->
        <button @click="changeA('changeA')">changeA</button>
    </div>
</template>

<script setup lang="ts">
// 从props中接收祖先组件传来的changeA方法
defineProps(['changeA'])
</script>
```

渲染后，点击按钮前：

![](image-20240330184834433.png)

点击按钮后：

![](image-20240330184856590.png)

## `$refs`和`$parent`

**`$refs`** ：

- 值为对象，包含所有被 `ref` 属性（模板引用）标识的 DOM 元素或组件实例。

- 可通过`$refs`实现父传子通信。

**`$parent`** ：

- `$parent`是当前组件的父组件的对象引用。
- 可通过`$parent`实现子传父通信。

`Father.vue`：

```html
<template>
    <div class="Father">
        <h1>Father</h1>
        <p>car: {{ car }}</p>
        <button @click="changeToy('ttooyy')">修改toy</button>

        <!-- $refs可以获得当前组件所有通过模板引用ref获取到的子组件，即c1和c2 -->
        <button @click="changeAll($refs)">changeAll</button>

        <Child ref="c1" />
        <Child2 ref="c2" />

    </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import Child from './Child.vue'
import Child2 from './Child2.vue'

const car = ref('car')
const c1 = ref()
const c2 = ref()

function changeToy(newValue: string) {
    c1.value.toy = newValue
}

function changeAll(refs: any) {
    for (const key in refs) {
        // 如果子组件中有book属性
        if ('book' in refs[key]) {
            // 将book改为'three body problems'
            refs[key]['book'] = 'three body problems'
        }

        // 如果子组件中有game属性
        if ('game' in refs[key]) {
            // 将game改为'palworld'
            refs[key]['game'] = 'palworld'
        }
    }
}

// 将car暴露给子组件
defineExpose({ car })
</script>

<style scoped></style>
```

`Child.vue`：

```html
<template>
    <div class="Child">
        <h1>Child</h1>
        <p>toy: {{ toy }}</p>
        <p>book: {{ book }}</p>
        <!-- $parent 是父组件对象的引用 -->
        <button @click="changeCar($parent)">给爹换辆BMW</button>
    </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const toy = ref('toy')
const book = ref('book')

function changeCar(parent: any) {
    parent.car = 'BMW'
}

// 由于Vue的单向数据流，如果不使用defineExpose将数据暴露出去，
// 那么，父组件即便使用ref模板引用获取到子组件对象，也没法修改子组件的数据

// 将数据暴露给父组件
defineExpose({ toy, book })
</script>

<style scoped></style>
```

`Child2.vue`：

```html
<template>
    <div class="Child2">
        <h1>Child2</h1>
        <p>computer: {{ computer }}</p>
        <p>game: {{ game }}</p>
    </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const computer = ref('computer')
const game = ref('game')

// 将数据暴露给父组件
defineExpose({ computer, game })
</script>

<style scoped></style>
```

渲染后：

![](image-20240330194816099.png)

点击“修改toy”：

![](image-20240330194832278.png)

点击“changeAll”：

![](image-20240330194846111.png)

点击“给爹换辆BMW”：

![](image-20240330194922057.png)

## `provide`和`inject`

`GrandFather.vue`：

```html
<template>
    <div class="GrandFather">
        <h1>GrandFather</h1>
        <p>money: {{ money }}</p>
        <p>car: {{ car }}</p>

        <Father />
    </div>
</template>

<script setup lang="ts">
import { provide, reactive, ref } from 'vue'
import Father from './Father.vue';

const money = ref(100)
const car = reactive({
    brand: 'BMW',
    price: 100
})

function getMoney(val: number) {
    money.value += val
}

// 向后代组件提供数据
provide('money', money)
provide('car', car)
provide('giveMoney', getMoney)
</script>

<style scoped>
.GrandFather {
    background-color: aquamarine;
}
</style>
```

`Fahter.vue`：

```html
<template>
    <div class="Father">
        <h1>Father</h1>
        <p>car: {{ c }}</p>

        <Child />
    </div>
</template>

<script setup lang="ts">
import { inject } from 'vue';
import Child from './Child.vue'

// 取出GrandFather provide的car
const c = inject('car')
</script>

<style scoped>
.Father {
    background-color: yellow;
}
</style>
```

`Child.vue`：

```html
<template>
    <div class="Child">
        <h1>Child</h1>
        <p>money: {{ m }}</p>
        <p>car: {{ c }}</p>
        <button @click="giveMoney(100)">给爷爷钱</button>
    </div>
</template>

<script setup lang="ts">
import { inject } from 'vue';

// inject可以直接收一个参数，但如果没有provide则为undefined
// const m = inject('money') // 取出GrandFather provide的money
// const c = inject('car') // 取出GrandFather provide的car

// inject也可以接收默认值作为第二个参数，当第一个参数没有provide时，会返回默认值
const m = inject('money', '默认值')
const c = inject('car', { brand: 'mini', price: 50 })
const giveMoney = inject('giveMoney', (v: number) => { v })
</script>
<style scoped>
.Child {
    background-color: pink;
}
</style>
```

渲染后：

![](image-20240330225823642.png)

点击“给爷爷钱”：

![](动画.gif)

 ## Slots

### 默认插槽

`Father.vue`：

```html
<template>
    <div class="Father">
        <h1>Father</h1>
        <div class="content">
            <Child>
                <ul>
                    <li v-for="(game, index) in games" :key="index">
                        {{ game }}
                    </li>
                </ul>
            </Child>
            <Child>
                <ul>
                    <li v-for="(game, index) in games" :key="index">
                        {{ game }}
                    </li>
                </ul>
            </Child>
            <Child>
                <ul>
                    <li v-for="(game, index) in games" :key="index">
                        {{ game }}
                    </li>
                </ul>
            </Child>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue';
import Child from './Child.vue'

const games = reactive([
    'FIFA 23',
    "Palworld",
    "WoW",
    "Final Fantasy"
])

</script>

<style scoped>
.Father {
    background-color: yellow;
    height: 400px;
}

.content {
    display: flex;
    justify-content: space-evently;
}
</style>
```

`Child.vue`：

```html
<template>
    <div class="Child">
        <h1>Child</h1>
        <slot></slot>
    </div>
</template>

<script setup lang="ts">

</script>
<style scoped>
.Child {
    background-color: pink;
    width: 200px;
    height: 200px;
    margin: 20px;
    border: 2px solid white;
}
</style>
```

渲染后：

![](image-20240330234559849.png)

### 具名插槽

`Father.vue`：

```html
<template>
    <div class="Father">
        <h1>Father</h1>
        <div class="content">
            <Child>
                <template v-slot:s1>
                    <h2>热门游戏</h2>
                </template>
                <template v-slot:s2>
                    <ul>
                        <li v-for="(game, index) in games" :key="index">
                            {{ game }}
                        </li>
                    </ul>
                </template>
            </Child>
            <Child>
                <!-- #是v-slot的简便写法 -->
                <template #s3>
                    <h2>今日美食</h2>
                    <ul>
                        <li v-for="(item, index) in foods" :key="index">
                            {{ item }}
                        </li>
                    </ul>
                </template>
            </Child>
            <Child>
                <!-- #是v-slot的简便写法 -->
                <template #s4>
                    <h2>今日影视</h2>
                    <ul>
                        <li v-for="(item, index) in films" :key="index">
                            {{ item }}
                        </li>
                    </ul>
                </template>
            </Child>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue';
import Child from './Child.vue'

const games = reactive([
    'FIFA 23',
    "Palworld",
    "WoW",
    "Final Fantasy"
])

const foods = reactive([
    "KFC",
    "Burger King",
    "McDonald"
])

const films = reactive([
    'Mission Impossible',
    "007",
    "WoW",
    "Final Fantasy"
])

</script>

<style scoped>
.Father {
    background-color: yellow;
    height: 400px;
}

.content {
    display: flex;
    justify-content: space-evently;
}
</style>
```

`Child.vue`：

```html
<template>
    <div class="Child">
        <slot name="s1"></slot>
        <slot name="s2"></slot>
        <slot name="s3"></slot>
        <slot name="s4"></slot>
    </div>
</template>

<script setup lang="ts">
</script>

<style scoped>
.Child {
    background-color: pink;
    width: 200px;
    height: 200px;
    margin: 20px;
    border: 2px solid white;
}
</style>
```

渲染后：

![](image-20240331000844416.png)



### 作用域插槽

在某些场景下插槽的内容可能想要**在父组件中使用子组件作用域中的数据**，或者，**同时使用父组件域内和子组件域内的数据**。要做到这一点，我们需要使用作用域插槽，来让子组件在渲染时将一部分数据提供给插槽。

`Father.vue`：

```html
<template>
    <div class="Father">
        <h1>Father</h1>
        <div class="content">
            <Child>
                <!-- 获取子组件传给slot的props，这里变量名params可以随便起 -->
                <template v-slot="params">
                    <ul>
                        <!-- 现在便可以从params中取出子组件传给slot的propsGames -->
                        <li v-for="(game, index) in params.propsGames" :key="index">
                            {{ game }}
                        </li>
                    </ul>
                </template>
            </Child>
            <Child>
                <!-- 这里变量名可以随便起，例如上面是params，这里是args-->
                <template v-slot="args">
                    <ol>
                        <li v-for="(game, index) in args.propsGames" :key="index">
                            {{ game }}
                        </li>
                    </ol>
                </template>
            </Child>
            <Child>
                <!-- 也直接对象解构 -->
                <template v-slot="{ propsGames }">
                    <p v-for="(game, index) in propsGames" :key="index">
                        {{ game }}
                    </p>
                </template>
            </Child>

        </div>
    </div>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue';
import Child from './Child.vue'

</script>

<style scoped>
.Father {
    background-color: yellow;
    height: 400px;
}

.content {
    display: flex;
    justify-content: space-evently;
}
</style>
```

`Child.vue`：

```html
<template>
    <div class="Child">
        <h2>游戏列表</h2>
        <!-- 将games作为props传递给slot，这样就可以在父组件中获取到 -->
        <slot :propsGames="games"></slot>
    </div>
</template>

<script setup lang="ts">
import { reactive } from 'vue';
const games = reactive([
    'FIFA 23',
    "Palworld",
    "WoW",
    "Final Fantasy"
])
</script>

<style scoped>
.Child {
    background-color: pink;
    width: 200px;
    margin: 20px;
    border: 2px solid white;
}
</style>
```

渲染后：

![](image-20240331221005074.png)

### 具名作用域插槽

```html
<MyComponent>
  <template v-slot:header="headerProps">
    {{ headerProps }}
  </template>

  <template #default="defaultProps">
    {{ defaultProps }}
  </template>

  <template #footer="footerProps">
    {{ footerProps }}
  </template>
</MyComponent>
```

向具名插槽中传入 props：

```html
<slot name="header" message="hello"></slot>
```



# 其他常用API

## `shallowRef`与`shallowReactive`

**`shallowRef` **

- 作用：创建一个响应式数据，但是只对顶层属性进行响应式处理。
- 用法：

```ts
let myVar = shallowRef('initialValue');
```

- 特点：只跟踪引用值的变化，不关心值内部属性的变化。

**`shallowReactvie`**

- 作用：创建一个浅层响应式对象，只会使对象的最顶层属性变成响应式的，对象内部的嵌套属性则不会变成响应式的。
- 用法：

```ts
const myObj = shallowReactive({});
```

- 特点：对象的顶层属性是响应式的，但嵌套对象的属性不是。

## `readonly`和`shallowReadonly`

## `toRaw`和`markRaw`

## `customRef`

```ts
let initValue = 'initValue'
let msg = customRef((track, trigger)=>{
    return {
        // msg 被读取时调用
        get(){
            track() // 跟踪msg的变化，一旦msg变化就更新
            return initValue
        },
        // msg被修改时调用
        set(val){
            initValue = val
            trigger() // msg发生变化了，通知Vue
        }
    }
})
```

使用`customRef`封装一个hooks，实现输入框输入数据时延迟显示：

```ts
// useMsgRef.ts
import { customRef } from 'vue'

export default function(initValue: string, delay: number){
    let timer: number
    let msg = customRef((track, trigger)=>{
        return {
            // msg 被读取时调用
            get(){
                track() // 跟踪msg的变化，一旦msg变化就更新
                return initValue
            },
            // msg被修改时调用
            set(val){
                clearTimeout(timer)
                timer = setTimeout(()=>{
                    initValue = val
                	trigger() // msg发生变化了，通知Vue
                }, delay)
            }
        }
    })
    
    return {
        msg
    }
}

```

使用这个`useMsgRef.ts`：

```html
<template>
    <div class="app">
        <h2>{{ msg }}</h2>
        <input type="text" v-model="msg">
    </div>
</template>

<script setup lang="ts">
import { reactive } from 'vue';
import useMsgRef from './useMsgRef'

// 使用useMsgRef来定义一个响应式数据且延迟2秒显示
const { msg } = useMsgRef('你好', 2000);
</script>
```

效果：

![](adsfasd.gif)

