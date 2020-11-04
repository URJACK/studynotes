# Vue3

## 基础知识

### 生命周期

![img](.\1\1.png)

### Vue交互

#### axios通信

使用Vue进行前后端分离开发重要的就是发起跨域请求。

在main.js中，我们需要做两件事

```javascript
//引入系统的axios库
import axios from 'axios'
//将系统的axios库放入Vue原型中去
Vue.prototype.$axios = axios
```

之后就可以在代码中使用了

```javascript
      this.$axios
        .post(process.env.VUE_APP_AXIOS_URL + "/vps/search", {
          serverName: "lubada43.0",
          serverIp: "192.168.0.1",
          page: "1",
        })
        .then((res) => {
          const data = res.data.data;
          //打印返回头信息的相关数据
          console.log(res.headers);
          //打印返回的数据信息
          console.log(data);
        });
```

需要注意的是，这里的process.env.VUE_APP_AXIOS_URL是一个写在

".env.development"

```
VUE_APP_AXIOS_URL=http://localhost:8080
```

或者".env.production"的两个变量

```
VUE_APP_AXIOS_URL=http://120.77.151.214:8080
```

至此，axios在Vue中通信的相关配置就完成了。

##### 后端的配合

需要注意的是，这里需要后台也有相应的配合

比如在springboot中，我们需要在这里也使用一些跨域的配置

```java
package com.ffz.demo.configuration;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class CorsConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("*")
                .allowedMethods("GET", "HEAD", "POST", "PUT", "DELETE", "OPTIONS")
                .allowCredentials(true)
                .maxAge(3600)
                .allowedHeaders("*");
    }
}
```

#### Vue路由

对于Vue来说，Vue的路由是必须的。

##### router文件

我们可以新建一个router文件，存储位置放在哪里都行，因为最终是在main.js中去调用它

注意点有三个：

1·Vue 去使用 VueRouter

2·新建一个**route对象数组**。

route对象基础属性包括了path，name，component

其他属性包括了redirect，meta

3·使用**route对象数组**来创建一个VueRouter

```javascript
import Vue from 'vue'
import VueRouter from 'vue-router'
import BlogDetail from '../views/BlogDetail.vue'
import BlogEdit from '../views/BlogEdit.vue'
import Login from '../views/Login.vue'

Vue.use(VueRouter)

const routes = [
  {
    path: '/',
    name: 'Index',
    redirect: { name: 'Blogs' }
  },
  {
    path: '/login',
    name: 'Login',
    component: Login
  },
  {
    path: '/blogs',
    name: 'Blogs',
    component: () => import('../views/Blogs.vue')
  },
  {
    path: '/blog/add',
    name: 'BlogAdd',
    meta: {
      requireAuth: true
    },
    component: BlogEdit
  },
  {
    path: '/blog/:blogId',
    name: 'BlogDetail',
    component: BlogDetail
  },
  {
    path: '/blog/:blogId/edit'
    name: 'BlogEdit',
    meta: {
      requireAuth: true
    },
    component: BlogEdit
  }
]

const router = new VueRouter({
  mode: 'hash',
  base: process.env.BASE_URL,
  routes
})

export default router
```

##### Vue的引用

在这里使用<router-view>就可以了

```
<template>
  <div id="app">
    <router-view/>
  </div>
</template>
```

##### Vue路由传参

传参的时候使用<router-link>

```html
<router-link :to="{name:'BlogDetail',params:{blogId:blog.id}}">
	<h4>{{blog.title}}</h4>
</router-link>
```

接受参数使用this.$route.params.xxx

```javascript
created() {
	const blogId = this.$route.params.blogId;
}
```

### Vue组件

#### 组件的注册

这是局部注册组件，仅在当前组件可以使用。也是之前刚学使用.Vue文件开发Vue应用的时候最基础的方法。

```js
import ComponentA from './ComponentA'
import ComponentC from './ComponentC'

export default {
  components: {
    ComponentA,
    ComponentC
  }
  // ...
}
```

这是全局注册组件，注册后的组件应该是可以在整个App中进行使用

```js
const app = Vue.createApp({...})

app.component('my-component-name', {
  /* ... */
})
```

#### 组件之间的通信

props参数

```js
props: {
  title: String,
  likes: Number,
  isPublished: Boolean,
  commentIds: Array,
  author: Object,
  callback: Function,
  contactsPromise: Promise // 或任何其他构造函数
}
```

可以给组件定义props，进而可以作为从外部获取的参数

```html
<!--静态值-->
<blog-post title="My journey with Vue"></blog-post>

<!-- 动态赋予一个变量的值 -->
<blog-post :title="post.title"></blog-post>

<!-- 动态赋予一个复杂表达式的值 -->
<blog-post :title="post.title + ' by ' + post.author.name"></blog-post>
```

### Prop

#### 单向数据流

所有的prop都让父->子之间形成了一种**单向下行绑定**。父级prop的更新会向下流动到子组件中。

父级组件发生更新，会让子组件的prop值进行刷新。所以**不应该**在子组件中对prop的值进行修改。

```javascript
props: ['initialCounter'],
data() {
  return {
    counter: this.initialCounter
  }
}
```

```javascript
props: ['size'],
computed: {
  normalizedSize: function () {
    return this.size.trim().toLowerCase()
  }
}
```

#### Prop验证

```js
app.component('my-component', {
  props: {
    // 基础的类型检查 (`null` 和 `undefined` 会通过任何类型验证)
    propA: Number,
    // 多个可能的类型
    propB: [String, Number],
    // 必填的字符串
    propC: {
      type: String,
      required: true
    },
    // 带有默认值的数字
    propD: {
      type: Number,
      default: 100
    },
    // 带有默认值的对象
    propE: {
      type: Object,
      // 对象或数组默认值必须从一个工厂函数获取
      default: function() {
        return { message: 'hello' }
      }
    },
    // 自定义验证函数
    propF: {
      validator: function(value) {
        // 这个值必须匹配下列字符串中的一个
        return ['success', 'warning', 'danger'].indexOf(value) !== -1
      }
    },
    // 具有默认值的函数
    propG: {
      type: Function,
      // 与对象或数组默认值不同，这不是一个工厂函数 —— 这是一个用作默认值的函数
      default: function() {
        return 'Default function'
      }
    }
  }
})
```

还可以进行自定义类型的验证

```js
function Person(firstName, lastName) {
  this.firstName = firstName
  this.lastName = lastName
}
```

```js
app.component('blog-post', {
  props: {
    author: Person
  }
})
```

#### Prop大小写

HTML 中的 attribute 名是大小写不敏感的，所以浏览器会把所有大写字符解释为小写字符。

这意味着当你使用 DOM 中的模板时，camelCase (驼峰命名法) 的 prop 名需要使用其等价的 kebab-case (短横线分隔命名) 命名：

```js
const app = Vue.createApp({})

app.component('blog-post', {
  // camelCase in JavaScript
  props: ['postTitle'],
  template: '<h3>{{ postTitle }}</h3>'
})
```

```html
<!-- kebab-case in HTML -->
<blog-post post-title="hello!"></blog-post>
```

## 易错细节

### 访问DOM

```html
<template>
  <div class="container" id="app">
    Canvas
    <canvas id="oCanvas" width="600" height="600"></canvas>
    <button v-on:click="debug">click</button>
  </div>
</template>
```

```javascript
  created() {
    console.log("CanvasPoint created");
    app = this;
    window.onload = function() {
      //在vue的函数中，使用oCanvas就默认使用了document.getElementById(oCanvas)
      initCanvas(oCanvas);
    };
  },
```

### 函数命名

```javascript
    getDrawArrayItemValue(y, x) {
      //正确的命名方式
      return this.drawArray[app.width * y + x];
    },
    getDrawArrayItemValue: (y, x) => {
      //该种命名方式，会导致使用this无法访问自己的成员变量与函数
      return this.drawArray[app.width * y + x];
    },
```

