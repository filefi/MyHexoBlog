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



