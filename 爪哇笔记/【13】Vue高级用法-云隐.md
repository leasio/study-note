# 课程目标

vue的高级用法

# 知识要点

## vue进阶用法

### 特征一：模块化

#### 插槽

##### 默认插槽

组件外部维护参数以及结构，内部安排位置

##### 具名插槽

以name标识插槽的身份，从而在组件内部做到可区分

##### 作用域插槽

slot-scope (2.6 before)

v-slot (after)

外部做结构描述勾勒，内部做传参

#### 模板数据的二次加工

1. watch、computed => 响应流过于复杂（computed赋值）

2. 方案一：函数 - 独立、管理 / 无法响应式

   方案二：v-html

   方案三：过滤器

   ```js
   {{ time | format }}
   ```

#### jsx 更自由地基于js书写

* 1. v-model 如何实现 => 双向绑定 => 外层bind:value，内层@input
* 2. 写jsx的好处、劣势 => vue的编译路径：template->render->vm.render->vm.render() diff => 可以使用性能优化方案

#### 组件化

### 特征二：组件化

#### 传统模板化

```js
Vue.component('component', {
    template: '<h1>组件</h1>'
})
new Vue({
    el: '#app'
})
```

* 1. 抽象复用
* 2. 精简 & 聚合

#### 混入mixin - 逻辑混入

* 1. 应用：抽离公共逻辑（逻辑相同，模板不同，可用mixin）

* 2. 缺点：数据来源不明确

* 3. 合并策略

     a. 递归合并

     b. data合并冲突时，以组件优先

     c. 生命周期回调函数不会覆盖，会先后执行，优先级为先mixin后组件

#### 继承拓展extends - 逻辑拓展

* 1. 应用：拓展独立逻辑

* 2. 与mixin的区别，传值mixin为数组

* 3. 合并策略

     a. 同mixin，也是递归合并

     b. 合并优先级：组件 > mixin > extends

     c. 回调优先级：extends > mixin

#### 整体拓展类extend

从预定义的配置中拓展出来一个独立的配置项，进行合并

#### Vue.use - 插件

* 1. 注册外部插件，作为整体实例的补充

* 2. 会除重，不会重复注册

* 3. 手写插件

     a. 外部使用Vue.use(myPlugin, options)

     b. 默认调用的是内部的install方法

#### 组件的高级引用

* 1. 递归组件 - es6 vue-tree
* 2. 动态组件 - <component :name='name'></component>
* 3. 异步组件 - router

# 补充知识点