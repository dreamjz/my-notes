---
title: 'Vue Router'
date: '2021-12-07'
categories:
 - frontend
 - vue
tags:
 - vue router
publish: true
---

# Vue Router

## 1. 安装

### 1.1 直接下载 / CDN

[https://unpkg.com/vue-router/dist/vue-router.js(opens new window)](https://unpkg.com/vue-router/dist/vue-router.js)

[Unpkg.com (opens new window)](https://unpkg.com/)提供了基于 NPM 的 CDN 链接。上面的链接会一直指向在 NPM 发布的最新版本。你也可以像 `https://unpkg.com/vue-router@2.0.0/dist/vue-router.js` 这样指定 版本号 或者 Tag。

在 Vue 后面加载 `vue-router`，它会自动安装的：

```html
<script src="/path/to/vue.js"></script>
<script src="/path/to/vue-router.js"></script>
```

### 1.2 NPM

```bash
npm install vue-router
```

如果在一个模块化工程中使用它，必须要通过 `Vue.use()` 明确地安装路由功能：

```js
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)
```

如果使用全局的 script 标签，则无须如此 (手动安装)

### 1.3 Vue CLI

如果你有一个正在使用 [Vue CLI (opens new window)](https://cli.vuejs.org/zh/)的项目，你可以以项目插件的形式添加 Vue Router。CLI 可以生成上述代码及两个示例路由。**它也会覆盖你的 `App.vue`**，因此请确保在项目中运行以下命令之前备份这个文件：

```sh
vue add router
```

### 1.4 构建开发版

如果你想使用最新的开发版，就得从 GitHub 上直接 clone，然后自己 build 一个 `vue-router`。

```bash
git clone https://github.com/vuejs/vue-router.git node_modules/vue-router
cd node_modules/vue-router
npm install
npm run build
```

## 2. 基础

用 Vue.js + Vue Router 创建单页应用，感觉很自然：使用 Vue.js ，我们已经可以通过组合组件来组成应用程序，当你要把 Vue Router 添加进来，我们需要做的是，将组件 (components) 映射到路由 (routes)，然后告诉 Vue Router 在哪里渲染它们

### 2.1 起步

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <!-- import vue -->
    <script src="https://unpkg.com/vue/dist/vue.js"></script>
    <!-- import vue router -->
    <script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>

</head>

<body>
    <div id="app">
        <h1>Hello Vue Router</h1>
        <p>
            <!-- 使用 router-link 组件来导航, 默认渲染成 <a> 标签-->
            <!-- to 属性用于指定链接 -->
            <router-link to="/foo">Go to Foo</router-link>
            <br>
            <router-link to="/bar">Go to Bar</router-link>
        </p>
        <!-- 路由出口 -->
        <!-- 路由匹配到的组件将会渲染在此 -->
        <router-view></router-view>
    </div>
    <!-- app script -->
    <script src="app.js"></script>
</body>

</html>
```

```js
// 0 若使用模块化机制编程，导入 Vue 和 VueRouter 需要使用 Vue.use(Vuerouter)

// 1. 定义路由组件
// 可以从其他文件中导入
const Foo = { template: "<div>foo</div>" };
const Bar = { template: "<div>bar</div>" };

// 2. 定义路由
// 每个路由映射一个组件
// compoent 可以是 Vue.extend() 创建的组件构造器
// 也可以使组件配置对象
const routes = [
    { path: "/foo", component: Foo },
    { path: "/bar", component: Bar },
];

// 3. 创建 router 实例
// 传入 routes 配置
const router = new VueRouter({
  // 可缩写为 routes
  routes: routes,
});

// 4. 创建和挂载根实例
// 使用 router 配置参数注入路由
// 从而让整个应用都有路由功能
const app = new Vue({
  router,
}).$mount("#app");

```

接下来使用 Vue CLI 创建项目并引入 Vue Router

创建项目

```sh
vue create vue-router-note
```

引入 Vue Router

```sh
npm install vue-router
```

`src`下创建`views/foo/index.vue`

```vue
<template>
    <div class="foo">
        Foo
    </div>
</template>
```

`src`下创建`router/index.js`

```js
import Vue from "vue";
import VueRouter from "vue-router";

Vue.use(VueRouter);

const routes = [{ path: "/foo", component: () => import("@/views/foo") }];

const router = new VueRouter({
  routes: routes,
});

export default router;
```

修改`src/main.js`

```js
import Vue from "vue";
import App from "./App.vue";

import router from "./router";

Vue.config.productionTip = false;

new Vue({
  router: router,
  render: (h) => h(App),
}).$mount("#app");
```

修改`App.vue`

```vue
<template>
  <div id="app">
    <router-link to="/foo">Go to foo</router-link>
    <br>
    <router-view></router-view>
  </div>
</template>

<script>
export default {
  name: "App",
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
```

接下来学习的内容都会以这个为基础

### 2.2 动态路由匹配

我们经常需要将某种模式匹配到所有路由并全都映射到同一个组件

例如： 有一个`User`组件，对于不同 ID 的用户都需要使用这个组件进行渲染，那么可以使用动态路径参数 (dynamic segment)

`src/router/index.js`

```js
const routes = [
    { path: "/user/:id", component: () => import("@/views/user") }
];
```

- `/user/:id`: id为动态路径参数，以`:`开头, 当匹配到路由时，参数值会被设置到`this.$route.params`中

创建`src/views/user/index.vue`

```vue
<template>
    <div class="user">
        Welcome! User: {{ $route.params.id }}
    </div>
</template>
```

- `$route.params.id`:获取路径中的`id`

修改`App.vue`:

```vue
<template>
  <div id="app">
      <router-link to="/user/foo">User foo</router-link>
      <router-link to="/user/bar">User bar</router-link>
    <router-view></router-view>
  </div>
</template>
```

可以在路由中设置多段路径参数，对应的值均会设置到`$route.params`中

| 模式                          | 匹配路径            | $route.params                          |
| ----------------------------- | ------------------- | -------------------------------------- |
| /user/:username               | /user/evan          | `{ username: 'evan' }`                 |
| /user/:username/post/:post_id | /user/evan/post/123 | `{ username: 'evan', post_id: '123' }` |

除了 `$route.params` 外，`$route` 对象还提供了其它有用的信息，例如，`$route.query` (如果 URL 中有查询参数)、`$route.hash` 等等,可以查看 [API 文档](https://router.vuejs.org/zh/api/#路由对象) 的详细说明

#### 2.2.1 响应路由参数的变化

当使用路由参数时（例如从`/user/foo`到`/user/bar`）组件实例会被复用（两个路由都渲染同个组件，比起销毁再创建，复用则显得更加高效），但是也意味着组件的生命周期钩子不会再被调用

复用组件时，可以使用 `watch` 检测 `$route` 对象实现对路由参数的变化做出响应

```js
export default {
    watch: {
        $route(to,from) {
            console.log(from)
            console.log(to)
        }
    }
}
```

或者使用`beforeRouteUpdate` 导航守卫

```js
export default {
    beforeRouteUpdate(to,from,next){
        console.log("from: ",from.fullPath)
        console.log("to",to.fullPath)
        next()
    }
}
```

#### 2.2.2 捕获所有路由或 404 Not found 路由

常规参数只会匹配被`/`分隔的 URL 片段中的字符，若想要匹配任意路径可以使用通配符`*`

```js
{
  // 会匹配所有路径
  path: '*'
}
{
  // 会匹配以 `/user-` 开头的任意路径
  path: '/user-*'
}
```

当使用通配路由时，应该将其放在最后，路由`{path: '*'}`通常用于客户端 404 错误

当使用通配符时，`$route.params` 内会自动添加一个名为 `pathMatch` 参数，包含了 URL 通过通配符被匹配的部分

```js
// 给出一个路由 { path: '/user-*' }
this.$router.push('/user-admin')
this.$route.params.pathMatch // 'admin'
// 给出一个路由 { path: '*' }
this.$router.push('/non-existing')
this.$route.params.pathMatch // '/non-existing'
```

#### 2.2.3 高级匹配模式

`vue-router` 使用 [path-to-regexp](https://github.com/pillarjs/path-to-regexp/tree/v1.7.0)作为路径匹配引擎，所以支持很多高级的匹配模式，例如：可选的动态路径参数、匹配零个或多个、一个或多个，甚至是自定义正则匹配。查看它的[文档](https://github.com/pillarjs/path-to-regexp/tree/v1.7.0#parameters)学习高阶的路径匹配，还有[这个例子 ](https://github.com/vuejs/vue-router/blob/dev/examples/route-matching/app.js)展示 `vue-router` 怎么使用这类匹配

#### 2.2.4 匹配优先级

有时候，同一个路径可以匹配多个路由，此时，匹配的优先级就按照路由的定义顺序：路由定义得越早，优先级就越高

### 2.3 嵌套路由

实际生活中的应用页面，通常由多层嵌套的组件组合而成， URL 中各段动态路径也按某种结构对应嵌套的各层组件

```
/user/foo/profile                     /user/foo/posts
+------------------+                  +-----------------+
| User             |                  | User            |
| +--------------+ |                  | +-------------+ |
| | Profile      | |  +------------>  | | Posts       | |
| |              | |                  | |             | |
| +--------------+ |                  | +-------------+ |
+------------------+                  +-----------------+
```

使用嵌套路由配置可以很简单的表达这种关系

```vue
<template>
  <div id="app">
    <router-view></router-view>
  </div>
</template>
```

这里的 `<router-view>`是最顶层的出口，渲染最高级路由匹配到的组件

一个被渲染组件可以包含自己的嵌套 `<router-view>`

`src/views/user/index.vue`

```vue
<template>
  <div class="user">
    <h2>Welcome! User: {{ $route.params.id }}</h2>
    <router-view></router-view>
  </div>
</template>
```

上例在 `user` 组件中添加了一个嵌套的`router-view`

要在嵌套的出口中渲染组件，需要在`VueRouter`的参数中使用 `children`配置

这里先在`src/user/`下创建两个组件 `profile/index.vue`和 `post/index.vue`

```vue
<template>
    <div>
        User's profile is here ~
    </div>
</template>
```

```vue
<template>
    <div>
        User's post is here ~
    </div>
</template>
```

`src/router/index.js`

```js
const routes = [
    { path: "/foo", component: () => import("@/views/foo") },
    {
        path: "/user/:id",
        component: () => import("@/views/user"),
        children: [
            {
                path: "profile",
                component: () => import("@/views/user/profile")
            },
            {
                path: "post",
                component: () => import("@/views/user/post")
            }
        ]
    }
];
```

- `/user/:id/profile`： 匹配成功时，`profile`组件会被渲染在 `user` 的`<router-view>`中
- `/user/:id/post`： 匹配成功时，`post`组件会被渲染在 `user` 的`<router-view>`中

- `children`配置和`routes`配置一样，所以可以嵌套多层

- 当访问`/user/foo`时是不会渲染子路由的组件的，若想要渲染内容的化可以配置空的子路由

  ```js
  children: [
          // 当 /user/:id 匹配成功，
          // UserHome 会被渲染在 User 的 <router-view> 中
          { path: '', component: UserHome }
  
          // ...其他子路由
        ]
  ```

### 2.4 编程式导航

#### 2.4.1 Push

  除了使用 `<router-link>`创建 a 标签定义导航链接还可以借助  router 的实例方法

  ```js
  router.push(location, onComplete?, onAbort?)
  ```

在 Vue 实例内部，可以通过 `$router`访问路由示例，所以可通过 `this.$router.push`调用

`push` 方法可以导航到不同的路由，其会向 history 栈添加一个新的记录，所以当用户点击浏览器后退按钮时，可回到之前的 URL

当点击 `<router-link>` 时，这个方法会在内部调用，即点击 `<router-link to="...">` 等同于调用`router.push(...)`

|          声明式           |       编程式       |
| :-----------------------: | :----------------: |
| `<router-link :to="...">` | `router.push(...)` |

  方法参数可以是一个字符串路径或一个描述地址的对象

```js
// 字符串
router.push('home')

// 对象
router.push({ path: 'home' })

// 命名的路由
router.push({ name: 'user', params: { userId: '123' }})

// 带查询参数，变成 /register?plan=private
router.push({ path: 'register', query: { plan: 'private' }})
```

值得注意的是，如果提供了 `path`, `params` 会被忽略

```js
const userId = '123'
router.push({ name: 'user', params: { userId }}) // -> /user/123
router.push({ path: `/user/${userId}` }) // -> /user/123
// 这里的 params 不生效
router.push({ path: '/user', params: { userId }}) // -> /user
```

规则同样使用于 `<router-link>` 的 to 属性

同样的规则也适用于 `router-link` 组件的 `to` 属性。

在 2.2.0+，可选的在 `router.push` 或 `router.replace` 中提供 `onComplete` 和 `onAbort` 回调作为第二个和第三个参数。这些回调将会在导航成功完成 (在所有的异步钩子被解析之后) 或终止 (导航到相同的路由、或在当前导航完成之前导航到另一个不同的路由) 的时候进行相应的调用。在 3.1.0+，可以省略第二个和第三个参数，此时如果支持 Promise，`router.push` 或 `router.replace` 将返回一个 Promise。

**注意**： 如果目的地和当前路由相同，只有参数发生了改变 (比如从一个用户资料到另一个 `/users/1` -> `/users/2`)，你需要使用 [`beforeRouteUpdate`](https://router.vuejs.org/zh/guide/essentials/dynamic-matching.html#响应路由参数的变化) 来响应这个变化 (比如抓取用户信息)

#### 2.4.2 Replace

```
router.replace(location, onComplete?, onAbort?)
```

和 `router.push` 类似，但是不会向 history 中添加新的记录，而是替换当前的 history 记录

| 声明式                            | 编程式                |
| --------------------------------- | --------------------- |
| `<router-link :to="..." replace>` | `router.replace(...)` |

#### 2.4.3 Go

```
router.go(n)
```

此方法的参数为整数，表示在 history 记录中向前或向后多少步，类似于 `window.history.go(n)`

```js
// 在浏览器记录中前进一步，等同于 history.forward()
router.go(1)

// 后退一步记录，等同于 history.back()
router.go(-1)

// 前进 3 步记录
router.go(3)

// 如果 history 记录不够用，那就默默地失败呗
router.go(-100)
router.go(100)
```

#### 2.4.4 操作 History

你也许注意到 `router.push`、 `router.replace` 和 `router.go` 跟 [`window.history.pushState`、 `window.history.replaceState` 和 `window.history.go` (opens new window)](https://developer.mozilla.org/en-US/docs/Web/API/History)好像， 实际上它们确实是效仿 `window.history` API 的。

因此，如果你已经熟悉 [Browser History APIs (opens new window)](https://developer.mozilla.org/en-US/docs/Web/API/History_API)，那么在 Vue Router 中操作 history 就是超级简单的。

还有值得提及的，Vue Router 的导航方法 (`push`、 `replace`、 `go`) 在各类路由模式 (`history`、 `hash` 和 `abstract`) 下表现一致

### 2.5 命名路由

在创建 routes 时，可以给某个路由设置名称

```js
const router = new VueRouter({
  routes: [
    {
      path: '/user/:userId',
      name: 'user',
      component: User
    }
  ]
})
```

要链接到一个命名路由，可以给 `router-link` 的 `to` 属性传一个对象：

```html
<router-link :to="{ name: 'user', params: { userId: 123 }}">User</router-link>
```

这跟代码调用 `router.push()` 是一回事：

```js
router.push({ name: 'user', params: { userId: 123 } })
```

这两种方式都会把路由导航到 `/user/123` 路径。

### 2.6 命名视图

有时需要同级展示多个视图，而不是嵌套展示

例如，创建一个布局的，有`sidebar`和 `main` 两个视图，可以在界面中有多个单独命名的视图，而不是单独的路由出口

```html
<router-view class="view one"></router-view>
<router-view class="view two" name="a"></router-view>
<router-view class="view three" name="b"></router-view>
```

默认的 `<router-view>` 名称为 `default`

一个路由对应多个组件，需要使用 `components`配置

```js
const router = new VueRouter({
  routes: [
    {
      path: '/',
      components: {
        default: Foo,
        a: Bar,
        b: Baz
      }
    }
  ]
})
```

#### 2.6.1 嵌套命名视图

我们也有可能使用命名视图创建嵌套视图的复杂布局。这时你也需要命名用到的嵌套 `router-view` 组件。我们以一个设置面板为例

```
/settings/emails                                       /settings/profile
+-----------------------------------+                  +------------------------------+
| UserSettings                      |                  | UserSettings                 |
| +-----+-------------------------+ |                  | +-----+--------------------+ |
| | Nav | UserEmailsSubscriptions | |  +------------>  | | Nav | UserProfile        | |
| |     +-------------------------+ |                  | |     +--------------------+ |
| |     |                         | |                  | |     | UserProfilePreview | |
| +-----+-------------------------+ |                  | +-----+--------------------+ |
+-----------------------------------+                  +------------------------------+
```

- `Nav` 只是一个常规组件。
- `UserSettings` 是一个视图组件。
- `UserEmailsSubscriptions`、`UserProfile`、`UserProfilePreview` 是嵌套的视图组件

`UserSettings` 组件的 `<template>` 部分应该是类似下面的这段代码：

```html
<!-- UserSettings.vue -->
<div>
  <h1>User Settings</h1>
  <NavBar/>
  <router-view/>
  <router-view name="helper"/>
</div>
```

然后你可以用这个路由配置完成该布局：

```js
{
  path: '/settings',
  // 你也可以在顶级路由就配置命名视图
  component: UserSettings,
  children: [{
    path: 'emails',
    component: UserEmailsSubscriptions
  }, {
    path: 'profile',
    components: {
      default: UserProfile,
      helper: UserProfilePreview
    }
  }]
}
```

### 2.7 重定向和别名

#### 2.7.1 重定向

重定向可以通过 `routes` 配置完成

例如从`/a`重定向到`/b`

```js
const router = new VueRouter({
  routes: [
    { path: '/a', redirect: '/b' }
  ]
})
```

重定向的目标也可以是一个命名的路由：

```js
const router = new VueRouter({
  routes: [
    { path: '/a', redirect: { name: 'foo' }}
  ]
})
```

甚至是一个方法，动态返回重定向目标：

```js
const router = new VueRouter({
  routes: [
    { path: '/a', redirect: to => {
      // 方法接收 目标路由 作为参数
      // return 重定向的 字符串路径/路径对象
    }}
  ]
})
```

注意[导航守卫](https://router.vuejs.org/zh/guide/advanced/navigation-guards.html)并没有应用在跳转路由上，而仅仅应用在其目标上。在下面这个例子中，为 `/a` 路由添加一个 `beforeEnter` 守卫并不会有任何效果

#### 2.7.2 别名

“重定向”的意思是，当用户访问 `/a`时，URL 将会被替换成 `/b`，然后匹配路由为 `/b`，那么“别名”又是什么呢？

**`/a` 的别名是 `/b`，意味着，当用户访问 `/b` 时，URL 会保持为 `/b`，但是路由匹配则为 `/a`，就像用户访问 `/a` 一样。**

上面对应的路由配置为：

```js
const router = new VueRouter({
  routes: [
    { path: '/a', component: A, alias: '/b' }
  ]
})
```

“别名”的功能让你可以自由地将 UI 结构映射到任意的 URL，而不是受限于配置的嵌套路由结构

### 2.8 路由组件传参

在组件中使用 `$route` 会使之与其对应路由形成高度耦合，从而使组件只能在某些特定的 URL 上使用，限制了其灵活性

使用 `props` 将组件和路由解耦：

**取代与 `$route` 的耦合**

```js
const User = {
  template: '<div>User {{ $route.params.id }}</div>'
}
const router = new VueRouter({
  routes: [{ path: '/user/:id', component: User }]
})
```

**通过 `props` 解耦**

```js
const User = {
  props: ['id'],
  template: '<div>User {{ id }}</div>'
}
const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User, props: true },

    // 对于包含命名视图的路由，你必须分别为每个命名视图添加 `props` 选项：
    {
      path: '/user/:id',
      components: { default: User, sidebar: Sidebar },
      props: { default: true, sidebar: false }
    }
  ]
})
```

这样你便可以在任何地方使用该组件，使得该组件更易于重用和测试

#### 2.8.1布尔模式

如果 `props` 被设置为 `true`，`route.params` 将会被设置为组件属性

#### 2.8.2 对象模式

如果 `props` 是一个对象，它会被按原样设置为组件属性。当 `props` 是静态的时候有用。

```js
const router = new VueRouter({
  routes: [
    {
      path: '/promotion/from-newsletter',
      component: Promotion,
      props: { newsletterPopup: false }
    }
  ]
})
```

#### 2.8.3 函数模式

你可以创建一个函数返回 `props`。这样你便可以将参数转换成另一种类型，将静态值与基于路由的值结合等等。

```js
const router = new VueRouter({
  routes: [
    {
      path: '/search',
      component: SearchUser,
      props: route => ({ query: route.query.q })
    }
  ]
})
```

URL `/search?q=vue` 会将 `{query: 'vue'}` 作为属性传递给 `SearchUser` 组件。

请尽可能保持 `props` 函数为无状态的，因为它只会在路由发生变化时起作用。如果你需要状态来定义 `props`，请使用包装组件，这样 Vue 才可以对状态变化做出反应

### 2.9 HTML History 模式

`vue-router` 默认 hash 模式 —— 使用 URL 的 hash 来模拟一个完整的 URL，于是当 URL 改变时，页面不会重新加载

如果不想要很丑的 hash，我们可以用路由的 **history 模式**，这种模式充分利用 `history.pushState` API 来完成 URL 跳转而无须重新加载页面

```js
const router = new VueRouter({
  mode: 'history',
  routes: [...]
})
```

当你使用 history 模式时，URL 就像正常的 url，例如 `http://yoursite.com/user/id`，也好看！

不过这种模式要玩好，还需要后台配置支持。因为我们的应用是个单页客户端应用，如果后台没有正确的配置，当用户在浏览器直接访问 `http://oursite.com/user/id` 就会返回 404，这就不好看了。

所以呢，你要在服务端增加一个覆盖所有情况的候选资源：如果 URL 匹配不到任何静态资源，则应该返回同一个 `index.html` 页面，这个页面就是你 app 依赖的页面

下面是 nginx 的配置

```
location / {
  try_files $uri $uri/ /index.html;
}
```

**警告**

给个警告，因为这么做以后，你的服务器就不再返回 404 错误页面，因为对于所有路径都会返回 `index.html` 文件。为了避免这种情况，你应该在 Vue 应用里面覆盖所有的路由情况，然后再给出一个 404 页面。

```js
const router = new VueRouter({
  mode: 'history',
  routes: [
    { path: '*', component: NotFoundComponent }
  ]
})
```

或者，如果你使用 Node.js 服务器，你可以用服务端路由匹配到来的 URL，并在没有匹配到路由的时候返回 404，以实现回退

## 3. 进阶

### 3.1 导航守卫

正如其名，`vue-router` 提供的导航守卫主要用来通过跳转或取消的方式守卫导航。有多种机会植入路由导航过程中：全局的, 单个路由独享的, 或者组件级的。

记住**参数或查询的改变并不会触发进入/离开的导航守卫**。你可以通过[观察 `$route` 对象](https://router.vuejs.org/zh/guide/essentials/dynamic-matching.html#响应路由参数的变化)来应对这些变化，或使用 `beforeRouteUpdate` 的组件内守卫

#### 3.1.1 全局前置守卫



## Reference

1. [Vue Router](https://router.vuejs.org/zh/installation.html) vue docs