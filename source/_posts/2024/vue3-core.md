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



## 标签中的ref属性

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



