# 课程目标

* 仿知乎页面架构

# 知识要点

[headless ui](https://headlessui.dev/)

[heroicons](https://heroicons.com/)

* React Router V6 的一些新的特性

  * Outlet
  * use API

* Tailwind CSS

  * 工程化的思路，核心就是解决冗余的问题，实现复用
  * 原子化的CSS，粒度：bootsrtap  > tailwind > css
  * 解决一些CSS的复杂性，类名约定的心智负担

* headless ui

  * antd element ui -> 风格，对于一些复杂界面设计的网站，解决不了
  * headles ui：没有样式的，styleles的

* 组件封装

  * 两种state组件维护的方式：
    * 构建一个 list；splice concat 不断地更新
    * 每个回答，单独维护一个状态

* 定位的多种用法

  * CSS 常考题：BFC成因和解决方案 / 布局IFC、BFC、FFC和GFC / 绝对定位 / flex 容器，属性
  * 定位：`static`,`absolute`,`relative`,`sticky`,`fixed`,`inherit`
    * static：默认值，是没有脱离文档流
    * relative：相对于文档流中原先的元素进行调整
    * fixed：绝对，相对于浏览器进行定位的
    * absolute：相对于最近的祖先节点(fixed/sticky/relative)的定位
    * 当 absolute 的祖先元素均为 static 布局，此时和 fixed 的区别在于，是否随滚动条移动
    * sticky：CSS3 的特性，粘性。在 relative 和 fixed 之间切换

* 响应式布局

  * 传统的响应式布局的思路

    * float 布局

      * 双飞翼布局
      * 圣杯布局

    * 相对单位布局

      * em rem

      ```js
      // 淘宝的方案
      
      // document.documentElement.clientWidth 一般在手机端，是375px；
      document.style.fontSize = document.documentElement.clientWidth / 3.75;
      // 100px 1rem;
      // font -- 12px; 0.12rem
      ```

    * flex 布局
    * grid 布局
    * 使用 JavaScript 的控制
    * 媒体查询

  * bootstrap / antd / element -- column / row 24栅格布局 -- 都是局域媒体查询的

    ```html
    <Col md={12} sm={24}></Col>
    <!-- 转译成 -->
    <Col class=‘ant-col-md-12 ant-col-sm-24’></Col>
    ```

    实现原理

    ```css
    @media screen and (min-width: 768px) {
        .ant-col-md-1 {
            display: block;
            flex: 0 0 4.1667%;
            max-width: 0 0 4.1667%;
        }
        .ant-col-md-2 {
            display: block;
            flex: 0 0 8.333%;
            max-width: 0 0 8.333%;
        }
        ...
        .ant-col-md-24 {
            display: block;
            flex: 0 0 100%;
            max-width: 0 0 100%;
        }
    }
    @media screen and (max-width: 768px) {
        .ant-col-sm-2 {
            display: block;
            flex: 0 0 8.333%;
            max-width: 0 0 8.333%;
        }
        .ant-col-sm-4 {
            display: block;
            flex: 0 0 16.666%;
            max-width: 0 0 16.666%;
        }
        ...
        .ant-col-sm-24 {
            display: block;
            flex: 0 0 100%;
            max-width: 0 0 100%;
        }
    }
    ```

* 接触滚动

  * getboundingClientRect()
    * 节流，获取Tab框距离屏幕右上角的位置，从而显示
  * Observer 的原理和总结
    * intersectionObserver 主要是用来监听组件被遮挡的监听器
    * 一个元素，从可见到不可见，或者反之，我们就可以使用它来监听
  * Observer
    * IntersectionObserver：监听元素的可见变化
    * MutationObserver：监听元素属性和节点的变化
    * ResizeObserver：监听大小的改变
    * ReportingObserver：
    * PerformanceObserver：监听网页性能

* 主题切换

  * tailwind css 如何实现主题切换
  * postcss 的插件，了解一些css ast 能力

# 补充知识点

## 关于埋点

- 埋点分类：可视化埋点、无埋点、业务埋点
- 要在 埋点的入侵性，完整性之间做一个 trade off
- 收集 SDK 中间层。收集你的所有接口 节流 localstorage
- 数据存储、数据清洗、分桶、数据可视化、分析。