# 课程目标

- 插槽 & 过滤器 & 模板的二次加工
- mixin复用 & extends拓展
- 整体拓展类extend & 插件Vue.use
- 组件的高级使用

# 知识要点

## vue进阶用法

### 特征一：模板化

#### 插槽

##### 默认插槽

组件外部维护参数以及结构，内部安排位置

```vue
<template>
	<div id="app">
        <hello-world>
            <p>{{ msg }}</p>
    	</hello-world>
    </div>
</template>

<script>
    import HelloWorld from './components/HelloWorld.vue'
    export default {
        components: {
            HelloWorld
        },
        name: 'App',
        data () {
            return {
                msg: 'zhaowa start'
            }
        }
    }
</script>
```

```vue
<!-- ./components/HelloWorld.vue -->
<template>
	<div class="hello">
        <slot></slot>
    </div>
</template>
```

##### 具名插槽

以name标识插槽的身份，从而在组件内部做到可区分

```vue
<template>
	<div id="app">
        <hello-world>
            <template v-slot:header>{{ header }}</template>
			<template v-slot:body>{{ body }}</template>
			<template v-slot:footer>{{ footer }}</template>
    	</hello-world>
    </div>
</template>

<script>
    import HelloWorld from './components/HelloWorld.vue'
    export default {
        components: {
            HelloWorld
        },
        name: 'App',
        data () {
            return {
                header: 'zhaowa header',
                body: 'zhaowa body',
                footer: 'zhaowa footer'
            }
        }
    }
</script>
```

```vue
<!-- ./components/HelloWorld.vue -->
<template>
	<div class="hello">
        <slot name="header"/>
        <slot name="body"/>
        <slot name="footer"/>
    </div>
</template>
```

##### 作用域插槽

slot-scope (v2.6 before)

v-slot (v2.6 after)

外部做结构描述勾勒，内部做传参

```vue
<template>
	<div id="app">
        <hello-world>
            <!-- 老版 -->
            <template slot="content" slot-scope="{ slotProps }">
                {{ slotProps }}
            </template>
            <!-- 新版 -->
            <template v-slot:slotProps="slotProps">
                {{ slotProps }}
            </template>
    	</hello-world>
    </div>
</template>

<script>
    import HelloWorld from './components/HelloWorld.vue'
    export default {
        components: {
            HelloWorld
        },
        name: 'App'
    }
</script>
```

```vue
<!-- ./components/HelloWorld.vue -->
<template>
	<div class="hello">
        <slot name="content" :slotProps="slotProps"></slot>
    </div>
</template>

<script>
    export default {
        name: 'HelloWorld',
        data () {
            return {
                slotProps: 'zhaowa start'
            }
        }
    }
</script>
```

#### 模板数据的二次加工

1. watch、computed => 响应流过于复杂（尤其在computed赋值）

2. 方案一：函数 - 独立、管理 / 无法响应式

   方案二：v-html

   ```vue
   <template>
       <h1 v-html="money > 99 ? 99 : money"></h1>
   </template>
   ```
   
   方案三：过滤器 - 无上下文
   
   ```vue
   <template>
   	<h1>{{ money | moneyFilter }}</h1>
   </template>
   
   <script>
       export default {
           name: 'HelloWorld',
           data () {
               return {
                   money: 100
               }
           },
           filters: {
               // 过滤器
               moneyFilter(money) {
                   return money > 99 ? 99 : money
               }
           },
       }
   </script>
   ```

#### jsx  更自由地基于js书写

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

```vue
<script>
    import demoMixin from './fragments/demoMixin'
    export default {
        mixins: [demoMixin],
        data () {
            return {
                msg: 'Welcome to Your Vue.js App',
                obj: {
                    number: 123,
                    text: 'text',
                    header: 'header'
                }
            }
        },
        created() {
            console.log(this.$data)
            console.log('component created')
        }
    }
</script>
```

```js
// ./fragments/demoMixin.js
export default {
    data() {
        return {
            msg: '我是mixin',
            obj: {
                title: 'mixinTitle',
                header: 'mixinHeader'
            }
        }
    },
    created() {
        console.log('mixin created');
    }
}
```

```js
// 输出
mixin created
{
    msg: 'Welcome to Your Vue.js App',
    obj: {
        header: 'header',
        number: 123,
        text: 'text',
        title: 'mixinTitle'
    }
}
component created
```

* 1. 应用：抽离公共逻辑（逻辑相同，模板不同，可用mixin）

* 2. 缺点：数据来源不明确

* 3. 合并策略

     a. 递归合并

     b. data合并冲突时，以组件优先

     c. 生命周期回调函数不会覆盖，会先后执行，优先级为先mixin后组件

#### 继承拓展extends - 逻辑拓展

```vue
<script>
    import demoExtends from './fragments/demoExtends'
    export default {
        extends: demoExtends,
        data () {
            return {
                msg: 'Welcome to Your Vue.js App',
                obj: {
                    number: 123,
                    text: 'text',
                    header: 'header'
                }
            }
        },
        created() {
            console.log(this.$data)
            console.log('component created')
        }
    }
</script>
```

```js
// ./fragments/demoExtends
export default {
    data() {
        return {
            msg: '我是extends',
            obj: {
                title: 'extendsTitle',
                header: 'extendsHeader'
            }
        }
    },
    created() {
        console.log('extends created');
    }
}
```

```js
// 输出
extends created
{
    msg: 'Welcome to Your Vue.js App',
    obj: {
        header: 'header',
        number: 123,
        text: 'text',
        title: 'extendsTitle'
    }
}
component created
```

* 1. 应用：拓展独立逻辑

* 2. 与mixin的区别，传值mixin为数组

* 3. 合并策略

     a. 同mixin，也是递归合并

     b. 合并优先级：组件 > mixin > extends

     c. 回调优先级：extends > mixin > 组件

#### 整体拓展类extend

从预定义的配置中拓展出来一个独立的配置项，进行合并

```js
import Vue from 'vue'

// 拓展一个构造器
let _baseOptions = {
    data: function() {
        return {
            course: 'zhaowa',
            session: 'vue',
            teacher: 'yy'
        }
    },
    created () {
        console.log('extend base')
    }
}

const BaseComponent = Vue.extend(_baseOptions)
// 基于BaseComponent，再拓展逻辑
new BaseComponent({
    created() {
        console.log('extend created')
    }
})
```

#### Vue.use - 插件

```js
import MyPlugin from './plugins/myPlugin'

// 加载拓展插件
const _options = {
    name: 'my baby plugin'
}

Vue.use(MyPlugin, _options)
```

```js
// ./plugins/myPlugin
export default {
    install: (Vue, option) => {
        Vue.globalMethod = function() {
            // 主函数
        }
        Vue.directive('my-directive', {
            bind(el, binding, vnode, oldVnode) {
                // 全局资源
            }
        })
        Vue.mixin({
            created: function() {
                console.log(option.name + 'created');
            }
        })
        Vue.prototype.$myMethod = function() {}
    }
}
```

* 1. 注册外部插件，作为整体实例的补充

* 2. 会除重，不会重复注册

* 3. 手写插件

     a. 外部使用Vue.use(myPlugin, options)

     b. 默认调用的是内部的install方法

#### 组件的高级引用

* 1. 递归组件 - es6 vue-tree

* 2. 动态组件 - <component :is='name'></component>

* 3. 异步组件 - router

  ```js
  import Vue from 'vue'
  import Router from 'vue-router'
  import HelloWorld from '@/components/HelloWorld'
  
  Vue.use(Router)
  
  export default new Router({
    routes: [
      {
        path: '/',
        name: 'HelloWorld',
        component: HelloWorld
      }
    ]
  })
  ```

  

