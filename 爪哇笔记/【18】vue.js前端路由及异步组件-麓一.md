# 课程目标

- 针对 react / vue，能够根据业务需求口喷 router 的关键配置，包括但不限于：路由的匹配规则、路由守卫、路由分层等
- 能够描述清楚 history 的主要模式，知道 history 和 router 的边界
- 在么有路由的情况下，也可以根据业务需要，实现一个简单的路由
- 读过 router 底层的源码，不要求每行都读，可以口喷关键代码即可

# 知识要点

## 1. 什么是 Router，以及 Router 发展历史。

- 路由的概念，是伴随着SPA（单页面应用）出现的。在此之前，页面的跳转是通过服务器端进行控制的。
  - 传统页面的跳转，是通过前端向后台发送请求；
  - 后台通过模板引擎的渲染，将一个新的HTML界面再返回给前台；
  - 比如页面跳转时：
    - a 标签的 href
    - from 表单
    - location.href
- 在SPA的出现后，前端可以自由地控制组件的渲染，来模拟页面的跳转。
  - hash 路由和 history 路由

总结：

* 传统的路由，是根据`url`访问相关的`controller`进行数据资源和模板引擎的拼接，返回前端；
* 前端路由使用过`js`根据`url`返回对应的组件加载
  * 所以，前端路由包含两个部分；
    * `url`的处理；
    * 组件加载。

## 2.动态路由

其实，动态路由包含`React.lazy`、``

## 2.路由的分类

- history 路由
- hash 路由
- memory 路由 *

## 3.Router 异步组件

## 4.路由守卫

## 5.路由的实现手写

history部分。

## 题目总结

### 1.hash 路由和history 路由的区别？

* hash 路由一般会携带一个`#`号，不够美观；history 路由不存在这个问题；
* 默认 hash 路由是不会向浏览器端发出请求的，主要一般是用于锚点：history 中 go / back / forward 以及浏览器中的前进和后退按钮，**一般**都会向服务器端发起请求；pushState / replaceState
* 基于此，hash 路由是不支持 SSR 的，但是 history 路由是可以的；
* history 路由在部署的时候，如 nginx，只需要渲染首页，让首页根据路径重新跳转
* hash 路由的监听，一般用 onHashChange 事件监听； history 路由的监听
* Nginx 如何部署：

```bash
# 单个的服务器的部署
location / {
    try_files uri #uir /xxx/main/index.html
}
# 存在代理的情况
location / {
    rewrite ^ /file/index.html break; # 这里代表的是 xxx.cdn 的资源路径
    proxy_pass http://www.xxx.cdn.com;
}
```

### 2.history

history 是一个 BOM Api，里面有 go / forward / back 三个 API，以及 pushState / replaceState

* pushState / replaceState 都不会触发 popState 事件，（不一定会重新渲染）
* popState 什么时候触发？
  * 点击浏览器的前进、后退按钮
  * back / forward / go

### 路由守卫的触发流程

1. 【组件】 - 前一个组件 beforeRouteLeave
2. 【全局】 - router.beforeEach
3. 【组件】 - 如果是路由的参数变化，触发 beforeRouteUpdate；
4. 【配置文件】里，下一个 beforeEnter
5. 【组件】内部声明的 beforeRouterEnter
6. 【全局】调用 beforeResolve
7. 【全局】的 router.afterEach

# 补充知识点