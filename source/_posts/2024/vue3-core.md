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
<div id="app">{{ message }}</div>

<script type="module">
  import { createApp, ref } from 'https://unpkg.com/vue@3/dist/vue.esm-browser.js'

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



## `setup()`

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

`expose` 函数用于显式地限制该组件暴露出的属性，当父组件通过[模板引用](https://cn.vuejs.org/guide/essentials/template-refs.html#ref-on-component)访问该组件的实例时，将仅能访问 `expose` 函数暴露出的内容：

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

## getters

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



# 组件通信



