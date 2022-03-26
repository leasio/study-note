# 课程目标

p6：

* 知道react大致实现思路，能对比react和js控制原生dom的差异，能口喷一个简化版的react。
* 知道diff算法大致实现思路。
* 对state和props有自己的使用心得，结合受控组件、hoc等特性描述，需要说明各种方案的适用场景。

p6+~p7：

* 能说明白为什么要实现fiber。
* 能说明白为什么要实现hook。
* 知道react不常用的特性、比如context、portal、errorBoundry。

# 知识要点

## diff算法

### 为什么需要diff，为什么现在对diff好像提及的不是那么多？

### 为什么传统的是O(n^3)

### react是如何将diff算法的复杂度降下来的？

其实就是在算法复杂度、虚拟dom

b. 跨层级移动子tree结构的情况比较少见，或者可以培养用户使用习惯来规避这种情况，遇到这种情况同样是采用先[删除]再[插入]的方式，这样就避免了跨层级移动

d. 完全相同的节点，其虚拟dom也是完成一致的

## 调度

`concurrent`模式 / 18里面 -- 调度。

`scheduler` -- 有一个任务，耗时很长，filter。

--> 把任务放进一个队列，然后开始以一种节奏进行执行。

\---- ---- ----

`requestIdleCallback` -> 

* 兼容性；
* 50ms渲染问题；

chrome 60hz 每16.666ms执行一次时间循环。

\| --- task queue --- | --- micro task --- | --- raf --- | --- render --- | --- requestIdleCallback --- |

为什么没有用`generator`

为什么没有用`setTimeout` -- 4-5ms延时

# 补充知识点