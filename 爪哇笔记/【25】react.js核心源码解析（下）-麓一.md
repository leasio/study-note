# 课程目标

p6：

* 知道react大致实现思路，能对比react和js控制原生dom的差异，能口喷一个简化版的react
* 知道diff算法大致实现思路
* 对state和props有自己的使用心得，结合受控组件、hoc等特性描述，需要说明各种方案的适用场景

p6+~p7：

* 能说明白为什么要实现fiber
* 能说明白为什么要实现hook
* 知道react不常用的特性、比如context、portal、errorBoundry

# 知识要点

## diff算法

* react和vue的diff算法有什么异同？
* react为什么不去优化diff算法？
* 传统diff O(n^3)，React Diff O(n)？怎么来的？还可以优化吗？
* 最好时间复杂度、最坏时间复杂度、平均时间复杂度、均摊时间复杂度：《数据结构和算法之美》——极客时间

diff算法并不是近年才有的，早在多年以前就已经有人在研究diff算法了。最早复杂度基本是O(m^3n^3)，然后优化了30多年，终于在2011年把复杂度降低到O(n^3)。这里的n指的是节点总数，所以1000个节点，要进行10亿次操作。

而今天，站在巨人的肩膀上，我们将探究的diff算法主要指react横空出世之后提出的近代同层比较的diff算法。因为同层嘛，复杂度就到了O(n)。

### 为什么需要diff，为什么现在对diff好像提及的不是那么多？

本质上就是为了性能，性能，性能

### 为什么传统的是O(n^3)

在计算机中，比较两棵树的区别；——借鉴字符串的编辑距离，莱温斯坦最短距离算法

比如：字符串"hello"->“hallo”

### react是如何将diff算法的复杂度降下来的？

其实就是在算法复杂度、虚拟dom渲染机制、性能中找了一个平衡，react采用了启发式的算法，做了如下最优假设：

a. 如果节点类型相同，那么以该节点为根节点的tree结构，大概率是相同的，所以如果类型不同，可以直接「删除」原节点，「插入」新节点

b. 跨层级移动子tree结构的情况比较少见，或者可以培养用户使用习惯来规避这种情况，遇到这种情况同样是采用先「删除」再「插入」的方式，这样就避免了跨层级移动

c. 同一层级的子元素，可以通过key来缓存实例，然后根据算法采取「插入」「删除」「移动」的操作，尽量复用，减少性能开销

d. 完全相同的节点，其虚拟dom也是完成一致的

### 实现一个简化版react

#### index.js

react的使用

```js
import { render } from "./render";
import { createElement } from "./react";

const vnode = createElement(
    "ul",
    {
        id: "ul-test",
        className: "padding-20",
        style: {
            padding: "10px",
        },
    },
    createElement("li", { key: "li-0" }, "this is li 01")
);

const nextVNode = createElement(
    "ul",
    {
        style: {
            width: "100px",
            height: "100px",
            backgroundColor: "green",
        },
    },
    [
        createElement("li", { key: "li-a" }, "this is li a"),
        createElement("li", { key: "li-b" }, "this is li b"),
        createElement("li", { key: "li-c" }, "this is li c"),
        createElement("li", { key: "li-d" }, "this is li d"),
    ]
);

const lastVNode = createElement(
    "ul",
    {
        style: {
            width: "100px",
            height: "200px",
            backgroundColor: "pink",
        },
    },
    [
        createElement("li", { key: "li-a" }, "this is li a"),
        createElement("li", { key: "li-c" }, "this is li c"),
        createElement("li", { key: "li-d" }, "this is li d"),
        createElement("li", { key: "li-f" }, "this is li f"),
        createElement("li", { key: "li-b" }, "this is li b"),
    ]
);

setTimeout(() => render(vnode, document.getElementById("app")))
setTimeout(() => render(nextVNode, document.getElementById("app")),6000)
setTimeout(() => render(lastVNode, document.getElementById("app")),8000)
```

#### react.js

index.js中``createElement`的实现

```JS

const normalize = (children = []) => 
	children.map(child => typeof child === 'string' ? createVText(child) : child)

export const NODE_FLAG = {
    EL: 1, // 元素 element
    TEXT: 1 << 1
};
// El & TEXT  = 0

const createVText = (text) => {
    return {
        type: "",
        props: {
            nodeValue: text + ""
        },
        $$: { flag: NODE_FLAG.TEXT }
    }
}

const createVNode = (type, props, key, $$) => {
    return {
        type, 
        props,
        key,
        $$,
    }
}

export const createElement = (type, props, ...kids) => {
    props = props || {};
    let key = props.key || void 0;
    kids = normalize(props.children || kids);

    if(kids.length) props.children = kids.length === 1? kids[0] : kids;

    // 定义一下内部的属性
    const $$ = {};
    $$.staticNode = null;
    $$.flag = type === "" ? NODE_FLAG.TEXT : NODE_FLAG.EL;

    return createVNode(type, props, key, $$)
}
```

#### render.js

index.js中`render`的实现

```js
import { mount } from "./mount";
import { patch } from "./patch";

export function render(vnode, parent) {
    let prev = parent.__vnode;
    if(!prev) {
        mount(vnode, parent);
        parent.__vnode = vnode;
    } else {
        if(vnode) {
            // 新旧两个
            patch(prev, vnode, parent);
            parent.__vnode = vnode;
        } else {
            parent.removeChild(prev.staticNode)
        }
    } 
}
```

#### mount.js

render.js中`mount`的实现

```js
import { patchProps } from "./patch";
import { NODE_FLAG } from "./react";

export function mount(vnode, parent, refNode) {
    // 为什么会有一个 refNode?
    /**                   |
     * 假如： ul ->  li  li  li(refNode) 
     */
    if(!parent) throw new Error('no container');
    const $$ = vnode.$$;

    if($$.flag & NODE_FLAG.TEXT) {
        // 如果是一个文本节点
        const el = document.createTextNode(vnode.props.nodeValue);
        vnode.staticNode = el;
        parent.appendChild(el);
    } else if($$.flag & NODE_FLAG.EL) {
        // 如果是一个元素节点的情况，先不考虑是一个组件的情况；
        const { type, props } = vnode;
        const staticNode = document.createElement(type);
        vnode.staticNode = staticNode;

        // 我们再来处理，children 和后面的内容
        const { children, ...rest} = props;
        if(Object.keys(rest).length) {
            for(let key of Object.keys(rest)) {
                // 属性对比的函数
                patchProps(key, null, rest[key], staticNode);
            }
        }

        if(children) {
            // 递归处理子节点
            const __children = Array.isArray(children) ? children : [children];
            for(let child of __children) {
                mount(child, staticNode);
            }
        }
        refNode ? parent.insertBefore(staticNode, refNode) : parent.appendChild(staticNode);
    }   
}
```

#### patch.js

render.js中`patch`的实现；mount.js中`patchProps`的实现

```js
import { mount } from "./mount";
import { diff } from './diff';

function patchChildren(prev, next, parent) {
    // diff 整个的逻辑还是耗性能的，所以，我们可以先提前做一些处理。
    if(!prev) {
        if(!next) {
            // nothing
        } else {
            next = Array.isArray(next) ? next : [next];
            for(const c of next) {
                mount(c, parent);
            }
        }
    } else if(!next) parent.removeChild(prev.staticNode);
    else if (!Array.isArray(prev)) {
        // 只有一个 children
        if(!Array.isArray(next)) {
            patch(prev, next, parent)
        } else {
            // 如果prev 只有一个节点，next 有多个节点
            parent.removeChild(prev.staticNode);
            for(const c of next) {
                mount(c, parent);
            }
        }
    } else {
        if(!Array.isArray(next)) {
            parent.removeChild(prev.staticNode);
            mount(next, parent);
        } else diff(prev, next, parent);
    } 
}

export function patch (prev, next, parent) {
    // type: 'div' -> 'ul'
    if(prev.type !== next.type) {
        parent.removeChild(prev.staticNode);
        mount(next, parent);
        return;
    }

    // type 一样，diff props 
    // 先不看 children 
    const { props: { children: prevChildren, ...prevProps}} = prev;
    const { props: { children: nextChildren, ...nextProps}} = next;
    // patch Porps
    const staticNode = (next.staticNode = prev.staticNode);
    for(let key of Object.keys(nextProps)) {
        let prev = prevProps[key],
        next = nextProps[key]
        patchProps(key, prev, next, staticNode)
    }

    for(let key of Object.keys(prevProps)) {
        if(!nextProps.hasOwnProperty(key)) patchProps(key, prevProps[key], null, staticNode);
    }

    // patch Children ！！！
    patchChildren(
        prevChildren,
        nextChildren,
        staticNode
    )

}


export function patchProps(key, prev, next, staticNode) {
    // style 
    if(key === "style") {
        // margin: 0 padding: 10
        if(next) {
            for(let k in next) {
                staticNode.style[k] = next[k];
            }
        }
        if(prev) {
        // margin: 10; color: red
            for(let k in prev) {
                if(!next.hasOwnProperty(k)) {
                    // style 的属性，如果新的没有，老的有，那么老的要删掉。
                    staticNode.style[k] = "";
                }
            }
        }
    }

    else if(key === "className") {
        if(!staticNode.classList.contains(next)) {
            staticNode.classList.add(next);
        }
    }

    // events
    else if(key[0] === "o" && key[1] === 'n') {
        prev && staticNode.removeEventListener(key.slice(2).toLowerCase(), prev);
        next && staticNode.addEventListener(key.slice(2).toLowerCase(), next);

    } else if (/\[A-Z]|^(?:value|checked|selected|muted)$/.test(key)) {
        staticNode[key] = next

    } else {
        staticNode.setAttribute && staticNode.setAttribute(key, next);
    }
}
```

#### diff.js

patch.js中`diff`的实现

```js
import { mount } from './mount.js'
import { patch } from './patch.js'

export const diff = (prev, next, parent) => {
    let prevMap = {}
    let nextMap = {}

    // 遍历我的老的 children
    for (let i = 0; i < prev.length; i++) {
        let { key = i + '' } = prev[i]
        prevMap[key] = i
    }

    let lastIndex = 0
    // 遍历我的新的 children
    for (let n = 0; n < next.length; n++) {
        let { key = n + '' } = next[n]
        // 老的节点
        let j = prevMap[key]
        // 新的 child
        let nextChild = next[n]
        nextMap[key] = n
        
        if (j == null) {
            // 老的children      新的children
            // [b, a]           [c, d, a]  =>  [c, b, a]  --> c
            // [b, a]           [c, d, a]  =>  [c, d, b, a]  --> d
            // 从老的里面没有找到,新插入
            let refNode = n === 0 ? prev[0].staticNode : next[n - 1].staticNode.nextSibling
            mount(nextChild, parent, refNode)
        } else {
            // [b, a]           [c, d, a]  =>  [c, d, a, b]  --> a
            // 如果找到了，我 patch 
            patch(prev[j], nextChild, parent)

            if (j < lastIndex) {
                // 上一个节点的下一个节点的前面，执行插入
                let refNode = next[n - 1].staticNode.nextSibling;
                parent.insertBefore(nextChild.staticNode, refNode)
            } else {
                lastIndex = j
            }
        }
    }
    // [b, a]           [c, d, a]  =>  [c, d, a]  --> b
    for (let i = 0; i < prev.length; i++) {
        let { key = '' + i } = prev[i]
        if (!nextMap.hasOwnProperty(key)) parent.removeChild(prev[i].staticNode)
    }
}
```

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

```js
/**
 * schedule —> 把我的任务放进一个队列里，然后以某一种节奏进行执行；
 * 
 */

// task 的任务队列
const queue = [];
const threshold = 1000 / 60;

const transtions = [];
let deadline = 0;

// 获取当前时间， bi  date-now 精确
const now = () => performance.now(); // 时间 ，精确
// 从任务queue中，选择第一个 任务 
const peek = arr => arr.length === 0 ? null : arr[0];

// schedule —> 把我的任务放进一个队列里，然后以某一种节奏进行执行；
export function schedule (cb) {
    queue.push(cb);
    startTranstion(flush);
}

// 此时，是否应该交出执行权
function shouldYield() {
    return navigator.scheduling.isInputPending() || now() >= deadline;
}

// 执行权的切换
function startTranstion(cb) {
    transtions.push(cb) && postMessage();
}

// 执行权的切换
const postMessage = (() => {
    const cb = () => transtions.splice(0, 1).forEach(c => c());
    const { port1, port2 } = new MessageChannel();
    port1.onmessage = cb;
    return () => port2.postMessage(null);
})()

// 模拟实现 requestIdleCallback 方法
function flush() {
    // 生成时间，用于判断
    deadline = now() + threshold;
    let task = peek(queue);

    // 我还没有超出 16.666ms 同时，也没有更高的优先级打断我
    while(task && !shouldYield()) {
        const { cb } = task;
        const next = cb();
        // 相当于有一个约定，如果，你这个task 返回的是一个函数，那下一次，就从你这里接着跑
        // 那如果 task 返回的不是函数，说明已经跑完了。不需要再从你这里跑了
        if(next && typeof next === "function") {
            task.cb = next;
        } else {
            queue.shift()
        }
        task = peek(queue);
    }

    // 如果我的这一个时间片，执行完了，到了这里。
    task && startTranstion(flush)
}
```



# 补充知识点

## react和Vue的diff算法有什么区别？

**相同点**：

Vue和react的diff算法，都是不进行跨层级比较，只做同级比较。

**不同点**：

1. Vue进行diff时，调用patch打补丁函数，一边比较一边给真实的DOM打补丁

2. Vue对比节点，当节点元素类型相同，但是className不同时，认为是不同类型的元素，删除重新创建，而react则认为是同类型节点，进行修改操作

3. ①Vue的列表比对，采用从两端到中间的方式，旧集合和新集合两端各存在两个指针，两两进行比较，如果匹配上了就按照新集合去调整旧集合，每次对比结束后，指针向队列中间移动	②而react则是从左往右依次对比，利用元素的index和标识lastindex进行比较，如果满足index < lastIndex就移动元素，删除和添加则各自按照规则调整

   ③当一个集合把最后一个节点移动到最前面，react会把前面的节点依次向后移动，而Vue只会把最后一个节点放在最前面，这样的操作来看，Vue的diff性能是高于react的

## 目前的diff是什么？真的是O(n)吗？

准确的来说，React的diff的最好时间复杂度是O(n)，最差的话，是O(mn)

## diff比较的是什么？

比较的是current fiber和vdom，比较之后生成workInprogress Fiber

## 为什么要有key？

在比较时，会以key和type是否相同进行比较，如果相同，则直接复制。

## react为什么要用fiber？

stack reconciler -> fiber reconciler

在V16版本之前，协调机制是Stack reconciler，V16版本发布Fiber架构后是Fiber reconciler。

在setState后，react会立即开始recociliation过程，从父节点（Virtual DOM）开始递归遍历，以找出不同。将所有的Virtual DOM遍历完成后，reconciler才能给出当前需要修改真实DOM的信息，并传递给renderer，进行渲染，然后屏幕上才会显示此次更新内容。

对于特别庞大的DOM树来说，reconciliation过程会很长（x00ms），在这期间，主线程是被js占用的，因此任何交互、布局、渲染都会停止，给用户的感觉就是页面被卡住了。

在这里我们想解决这个问题的话，来引入一个概念，就是任务可中断，以及任务优先级。也就是说我们的reconciliation的过程中会生成一些任务和子任务，用户的操作的任务优先级是高于reconciliation产生的任务的，也就是说用户操作的任务是可以打断reconciliation中产生的任务的，它会优先执行。

## react为什么要用hook？

[Hook 简介 – React (docschina.org)](https://react.docschina.org/docs/hooks-intro.html#motivation)

## hook是怎么玩的？

我的diff，是不是current fiberhevdom比较？

```jsx
function App() {
    const [state, setState] = useState(0);
    const divRef = useRef(null);
    return <div>XXX</div>
}
// App 是不是也是⼀个 Fiber
// 在beginWork的时候 App(); -> vdom

// AppFiber.memoizedState -> hook.next -> hook.next -> hook
// 							 hook.memoizedState - 保存了 hook 对应的属性
 
class Main extends Component {
    render(){
        return <div>xxx</div>
    }
}
```

