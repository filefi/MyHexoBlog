---
title: Vue3 核心
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



## 3.9 watch

- 作用：监视数据的变化（和Vue2中的`watch`作用一致）；

- 特点：Vue3中的`watch`只能监视以下4种数据：

  - `ref`定义的数据（基本类型、对象）；

  - `reactive`定义的对象；
  
  - getter/effect函数，即返回一个值的函数；
  
  - 或者以上类型的数组；

1. `watch`监视`ref`：

```vue

```

   



## 3.10 watchEffet

- 立即运行一个函数，同时响应式地追踪器其中所使用的响应式变量，并在其中的响应式变量更该时重新执行该函数。
- 对比`watch`：
  - `watch`和`watchEffect`都能监听响应式数据的变化；
  - `watch`需要手动指定监视的数据；
  - `watchEffet`不需要手动指定监视的数据，而是通过识别函数中所使用的响应式数据，自动判断需要监视的数据。



## 3.11 标签中的ref属性

- `ref`属性在HTML原生标签中时，获取`ref`属性所引用的标签；

```vue
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

```vue
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

## 3.12 props

`defineProps`是编译器宏，不需要导入就可使用，用来定义组件的属性：

```vue
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

```vue
<template>
	<Person msg="haha" /> <!-- 父组件为Person组件msg属性设置值 -->
</template>
```

使用泛型指定props类型：

```vue
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

```vue
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

```vue
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

```vue
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

## 3.13 生命周期

### 3.13.1 Vue2生命周期

1. `beforeCreate`：初始化选项式API前；
2. `created`：初始化选项式API后；
3. `beforeMount`: 初始渲染前；
4. `mounted`：初始渲染、创建和插入DOM节点后；
5. `beforeUpdate`：重新渲染前；
6. `updated`：重新渲染后；
7. `beforeDestroy`：组件销毁前；
8. `destroyed`：组件销毁后；

```vue
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



### 3.13.2 Vue3生命周期

1. `setup`：将 Vue2 的`beforeCreate`和`created`合并了；
2. `onBeforeMount`： 初始渲染前；
3. `onMounted`：初始渲染、创建和插入DOM节点后；
4. `onBeforeUpdate`：重新渲染前；
5. `onUpdated`：重新渲染后；
6. `onBeforeUnmount`：组件取消挂载前，相当于 Vue2 的`beforeDestroy`；
7. `onUnmounted`：组件取消挂载后，相当于 Vue2 的`destroyed`；

```vue
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

```vue
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

## 3.14 Hooks

# 4 路由 vue-router

## 4.1 基本使用步骤

使用`vue-router`创建SPA基本步骤：

1. 创建路由View组件；
2. `createRouter`创建`router`并配置路由参数；
3. `app.use(router)`引入创建好的`router`；
4. 在路由视图View呈现页添加`<RouterLink>`导航和`<RouterView>`；

步骤1：创建路由视图组件。路由视图组件通常放在项目目录的`views/`目录或`pages/`目录中；一般组件放在`components/`目录中。

```vue
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

```vue
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

```vue
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

## 4.2 路由器模式

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

## 4.3 命名路由

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



## 4.4 `RouterLink`中`to`属性的2种写法
**字符串写法：**

```vue
<RouterLink to="/">首页</RouterLink>
<RouterLink to="/about">关于</RouterLink> 
```

**对象写法：**

```vue
<!-- 路径跳转 -->
<RouterLink :to="{path: '/'}">关于</RouterLink>
<RouterLink :to="{path: '/about'}">关于</RouterLink>

<!-- 命名路由跳转 -->
<RouterLink :to="{name: 'home'}">关于</RouterLink>
<RouterLink :to="{name: 'about'}">关于</RouterLink>
```



## 4.5 嵌套路由

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
              component: HomeView,
          },
          {
              path: 'economy',
              name: 'economy',
              component: HomeView,
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

```vue
<RouterLink to="/news/sport">体育</RouterLink>
<RouterLink to="/news/economy">财经</RouterLink>
```

## 4.6 路由的querystring参数

**在`RouterLink`的`to`属性中传参：**

- 写法1：

```vue
<RouterLink to="/news/sport?id=1&title=sportnews&content=arsenalwin">体育</RouterLink>
<RouterLink to="/news/economy?id=2&title=economynews&content=btcfalldown">财经</RouterLink>
```

- 写法2：

```vue
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

```vue
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

## 4.7 路由的props







