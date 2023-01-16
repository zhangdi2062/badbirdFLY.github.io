---
title: vue初学者
date: 2019-06-29 10:31:56
tags:
- vue
---

1. 关于npm安装

当你新建一个项目，执行`npm install vue`报错`no such file or directory, open 'package.json'`，

看下[npm官网](https://www.npmjs.cn/getting-started/using-a-package.json/)，管理本地安装的npm包的最佳方法是创建package.json文件，输入命令`npm init`,接下来根据提示输入信息，会生成package.json文件，此时再安装vue就ok啦

2. webpack部署
```
npm install --global vue-cli

vue init webpack my-project
(vue init webpack-simple my-project 省略了路由、eslint等安装提示)
```
![](https://images2017.cnblogs.com/blog/1062254/201801/1062254-20180131161819546-387251641.png)

3. scoped  css样式的局部作用域

4. v-on:事件  v-bind:属性
4. 组件注册到父组件中

     1. `import Users from './components/Users'`
     2. ```js
        components: {
            //Users, // or "users": Users
            "users": Users
        }
        ```
    3. `在templete中引用<users></users>`
    

5. 父组件给子组件传值

    1. 在父组件的data中定义好数据，在父组件的templete中添加绑定`<users v-bind:usersname="userData"/> <!-- userData代表父组件中对应data中的名字，username为绑定该值代表的属性  -->`

    2. 在子组件中注册该属性
    
    ```js
    props:['username'] //或者更详细一点
    props: {
        username: {
            type: Array,
            required: true
        }
    },
    ``` 

6. 子组件给父组件传值（自定义事件）

    1. 子组件中发射事件

    ```js
    //子向父传值,第一个参数定义事件的名字，第二个参数传递的值
      this.$emit("title-change", "这是子组件的值")
    ```

    2. 父组件中绑定自定义事件，并接收子组件的值

    ```html
    <!-- 在该子组件中绑定事件,事件名称和上面保持一致，后面的方法名任意 -->
    <!-- 注意参数一定是$event -->
    <app-header v-on:title-change="updateTitle($event)"></app-header>
    ```
    ```js
    //在updateTitle方法中接收子组件传递的参数
    methods: {
        updateTitle(title) {
        this.title = title
        }
    }
    ```

7. 生命周期，钩子函数和methods、data同级

8. 安装路由，页面跳转

`cnpm install vue-router --save-dev`

注册路由

main.js中配置

```js
import VueRouter from 'vue-router'
import Home from './components/Home'
import HelloWorld from './components/HelloWorld'

Vue.use(VueRouter)

//配置路由,是component不加s,将组件引入到main.js中到
const router = new VueRouter({
    routes: [
        {path: "/", component: Home},
        {path: "/helloworld", component: HelloWorld}
    ],
    //去掉 地址栏标记/#/
    mode: 'history'
})
//在vue中引入路由
new Vue({
    rooter,
    el: '#app',
    ...
})
```

在App.vue中引入路由
```html
<router-view></router-view>
```

此时页面已经可以通过地址栏跳转页面，但地址栏出现`/#/`，去掉方法： 在router配置中添加这句话
`mode: 'history'`


在app中实现导航

使用a标签每次都会重新加载页面，采用路由代替`<router-link to='/'>home</router-link>`

9. 安装http,vue-resource`cnpm install vue-resource --save-dev`

同样，在main.js中引入并使用
`import VueResource from 'vue-resource'`
`Vue.use(VueResource)`

需要http请求得到数据，一般在dom创建前，在created阶段，写法如下：
```js
created() {
    this.$http.get('http://jsonplaceholder.typicode.com/users')
    .then((data) => {
      this.users = data.body
      }
    )
  }
```

10. 搭建脚手架

